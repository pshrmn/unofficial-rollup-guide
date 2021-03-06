# Big List of Rollup Node Types and Their Behavior

## Shared Types

### Node

#### Behaviors

* assignExpression - stub
* bind - bind each child node
* eachChild - iterate over child nodes by using the node's `keys` property, calling the callback for each
* gatherPossibleValues - add `UNKNOWN_ASSIGNMENT` to values set
* getValue - return `UNKNOWN_VALUE`
* hasEffects - return true if node's `included` property is true or any child nodes have effects
* hasEffectsWhenAssigned - return false
* hasEffectsWhenMutated - return false
* includeDeclaration - return result of calling node's includeInBundle method
* includeInBundle
  * if already fully included, return false
  * iterate over child nodes and call their `includeInBundle` method. If any of them return true, set node's `included property to true.
  * return false if already included and no child node's `includeInBundle` returned true, otherwise return true
* initialise - run in order: `intialiseScope`, `initialiseNode`, `initialiseChildren`
* initialiseChildren - call each child node's `initialise` method, passing it the node's scope
* initialiseNode - stub
* initialiseScope - assign provided scope
* insertSemicolon - add a semicolon to the code string
* isFullyIncluded - is node's `_isFullyIncluded` is already true, return true. Otherwise, set node's `_isFullyIncluded` value using node's `included` property and calling all child nodes' `isFullyIncluded` methods.
* locate - return an object that describes the location of the node
* render - call each child node's render method
* shouldBeIncluded - return result of calling node's `hasEffects` method
* someChild - similar to eachChild, but using `some` instead of `forEach`
* toString - returns the node's code string

### Function

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/shared/Function.js)

#### Behaviors

* bind - bind identifier (if there is one), params, and body
* hasEffects - return node's `included` value
* initialiseChildren
  * for each param, initialize it and add as declaration to local scope
  * initialize the body, possibly replacing body's existing scope
* initialiseScope - create a new scope that is a lexical boundary

### Statement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/shared/Statement.js)

#### Behaviors

* render - if treeshaking and `included` is false, remove the code. otherwise, render it

### Class

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/shared/Class.js)

#### Behaviors

* addReference - stub
* getName - return the class's name
* initialiseChildren - If there is a super class, initialize that. Then, initialize the body node
* initializeScope - Create a new scope that is a block scope

====

## Types

### ArrayPattern

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ArrayPattern.js)

#### Example 

```js
// the array param
function thing([a, b, c]) {}
```

#### Properties

* elements - array of `Identifier`s and `AssignmentPattern`s

#### Behaviors

* assignExpression - for each child node, assign them an `UNKNOWN_ASSIGNMENT` expression
* hasEffectsWhenAssigned - 

### ArrowFunctionExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ArrowFunctionExpression.js)

