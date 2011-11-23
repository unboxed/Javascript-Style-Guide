# Javascript Style Guide - Unboxed Consulting

First Draft

## Preamble

This is intended as a set of guidelines only. It is not intended to be a 
book of absolute rules and consistency with any existing code base should 
always be held as more important that this document. 

It is also intended to be a living document and should be continuously 
updated where experience shows it to be incorrect. 

## TLDR;

*   FullCamel case for namespaces and constructors.
*   lowercase_underscored for variables and functions.
*   UPPERCASE_UNDERSCORED for constants.
*   Name verbosely, try to make function names explain what the function 
    does.
*   2 Spaces for indentation.
*   Use the constructor pattern.
*   Use named parameters where parameter meaning is not implicit in 
    function calls.
*   Use 'self' for holding construction object in closure.
*   Use exceptions. Never code to fail silently.

## House Style

### Naming conventions

#### Cases

<table>
  <tbody>
    <tr>
      <td>Namespaces</td>
      <td>FullCamel</td>
      <td>UbxdWeb</td>
    </tr>
    <tr>
      <td>Constructors</td>
      <td>FullCamel</td>
      <td>UbxdWeb.Carousel</td>
    </tr>
    <tr>
      <td>Variables</td>
      <td>lowercase_underscored</td>
      <td>price_of_cheese</td>
    </tr>
    <tr>
      <td>Functions</td>
      <td>lowercase_underscored</td>
      <td>get_price_of_cheese()</td>
    </tr>
    <tr>
      <td>Constants</td>
      <td>UPPERCASE_UNDERSCORED</td>
      <td>KG_PER_POUND</td>
    </tr>
  </tbody>
</table>

Given we are predominately a ruby house, these have been chosen to reflect
the ruby conventions. 

#### Functions

Avoid using anonymous functions. Note that a function assigned to an 
object attribute is anonymous unless explicitly given a name:

bad:

```javascript
this.do_the_macarana = function () {
  //...
}
```

Although assigned to a meaningful attribute, this function itself is 
anonymous.

good:

```javascript
this.do_the_macarana = function do_the_macarana() {
  //...
}
```

An application made up of entirely anonymous functions is almost 
impossible to effectively profile, and stack traces end up being pretty 
meaningless. 

Some small functions passed as arguments (maybe as callbacks or event 
handlers) may not really have a suitable name, in which case leaving them 
anonymous is appropriate. 

#### More about naming

Function naming is extremely important. Descriptive function names can 
really make the difference between readable and unreadable code. Be 
verbose. Having really long function or variable names is not a problem; 
Javascript can be minified.

#### Commenting

Feel free to comment anything obscure, but there is no need to comment the
obvious.

If you find yourself needing to comment a lot, consider breaking the code 
up into named functions which serve to describe what the code is doing.

If working on a previously commented piece of code, be sure to change the 
comments to ensure they are still accurate.

### Code Formatting

#### Curly Braces

Because of implicit semicolon insertion, always start your curly braces on
the same line as whatever they're opening. For example:

```javascript
if (something) {
  // ...
}
else {
  // ...
}
```

#### Array and Object Initializers

Single-line array and object initializers are allowed when they fit on a 
line:

```javascript
var arr = [1, 2, 3];  // No space after [ or before ].
var obj = {a: 1, b: 2, c: 3};  // No space after { or before }.
```

Multiline array initializers and object initializers are indented 2 
spaces, just like blocks.

```javascript
var inset = {
  top:    10,
  right:  20,
  bottom: 15,
  left:   12
};
```

Note that values are aligned with no space between the key and the colon.

```javascript
this.rows_ = [
  '"Slartibartfast" ',
  '"Zaphod Beeblebrox" ',
  '"Ford Prefect" ',
  '"Arthur Dent" ',
  '"Marvin the Paranoid Android" ',
  'the.mice@magrathea.com'
];
```

#### Function Calls

Simple function calls should all be on one line:

```javascript
activate(the_laser, '100000V');
```

Function calls with extremely long arguments or lots of them should be 
split into 1 line per argument, starting on the second line:

```javascript
activate(
  the_super_multi_laser_array,
  '100000000000000000000000000000000000000000V'
);
```

