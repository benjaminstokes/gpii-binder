# `gpii-binder`

This package provides a mechanism for binding [Fluid view component](http://docs.fluidproject.org/infusion/development/tutorial-gettingStartedWithInfusion/ViewComponents.html) model variables to DOM elements using [selectors](http://docs.fluidproject.org/infusion/development/tutorial-gettingStartedWithInfusion/ViewComponents.html#selectors).

You can bind to any DOM element whose value can be read and set using
[`fluid.value`](http://docs.fluidproject.org/infusion/development/ViewAPI.html#fluid-value-nodein-newvalue-), but the
primary (and tested) use case is binding model variables to form elements, specifically:

* text `<input>` fields
* radio `<input>` fields
* checkbox `<input>` fields
* `<textarea>` fields
* `<select>` fields

Once you run `gpii.binder.applyBinding(component)` (see "Static Functions" below), a binding is created
between any selectors and model variables referenced in`options.binding` (see "Supported options" for the format).

Once a binding exists, changes to a bound model sent using [the change applier](http://docs.fluidproject.org/infusion/development/ChangeApplier.html)
are used to update the DOM element's value.

The binding is bidirectional.  Change events to a bound DOM element's value are also relayed to the associated model
variable.  Note that change events are not generated when you directly set the element's value, but only when you have
updated the value using browser events *and* change focus.  For more details, see the
[jQuery documentation for the change event](https://api.jquery.com/change/).


# Supported options

The `gpii.binder.applyBinding` function provided by this package can only do its work if you have the
following options defined:

| Option             | Type     | Description |
| ------------------ | -------- | ----------- |
| `selectors` | `{Object}` | You must define one or more [selectors](http://docs.fluidproject.org/infusion/development/tutorial-gettingStartedWithInfusion/ViewComponents.html#selectors) that can be used in a binding. |
| `bindings` | `{Object}` | Defines the relationship between selectors and model variables.  The full notation for this option is outlined below. |

## Long notation

There are two ways of specifying bindings.  The "long form" has named keys (as in the first example above) and
supports the following options:

* selector: A valid selector for your component.  Must be able to be resolved using `that.locate(selector)`
* path: A valid path for the model variable whose value will be watched.  Must be able to be resolved using `fluid.get(that.model, path)`.

The "long form" looks like:

    bindings: {
        "<key>": {
            selector: "<selector1>",
            path:     "<path1>"
        }
    }


## Short notation

The "short form" uses the selector as the key, and the path as a string value (as in the second example above).

    bindings: {
        "<selector2>": "<path2>"
    }


## Combining the two notations

You can use both forms together, as in:

    bindings: {
        "<key>": {
            selector: "<selector1>",
            path:     "<path1>"
        },
        "<selector2>": "<path2>"
    }

## Additional long notation options

`unidirectional` when true creates one way (model to page) data binding
* type: boolean. 
* default: false

`method` - The function used to access the page element.
* type: function. 
* default: fluid.value
* Must have signature of `function(element, value)`.

    bindings: {
        "<key>": {
            selector: "<selector1>",
            path:     "<path1>",
            unidirectional: true|false,
            method: <fluid.value|fluid.text|fluid.html>
        }
    }



# Static Functions

## `gpii.binder.applyBinding(component)`
* `component` `{Object}` - A fluid `viewComponent` with both `selectors` and `bindings` options defined (see above).
* Returns: Nothing.

You must explicitly invoke this function to create bindings.  Generally you wil do this from a
[listener definition](http://docs.fluidproject.org/infusion/development/InfusionEventSystem.html#registering-a-listener-to-an-event).
For example, if all required markup already exists on startup, you can simply bind to the "onCreate" event, as in:

    listeners: {
        "onCreate.applyBindings": {
            "funcName": "gpii.binder.applyBinding",
            "args":     "{that}"
        }
    }

For an example of using the binder with static markup, see the tests in this package.

### Bindings and dynamic markup

If your component generates or regenerates markup, you will need to call `gpii.binder.applyBinding(component)`
whenever it changes.  Best practice is to call `fluid.initDomBinder` when you have finished manipulating the DOM and
then listen for the `viewComponent`'s `onDomBind` event, as in:

    listeners: {
        "onCreate.applyBindings": {
            "funcName": "gpii.binder.applyBinding",
            "args":     "{that}"
        }
    }

See the `templateAware` grade in [the `gpii-handlebars` package](https://github.com/GPII/gpii-handlebars) for a working
example of dynamically updating bindings.