Abstract Syntax Tree
====================

The CoffeeScript AST format is much pretty much exactly like JavaScript's. Only:
it has a few new node types, a few new properties in existing types, and some
unsupported ones are removed.

- New node types are always prefixed with *Coffee*, eg: `CoffeeLoopStatement`.

- New properties are always prefixed with underscore, eg: `_prefix` in
  `ThisExpression`.

- Types supported in JS but not Coffee are removed, eg: `WithStatement`.

Refer to the Mozilla's [Parser API spec] for the JavaScript version.

About this document
-------------------

A few pointers on notation:

- Array properties are denoted with `[ ]`. (eg: expressions in an ArrayExpression)

- Optional properties are denoted with `*`, meaning they can be `undefined` in
some cases. (eg: argument of a ReturnStatement)

Inspecting the syntax tree
--------------------------

In the command line, you can use the `--ast` option:

```sh
js2c file.coffee --ast
```

Programatically, just check the result's `ast` property:

```js
result = js2coffee.build(source);
console.log(result.ast);
```

An example of an AST:

```js
/* echo "alert()" | js2c --ast */

{ type: 'Program',
  body:
   [ { type: 'ExpressionStatement',
       expression:
        { type: 'CallExpression',
          callee: { type: 'Identifier', name: 'alert' },
          arguments: [],
          _isStatement: true } } ],
  comments: [] }
```

Node types
----------

These are nodes available in both CoffeeScript and JavaScript.

### Program
The root node.

   - `body` : [ Statement, ... ]

### BlockStatement
A sequence of statements. Usually belongs to a loop like WhileStatement, or an 
[IfStatement], or some other.

   - `body` : [ Statement, ... ]

### ExpressionStatement
A statement with one expression in it.

   - `expression` : Expression

### Identifier
Just an identifier.

   - `name` : String

### IfStatement
A conditional. This encompasses both the `if` and `else` parts.

   - `test` : Expression
   - `consequent` : [BlockStatement]
   - `alternate` : [BlockStatement] *

### BreakStatement
Just `break`, for breaking out of loops. No properties.

### ContinueStatement
Just `continue`, for breaking out of loops. No properties.

### SwitchStatement

### ReturnStatement
Can have its argument missing (eg: `return`).

   - `argument` : Expression *

### ThrowStatement

   - `argument` : Expression

### TryStatement
A try block. Encompasses all `try`, `catch`, and `finally`.

   - `block` : [BlockStatement] (the *try*)
   - `handlers` : [ CatchClause, ... ] ? (the *catch*)
   - `finalizer` : [BlockStatement] ? (the *finally*)

### CatchClause
A handler in a TryStatement. (eg: `catch err`)

   - `param` : Identifier
   - `body` : BlockStatement

### WhileStatement

### DebuggerStatement
Invokes a debugger. No properties.

### ThisExpression
Just `this`.

   - `_prefix` : Boolean (true if rendered as `@`)

### ArrayExpression

   - `elements` : [ Expression, ... ] *

### ObjectExpression

   - `properties` : [ [Property], ... ] *
   - `_braced` : Boolean (true if braced, eg `{ a: 1 }`)

### Property
Inside ObjectExpression.

   - `kind` : "init" (not sure what this is)
   - `key` : Expression (usually Identifier or Literal)
   - `value` : Expression

### FunctionExpression
(note: `id` is removed from function expressions.)

   - `params` : [ Identifier, ... ]
   - `body` : BlockStatement
   - `_parenthesized` : Boolean (true if parenthesized, eg `(-> ...)`)

### UnaryExpression
A prefix expression. Note that this *operator* also be `not`, which is not
available in JS.

   - `operator` : String (eg: "void", "!")
   - `argument` : Expression

### BinaryExpression

   - `left` : Expression
   - `operator` : String (eg: `+`)
   - `right` : Expression

### AssignmentExpression
An assignment. Also covers shorthand like `+=`.

   - `left` : Expression
   - `operator` : String (eg: `+=`)
   - `right` : Expression