Additionally, if the arguments contain an object (hashlike), it should 
always be written multiline:

```javascript
activate(
  the_super_multi_laser_array,
  settings: {
    voltage: 100000000000000000000000000,
    target:  "John Prescot's bald spot"
  }
);
```

#### Indentation

Use 2 spaces for indentation.

### Constructors

There are several different ways of constructing objects. In general the 
constructor pattern is recommended:

```javascript
function Laser() {
  this.fire = function fire() {
    //...
  }
}

window.the_laser = new Laser();
```

Nested constructors are also fine but try to keep stuff separate where 
possible.

```javascript
function Laser() {
  var power_supply = new PowerSupply();

  this.fire = function fire() {
    power_supply.turn_on();
    //...
  }

  function PowerSupply() {
    //...
  }
}
```

### Modular Behaviour

Common behaviour can be bundled into modules:

```javascript
function PoweredThing() {
  this.set_voltage = function set_voltage() {
    //...
  }
}

function Laser() {
  PoweredThing.apply(this);
}

var laser = new Laser();
laser.set_voltage(100000000);
```

### Self

Often, it's required to hold the constructed object in closure for use by 
private functions. Use the variable name self for this:

```javascript
function Laser() {
  var self = this;

  function fire() {
    set_voltage(self.voltage);
  }
}
```

Note that this should always be the first line of the constructor.

### Using named parameters

Using named parameters reduces connescence and makes code more readable. 
It is encouraged when a function's parameters are not immediately obvious 
from the function name.

Bad:

```javascript
function do_the_macarana(tempo, start_at_step, repeats) {
  //...
}

do_the_macarana(120, 5, 3);
```

Here, it's not obvious reading the function call what the parameters are 
doing and anyone looking at the code would have to look up the function 
definition to understand the code.

Good:

```javascript
function do_the_macarana(params) {
    //params: tempo, start_at_step, repeats
  }

  do_the_macarana({
    tempo:         120,
    start_at_step: 5,
    repeats:       3
  });
```

or

```javascript
function do_the_macarana(params) {
  var tempo =         params.tempo;
  var start_at_step = params.start_at_step;
  var repeats =       params.repeats;
}

do_the_macarana({
  tempo: 120,
  start_at_step: 5,
  repeats: 3
});
```

Here, it's immediately apparent what the purpose of each parameter is when
reading the function call.

Which parameters are consumed should be immediately visible at the start 
of the function definition. When defining a function, either var all the 
members of params which you are interested in using, or leave a comment 
saying which params are being used.

## Code Layout

### Files

In general, one file should be created for each constructor. All files 
should be named in lower_underscored case and should reflect the 
constructor.

Vendor stuff (jQuery, CKEditor, etc.) should be in the /vendor folder

Lib stuff (Utilities, any other non-vendor libraries which are not 
specific to a single part of the application) should be in the /lib 
folder.

`application.js` should only include code to initialize the application.

### Code in views

It is acceptable to have inline javascript code, but only initializers 
related to the page content. In general, wherever possible, javascript 
code should be kept out of the view.

It's acceptable to set any required application constants inline.

Of course, drop-in code like google analytics is also acceptable.

Use of javascript in html attributes is bad:

```html
<a onclick="do_something();">Do Something</a>;
```

