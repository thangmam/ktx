# KTX: general `Scene2D` utilities

Extensions and utilities for stages, actors, actions and event listeners.

### Why?

`Scene2D` API is not that easy to extend in the first place - due to hidden fields and methods, sometimes copy-pasting
a widget class and modifying the relevant lines is the only way to implement an actor "extension". Most event listeners
cannot be used with Kotlin's pleasant lambda syntax. Actions API is pretty powerful, but can be nearly unreadable when
it comes to chaining.

### Guide

#### Actors

- Null-safe `Actor.isShown()` method was added to make it possible to check if actor is currently on a `Stage`.
- `Actor.centerPosition` extension method was added to allow quick actor centering, hiding the necessary math.
- `Group.contains(Actor)` method was added to support `in` operator. You can check if an `Actor` is a direct child of
a `Group` with `actor in group` syntax.
- `Group` and `Stage` support actor adding and removal through `+` and `-` operators.
- `Stage.contains(Actor)` method was added to support `in` operator. This will report `true` if the `Actor` is on the
`Stage` (it does not have to be a direct child of `Stage` root group).
- `Actor.alpha` and `Stage.alpha` inlined extension properties were added to support easy modification of `Color.a` value.
- `Actor.setKeyboardFocus` and `.setScrollFocus` allow to quickly (un)focus the actor on its stage.

#### Event listeners

- Lambda-compatible `Actor.onChange` method was added. Allows to listen to `ChangeEvents`.
- Lambda-compatible `Actor.onClick` method was added. Attaches `ClickListeners`.
- Lambda-compatible `Actor.onKey` method was added. Allows to listen to `InputEvents` with `keyTyped` type.
- Lambda-compatible `Actor.onKeyDown` and `Actor.onKeyUp` methods were added. They allow to listen to `InputEvents`
with `keyDown` and `keyUp` type, consuming key code of the pressed or released key (see LibGDX `Keys` class).
- Lambda-compatible `Actor.onScrollFocus` method was added. Allows to listen to `FocusEvents` with `scroll` type.
- Lambda-compatible `Actor.onKeyboardFocus` method was added. Allows to listen to `FocusEvents` with `keyboard` type.
- `KtxInputListener` is an open class that extends `InputListener` with no-op default implementations and type
improvements (nullability data).
- `onChangeEvent`, `onClickEvent`, `onKeyEvent`, `onKeyDownEvent`, `onKeyUpEvent`, `onScrollFocusEvent` and
`onKeyboardFocusEvent` `Actor` extension methods were added. They consume the relevant `Event` instances as lambda
parameters. Both listener factory variants are inlined, but the ones ending with *Event* provide more lambda parameters
and allow to inspect the original `Event` instance that triggered the listener. Regular listener factory methods should
be enough for most use cases.

#### Actions

- Global actions can be added and removed from `Stage` with `+` and `-` operators.
- `Action.then` *infix* extension function allows to easily create action sequences with pleasant syntax.

#### Widgets

- `txt` inlined extension properties added to `Label` and `TextButton` widgets. Since types of `getText` and `setText`
methods in both of this widgets are not compatible (get returns `StringBuilder`, set consumes a `CharSequence`), an
extension was necessary to let these widgets fully benefit from idiomatic Kotlin properties syntax. Since Kotlin
properties cannot overshadow Java methods, property was renamed to still hopefully readable `txt`.

### Usage examples

Centering actor on a stage:
```Kotlin
import ktx.actors.*

window.centerPosition()
```

Adding and removing actors with operators:
```Kotlin
import ktx.actors.*

table + button
table - label
stage + table
```

Checking if actor is on a stage or in a group:
```Kotlin
import ktx.actors.*

button in stage
button in table
```

Quickly accessing actor and stage alpha color value:
```Kotlin
import ktx.actors.*

label.alpha = 0.5f
stage.alpha = 0.2f
```

Focusing events on actors:
```Kotlin
import ktx.actors.*

textField.setKeyboardFocus(true)
scrollPane.setScrollFocus(false)
```

Adding a `ChangeListener` to an `Actor`:

```Kotlin
import ktx.actors.*

button.onChange {
  println("Button changed!")
}

button.onChangeEvent { changeEvent, actor ->
  // If you need access to the original ChangeEvent, use this expanded method variant.
  println("$actor changed by $changeEvent!")
}
```

Adding a `ClickListener` to an Actor:

