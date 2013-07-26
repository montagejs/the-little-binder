
FRB is like a parfait (everyone likes parfaits)—it has layers.
Throughout the following documentation, there will be FRB expressions,
written in a simplified form, that apply equally well at any layer.

For example, this is a simple binding from a source property to a target
property on an object.

```
target <- source;
```

If you import the `frb` module, represented within the package as 
`frb/bindings.js`, you will get an interface that can define, enumerate,
inspect, and cancel bindings that are associated with a particular
object.

```javascript
var Bindings = require("frb");
Bindings.defineBinding(object, "target", {"<-": "source"});
Bindings.cancelBinding(object, "target");
```

If you are using MontageJS, all types that specialize the `Montage` type
have `defineBinding` as a method.  It is common to use this layer in
Montage constructors.  This constructor defines a
``training <- trainer.trainees.get(id)`` binding.

```javascript
var Montage = require("montage").Montage;
exports.TrainingMontage = Montage.specialize({
    constructor: {
        value: function TrainingMontage() {
            this.super();
            this.defineBinding("trainee", {
                "<-": "trainer.trainees.get(id)"
            });
        }
    }
});
```

In Montage, you can also declare bindings in the Montage component
description, which you will often hear called "the serialization".  This
is a JSON-based language where you can define classes of objects and
connect them with properties, bindings, event listeners, and
localization information.  This example establishes that the component
will have the same binding, ``trainee <- trainer.trainees.get(id)``.

```json
{
    "owner": {
        "prototype": "ui/training.reel",
        "properties": {
            "element": {"#": "training"}
        },
        "bindings": {
            "trainee": {
                "<-": "trainer.trainees.get(id)"
            }
        }
    }
}
```

The `Bindings` interface is a thin layer that is responsible for
tracking the bindings associated with an object.  It is possible to
create bindings on an object without the per-object tracking.  The
disadvantage is that you will need to keep track of the `cancel`
function for your binding yourself or you will never be able to find and
cancel it.

Bindings declared at this layer do not appear in the binding descriptor
table for the object, do not interfere with attempts to declare bindings
on the same target path, and calling the `cancel` method *does not*
implicitly remove the binding from the object's binding descriptor
table.

With that taken into account, FRB provides a `bind` function that is
capable of expressing all the same bindings.  This is another rendition
of a ``target <- source`` binding.

```javascript
var bind = require("frb/bind");
var cancel = bind(object, "target", {"<-": "source"});
cancel();
```

This form strikes deeper into how FRB works internally and can be useful
if you are working on the same layer.  For a related but somewhat
tangential example, you can override the way that FRB observes
properties on a particular object.  FRB delegates to `observeProperty`
on objects that implement it.  The `observeProperty` method may return a
canceler, and the `emit` function may provide one.  This makes observers
very plugable, so canceling the parent observer implicitly cancels all
of its transitive children.

```javascript
var Observers = require("frb/observers");
Nachos.prototype.observeProperty = function (name, emit, scope) {
    if (name === "delicious") {
        // there is no need to set up a property change listener, nor
        // even check the actual value.
        return emit(true);
    } else {
        return Observers.observeProperty(name, emit, scope);
    }
};
```

