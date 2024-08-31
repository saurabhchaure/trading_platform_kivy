# KV Language

## Concept behind the language
As your application grows more complex, it's common that the costruction of widget trees and explicit declaration of bindings becomes verbose and hard to maintain. The KV language is an attempt to overcome these shortcomings.

## How to load KV
There are two ways to load KV Code into your application
1. By name convention:
    Kivy looks for a kv file with the same name as your App class in lowercase, minus "App" if it ends with 'App' e.g.
    ```
    MyApp -> my.kv
    ```
    If this file defines a Root Widget it will be attached to the App's root attribute and used as the base of the application widget tree.

2. `Builder`: You can tell Kivy to directly load a string or a file. If this string or file defines a root widget, it will be returned by the method:
    ```
    Builder.load_file('path/to/file.kv')
    ```
    or:
    ```
    Builder.load_string(kv_string)
    ```

## Rule Context
A Kv source constitutes of rules which are used to describe the content of a Widget. You can have one root rule, and any number of class or template rules.

The root rule is declared by declaring the class of your root widget, without any indentation, followed by : and will be set as the root attribute of the App instance:
```
Widget:
```
A class rule, declared by the name of a widget class between < > and followed by :, defines the appearance and behavior of any instance of that class:
```
<MyWidget>:
```
Rules use indentation for delimitation, like Python. Indentation should be four spaces per level, like the Python style guide recommends.

There are three keywords specific to the Kv language:
- app: always refers to the instance of your application.
- root: refers to the base widget/template in the current rule
- self: always refer to the current widget

## Special Syntax
There is a special syntax to define values for the whole Kv context.
To access Python modules and classes from kv, use `#:import`
```
#:import name x.y.z
#:import isdir os.path.isdir
#:import np numpy
```
is equivalent to:
```
from x.y import z as name
from os.path import isdir
import numpy as np
```
in Python.

To set a global value, use `#:set`
```
#:set name value
```
is equivalent to:
```
name = value
```
in Python.

## Instantiate children
To declare a widget instance of some class as a child widget, just declare that child inside the rule:
```
MyRootWidget:
    BoxLayout:
        Button:
        Button:
```
The example above defines that our root widget, an instance of MyRootWidget, has a child that is an instance of the `BoxLayout`, and that BoxLayout further has two children, instances of the `Button` class.

The Python equivalent of this code might be:
```
root = MyRootWidget()
box = BoxLayout()
box.add_widget(Button())
box.add_widget(Button())
root.add_widget(box)
```
Which you may find less nice, both to read and to write.
Of course, in Python, you can pass keyword arguments to your widgets at creation to specify their behaviour. For example, to set the number of columns of a gridlayout, we would do:
```
grid = GridLayout(cols=3)
```
To do the same thing in kv, you can set properties of the child widget directly in the rule:
```
GridLayout:
    cols: 3
```

The value is evaluated as a Python expression, and all the properties used in the expression will be observed, that means that if you had something like this in Python (this assume self is a widget with a data `ListProperty`):
```
grid = GridLayout(cols=len(self.data))
self.bind(data=grid.setter('cols'))
```
To have your display updated when your data change, you can now have just:
```
GridLayout:
    cols: len(root.data)
```
---
**Note**

Widget names should start with upper case letters while property names should start with lower case ones. Following the PEP8 Naming Conventions is encouraged.
---

## Event Bindings
You can bind to events in Kv using the “:” syntax, that is, associating a callback to an event:
```
Widget:
    on_size: my_callback()
```
You can pass the values dispatched by the signal using the args keyword:
```
TextInput:
    on_text: app.search(args[1])
```
More complex expressions can be used, like:
```
pos: self.center_x - self.texture_size[0] / 2., self.center_y - self.texture_size[1] / 2.
```
This expression listens for a change in `center_x`, `center_y`, and `texture_size`. If one of them changes, the expression will be re-evaluated to update the pos field.

you can also handle`on_` events inside your kv language. For example the TextInput class has a `focus` propery whose auto-generated `on_focus` event can be accessed inside the kv language like so:
```
TextInput:
    on_focus: print(args)
```

## Extend Canvas
Kv lang can be used to define the canvas instructions of your widget like this:
```
MyWidget:
    canvas:
        Color:
            rgba: 1, .3, .8, .5
        Line:
            points: zip(self.data.x, self.data.y)
```
And they get updated when properties values change.
Of course you can use canvas.before and canvas.after.

## Referncing Widgets
In a widget tree there is often a need to access/reference other widgets. The Kv Language provides a way to do this using id’s. Think of them as class level variables that can only be used in the Kv language. Consider the following:
```
<MyFistWidget>:
    Button:
        id: f_but
    TextInput:
        text: f_but.state

<MySecondWidget>:
    Button:
        id: s_but
    TextInput: s_but.state
```
An `id` is limited in scope to the rule it is declared in, so in the code above `s_but` can not be accessed outside the `<MySecondWidget>` rule.