Inherits from [Function](#function)

##### Example 

```js
() => {}
```

##### Properties

* params - array of `Identifier`s or patterns
* body - `BlockStatement`, expression, or `Literal`

##### Behaviors

* initialiseScope - creates a new scope that is neither a block scope nor a lexical boundary

### AssignmentExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/AssignmentExpression.js)

#### Example 

```js
x = 5
```

#### Properties

* left - `Identifier` or pattern
* right - a `Literal` or expression
* operator - `=`, `+=`, etc.

#### Behaviors

* bind
  * call `Node.bind`
  * throw an error if assignment is not allowed
  * assign the `right` node to the `left` node
* hasEffects - return `Node.hasEffects` value or if `left` node has effects when assigned
* hasEffectsWhenMutated - return true

### BinaryExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/BinaryExpression.js)

#### Example 

```js
variable === 24
```

#### Properties

* left - `Literal` or expression
* right - `Literal` or expression
* operator - `==`, `+`, etc.

#### Behaviors

* getValue - if left, right, or operator is unknown, return `UNKNOWN_VALUE`. Otherwise, return result of executing expression
* hasEffectsWhenMutated - return true

### BlockStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/BlockStatement.js)

Inherits from [Statement](#statement)

#### Example 

```js
{
  const x = 'hello';
  console.log(x);
}
```

#### Properties

* body - array of statements and declarations

#### Behaviors

* bind - bind each node in the body array
* includeInBundle
  * if already fully included, return false
  * iterate over nodes in body array, tracking if any that aren't already included should be
  * if the node is not already included or there are new nodes to include, set node's `included` property to true and return true. otherwise, return false
* initialiseAndReplaceScope - similar to initialise (from Node), but sets scope to the provided scope instead of creating a new scope.
* initialiseChildren - initialize each node in the body array and link them together by setting `node.next` to the the next node's `start`
* initialiseScope - create a new scope that is a block scope
* render - if there are body nodes, iterate over them and render each. Otherwise, call `Statement.render`

### CallExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/CallExpression.js)

#### Example 

```js
fn(one, two)
```

#### Properties

* callee - `Identifier` or `MemberExpression`
* arguments - array of `Literal`s and expressions

#### Behaviors

* bind - If the callee is an `Identifier`, find the associated declaration, and throw an error if the declaration is a namespace or log a warning for "eval" calls. Then, call `Node.bind`
* hasEffects - return true if node's `included` property is true, any of its arguments have effects, or the call has effects
* hasEffectsWhenMutated - return true

### CatchClause

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/CatchClause.js)

#### Example 

```js
catch (e) {
  ...
}
```

#### Properties

* param - an `Identifier`
* body - a `BlockStatement`

#### Behaviors

* initialiseChildren
  * if there is a param, initialise it and add it to scope's declarations
  * replace body node's scope with the node's scope
* initialiseScope - create a new scope that is a block scope

### ClassDeclaration

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ClassDeclaration.js)

Inherits from [Class](#class)

#### Example 

```js
class Thing {}
class AnotherThing extends Thing {}
// the class part of:
export default class {}
```

#### Properties

* id - an `Identifier`
* superClass - an `Identifier` or null
* body - a `ClassBody`

#### Behaviors

* gatherPossibleValues - add node to values set
* hasEffects - return node's `included` property
* initialiseChildren - if the class has an `id`, set the node's name to its name, add the name to the scope's declarations, and initialize the `id` `Identifier` (`id` can be null for default exports). Then, call `Class.initialiseChildren`.
* render - if tree shaking and `included` is false, remove the code

### ClassExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ClassExpression.js)

Inherits from [Class](#class)

#### Example 

```js
// the right node of
var x = class Thing {};
```

#### Properties

* id - an `Identifier` (optional)
* superClass - an `Identifier` or null
* body - a `ClassBody` 

#### Behaviors

* initialiseChildren - if the class has an `id`, set the class's name to its name, add the name to the parent scope's declarations, and initialize the `id` `Identifier`. Then, call `Class.initialiseChildren`

###  ConditionalExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ConditionalExpression.js)

#### Example 

```js
test ? consequent : alternate
```

#### Properties

* test - an expression or `Identifier`
* consequent - an expression or `Literal`
* alternate - an expression or `Literal`

#### Behaviors

* initialiseChildren
  * if tree shaking, get the actual value for test. If we cannot determine the value at this time, initialize both consequent and alternate. If test value is truthy, initialize consequent and set alternate to null. Otherwise, if there is an alternate, initialize it and set consequent to null.
  * else, call `Node.initialiseChildren`
* gatherPossibleValues - When test value is unknown, add both consequent and alternate to values set. Otherwise, execute conditional expression and only add the returned node.
* getValue - Execute test and return unknown if result of test is unknown, otherwise return the value for consequent or alternate (depending on test value).
* hasEffectsWhenMutated - return true
* render - If tree shaking:
  * If test value is unknown, render everything
  * If test value is truthy, only render consequent (`true ? 'a' : 'b'` rendered as `'a'`)
  * if test value is falsy, only render alternate

### EmptyStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/EmptyStatement.js)

#### Example 

```js
;
```
#### Behaviors

* render - if the parent is a `BlockStatement` or `Program`, remove the empty statement

### ExportAllDeclaration

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ExportAllDeclaration.js)

#### Example 

```js
export * from 'another-module';
```

#### Properties

* source - a `Literal` string

#### Behaviors

* initialiseNode - set `isExportDeclaration = true`
* render - remove the code

### ExportDefaultDeclaration

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ExportDefaultDeclaration.js)

