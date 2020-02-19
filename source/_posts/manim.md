---
title: 使用manim制作数学动画
tags: ["animation", "python"]
categories: ["tools"]
reward: true
copyright: true
date: 2020-02-14 15:52:55
thumbnail: manim/manim-logo.png
---





本文介绍如何使用manim制作动画。

<!--more-->



## 基本结构

```python
from manimlib.imports import *

class NameOfScene(Scene):
    def construct(self):
        # Animation progress
```

## 基本运行方式

```shell
python -m manim example_scenes.py NameOfScene -pl
```

+ `p` 表示preview，即在渲染完成后立即播放
+ `l` 表三 `low quality` ，以 480p 15fps 渲染

更多选项：

```
-h,    --help show this help message and exit
-p,    --preview
-w,    --write_to_movie
-s,    --show_last_frame
-l,    --low_quality
-m,    --medium_quality
-g,    --save_pngs
-f,    --show_file_in_finder
-t,    --transparent
-q,    --quiet
-a,    --write_all
-o OUTPUT_NAME,    --output_name OUTPUT_NAME
-n START_AT_ANIMATION_NUMBER,    --start_at_animation_number START_AT_ANIMATION_NUMBER
-r RESOLUTION,    --resolution RESOLUTION
-c COLOR,    --color COLOR
-d OUTPUT_DIRECTORY,    --output_directory OUTPUT_DIRECTORY
```

## 基本形状

### 点

```python
class TutorialShapeDot(Scene):
    def construct(self):
        dot = Dot(ORIGIN, buff=0.1)
        self.add(dot)
```



### 线段

```python
class TutorialShapeLine(Scene):
    def construct(self):
        line = Line(LEFT, ORIGIN)
        self.add(line)
```



### 圆弧

```python
class TutorialShapeArc(Scene):
    def construct(self):
        arc = Arc(0, PI/2).scale(2)
        self.add(arc)
```



### 圆形

```python
class TutorialShapeCircle(Scene):
    def construct(self):
        circle = Circle()
        self.add(circle)
```



### 方形

```python
class TutorialShapeSquare(Scene):
    def construct(self):
        square = Square()
        square.move_to(LEFT)
        self.add(square)
```



### 矩形

```python
class TutorialShapeRectangle(Scene):
    def construct(self):
        rectangle = Rectangle()
        rectangle.set_width(2)
        rectangle.set_height(1)
        self.add(rectangle)
```



### 多边形

```python
class TutorialShapePolygon(Scene):
    def construct(self):
        polygon = Polygon(np.array([-1,0,0]), np.array([1,0,0]), np.array([2,1,0]), np.array([2,2,0]))
        self.add(polygon)
```



### 箭头

```python
class TutorialShapeArrow(Scene):
    def construct(self):
        arrow = Arrow().scale(1.5)
        self.add(arrow)
```



### 双向箭头

```python
class TutorialShapeDoubleArrow(Scene):
    def construct(self):
        double_arrow = DoubleArrow().scale(1.5)
        double_arrow.move_to(UR) 
        self.add(double_arrow)
```



### 向量

```python
class TutorialShapeVector(Scene):
    def construct(self):
        vector = Vector().scale(1.5)
        vector.move_to(DR) 
        self.add(vector)
```



### 数学公式

```python
class TutorialShapeTex(Scene):
    def construct(self):
        tex = TexMobject(r"\frac{d}{dx} f(x)g(x)").set_color(RED)
        tex.move_to(UP)
        self.add(tex)
```



### 文本

```python
class TutorialShapeText(Scene):
    def construct(self):
        text = TextMobject(r"Text with LaTeX $\frac{1}{2}$", color=RED)
        text.move_to(DOWN)
        self.add(text)

        text = Text(r"Text without LaTeX 1/2", color=RED)
        text.move_to(UP)
        self.add(text)
```



## 基本动画

### 淡入 / 淡出

