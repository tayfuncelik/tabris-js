# Internal API

## tabris.registerType

This method registers a new native type (a proxy) that can be instantiated via `tabris.create`. If the registered type is a widget, use `tabris.registerWidget`.

Usage:

    tabris.registerType(type, members);

The parameter `type` is the type name used to create instances. Type names should begin with an upper-case character, internal types are prefixed with an underscore. The type name is available on instances in a field `type`. A constructor of this name will be created in the `tabris` object in order to enable instanceof checks, e.g. `widget instanceof tabris.MyType`. This constructor should never be used directly.

All fields on the `members` map will be inherited by the type instances (potentially overwriting members of `tabris.Proxy`), except for the following special keys. These keys will be attached to the constructor instead and change the proxy's behavior:

* `_type`: The native type to be used in the `create` operation. If missing, the `type` name will be used.
* `_supportsChildren`: Unless this is set to true, the instance will throw exceptions when attempting to append any children.
* `_properties`: A map where the keys are all names of the supported properties, and the values determine how the property values are checked and encoded for `set` operations. (See list below.) For properties not in the list no `set` operations will be issued and a console warning is logged instead.
* `_internalProperties`: A map with key-value pairs that are added to any `create` operation for that type, bypassing all checks, encoding and `_setProperty` handler.
* `_setProperty`: A map where the keys are the names of any of the supported properties (as given in `_properties`), and the value is a function that is called instead of creating a `set` operation for this property change. The function is called with the (encoded) property value as the sole argument and the proxy as the context (`this`). It can use the `_setPropertyNative` method of the proxy is needed.
* `_getProperty`: A map where the keys are the names of any of the supported properties (as given in `_properties`), and the value is a function that is called instead of creating a `get` operation for this property. The function is called with the proxy as the context (`this`) and must return the current value of this property. It can use the `_getPropertyNative` method of the proxy is needed.
* `_listen`: A map where the keys are the names of all supported events, and the value determines how the native `listen` operation is issued. (See list below.) For events that are not in the map no `listen` operations are issued and a console info logged instead.
* `_trigger`: A map where the keys are the names of events that may be issued in a `notify` operation, and the value determines how the operation is handled (see list below). For events that are not in the map, the operation's event name and parameters will be passed to the proxy's `trigger` method without any changes.

Valid values for the `_properties` map are:

* `true`: Accepts any value.
* `"boolean"`: Accepts `true` or `false`, all other values are converted as determined by the JavaScript language ("truthy"/"falsy" values).
* `"string"`:  Accepts any strings, any non-string is converted as determined by the JavaScript language (e.g. `23.0` becomes `"23"`).
* `"natural"`: Accepts any number equal or greater than zero and smaller than infinity, with any non-round number being rounded. Any other value (including `NaN`) is rejected.
* `"integer"`: Accepts any number equal or greater than -infinity and smaller than infinity, with any non-round number being rounded. Any other value (including `NaN`) is rejected.
* `"color"`: Accepts any value describing a color as outlined in the Tabris.js documentation, other values are rejected.
* `"font"`: Accepts any value describing a color as outlined in the Tabris.js documentation, other values are rejected.
* `"image"`: Accepts any value describing an image as outlined in the Tabris.js documentation, other values are rejected.
* `"layoutData"`: Accepts any object describing layoutData as outlined in the Tabris.js documentation, with invalid keys being ignored.
* `"bounds"`: Accepts any any object describing bounds as outlined in the Tabris.js documentation. Invalid values are currently not rejected or converted.
* `["choice", ["op1", "op2", ...]]` allows `"op1"`, `"op2"`, etc. Other values are rejected.
* `["choice", ["op1": "encodedOp1", ...}]` allows `"op1"` and encodes it as `"encodedOp1"`. Other values are rejected.

Rejected values will not be passed on in a `set` operation and cause a console warning to be logged.

Valid values for the `_listen` map are:

* `true`: passes the name of the event to the `listen` operation without any changes.
* A *string*: will be used as the event name in the `listen` operation instead of the public event name.
* A *function*: will be called with the new listen state (`true`/`false`) as the sole argument and the proxy as the context (`this`). It can use the `_nativeListen` method of the proxy if needed.

Valid values for the `_trigger` map are:

* A *string*: will be used as a public event name (as given in `_listen`) instead of the `notify` operation's event name.
* A *function*: will be called instead of the proxy's `trigger` method, with the event object as the sole argument and the proxy as the context (`this`). It can use the `trigger` method of the proxy if needed.

## tabris.registerWidget

Acts as wrapper for `tabris.registerType`. The API is identical, except that the `_properties`, `_setProperty`, `_getProperty`, `_listen` and `_trigger` maps are extended to add support for  properties and events common to all widgets.

## tabris.Proxy

All objects created with `tabris.create` extend this type. Aside from the public methods described in the Tabris.js documentation as widget API, it has the following internal methods that may be useful:

 * `_create(properties)`: Called once to issue the `create` operation for this instance. May be overwritten to execute some additional tasks or modify the operation, but a super call should always be included in the new implementation, e.g. `this.super("_create", properties)`.
 * `_setParent(parent)`: Called when the proxy is appended to another proxy. Issues a `set` parent operation and adds the proxy to the parent's children list. May be overwritten to execute some additional tasks or to replace/reject the parent. A super call may be included in the new implementation, e.g. `this.super("_setParent", parent)`.
 * `_checkDisposed()`: Throws an exception if the proxy has been disposed. Should be called before issuing any operations using the methods listed below.
 * `_nativeListen(event, state)`: Issues a `listen` operation. May be useful for any function in a `_listen` map.
 * `_setPropertyNative(name, value)`: Appends a property to the next `create` or `set` operation to be issued, without any checks or encoding. May be useful for any function in a `_setProperty` map or in `_create`.
 * `_getPropertyNative(name)`: Issues a `get` operation and returns the result. May be useful for any function in a `_getProperty` map.