#### Example 

```js
export default function() {...}
```

#### Properties

* declaration

#### Behaviors

* addReference - set the node's name to the reference's name and if there is an original property, add the reference to that too
* bind - get the name using the node's declaration and if there is a name, set node's original property by finding the declaration in the scope with that name. Then, call the node's declaration's bind method.
* gatherPossibleValues - call declaration's `gatherPossibleValues` method
* getName - if original property is set and not reassigned, return that node's name, otherwise return the node's name property.
* includeDeclaration - if node's `included` property is true, return false. Otherwise, set `included` to true, call declaration's `includeInBundle` method, and return true.
* includeInBundle - if the declaration's `shouldBeIncluded` method returns true, return result of calling declaration's `includeInBundle` method. Otherwise, return false.
* initialiseNode
  * set node properties `isExportDeclaration` and `isDefault` to true
  * set the name property using the node's declaration
  * set the scope's `declarations.default` to this node
* render - this behavior is fairly complicated. In essence, this does a couple of things:
  1. It removes `export default` because we just want a declaration in our code.
  2. For class and function exports, if they have no name, it will add `var <name>` (where name is derived from a reference to the function/class). For example, if we have `export default function() {}` and then a file imports that using `import fn from './module'`, Rollup will update the code to be `var fn = function() {}`.

### ExportNamedDeclaration

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ExportNamedDeclaration.js)

#### Example 

```js
export const x = 'Exit';  // declaration

const y = 'Why';
export { y }              // specifiers
```

#### Properties

* declaration
* specifiers - an array of export specifiers

**Note:** This is a one or the other type of situation

#### Behaviors

* bind - If this export has a declaration, bind it
* hasEffects - return node's `included` value or the result of calling declaration's `hasEffects` (if there is a declaration)
* initialiseNode - set `isExportDeclaration` to true
* render
  * for `export declaration`, remove the `export` from the code and render the declaration
  * for `export { ... }`, remove the whole line. If the export is renamed, include a line that creates a variable with the new name.

### ExpressionStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ExpressionStatement.js)