```python
class AnimationFadeInOut(GraphScene):
    CONFIG = {  # 设置坐标轴
        "x_min": -5,
        "x_max": 5,
        "y_min": -4,
        "y_max": 4,
        "graph_origin": ORIGIN,  # 原点
    }
    def construct(self):
        self.setup_axes() # 显示坐标轴
        polygon = Polygon(np.array([-1,0,0]), np.array([1,0,0]), np.array([2,1,0]), np.array([2,2,0]))
        self.play(FadeIn(polygon), run_time=2)
        self.play(FadeOut(polygon), run_time=2)

```



### 显示生成过程 (描边)

```python
class AnimationShowCreation(GraphScene):
    CONFIG = {  # 设置坐标轴
        "x_min": -5,
        "x_max": 5,
        "y_min": -4,
        "y_max": 4,
        "graph_origin": ORIGIN,  # 原点
    }
    def construct(self):
        self.setup_axes() # 显示坐标轴
        polygon = Polygon(np.array([-1,0,0]), np.array([1,0,0]), np.array([2,1,0]), np.array([2,2,0]))
        self.play(ShowCreation(polygon))

```



### 平移

```python
class AnimationMove(GraphScene):
    CONFIG = {  # 设置坐标轴
        "x_min": -5,
        "x_max": 5,
        "y_min": -4,
        "y_max": 4,
        "graph_origin": ORIGIN,  # 原点
    }
    def construct(self):
        self.setup_axes() # 显示坐标轴
        square = Square()
        self.play(ShowCreation(square))
        self.play(
            square.move_to, DOWN,
            rate_func=linear,
            run_time=1
        )
```



### 旋转

```python
class AnimationRotate(GraphScene):
    CONFIG = {  # 设置坐标轴
        "x_min": -5,
        "x_max": 5,
        "y_min": -4,
        "y_max": 4,
        "graph_origin": ORIGIN,  # 原点
    }
    def construct(self):
        self.setup_axes() # 显示坐标轴
        square = Square()
        self.play(ShowCreation(square))
        self.play(Rotate(square, PI, IN))  # 第二个参数为旋转角度，第三个参数为旋转轴(IN表示垂直屏幕向里), 旋转的方向为右手螺旋

```



### 变换

```python
class AnimationTransform(Scene):
    def construct(self):
        square = Square()
        circle1 = Circle()
        circle2 = Circle()
        circle1.set_color(GREEN)
        circle2.set_color(RED)
        circle2.next_to(circle1) # 让circle2在circle1右边
        self.play(ShowCreation(circle1), ShowCreation(circle2)) # 同时播放多个动画
        self.play(Transform(circle1, square)) # 变换过后 circle1和square是同一个对象

```



### Write

```python
class AnimationWrite(Scene):
    def construct(self):
        square = Square()
        square.set_color(WHITE)
        square.set_fill(RED, opacity=0.5)
        self.play(Write(square))
        self.wait(2)
```



### updater

```python
class UpdaterExample(Scene):
    def construct(self):
        text = Text("text")
        dot = Dot()
        text.next_to(dot)

        def update(obj):
            obj.next_to(dot) # 让obj近挨dot
        text.add_updater(update) # 将该updater添加给text，即随着dot的移动，text会紧跟dot，保持相对静止

        self.add(dot)
        self.add(text)
        self.play(dot.move_to, DOWN, run_time=2)
        
        text.remove_updater(update) # 将updater从text中移除
```



## 绘制函数曲线

```python
class AnimationFunction(GraphScene):
    CONFIG = {  # 设置坐标轴
        "x_min": -5,
        "x_max": 5,
        "y_min": -4,
        "y_max": 4,
        "graph_origin": ORIGIN,  # 原点
    }
    def construct(self):
        self.setup_axes() # 显示坐标轴
        graph = self.get_graph(lambda x: x**2, color = GREEN)
        self.play(ShowCreation(graph))
```



## 外部导入

### SVG

```python
class OutsideSVGImage(Scene):
    def construct(self):
        svg_image = SVGMobject("test.svg")
        svg_image_scale = svg_image.copy().scale(2)
        self.play(ShowCreation(svg_image), run_time=2)
        self.play(Transform(svg_image, svg_image_scale))
```

