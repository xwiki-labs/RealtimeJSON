# Listmap API Specification

The purpose of this specification is to create a standard way to synchronize JSON data types (specifically lists, maps and strings) in realtime across multiple computers for collaborative document editing.

The means by which the Listmap object is created is not part of the specification, it is specific to the implementation and it is where any configuration and setup must take place.

Examples of how a Listmap object might be created:

```javascript

var proxy = ListMap.create(configuration).proxy;

```

# Create a proxy

## Listmap.create -> Object

This function creates the realtime session and the proxy and requires a configuration object :

```javascript
ListMap.create(config)
```

The `config` object should contain everything the implementation need to work.

Listmap.create() returns an object containing at least the following property :
* proxy
  - the realtime list or map that was created based on your supplied `data` attribute

## Listmap.create().proxy -> Proxy

This function returns an object containing a proxy around an array or an object, this proxy will intercept any changes to this object and will make proxies out of any objects which are pointed to from this object.

When a field of the object is set with another value, for example:

```javascript

var proxy = Listmap.create(options).proxy; // Create a new proxy

var subObject = { value: 1 }; // Create a new non-realtime object

proxy["key"] = subObject; // Assign the non-realtime object to key of the proxy

```

subObject will be a proxy, changes to the proxy["key"] object will change subObject and changes to subObject will change proxy["key"]. 
All these modifications will also be replicated for all the realtime users.

# Proxy Events

These events must be applied on the root proxy. If you want an event listener ("change" or "remove") on a subobject, use the `path` parameter.

## create

Fired when the connection to the server/peers has been made but before the proxy is synced with the peers and is editable.

```javascript

proxy.on('create', handler);

```

## ready

Fired when the connection to the server/peers has been made and the proxy is fully synxed and ready to be edited.

```javascript

proxy.on('ready', handler);

```

## disconnect

Fired when the connection to the server/peers is lost.

```javascript

proxy.on('disconnect', handler);

```

## change

Fired whenever a change comes in to the object or one of it's sub-objects. `handler` is a callback, and `path` is an array of property names which would select the targeted property given the root object.

```javascript

proxy.on('change', path, handler);

```

For example:

```
var A = {
    b: [
        0,
        1,
        {
            c: 5
        }
    ]
};
```

`c` could be selected given the path `['b', 2, 'c']`.

Changes _bubble up_, so if you were to specify a path `['b', 2]`, changes to `['b', 2, 'c']` would trigger that listener.

Because of string immutibility it is considered that the string is replaced even if only a few characters in the string have changed.

## remove

Fired whenever a property is deleted from the proxy. If that property was an object, it fires a `change` event on every property of that object (which is changed to undefined)

```javascript

proxy.on('remove', path, handler);

```

# Example

In order to make better sense of the API, we have provided an example of it's usage below:

```html
<form id="myform" action="">
First Name: <input type="text" class="fname" name="fname" value="" />
Last Name: <input type="text" class="lname" name="lname" value="" />
</form>

<script type="text/javascript">

require([
    '/path/to/configuration',
    '/path/to/listmap/api.js'
], function (Config, Listmap) {

    var obj = Listmap.create(Config).proxy;
    
    var onReady = function() {
        var fname = jQuery("#myform .fname");
        var lname = jQuery("#myform .lname");
        
        fname.on('change', function() {
            if (fname.val() !== obj.fname) {
                obj.fname = fname.val();
            }
        });
        
        lname.on('change', function() {
            if (lname.val() !== obj.lname) {
                obj.lname = lname.val();
            }
        });
        
        var meta = obj.meta = { collaborators: [] };
        meta.collaborators.push('Test');
        
        obj.on('change', ['fname'], function (fname) {
            jQuery("#myform .fname").val(fname);
        });
        obj.on('change', ['lname'], function (lname) {
            jQuery("#myform .lname").val(lname);
        });
        obj.on('change', ['meta', 'collaborators'], function (collaborators) {
            console.log("Collaborator list changed");
        });
    };
    
    obj.on('ready', onReady);
    
});

</script>
```