Make use of [unobtrusive javascript](http://en.wikipedia.org/wiki/Unobtrusive_JavaScript "Unobtrusive JavaScript on Wikipedia") in order to separate the functionality (the "behaviour layer") from a Web page's structure/content and presentation.

## Language rules

### Var

Always use var. Never use implicit assignment.

If you wish something to be declared on window, be explicit:

wrong:

```javascript
foo = 'bar';
```

right:

```javascript
window.foo = 'bar';
```

### Constants

Constants should be defined as normal variables but should obey the naming
conventions for constants. The const keyword is not well supported and 
thus should not be used.

### Semicolons

Never rely on inferred insertion of semicolons. Semicolons are required 
after all assignments or function calls. Semicolons are not required after
blocks.

It's also recommended to use semicolon after function declarations even t
hough they are not required.

Semicolons should not be used after `for` / `while` / `switch` / `if` /
`else` blocks and other similar constructions for readability.

```javascript
var foo = 'bar';
// assignment - needs a semicolon.

function foo() {
  alert("don't you foo me");
  //function call, needs a semicolon.
};
// Doesn't need a semicolon, but should have one.

this.foo = function foo() {
  alert("don't you foo me");
};
// This must have a semicolon as it is an assignment.

if (foo) {
  foo();
}
// Should not have a semicolon.
```

### Nested Functions

Nested functions are a great way to break up a function and can be used 
liberally. If a function is used exclusively by another function, it 
should in general be declared within that function.

Bad:

```javascript
function foo_plus_bar() {
  return get_foo_for_foo_plus_bar() + 'bar';
};
function get_foo_for_foo_plus_bar() {
  return 'foo';
};
```

Good:

```javascript
function foo_plus_bar() {
  function get_foo() {
    return 'foo';
  };

  return get_foo() + 'bar';
};
```

As an exception to this, sometimes you may have to declare a 'child' 
function outside of the main function to avoid a circular reference (see 
section on closures).

### Declarations within blocks

In general, avoid making any declarations within blocks. Especially, 
absolutely do not define functions within blocks as this is not part of 
ECMAScript. ECMAScript only allows for Function Declarations in the root 
statement list of a script or function.

Terrible:

```javascript
if (foo_is_wibble) {
  function foo() { return 'wibble'; }
}
else {
  function foo() { return 'bar'; }
}
foo();
```

Ok:

```javascript
function wibble() {
  return 'wibble';
}
function bar() {
  return 'bar';
}

var foo;
if (foo_is_wibble) {
  foo = wibble;
}
else {
  foo = bar;
}
foo();
```

In general, variables should also not be declared in blocks:

bad:

```javascript
for (var i=0; i<5 i++) {
  var wibble = new Wibble(i);
  wibbles.push(bar);
}
```

good:

```javascript
var wibble;
for (var i=0; i<5 i++) {
  wibble = new Wibble(i);
  wibbles.push(wibble);
}
```

### Exceptions

Use exceptions. Throw stuff whenever you need to. The only thing to avoid 
is using try / catch blocks for 'happy path' execution. Try / catch blocks
are expensive and should only be used for exception handling.

Don't be afraid of throwing custom exceptions. If something is broken, 
it's broken and a good error message will save time in debugger. Never 
code to fail silently.

### Creating primitive objects

Do not use wrapper objects. Just declare the primitive directly:

```javascript
var str = 'foo';
var bool = false;
var arr = [];
var num = 10;
var obj = {};
```

Especially, never use the new keyword with primitive constructors. Here's
why:

```javascript
typeof(Boolean()) // ==> 'boolean'
typeof(new Boolean()) // ==> 'object'
```

There are only five primitive types in javascript, these are `undefined`, `null`, `boolean`, `number`, and `string`.

### Closures

Closures are great. Closures can inspire genuine awe. However, 
consideration must sometimes be taken in their use, especially in 
long-running code.

Consider this:

```javascript
function foo(element, a, b) {
  element.onclick = function handle_click() { /* uses a and b */ };
}
```

Here, we have created a circular reference which will cause a memory leak.
The function `handle_click` holds `element` in scope, and `element` holds 
`handle_click`, meaning that neither can be destroyed.

```javascript
function foo(element, a, b) {
  element.onclick = bar(a, b);
}
```

```javascript
function bar(a, b) {
  return function() { /* uses a and b */ }
}
```

Here a second closure not containing element is created, thus avoiding the
circular reference.

### Eval

In general, the only use for `eval` is to deserialize stuff. If you find 
yourself trying to use it for anything else, you're probably being too 
'meta'.

### With

In general, just don't. Use of the `with` keyword can easily mask scope 
and produce extremely confusing code to read.

### This

Use of `this` can be confusing. In general it should be limited to being 
used within constructors and in conjunction with `call` or `apply`.

Using this to target DOM elements in event calls generally hints at a bad 
code structure, and consideration should be given to how to achieve the 
same results in a different way.

### For … in … loops

This method should not be used to iterate over an array.

### Modifying prototypes of built-in objects

for example:

```javascript
Array.prototype.find = function () {....} 
```

Native wrapped constructors should never be modified. If you need to 
extend the functionality of a native type, create a new constructor which
creates an object and then attaches any extended stuff to it.
