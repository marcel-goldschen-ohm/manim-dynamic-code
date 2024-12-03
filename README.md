# manim-dynamic-code
Dynamic code animations in Manim.

## Why not just use the default `Code` Mobject?
Because editing the code in a `Code` Mobject after initial creation is absolute hell. There are multiple reasons for this including:
1. `Code` Mobjects have multiple references to individual SVG glyphs across multiple groups which makes removing the glyphs from the scene a nightmare.
2. Indexes of code string characters and `Code` Mobject SVG glyphs does not necessarily  match as there are no glyphs for whitespace, which makes editing the displayed code non-straightforward.
3. `Transform` animations from one `Code` Mobject to another look terrible (my opinion), which means custom animation sequences are required.
4. Animations which add/remove code glyphs create new references to the animated glyph objects which is an absolute nightmare if you need to keep track of those references (which you do if you want to perform additional edits to the code later). Thus, custom logic that resets glyph references after an animation where the code changes is needed.

`DynamicCode` does all of the above.

In my personal opinion, I find this solution incredibly hacky. It seems to me to be exposing some serious deficiencies in the way Manim does things including 1) the absolute horror of not being able to rely on an object reference after an animation, and 2) the nightmare of not having a straighforward scene hierarchy (e.g., cannot remove an object from the scene if it belongs to some other group you don't know about).

Caveat, I am rather new to Manim, so if I am just going about things the wrong way, please PLEASE let me know about it.

## Install
Just put `DynamicCode.py` where your project can find it and import it.

## Instantiate just like `Code` Mobjects
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
```

## Append code (with animation)
```python
# self is the parent Scene
dcode.append_code("append to last line", player=self, run_time=1)
dcode.append_code("append to 4th line", line_index=3, player=self, run_time=1)
dcode.append_code("""
append two
new lines
""", player=self, run_time=1)
```

## Animation or not?
For most of the actions described here, if `kwargs['player']` is defined and references the parent scene, then the action will be animated, otherwise it will be instantaneous. See above examples for appending code.

An unfortuante side-effect of this hacky way to animate code change sequences is that the animations cannot be played simultaneously with other animations. I'm not sure how to work around this at the moment.

## Prepend code
```python
dcode.prepend_code("prepend to first line")
dcode.prepend_code("prepend to 4th line", line_index=3)
dcode.prepend_code("""prepend two
new lines
""")
```

## Insert code
```python
dcode.insert_code((4, 2), "insert at the 3rd charachter of the 5th line")
dcode.insert_code((4, 2), """append to first two charachters of 5th line
new 6th line
new 7th line
prepend to the (5th line excluding the first two charachters)""")
```

## Remove code

## Clear code

## Set code