**Warning:**
When assigning a value to id, remember that the value isn’t a string. There are no quotes: good -> `id: value`, bad -> `id: 'value'`

An `id` is a `weakref` to the widget and not the widget itself. As a consequence, storing the id is not sufficient to keep the widget from being garbage collected. To demonstrate:
```
<MyWidget>:
    label_widget: label_widget
    Button:
        text: 'Add Button'
        on_press: root.add_widget(label_widget)
    Button:
        text: 'Remove Button'
        on_press: root.remove_widget(label_widget)
    Label:
        id: label_widget
        text: 'widget'
```
Although a reference to `label_widget` is stored in `MyWidget`, it is not sufficient to keep the object alive once other references have been removed because it’s only a `weakref`. Therefore, after the remove button is clicked (which removes any direct reference to the widget) and the window is resized (which calls the garbage collector resulting in the deletion of `label_widget`), when the add button is clicked to add the widget back, a `ReferenceError: weakly-referenced object no longer exists` will be thrown.

To keep the widget alive, a direct reference to the `label_widget` widget must be kept. This is achieved using `id.__self__` or `label_widget.__self__` in this case. The correct way to do this would be:
```
<MyWidget>:
    label_widget: label_widget.__self__
```

## Accessing Widget defined inside Kv lang in your Python Code
Consider the code below in my.kv:
```
<MyFirstWidget>:
    # both these variables can be the same name and this doesn't lead to
    # an issue with uniqueness as the id is only accessible in kv.
    txt_inpt: txt_inpt
    Button:
        id: f_but
    TextInput:
        id: txt_inpt
        text: f_but_state
        on_text: root.check_status(f_but)
```

In myapp.py
```
class MyFirstWidget(BoxLayout):
    txt_inpt = ObjectProperty(None)

    def check_status(self, btn):
        print('button state is: {state}'.format(state=btn.state))
        print('text input text is: {txt}'.format(txt=self.txt_inpt))
```
txt_inpt is defined as a `ObjectProperty` initialized to None inside the Class.
```
txt_inpt = ObjectProperty(None)
```
At this point self.txt_inpt is None. In Kv lang this property is updated to hold the instance of the `TextInput` referenced by the id txt_inpt.:
```
txt_inpt: txt_inpt
```

From this point onwards, self.txt_inpt holds a reference to the widget identified by the id txt_input and can be used anywhere in the class, as in the function check_status. In contrast to this method you could also just pass the id to the function that needs to use it, like in case of f_but in the code above.

There is a simpler way to access objects with id tags in Kv using the ids lookup object. You can do this as follows:
```
<Marvel>:
    Label:
        id: loki
        text: 'Loki: I AM YOUR GOD!'
    Button:
        id: hulk
        text: 'Press to smash loki'
        on_release: root.hulk_smash()
```
In your Python Code:
```
class Marvel(BoxLayout):
    def hulk_smash(self):
        self.ids.hulk.text = "hulk: puny god!"
        self.ids["loki"].text = "Loki: >_<!!!" #alternative syntax
```
When your kv file is parsed, kivy collects all the widgets tagged with id’s and places them in this self.ids dictionary type property. That means you can also iterate over these widgets and access them dictionary style:

```
for key, val in self.ids.items():
    print("key={0}, val={1}".format(key, val))
```

**Note**
Although the self.ids method is very concise, it is generally regarded as ‘best practice’ to use the ObjectProperty. This creates a direct reference, provides faster access and is more explicit.

## Dynamic Classes

Consider the code below:
```
<MyWidget>:
    Button:
        text: "Hello world, watch this text wrap inside the button"
        text_size: self.size
        font_size: '25sp'
        markup: True
    Button:
         text: "Even absolute is relative to itself"
        text_size: self.size
        font_size: '25sp'
        markup: True
    Button:
        text: "Repeating the same thing over and over ina comp = fail"
        text_size: self.size
        font_size: '25sp'
        markup: True
    Button:
```

Instead of having to repeat the same values for every button, we can just use a template instead, like so:
```
<MyBigButton@Button>:
    text_size: self.size
    font_size: '25sp'
    markup: True

<MyWidget>:
    MyBigButton:
        text: "Hello world, watch this text wrap inside the button"
    MyBigButton:
        text: "Even absolute is relative to itself"
    MyBigButton:
        text: "repeating the same thing over and over in a comp = fail"
    MyBigButton:
```
This class, created just by the declaration of this rule, inherits from the Button class and allows us to change default values and create bindings for all its instances without adding any new code on the Python side.


