# manim-dynamic-code
Drop-in replacement for `Code` Mobjects enabling dynamic code animations in Manim.

Developed for ManimCE, not tested with any other version of Manim.

## Why not just use the default `Code` Mobject?
Because editing the code in a `Code` Mobject after initial creation is absolute hell. There are multiple reasons for this including:
1. `Code` Mobjects have multiple references to individual SVG glyphs across multiple groups which makes removing the glyphs from the scene a nightmare.
2. Indexes of code string characters and `Code` Mobject SVG glyphs does not necessarily  match as there are no glyphs for whitespace, which makes editing the displayed code non-straightforward.
3. `Transform` animations from one `Code` Mobject to another look terrible if only a part of the code is changing, which means custom animation sequences are required.
4. Animations which add/remove code glyphs create new references to the animated glyph objects which is an absolute nightmare if you need to keep track of those references (which you do if you want to perform additional edits to the code later). Thus, custom logic that resets glyph references after an animation where the code changes is needed.

‼️ `DynamicCode` does all of the above and can be used as a drop-in replacement for `Code` Mobjects.

## How does `DynamicCode` work?
Basically, `DynamicCode` creates temporary `Code` Mobjects under the hood to generate the SVG glyphs with syntax highlighting and handles rearranging these glyphs as needed to edit the code. For animations that actually look like you are inserting or removing code chunks (compare with `Transform` from an original `Code` Mobject to a new `Code` Mobject which looks terrible in comparison), `DynamicCode` animates things in 3 steps:
1. Animates rearranging and/or hiding (set opacity to zero) current Mobjects as per the changes to the code to ensure smooth shifting of glyphs. No new glyphs are added or old glyphs are removed during this step as otherwise Manim defaults to a horrible looking transform (presumably due to the lack of a 1:1 mapping of points in the transform).
2. Outside of any animation, new glyphs are added in the correct positions or unneeded glyphs (these were hidden in the previous step) are removed. Newly added glyphs have opacity=0 so they are not visible yet.
3. Animates writing newly added glyphs by giving them a non-zero opacity one glyph at a time.
4. Updates new glyph references to the new references generated during the animation itself (seriously folks, this is crazy), and cleans up the scene's polluted mobjects where these new glyph references are found.

🤔 In my personal opinion, I find this solution incredibly hacky. It seems to me to be exposing some serious deficiencies in the way Manim does things including 1) the absolute horror of not being able to rely on an object reference after an animation, and 2) the nightmare of not having a straighforward scene hierarchy (e.g., cannot remove an object from the scene if it belongs to some other group you don't know about). The way I've implemented the sequential animations also means they don't quack like a normal animation and cannot be combined with other animations, which kind of sucks, but I don't know a good way around this.

⚠️ Caveat, I am rather new to Manim, so if I am just going about things the wrong way, please PLEASE let me know about it.

## Support
This is all done in my free time. If you find it useful, why not buy me a cup of coffee? Cheers!

<a href="https://www.buymeacoffee.com/marcel.goldschen.ohm" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

## Install
Just put `DynamicCode.py` where your project can find it and import it.

## Quick start
Run `DynamicCode.py` to see some examples of animating code changes in `DynamicCodeExampleScene`.

## Instantiate just like `Code` Mobjects
`DynamicCode` can be used as a drop-in replacement for `Code`.
```python
from DynamicCode import DynamicCode

class ExampleScene(Scene):
  def construct(self):
    mycode = """
from manim import Scene, Square

class FadeInSquare(Scene):
    def construct(self):
        s = Square()
        self.play(FadeIn(s))
        self.play(s.animate.scale(2))
        self.wait()
"""

    dcode = DynamicCode(code=mycode, language='python')
    self.add(dcode)
```

## Append code (no animation)
```python
dcode.append_code("append to last line")
dcode.append_code("append to 4th line", line_index=3)
dcode.append_code("""
append two
new lines
""")

# resize background to accomodate appended code.
dcode.append_code("append to last line", autosize=True)
```
Note, by default the background does not resize with code changes as a common approach is to create a larger background space and then add code to it dynamically. If you want the background to auto-adjust to your code changes, pass `autosize=True` or `autowidth=True` or `autoheight=True` as kwargs to any of the edit actions.

## Append code (with animation)
```python
# If the parent Scene (e.g., self) is given as the player kwarg,
# then the action will be animated.
dcode.append_code("append to last line", player=self, run_time=1)
dcode.append_code("append to 4th line", line_index=3, player=self, run_time=1)
dcode.append_code("""
append two
new lines
""", player=self, run_time=1)

# !! Do NOT do this!
#    You can only animate as shown above.
self.play(dcode.animate.append_code("append to last line"))
```

## Animation or not?
For most of the actions described here, if `player=scene` is passed as a kwarg, then the action will be animated, otherwise it will be instantaneous. See above examples for appending code.

An unfortuante side-effect of this hacky way to animate code change sequences is that the animations cannot be played simultaneously with other animations. I'm not sure how to work around this at the moment.

## Prepend code
Can be animated.
```python
dcode.prepend_code("prepend to first line")
dcode.prepend_code("prepend to 4th line", line_index=3)
dcode.prepend_code("""prepend two
new lines
""")
```

## Insert code
Can be animated.
```python
dcode.insert_code((4, 2), "insert at the 3rd charachter of the 5th line")
dcode.insert_code((4, 2), """append to first two charachters of 5th line
new 6th line
new 7th line
prepend to the (5th line excluding the first two charachters)""")
```

## Remove code
Can be animated.
```python
# Remove first char of first line through 2nd char of 6th line.
dcode.remove_code((0, 0), (5, 2))

# Remove 6th line.
dcode.remove_code((5, 0), (6, 0))
```

## Clear code
Cannot be animated, will always be instantaneous.
```python
dcode.clear_code()
```

## Set code
Can be animated.
```python
# Replaces current code with mycode.
dcode.set_code(code=mycode)
```

## Background
The background can be resized independently of the code. This is useful if you want to create a space and then add code into it dynamically.
```python
dcode.set_background_width(width)
dcode.set_background_height(height)
dcode.set_background_size(width, height)

# to fit the background to the current code
dcode.set_background_width('auto')
dcode.set_background_height('auto')
dcode.set_background_size('auto', 'auto')
```
Note, by default the background does not resize with code changes as a common approach is to create a larger background space and then add code to it dynamically. If you want the background to auto-adjust to your code changes, pass `autosize=True` or `autowidth=True` or `autoheight=True` as kwargs to any of the edit actions.

## TODO
- Fix slight vertical misalignment between newly inserted glyphs and previous glyphs which can sometimes occur.
- Scroll animation.
