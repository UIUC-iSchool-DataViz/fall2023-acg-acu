---
title: Traitlets, Widgets, Interactivity and bqplot
layout: lecture
tags:
  - bqplot
  - interactivity
  - ipywidgets
  - jupyter
  - traitlets
description: >-
  We start with the basics of using Traitlets and data binding in visualization, and then introduce the basics of bqplot.
---

## Traits and Data

There are two underlying libraries we utilize for interactivity in Jupyter. The first, `traitlets`, provides methods for datatype-verification and "watching" for changes.

```#python
import traitlets

class MusicAlbum(traitlets.HasTraits):
    name = traitlets.Unicode()
    artist = traitlets.Unicode()
    year_released = traitlets.Int()

speak_now = MyObject(name = "Speak Now",
				     artist = "Taylor Swift",
				     year_released = 2010)
```

---

## Watching Traitlets

Once we have an object that has traits, we can watch that object for changes.

```#python

def name_changed(change):
    print(change['new'])

my_obj.observe(name_changed, ['name'])
```

In this case, we are watching the trait `name` for changes.  When a change occurs, the function `name_changed` is called.  The argument is a dict with these values:

- `new`: the new value the trait has
- `old`: the previous value
- `type`: the type of change
- `owner`: the object that owns this trait
- `name`: the name of the trait

---

## Default Values

We can also specify "default" values for traits, so that they can be generated at runtime from other values.  We do this by "decorating" a function with the `traitlets.default` call:

```#python

class Student(traitlets.HasTraits):
	name = traitlets.Unicode()
	row_and_seat = traitlets.Tuple(
						traitlets.Unicode(),
						traitlets.Int()
					)

	@traitlets.default("row_and_seat")
	def random(self):
		return (random.choice("ABCDEFG"),
				random.randint(1, 15))
```

---

## Linking

Two objects can have linked values, so that when one changes, the other changes as well.

```#python
class Student(traitlets.HasTraits):
	name = traitlets.Unicode()

s1 = Student(name = "MyName")
s2 = Student(name = "")

traitlets.link((s1, "name"), (s2, "name"))

s1.name = "My Real Name"
print(s2.name)
```

---

	## Widgets

We can use the `ipywidgets` library to build out widgets in Jupyter notebooks. These widgets can be quite extensive with many different operations; additionally, they can have substantial CSS styling.

We've used simple examples before.  For instance, we can create an interactive function in just a few lines:

```#python
import ipywidgets

@ipywidgets.interact(name = ['Weezer', 'Nerf Herder', 'Mustard Plug'])
def print_bandname(name):
    print(name)
```

This creates a dropdown that we can select an item from, which is supplied. What this is doing implicitly is creating a widget with a `value` attribute, and whenever that `value` is changed, the function is called again.

---

## Widget Types 1

Automatically creating widgets using `@ipywidgets.interact` is very handy and useful for quick operations, but we can do this more deliberately as well. There are a number of widgets available in `ipywidgets` already:

- `IntSlider`, `FloatSlider`, `IntRangeSlider`, `FloatRangeSlider`, `IntProgress` and `FloatProgress` all display or allow the user to choose values.
- `IntText`, `FloatText`, `BoundedIntText` and `BoundedFloatText` let the user input explicit values to a widget.

---

## Widget Types 2

There are additional widget types that can provide indicators or restricted selections.

- `ToggleButton`, `Checkbox` and `Valid` provide boolean indicators; `Valid` is read-only.
- For selection, there are `Dropdown`, `RadioButtons`, `Select`,
  `SelectionSlider` and several others.
- Strings can be provided using `Text`, `TextArea` and `HTML`.
- Actions can be enabled through `Button` objects.

Widgets can also be laid out using `HBox`, `VBox`, `Tab`, and `Accordion`.

---

## Events and Linking

In addition to watching for changes, we can watch for events and we can link two (or more) values between different widgets.

The special method `on_click` on a `Button` allows for a function to be called when something is clicked.  We can also link using `ipywidgets.link` and supplying traits.  For example:

```#python
m = MyObject(name = "Weezer", age=26)
l = ipywidgets.Label()
ipywidgets.link((m, 'name'), (l, 'value'))

display(l)

m.name = 'Nerf Herder'
```

**Exercise:** Add a button and make this change occur when clicked.


---

## bqplot

Our first engine, `bqplot`, is a Jupyter-based interactive plotting system.

It presents two principal interfaces:

- `pyplot`-like interface, for making the transition from matplotlib easier
- An object-oriented API for constructing interactive visualizations

We will be using the latter far more frequently than the former.

---

## bqplot

Now that we've learned a bit about widgets, we can start to dig into `bqplot`. `bqplot` is based around traitlets and widget objects; every object you work with will have traits and may be represented as a widget.

When we use `bqplot` we will be constructing `Figure` objects, which will contain `marks` and `axes`.  To use these, we will build mark objects (`Bars`, `Lines`, `Scatter`, `Map`, etc) and describe the relationship between points using `Scale` objects.

We will be building out these objects and their relationships to develop interactivity.

---

## bqplot objects

- A mark is some mechanism for displaying data.  For example, we might have data that has a set of x and y values, which we can use `Lines` to represent.
- `Scale` objects describe relationships between visual attributes (position) and data values.
- `Axis` objects are where data are placed.
- `Figure` objects contain marks and axes, as well as interaction.

---

## bqplot: Very Simple

Our first example will be a simple lineplot.

```#python
import bqplot
import numpy as np

x = np.arange(100)
y = np.random.random(100) + 5

x_sc = bqplot.LinearScale()
y_sc = bqplot.LinearScale()
lines = bqplot.Lines(x = x, y = y, scales = {'x': x_sc, 'y': y_sc})
ax_x = bqplot.Axis(scale = x_sc, label = 'X value')
ax_y = bqplot.Axis(scale = y_sc, label = 'Y value', orientation = 'vertical')
fig = bqplot.Figure(marks = [lines], axes = [ax_x, ax_y])
display(fig)
```

---

## More bqplot

With bqplot, we construct a set of objects that are related:

- Scales
- Axes
- Marks
- Figures
- Interactions

---

## Scales in bqplot

We have dealt primarily with quantitative scales.  bqplot provides several scales we can utilize:

- `LogScale`
- `LinearScale`
- `DateScale`
- `OrdinalScale`
- `ColorScale`
- A few more as well.

([documentation](https://bqplot.readthedocs.io/en/latest/_generate/bqplot.scales.Scale.html))

---

## Marks

bqplot has several different marks we can explore.  We will utilize a few more today:

- `HeatMap`
- `GridHeatMap`
- `Bars`
- `Graph`

([documentation](https://bqplot.readthedocs.io/en/latest/_generate/bqplot.marks.Mark.html))

---

## bqplot interaction

bqplot widgets are all based on ipywidgets.  This means we use the same systems for describing the two.

We add an interaction to a given figure via the `interaction` keyword argument to a figure.

---

## bqplot interactors

We will be able to use these different interaction methods:

- `FastIntervalSelector`
- `IndexSelector`
- `BrushIntervalSelector` & `BrushSelector`
- `MultiSelector`
- `LassoSelector`
- `HandDraw`
- `PanZoom`
- `Tooltip`


---

## Today: Let's Try Things

Today we are going to build comparisons with our (virtual) hands.

- Getting a working jupyterlab environment
- Beginning bqplot
- A Bit More traitlets
- Making things display nicely
