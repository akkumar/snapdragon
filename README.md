Created by [jonschlinkert](https://github.com/jonschlinkert) and [doowb](https://github.com/doowb).

**Features**

- Bootstrap your own parser, get sourcemap support for free
- All parsing and rendering is handled by simple, reusable middleware functions
- Inspired by the parsers in [pug][] and the [css][] lib.

## Install
Install with [npm](https://www.npmjs.com/):

```sh
$ npm install --save snapdragon
```

## Usage examples

```js
var Snapdragon = require('snapdragon');
var snapdragon = new Snapdragon();
```

**Parse**

```js
var ast = snapdragon.parser('some string', options)
  // parser middleware that can be called by other middleware
  .set('foo', function () {})
  // parser middleware, runs immediately in the order defined
  .use(bar())
  .use(baz())
```

**Render**

```js
// pass the `ast` from the parse method
var res = snapdragon.renderer(ast)
  // renderer middleware, called when the name of the middleware
  // matches the `node.type` (defined in a parser middleware)
  .set('bar', function () {})
  .set('baz', function () {})
  .render()
```

See the [examples](./examples/).

## Getting started

**Parsers**

Parsers are middleware functions used for parsing a string into an ast node.

```js
var ast = snapdragon.parser(str, options)
  .use(function() {
    var pos = this.position();
    var m = this.match(/^\./);
    if (!m) return;
    return pos({
      // `type` specifies the renderer to use
      type: 'dot',
      val: m[0]
    });
  })
```

**AST node**

When the parser finds a match, `pos()` is called, pushing a token for that node onto the ast that looks something like:

```js
{ type: 'dot',
  val: '.',
  position:
   { start: { line: 1, column: 1 },
     end: { line: 1, column: 2 } }}
```

**Renderers**

Renderers are _named_ middleware functions that visit over an array of ast nodes to render a string.

```js
var res = snapdragon.renderer(ast)
  .set('dot', function (node) {
    console.log(node.val)
    //=> '.'
    return this.emit(node.val);
  })
```

**Source maps**

If you want source map support, make sure to emit the position as well.

```js
var res = snapdragon.renderer(ast)
  .set('dot', function (node) {
    return this.emit(node.val, node.position);
  })
```

## Docs

### Parser middleware

A parser middleware is a function that returns an abject called a `token`. This token is pushed onto the AST as a node.

**Example token**

```js
{ type: 'dot',
  val: '.',
  position:
   { start: { line: 1, column: 1 },
     end: { line: 1, column: 2 } }}
```

**Example parser middleware**

Match a single `.` in a string:

  1. Get the starting position by calling `this.position()`
  1. pass a regex for matching a single dot to the `.match` method
  1. if **no match** is found, return `undefined`
  1. if a **match** is found, `pos()` is called, which returns a token with:
      * `type`: the name of the [renderer] to use
      * `val`: The actual value captured by the regex. In this case, a `.`. Note that you can capture and return whatever will be needed by the corresponding [renderer].
      * The ending position: automatically calculated by adding the length of the first capture group to the starting position. 

## Renderer middleware

Renderers are run when the name of the renderer middleware matches the `type` defined on an ast `node` (which is defined in a parser).

**Example**

Exercise: Parse a dot, then render it as an escaped dot.

```js
var ast = snapdragon.parser('.')
  .use(function () {
    var pos = this.position();
    var m = this.match(/^\./);
    if (!m) return;
    return pos({
      // define the `type` of renderer to use
      type: 'dot',
      val: m[0]
    })
  })

var result = snapdragon.renderer(ast)
  .set('dot', function (node) {
    return this.emit('\\' + node.val);
  })
  .render()

//=> '\.'
```

## API

### Parse

### [Parser](lib/parser.js#L14)

Create a new `Parser` with the given `input` and `options`.

**Params**

* `input` **{String}**    
* `options` **{Object}**    

Push a `token` onto the `type` stack.

**Params**

* `type` **{String}**    
* `returns` **{Object}** `token`  

Pop a token off of the `type` stack

**Params**

* `type` **{String}**    
* `returns` **{Object}**: Returns a token  

Return true if inside a `stack` node. Types are `braces`, `parens` or `brackets`.

**Params**

* `type` **{String}**    
* `returns` **{Boolean}**  

Return true node is the given type.

**Params**

* `type` **{String}**    
* `returns` **{Boolean}**  

### Render

[css]: https://github.com/reworkcss/css
[pug]: http://jade-lang.com