```Kotlin
import ktx.actors.*

label.onClick { 
  println("Label clicked!")
}

label.onClickEvent { inputEvent, actor ->
  // If you need access to the original InputEvent, use this expanded method variant.
  println("$actor clicked by $inputEvent!")
}
label.onClickEvent { inputEvent, actor, x, y ->
  // If you need access to the local actor click coordinates, use this expanded method variant.
  println("$actor clicked by $inputEvent at ($x, $y)!")
}
```

Adding an `EventListener` which consumes typed characters:

```Kotlin
import ktx.actors.*

textField.onKey { key ->
  println("Typed $key char to text field!")
}

textField.onKeyEvent { inputEvent, actor, key ->
  // If you need access to the original InputEvent, use this expanded method variant.
  println("Typed $key char to $actor by $inputEvent!")
}
```

Adding `EventListeners` which listen to `FocusEvents`:

```Kotlin
import ktx.actors.*

// FocusEvent with scroll type:
scrollPane.onScrollFocus { focused ->
  println("Scroll pane is focused: $focused!")
}

scrollPane.onScrollFocusEvent { focusEvent, actor ->
  // If you need access to the original FocusEvent, use this expanded method variant.
  println("$actor is focused: ${focusEvent.isFocused}!")
}

// FocusEvent with keyboard type:
textField.onKeyboardFocus { focused ->
  println("Text field is focused: $focused!")
}

textField.onKeyboardFocusEvent { focusEvent, actor ->
  // If you need access to the original FocusEvent, use this expanded method variant.
  println("$actor is focused: ${focusEvent.isFocused}!")
}
```

Chaining actions (`SequenceAction` utility):
```Kotlin
import ktx.actors.*
import com.badlogic.gdx.scenes.scene2d.actions.Actions.*

val sequence = alpha(0f) then fadeIn(1f) then delay(1f) then fadeOut(1f)
actor + sequence // Adding action to the actor.
```

Adding and removing actions to stages and actors with operators:
```Kotlin
import ktx.actors.*

button + action - otherAction
stage + someAction // Adds action to stage root actor,
                   // affecting all actors on the stage.
```

Accessing and changing text of `Label` and `TextButton` widgets:

```Kotlin
import ktx.actors.*
import com.badlogic.gdx.scenes.scene2d.ui.Label
import com.badlogic.gdx.scenes.scene2d.ui.TextButton

val label = Label("text", skin)
label.txt // Returns "text".
label.txt = "new" // Changes current Label text to "new".

val button = TextButton("Click me!", skin)
button.txt // Returns "Click me!".
button.txt = "Drag me!" // Changes TextButton text to "Drag me!".
```

Extending `KtxInputListener`:

```Kotlin
import ktx.actors.KtxInputListener

class MyInputListener : KtxInputListener() {
  // Implement the methods that handle events you plan to listen to:
  override fun touchDown(event: InputEvent, x: Float, y: Float, pointer: Int, button: Int): Boolean {
    // Do something on mouse click.
    return true
  }
}
```

#### Migration guide

In **KTX** up to `1.9.6-b4`, extension methods `onChange`, `onClick`, `onKey`, `onKeyDown`, `onKeyUp`, `onScrollFocus`
and `onKeyboardFocus` consumed `Event` and `Actor` instances. This lead to common usage of `_, _ ->`, which was against
the goal of boilerplate-less listeners. That is why the existing listener factory methods where renamed with `Event`
suffix, and a new set of extension methods with the same names were added - this time consuming a minimal amount of
parameters. Add `Event` suffix to each of your listener methods or refactor them to the new API with less parameters.
For example, if you used `Actor.onChange` extension method, use `Actor.onChangeEvent` instead or remove the `event, actor`
parameters if you do not really need them at all.

### Alternatives

- [VisUI](https://github.com/kotcrab/vis-editor/wiki/VisUI) includes some `Scene2D` utilities, as well as some extended
widgets to address some of the LibGDX API problems. It is written in Java, though.
- [Kiwi](https://github.com/czyzby/gdx-lml/tree/master/kiwi) is a general purpose Guava-inspired LibGDX Java utilities
library, which contain some `Scene2D` helpers.
- [LibGDX Markup Language](https://github.com/czyzby/gdx-lml/tree/master/lml) makes it easier to build `Scene2D` views
thanks to its HTML-inspired syntax.

#### Additional documentation

- [Scene2D UI article.](https://github.com/libgdx/libgdx/wiki/Scene2d.ui)