Inherits from [Statement](#statement)

#### Example 

```js
variable = value;
fn();
```

#### Properties

* expression

#### Behaviors

* render - Call `Statement.render`, then make sure that the statement is terminated with a semicolon

### ForInStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ForInStatement.js)

Inherits from [Statement](#statement)

#### Example 

```js
for (let key in obj) {
  ...
}
```

#### Properties

* left - a `VariableDeclaration`
* right - an `Identifier` or expression
* body - a `BlockStatement`

#### Behaviors

* initialiseChildren
  * initialize left node using the scope
  * initialize right node using the parent scope
  * initialize body (possibly replacing scope)
* initialiseScope - create a new scope that is a block scope

### ForOfStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ForOfStatement.js)

#### Example 

```js
for (let key of iterable) {
  ...
}
```

#### Properties

* left - a `VariableDeclaration`
* right - an `Identifier` or expression
* body - a `BlockStatement`

#### Behaviors

* bind - call `Statement.bind`, then assign `left` node an `UNKNOWN_ASSIGNMENT` expression
* initialiseChildren
  * initialize left node using the scope
  * initialize right node using the parent scope
  * initialize body (possibly replacing scope)
* initialiseScope - create a new scope that is a block scope

### ForStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ForStatement.js)

#### Example 

```js
for (init; test; update) {
  ...
}
```

#### Properties

* init - a `VariableDeclaration` or null
* test - an expression
* update - an expression
* body - a `BlockStatement` or expression

#### Behaviors

* initialiseChildren - Initialize init, test, and update nodes. If body is a block statement, initialize its scope and children. For other node types, just initialize.
* initializeCScope - create a new scope that is a block scope

### FunctionDeclaration

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/FunctionDeclaration.js)

Inherits from [Function](#function)

#### Example 

```js
function name(param1, param2) {
  ...
}
```

#### Properties
* id - `Identifier`
* generator - boolean
* expression - boolean (???)
* params - an array of `Identifier`s or patterns
* body - a `BlockStatement`

#### Behaviors

* addReference - stub
* assignExpression - add expression to node's assigned expressions
* gatherPossibleValues - add self to values Set
* getName - return the node's name property
* hasEffectsWhenMutated - return node's `included` value
* initialiseChildren - If the node has an id: set node's name to id's name, add id to parent scope, and initialize the id node. Then, call `Function.initialiseChildren`
* initialiseNode - create a new set, saved as `assignedExpressions`, initialized with node as value
* render - if tree shaking and not `included`, remove the node. Otherwise, render

### FunctionExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/FunctionExpression.js)

Inherits from [Function](#function)

#### Example 

```js
// the right part of:
let fn = function() {}
```

#### Properties

* id - `Identifier` or null
* generator - boolean
* expression - boolean (???)
* params - an array of `Identifier`s or patterns
* body - a `BlockStatement`

#### Behaviors

* addReference - stub
* getName - return the node's name property
* intialiseChildren - If the node has an id: set node's name to id's name, add id to parent scope, and initialize the id node. Then, call `Function.initialiseChildren`

### Identifier

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/Identifier.js)

#### Example 

```js
// x in:
const x = 'hello';
```

#### Properties

* name - a string

#### Behaviors

* assignExpression - if the node has a declaration, add the expression to its assigned expressions
* bind - If the node is a reference or part of a left hand side assignment (default value), find and set the node's declaration and add the node to the declaration's references
* gatherPossibleValues - if the node is a reference (not a property or a rename), add it to set
* hasEffectsWhenAssigned - if there is a declaration, return its `included` value
* hasEffectsWhenMutated - if there is a declaration, return true if it has a truthy value for any of these properties: `included`, `isParam`, `isGlobal`, `isExternal`, `isNamespace`, or if any of the node's assigned expressions have effects when mutated.
* includeInBundle - if already included, return false. Otherwise, set `included` to true, call declaration's `includeDeclaration` method, and return true.
* render - If the node has a declaration, and the declaration has a different name than the node's name, overwrite code using declaration's name. For re-named property identifiers, include old name (`oldName: newName`).

### IfStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/IfStatement.js)

Inherits from [Statement](#statement)

#### Example 

```js
if (condition) {
  ...
} else {
  ...
}
```

#### Properties

* test - an `Identifier` or expression
* consequent - a `BlockStatement`
* alternate - An `IfStatement` or `BlockStatement`

#### Behaviors

* initialiseChildren
  * if not tree shaking, just initialize
  * otherwise, get the value of the test
    * if value cannot be determined, just initialize
    * if value is truthy, initialize the consequent, hoist any variable declarations in the alternate and set it to null
    * if value is falsy, initialize the alternate, hoist any variable declarations in the consequent and set it to null
* render
  * if not tree shaking, just render
  * otherwise, using the test value
    * set the test code to the test value
    * for any hoisted variables, add them to the code if they are `included`
    * if test value is truthy, only render the consequent
    * if test value is falsy, only render the alternate

### ImportDeclaration

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ImportDeclaration.js)

#### Example 

```js
import Name from 'module';
import { One, Two } from 'module';
import * as Grouped from 'module';
```

#### Properties

* specifiers - array of specifiers
* source - `Literal`

#### Behaviors

* bind - stub
* initialiseNode - set `isImportDeclaration` to true
* render - remove the node

### Literal

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/Literal.js)

#### Example 

```js
// the string in
const x = 'test';
```

#### Properties

* value - the literal value
* raw - the value as a string

#### Behaviors

* getValue - returns the node's value property
* getPossibleValues - add the node to the values set
* render - if value is a string, make sure not to mess with indentation

### LogicalExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/LogicalExpression.js)

#### Example 

```js
thing && thing.property
```

#### Properties

* left - an `Identifier`, `Literal`, or expression
* operator - '&&' or '||'
* right - an `Identifier`, `Literal`, or expression

#### Behaviors

* getValue - if either the left or right node's value is unknown, return `UNKNOWN_VALUE`. Otherwise, return the result of executing the expression

### MemberExpression

#### Example 

```js
object.property
object['property']
```

#### Properties

* object - an `Identifier` or expression
* property - an `Identifier` or `Literal`
* computed - boolean

#### Behaviors

* bind - walk through properties to create a "key path". Then, trace path to find the root declaration for the node, setting it as the node's declaration. (This is actually more complicated than this description, but will do for now.)
* getPossibleValues - add `UNKNOWN_ASSIGNMENT` to values set
* hasEffectsWhenAssigned - return value of calling `object` node's `hasEffectsWhenMutated`
* hasEffectsWhenMutated - return true
* includeInBundle - determine if there are any new nodes to add to bundle. If there is a declaration and it isn't already included, include it and set new nodes added to true. Return new nodes added value.
* render
  * if the node has a declaration and with a different, overwrite code with new name. 
  * if the node has a replacement name, overwite code with replacement name

### NewExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/NewExpression.js)

#### Example 

```js
new Thing(arg1, arg2)
```

#### Properties

* callee - `Identifier` or expression
* arguments - an arry of `Identifier`s or expressions

#### Behaviors

* hasEffects - return node's `included` value or result of passing scope and callee to `callHasEffects`

### ObjectPattern

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ObjectPattern.js)