### UpdateExpression
An increment or decrement. (`a++`)

   - `prefix` : Boolean
   - `operator` : String (eg: `++`)
   - `argument` : Identifier (eg: `a`)

### ConditionalExpression
The ternary operator. (`if a then b else c`)

  - `test` : Expression (the *if*)
  - `consequent` : Expression (the *then*)
  - `alternate` : Expression (the *else*)

### NewExpression
Instanciation (eg: `new X()`).

   - `callee` : Expression (usually an Identifier)
   - `arguments` : [ Expression, ... ]

### CallExpression
A function call.

   - `callee` : Expression (usually an Identifier)
   - `arguments` : [ Expression, ... ]
   - `_isStatement` : Boolean (true if it has no parentheses)

### MemberExpression
Scope resolution (eg: `this.foo`).

   - `computed` : Boolean (true if `a[b]`, false if `a.b`)
   - `object` : Expression (left side)
   - `property` : Expression (right side)

### SwitchStatement

   - `discriminant` : Expression
   - `cases` : [ [SwitchCase], ... ]

### SwitchCase
A case inside a SwitchStatement. Then `test` is not present, it's the `else`.
   
   - `type` : ??
   - `test` : Expression *
   - `consquent` : [BlockStatement]

### Identifier

   - `name` : String

### Literal
   - `raw` : raw code as a string (eg: `"\"hello\""`)
   - `value` : whatever the value is (eg: `hello`)

CoffeeScript-specific
---------------------

These are CoffeeScript-specific AST types:

### CoffeeListExpression
a comma-separated list (eg: `when a, b`).

   - `expressions` : [ Expression, ... ]

### CoffeePrototypeExpression
Prototype resolution operator. Similar to MemberExpression.

   - `object` : [BlockStatement]
   - `property` : Expression (usually Identifier)
   - `computed` : Boolean (*true* if `a::[b]`)

Examples:

```coffee
a::b

# { type: 'CoffeePrototypeExpression',
#   object: { type: 'Identifier', name: 'a' },
#   property: { type: 'Identifier', name: 'b' } }
```

### CoffeeLoopStatement
A forever loop. Compiles to `loop`.

   - `body` : [BlockStatement]

Examples:

```coffee
loop
  a()
  
# { type: 'CoffeeLoopExpression',
#   body: {
#     type: 'BlockStatement',
#     body: { ... } }
```

### CoffeeEscapedExpression
A backtick-wrapped JS expression.

   - `value` : the raw value

Example:

```coffee
`undefined`

# { type: 'CoffeeEscapedExpression',
#   value: 'undefined' }
```

### BlockComment
A `### ... ###` comment.

   - `value` : String

### LineComment
A `# ...` comment.

   - `value` : String

Only in JavaScript
------------------

These node types are not present in CoffeeScript ASTs, but are present in 
JavaScript ASTs.

### SequenceExpression
turned into a sequence of [ExpressionStatement]s.

### EmptyStatement
removed from the tree.

### FunctionDeclaration
converted into `a = ->` ([AssignmentExpression]).

### VariableDeclaration
converted into `a = ->` ([AssignmentExpression]).

### VariableDeclarator
converted into `a = ->` ([AssignmentExpression]).

### WithStatement
throws an error; unsupported in CoffeeScript.

### LabeledStatement
throws an error; unsupported in CoffeeScript.

### ForStatement
Converted to [WhileStatement] loops.

### ForInStatement
Converted to [WhileStatement] loops.

### DoWhileStatement
Converted to [WhileStatement] loops.

[Parser API spec]: https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Parser_API


[IfStatement]: #ifstatement
[ExpressionStatement]: #expressionstatement
[BlockStatement]: #blockstatement
[WhileStatement]: #whilestatement
[AssignmentExpression]: #assignmentexpression
[Property]: #property
[SwitchCase]: #switchcase