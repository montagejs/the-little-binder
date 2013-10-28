
# The Zen of Montage

Montage and Functional Reactive Bindings are born of the same school of
thought, which I have tried to distil in this, “The Zen of Montage”.  The Zen
subscribes to the form established by [The Zen of Python][PEP20].  The
philosophies are complementary, except when they are not.

[PEP20]: http://www.python.org/dev/peps/pep-0020/

- Draw only when drawing. View only when viewing.
- Views are consistent between draws.
- Models are consistent between events.
- Models need not be consistent between statements.
- A view-model is a model.
- Reactive state is better than pollable state.
- All state is expressible as properties and collections.
- Composition is better than inheritance.
- Declarativ is better than imperative.
- Bindings are better than change listeners.
- Bindings are not always sufficient.
- Sometimes, two bindings are better than one.
- Reactive state must gracefully handle transient inconsistencies.
- Null and undefined can mean uninitialized, inconsistent, or invalid.
- Null and undefined are contagious. Everything they touch turns to null, but
  not permanently.
- Null is better than NaN.
- Defaults should be null and should not imply any particular option.
- Constructor arguments should be optional and equivalent to setting initial
  state.
- A sequence of changes should arrive at the same state regardless of the order in which they occur. (“The Path Independence Principle”)
- Changes to any state should be observable, including side-effects.
- An operator's name's meaning may be universal, but it may not always work the same way.  (“The ‘Moar Ducks’ Principle”)




## Draw only when drawing. View only when viewing.

I would go on to say “Model only when modelling” apart from it being nonsense out of context. The model should not change while you’re drawing. Components depend on having a consistent model from the beginning of a draw to its end.

The model should be invariant during a draw, and the DOM should be invariant between draws. The only good time to draw is when your component is instructed to draw.

In the Flow, we perform frustum culling.