#### Example 

```js
{ one: 1, two: 2 }
```

#### Properties

* properties - an array of `Property` nodes

#### Behaviors

* assignExpression - assign `UNKNOWN_ASSIGNMENT` to all child node's
* hasEffectsWhenAssigned - return true if any child has effects when assigned

### Property

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/Property.js)

#### Example

```
// any of the lines below (a,b,c, d)
{
  a: 'Ayy',
  b() {...},
  c,
  [d]: 'Dee'
}
```

#### Properties

* method - true if if `key()` format
* shorthand - true if in `{ key }` format
* computed - true if in `{ [key]: ... }` format
* key - an `Identifier`
* value - a `Literal`, pattern, or expression
* kind - "init"??

#### Behaviors

* assignExpression - assign expression to value node
* hasEffectsWhenAssigned - return result of calling value node's `hasEffectsWhenAssigned`
* render - if not shorthand, render the key. render the value.

### RestElement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/RestElement.js)

#### Example

```js
// the "...rest" element
const [first, ...rest] = [1,2,3,4];
```

#### Properties

* argument - an `Identifier`

#### Behaviors

* assignExpression - assign `UNKNOWN_ASSIGNMENT` to argument node
* hasEffectsWhenAssigned - return result of calling argument node's `hasEffectsWhenAssigned`

### ReturnStatement 

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ReturnStatement.js)

#### Example

```js
return something;
```

#### Properties

* argument - an `Identifier`, `Literal`, or expression

#### Behaviors

* shouldBeIncluded - return true

### SwitchStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/SwitchStatement.js)

#### Example 

```js
switch (discriminant) {
case 1:
  ...
  break;
case 2:
  ...
  break;
default:
  ...
  break;
}
```

#### Properties

* discriminant - an `Identifier` or expression
* cases - an array of `SwitchCase` nodes

#### Behaviors

* initialiseScope - create a new scope that is a block scope

### TaggedTemplateExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/TaggedTemplateExpression.js)

#### Example 

```js
tag\`template string\`
```

#### Properties

* tag - an `Identifier` (possibly a function expression?)
* quasi - a `TemplateLiteral`

#### Behaviors

* bind - if tag is an `Identifier`, find the declaration and:
  * throw an error is the declaration is a namespace
  * log a warning if the tag is "eval" 
* hasEffects - return true if the quasi has effects or the tag has effects

### TemplateLiteral

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/TemplateLiteral.js)

#### Example 

```js
`I am a template literal`
`Name: ${obj.name}\nAge: ${obj.age}`
```

#### Properties

* expressions - an array of `Identifier`s or expressions
* quasis - an array of `TemplateElement`s

#### Behaviors

* render - ensure that the node's indentation does not get messed with

### ThisExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ThisExpression.js)

#### Example 

```js
// just the "this" part of the following lines
this.value = arg => { console.log(arg); };
this.value('hi!');
```

#### Behaviors

* hasEffectsWhenMutated - return true
* initialiseNode - find the node's scope's "lexical boundary" scope. If that is a module scope, set the node's alias to the module's context. Log a warning if that alias is undefined.
* render - if the node has an alias (lexical boundary scope is module scope), overwrite "this" with alias

