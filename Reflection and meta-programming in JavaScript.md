Reflection and meta-programming in JavaScript
Labels: dev, javascript, jslang, pl fundamentals
JavaScript is relatively weak when it comes to reflection and metaprogramming. Let us see what you can and cannot do. This post uses some of the standard methods of ECMAScript 5 (ES5).
Reflection
Reflection means examining the structure of a program and its data. The language of the program is the programming language PL, the language in which the examination is done is the meta programming language MPL. PL and MPL can be the same language.
Examining properties. When examining the properties of an object, JavaScript’s default is to include inherited properties. Thus, you have to make sure that a method does what you want and that will usually be to ignore inherited properties. One clue is to look for the presence or absence of “own” in the method name. For example, ES5’s Object.getOwnPropertyNames() returns all direct (non-inherited) property descriptors of an object.

    > var obj = { prop: "abc", method: function(x,y,z) {} };
    > Object.getOwnPropertyNames(obj)
    [ 'prop', 'method' ]
    > Object.getOwnPropertyDescriptor(obj, "prop")
    { value: 'abc'
    , writable: true
    , enumerable: true
    , configurable: true
    }
    > Object.getOwnPropertyDescriptor(obj, "method")
    { value: [Function]
    , writable: true
    , enumerable: true
    , configurable: true
    }
Writable means that the value of a property can be changed, enumerable that it is hidden in some contexts, configurable that it can be deleted or its property descriptor be changed. Similarly to getOwnPropertyNames(), Object.keys() returns only the enumerable “own” properties of an object. You can also use property descriptors to add properties, for example via:
    Object.defineProperty(obj, propname, desc)
List function parameter names. Having the names of functions is sometimes useful, e.g. if you have a function-valued argument and want to let that function control how it is used via its argument names. Functions are objects, so you would expect them to hold a list of parameter names somewhere. Alas, that is not the case. As a work-around one can use Function.prototype.toString() and parse its output. Which is what the following function does (it is borrowed from Prototype’s function.js).
    function argumentNames(fun) {
        var names = fun.toString().match(/^[\s\(]*function[^(]*\(([^)]*)\)/)[1]
            .replace(/\/\/.*?[\r\n]|\/\*(?:.|[\r\n])*?\*\//g, '')
            .replace(/\s+/g, '').split(',');
        return names.length == 1 && !names[0] ? [] : names;
    }
Using this function looks as follows:
    > function foo(bar, baz) {}
    > argumentNames(foo)
    [ 'bar', 'baz' ]
Meta-programming
Meta-programming means using a meta programming language MPL to influence a program written in a programming language PL. Examples include generating part of a PL program or changing how the PL program is executed. Note that MPL and PL can be the same programming language. JavaScript being a dynamic language, makes it easier to generate parts of a program at runtime. For example, many JavaScript libraries allow one to define classes (which are not native JavaScript constructs) that act as factories and dynamically construct instances.
Reacting to unknown methods. One use case is to have an object with no methods and to turn every method invocation into a request made to a server. Another use case is to use an object B to intercept every method call to an object A and to log it. This can be done by making A the prototype of B and by letting it react to unknown (i.e., all) methods. The pseudo-method __noSuchMethod__ allows one to do that. It is supported by Firefox and Rhino, but not by V8 (Chrome, node.js). It is on the list of features that might make it into V8, though. Given the following definition.

    var obj = {
        __noSuchMethod__: function(name, args) {
            print("Method " + name);
        }
    };
Interaction:
    > obj.foo();
    Method foo
What is missing from JavaScript is a corresponding __noSuchProperty__() method. Note that such a method could also be used to react to indexed reads and writes such as obj[33] or obj["key"] = true.
Getters and setters. ECMAScript 5 allows one to define functions that implement the getting or the setting of a property. That means that a property can be completely virtual. For example, one could forbid setting it via its property descriptor and always compute its value. Given the following definition.

    var obj = {
        get foo() {
            return "getter";
        },
        set foo(value) {
            print("setter: "+value);
        }
    };
Interaction:
    > obj.foo
    getter
    > obj.foo = "bla";
    setter: bla
    bla
    > Object.getOwnPropertyDescriptor(obj, "foo")
    { get: [Function: foo]
    , set: [Function: foo]
    , enumerable: true
    , configurable: true
    }
Obviously, We have barely scratched the surface what reflection and meta-programming are about, but the above techniques are already quite useful. Related reading:
“The Art of the Metaobject Protocol” [uk, de] by Gregor Kiczales, Jim des Rivieres, Daniel G. Bobrow. This can be seen as a precursor to Kiczales’ work an aspect-orientation.