### ThrowStatement

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/ThrowStatement.js)

#### Example 

```js
throw new Error();
```

#### Properties

* argument - an `Identifier` or expression

#### Behaviors

* hasEffects - return true

### UnaryExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/UnaryExpression.js)

#### Example 

```js
+plus
!not
~flipBits
typeof joke
```

#### Properties

* operator - `+`, `-`, `!`, `~`, `typeof`, `void`, `delete`
* prefix - true
* argument - an `Identifier` or expression

#### Behaviors

* bind - only bind when the node's value is unknown
* getValue - get the value from the node's arguments property. If that is unknown, return `UNKNOWN_VALUE`, otherwise execute the operator and return that value.
* hasEffects - return true if `included`, argument node has effects, or the operator is `delete` and either the argument is not a `MemberExpression` or the argument's object node has effects when mutated.
* initialiseNode - set node's value property by calling node's `getValue` method.

### UpdateExpression

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/UpdateExpression.js)

#### Example 

```js
variable++
```

#### Properties

* operator - '++' or '--'
* prefix - false
* argument - an `Identifier` or expression

#### Behaviors

* bind - Throw error if the update is not legal. If the subject is an `Identifier`, find the id's declaration node, set that node's `isReassigned` property to true. Finally, call `Node.bind`
* hasEffects - return `included` or result of calling argument node's `hasEffectsWhenAssigned`

### VariableDeclaration

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/VariableDeclaration.js)

#### Example 

```js
var one = 1;
let two = 2;
const three = 3, four = 4;
```

#### Properties

* kind - 'var', 'let', or 'const'
* declarations - an array of `VariableDeclarator`s

#### Behaviors

* assignExpression - for each child, assign them the `UNKNOWN_ASSIGNMENT` expression
* includeInBundle - if already fully included, return false. Otherwise, iterate over declarations to determine if any are new. If `included` is false or there are new nodes, set `included` to true and return true. Otherwise, return false.
* render - This is pretty complex
  * Determine if we should separate the declarators. By default Rollup doesn't separate declarators, but it does for module level declarations (unless the are part of a for statement).
  * Keep an "empty" variable with the default value of true and c, which is an index in the code
  * For each declarator in the declarations array
    * if the node is an identifier
      * get the declarator's proxy
      * if the declarator is exported and its value is reassigned
        * if the declarator has a truthy init property, prepend the prefix to the code and set "empty" to false
      * otherwise, if we are not treeshaking or the proxy is included
        * if Rollup should separate, insert prefix and kind (ie turn `x = 2` to `const x = 2`)
    * otherwise
      * set "included" to false
      * get all of the names from the declarator. This would be for the pattern types. For example, from `{ a, b } = {...}`, we would get `a` and `b`. There is some unimplemented code here, but if the declarator is included, then set "included" to true.
      * if not tree shaking or included is true
        * if declarator should be separated, insert prefix and kind into code
        * set "empty" to false
    * render the declarator
  * if tree shaking and "empty" is true, remove the declaration
  * otherwise, insert a semicolon at end of declaration's code (unless in a for loop head)

### VariableDeclarator

[Source](https://github.com/rollup/rollup/blob/master/src/ast/nodes/VariableDeclarator.js)

#### Example 

```js
x = 'exit'
{ y, z } = obj
[ a, b ] = arr
```

#### Properties

* id - an `Identifier` or pattern
* init - a `Literal`, `Identifier`, or expression

#### Behaviors

* assignExpression - for each proxy, assign them the `UNKNOWN_ASSIGNMENT` expression
* hasEffects - return result of calling init node's `hasEffects` method or extract all of the names from the `id` node and return true if any of their proxies are `included`.
* initialiseNode
  * create a proxies map
  * find the node's lexical boundary scope
  * for each name (multiple names for patterns), create a `DeclaratorProxy` node, add that node to the proxies map, and add the proxy to the scope's declarations
* render - for each name
  * get the declaration from the proxies map
  * if not an es build, the declaration has an export name, and the declaration is re-assigned
    * if init is truthy, overwrite the code with the declaration's code
    * otherwise if tree shaking, remove the code
