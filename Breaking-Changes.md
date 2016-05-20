These changes list where implementation differs between versions as the spec and compiler are simplified and inconsistencies are corrected.

> For breaking changes to the compiler/services API, please check the [[API Breaking Changes]] page.

# TypeScript 1.8

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.8%22+label%3A%22Breaking+Change%22+is%3Aclosed).

#### Modules are now emitted with a `"use strict";` prologue

Modules were always parsed in strict mode as per ES6, but for non-ES6 targets this was not respected in the generated code. Starting with TypeScript 1.8, emitted modules are always in strict mode. This shouldn't have any visible changes in most code as TS considers most strict mode errors as errors at compile time, but it means that some things which used to silently fail at runtime in your TS code, like assigning to `NaN`, will now loudly fail. You can reference the [MDN Article](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) on strict mode for a detailed list of the differences between strict mode and non-strict mode.

To disable this behavior, pass `--noImplicitUseStrict` on the command line or set it in your tsconfig.json file.

#### Exporting non-local names from a module

In accordance with the ES6/ES2015 spec, it is an error to export a non-local name from a module.

**Example**

```ts
export { Promise }; // Error
```

**Recommendation**

Use a local variable declaration to capture the global name before exporting it.

```ts
const localPromise = Promise;
export { localPromise as Promise }; 
```

#### Reachability checks are enabled by default

In TypeScript 1.8 we've added a set of [reachability checks](https://github.com/Microsoft/TypeScript/pull/4788) to prevent certain categories of errors. Specifically

1. check if code is reachable (enabled by default, can be disabled via `allowUnreachableCode` compiler option)
   ```ts
      function test1() {
          return 1;
          return 2; // error here
      }
      
      function test2(x) {
          if (x) {
              return 1;
          }
          else {
              throw new Error("NYI")
          }
          var y = 1; // error here
      }
   ```
2. check if label is unused (enabled by default, can be disabled via `allowUnusedLabels` compiler option)
   ```ts
   l: // error will be reported - label `l` is unused
   while (true) { 
   }
   
   (x) => { x:x } // error will be reported - label `x` is unused
   ```
3. check if all code paths in function with return type annotation return some value (disabled by default, can be enabled via `noImplicitReturns` compiler option)

   ```ts
   // error will be reported since function does not return anything explicitly when `x` is falsy.
   function test(x): number {
      if (x) return 10;
   }
   ```
4. check if control flow falls through cases in switch statement (disabled by default, can be enabled via `noFallthroughCasesInSwitch` compiler option). Note that cases without statements are not reported.

   ```ts
   switch(x) {
      // OK
      case 1: 
      case 2: 
          return 1; 
   }
   switch(x) {
      case 1:
          if (y) return 1;
      case 2: 
          return 2;
   }
   ```

If these errors are showing up in your code and you still think that scenario when they appear is legitimate you can suppress errors with compiler options.

#### `--module` is not allowed alongside `--outFile` unless `--module` is specified as one of `amd` or `system`.

Previously specifying both while using modules would result in an empty `out` file and no error.

#### Changes to DOM API's in the standard library

* **ImageData.data** is now of type `Uint8ClampedArray` instead of `number[]`. See [#949](https://github.com/Microsoft/TypeScript/issues/949) for more details.
* **HTMLSelectElement .options** is now of type `HTMLCollection` instead of `HTMLSelectElement`. See [#1558](https://github.com/Microsoft/TypeScript/issues/1558) for more details.
* **HTMLTableElement.createCaption**, **HTMLTableElement.createTBody**,  **HTMLTableElement.createTFoot**,  **HTMLTableElement.createTHead**, **HTMLTableElement.insertRow**, **HTMLTableSectionElement.insertRow**, and **HTMLTableElement.insertRow** now return `HTMLTableRowElement` instead of `HTMLElement`. See [#3583](https://github.com/Microsoft/TypeScript/issues/3583) for more details.
* **HTMLTableRowElement.insertCell** now return `HTMLTableCellElement` instead of `HTMLElement`. See [#3583](https://github.com/Microsoft/TypeScript/issues/3583) for more details.
* **IDBObjectStore.createIndex** and **IDBDatabase.createIndex** second argument is now of type `IDBObjectStoreParameters` instead of `any`. See [#5932](https://github.com/Microsoft/TypeScript/issues/5932) for more details.
* **DataTransferItemList.Item** returns type now is `DataTransferItem` instead of `File`. See [#6106](https://github.com/Microsoft/TypeScript/issues/6106) for more details.
* **Window.open** return type now is `Window` instead of `any`. See [#6418](https://github.com/Microsoft/TypeScript/issues/6418) for more details.
* **WeakMap.clear** as removed. See [#6500](https://github.com/Microsoft/TypeScript/issues/6500) for more details.

#### Disallow `this` accessing before super-call
ES6 disallows accessing `this` in a constructor declaration.

For example:
```typescript
class B {
    constructor(that?: any) {}
}

class C extends B {
    constructor() {
        super(this);  // error;
    }
}

class D extends B {
    private _prop1: number;
    constructor() {
        this._prop1 = 10;  // error
        super();
    }
}
```

# TypeScript 1.7

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.7%22+label%3A%22breaking+change%22).

#### Changes in inferring the type from `this`

In a class, the type of the value `this` will be inferred to the `this` type.
This means subsequent assignments from values the original type can fail.

**Example:**

```TypeScript
class Fighter {
    /** @returns the winner of the fight. */
    fight(opponent: Fighter) {
        let theVeryBest = this;
        if (Math.rand() < 0.5) {
            theVeryBest = opponent; // error
        }
        return theVeryBest
    }
}
```

**Recommendations:**

Add a type annotation:

```TypeScript
class Fighter {
    /** @returns the winner of the fight. */
    fight(opponent: Fighter) {
        let theVeryBest: Fighter = this;
        if (Math.rand() < 0.5) {
            theVeryBest = opponent; // no error
        }
        return theVeryBest
    }
}
```

#### Automatic semicolon insertion after class member modifiers

The keywords `abstract, public, protected` and `private` are *FutureReservedWords* in ECMAScript 3 and are subject to automatic semicolon insertion. Previously, TypeScript did not insert semicolons when these keywords were on their own line. Now that this is fixed, `abstract class D` no longer correctly extends `C` in the following example, and instead declares a concrete method `m` and an additional property named `abstract`.

Note that `async` and `declare` already correctly did ASI. 

**Example:**

```TypeScript
abstract class C {
    abstract m(): number;
}
abstract class D extends C {
    abstract
    m(): number;
}
```

**Recommendations:**

Remove line breaks after keywords when defining class members. In general, avoid relying on automatic semicolon insertion. 

# TypeScript 1.6

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.6%22+label%3A%22breaking+change%22).

#### Strict object literal assignment checking

It is an error to specify properties in an object literal that were not specified on the target type, when assigned to a variable or passed for a parameter of a non-empty target type.

This new strictness can be disabled with the [--suppressExcessPropertyErrors](https://github.com/Microsoft/TypeScript/pull/4484) compiler option.

**Example:**

```typescript
var x: { foo: number };
x = { foo: 1, baz: 2 };  // Error, excess property `baz`

var y: { foo: number, bar?: number };
y = { foo: 1, baz: 2 };  // Error, excess or misspelled property `baz`
```

**Recommendations:**

To avoid the error, there are few remedies based on the situation you are looking into:

**If the target type accepts additional properties, add an indexer:**

```typescript
var x: { foo: number, [x: string]: any };
x = { foo: 1, baz: 2 };  // OK, `baz` matched by index signature
```

**If the source types are a set of related types, explicitly specify them using union types instead of just specifying the base type.**

```ts
let animalList: (Dog | Cat | Turkey)[] = [    // use union type instead of Animal
    {name: "Milo", meow: true }, 
    {name: "Pepper", bark: true},
    {name: "koko", gobble: true} 
];
```

**Otherwise, explicitly cast to the target type to avoid the warning message:**
```ts
interface Foo {
    foo: number;
}
interface FooBar {
    foo: number;
    bar: number;
}
var y: Foo;
y = <FooBar>{ foo: 1, bar: 2 };
```

#### CommonJS module resolution no longer assumes paths are relative

Previously, for the files `one.ts` and `two.ts`, an import of `"one"` in `two.ts` would resolve to `one.ts` if they resided in the same directory.

In TypeScript 1.6, `"one"` is no longer equivalent to "./one" when compiling with CommonJS. Instead, it is searched as relative to an appropriate `node_modules` folder as would be resolved by runtimes such as Node.js. For details, see [the issue that describes the resolution algorithm](https://github.com/Microsoft/TypeScript/issues/2338).

**Example:**

`./one.ts`
```TypeScript
export function f() {
    return 10;
}
```

`./two.ts`
```TypeScript
import { f as g } from "one";
``` 

**Recommendations:**

**Fix any non-relative import names that were unintended (strongly suggested).**

`./one.ts`
```TypeScript
export function f() {
    return 10;
}
```

`./two.ts`
```TypeScript
import { f as g } from "./one";
``` 

**Set the `--moduleResolution` compiler option to `classic`.**

#### Function and class default export declarations can no longer merge with entities intersecting in their meaning

Declaring an entity with the same name and in the same space as a default export declaration is now an error; for example,  

```TypeScript
export default function foo() {
}

namespace foo {
    var x = 100;
}
```

and

```TypeScript
export default class Foo {
    a: number;
}

interface Foo {
    b: string;
}
```

both cause an error.

However, in the following example, merging is allowed because the namespace does does not have a meaning in the value space:

```TypeScript
export default class Foo {
}

namespace Foo {
}
```

**Recommendations:**

Declare a local for your default export and use a separate `export default` statement as so:

```TypeScript
class Foo {
    a: number;
}

interface foo {
    b: string;
}

export default Foo;
```

For more details see [the originating issue](https://github.com/Microsoft/TypeScript/issues/3095).

#### Module bodies are parsed in strict mode

In accordance with [the ES6 spec](http://www.ecma-international.org/ecma-262/6.0/#sec-strict-mode-code), module bodies are now parsed in strict mode. module bodies will behave as if `"use strict"` was defined at the top of their scope; this includes flagging the use of `arguments` and `eval` as variable or parameter names, use of future reserved words as variables or parameters, use of octal numeric literals, etc..

#### Changes to DOM API's in the standard library

* **MessageEvent** and **ProgressEvent** constructors now expect arguments; see [issue #4295](https://github.com/Microsoft/TypeScript/issues/4295) for more details.
* **ImageData** constructor now expects arguments; see [issue #4220](https://github.com/Microsoft/TypeScript/issues/4220) for more details.
* **File** constructor now expects arguments; see [issue #3999](https://github.com/Microsoft/TypeScript/issues/3999) for more details.

#### System module output uses bulk exports

The compiler uses the [new bulk-export](https://github.com/ModuleLoader/es6-module-loader/issues/386) variation of the `_export` function in the System module format that takes any object containing key value pairs (optionally an entire module object for export *) as arguments instead of key, value.

The module loader needs to be updated to [v0.17.1](https://github.com/ModuleLoader/es6-module-loader/releases/tag/v0.17.1) or higher.

#### .js content of npm package is moved from 'bin' to 'lib' folder

Entry point of TypeScript npm package was moved from `bin` to `lib` to unblock scenarios when 'node_modules/typescript/bin/typescript.js' is served from IIS (by default `bin` is in the list of hidden segments so IIS will block access to this folder).

#### TypeScript npm package does not install globally by default

TypeScript 1.6 removes the `preferGlobal` flag from package.json. If you rely on this behaviour please use `npm install -g typescript`.

#### Decorators are checked as call expressions

Starting with 1.6, decorators type checking is more accurate; the compiler will checks a decorator expression as a call expression with the decorated entity as a parameter. This can cause error to be reported that were not in previous releases.

# TypeScript 1.5

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.5%22+label%3A%22breaking+change%22).

#### Referencing `arguments` in arrow functions is not allowed
This is an alignment with the ES6 semantics of arrow functions. Previously arguments within an arrow function would bind to the arrow function arguments. As per [ES6 spec draft](http://wiki.ecmascript.org/doku.php?id=harmony:specification_drafts) 9.2.12, arrow functions do not have an arguments objects. 
In TypeScript 1.5, the use of arguments object in arrow functions will be flagged as an error to ensure your code ports to ES6 with no change in semantics.

**Example:**
```ts
function f() {
    return () => arguments; // Error: The 'arguments' object cannot be referenced in an arrow function. 
}
```

**Recommendations:**
```ts
// 1. Use named rest args 
function f() {
    return (...args) => { args; }
}

// 2. Use function expressions instead
function f() {
    return function(){ arguments; }
}
```

#### Enum reference in-lining changes
For regular enums, pre 1.5, the compiler *only* inline constant members, and a member was only constant if its initializer was a literal. That resulted in inconsistent behavior depending on whether the enum value is initalized with a literal or an expression. Starting with Typescript 1.5 all non-const enum members are not inlined.

**Example:**
```ts
var x = E.a;  // previously inlined as "var x = 1; /*E.a*/"

enum E {
   a = 1
}
```

**Recommendation:**
Add the `const` modifier to the enum declaration to ensure it is consistently inlined at all consumption sites.

For more details see issue [#2183](https://github.com/Microsoft/TypeScript/issues/2183).


#### Contextual type flows through `super` and parenthesized expressions
Prior to this release, contextual types did not flow through parenthesized expressions. This has forced explicit type casts, especially in cases where parentheses are *required* to make an expression parse.

In the examples below, `m` will have a contextual type, where previously it did not.
```ts
var x: SomeType = (n) => ((m) => q); 
var y: SomeType = t ? (m => m.length) : undefined; 

class C extends CBase<string> {
    constructor() {
        super({
            method(m) { return m.length; }
        });
    }
}
```

See issues [#1425](https://github.com/Microsoft/TypeScript/issues/1425) and [#920](https://github.com/Microsoft/TypeScript/issues/920) for more details.

#### DOM interface changes
TypeScript 1.5 refreshes the DOM types in lib.d.ts. This is the first major refresh since TypeScript 1.0; many IE-specific definitions have been removed in favor of the standard DOM definitions. as well as adding missing types like Web Audio and touch events.

**Workaround:**

You can keep using older versions of the library with newer version of the compiler. You will need to include a local copy of a previous version in your project. Here is the [last released version before this change (TypeScript 1.5-alpha)](https://github.com/Microsoft/TypeScript/blob/v1.5.0-alpha/bin/lib.d.ts).

**Here is a list of changes:**
- Property ``selection`` is removed from type ``Document``
- Property ``clipboardData`` is removed from type ``Window``
- Removed interface ``MSEventAttachmentTarget``
- Properties ``onresize``, ``disabled``, ``uniqueID``, ``removeNode``, ``fireEvent``, ``currentStyle``, ``runtimeStyle`` are removed from type ``HTMLElement``
- Property ``url`` is removed from type ``Event``
- Properties ``execScript``, ``navigate``, ``item`` are removed from type ``Window``
- Properties ``documentMode``, ``parentWindow``, ``createEventObject`` are removed from type ``Document``
- Property ``parentWindow`` is removed from type ``HTMLDocument``
- Property ``setCapture`` does not exist anywhere now
- Property ``releaseCapture`` does not exist anywhere now
- Properties ``setAttribute``, ``styleFloat``, ``pixelLeft`` are removed from type ``CSSStyleDeclaration``
- Property ``selectorText`` is removed from type ``CSSRule``
- ``CSSStyleSheet.rules`` is of type ``CSSRuleList`` instead of ``MSCSSRuleList``
- ``documentElement`` is of type ``Element`` instead of ``HTMLElement``
- ``Event`` has a new required property ``returnValue``
- ``Node`` has a new required property ``baseURI``
- ``Element`` has a new required property ``classList``
- ``Location`` has a new required property ``origin``
- Properties ``MSPOINTER_TYPE_MOUSE``, ``MSPOINTER_TYPE_TOUCH`` are removed from type ``MSPointerEvent``
- ``CSSStyleRule`` has a new required property ``readonly``
- Property ``execUnsafeLocalFunction`` is removed from type ``MSApp``
- Global method ``toStaticHTML`` is removed
- ``HTMLCanvasElement.getContext`` now returns ``CanvasRenderingContext2D | WebGLRenderingContex``
- Removed extension types ``Dataview``, ``Weakmap``, ``Map``, ``Set``
- ``XMLHttpRequest.send`` has two overloads ``send(data?: Document): void;`` and ``send(data?: String): void;``
- ``window.orientation`` is of type ``string`` instead of ``number``
- IE-specific `attachEvent` and `detachEvent` are removed from `Window`

**Here is a list of libraries that are partly or entirely replaced by the added DOM types:**
- ``DefinitelyTyped/auth0/auth0.d.ts``
- ``DefinitelyTyped/gamepad/gamepad.d.ts``
- ``DefinitelyTyped/interactjs/interact.d.ts``
- ``DefinitelyTyped/webaudioapi/waa.d.ts``
- ``DefinitelyTyped/webcrypto/WebCrypto.d.ts``

For more details, please see the [full change](https://github.com/Microsoft/TypeScript/pull/2739).

#### Class bodies are parsed in strict mode

In accordance with [the ES6 spec](http://www.ecma-international.org/ecma-262/6.0/#sec-strict-mode-code), class bodies are now parsed in strict mode. Class bodies will behave as if `"use strict"` was defined at the top of their scope; this includes flagging the use of `arguments` and `eval` as variable or parameter names, use of future reserved words as variables or parameters, use of octal numeric literals, etc..

# TypeScript 1.4

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.4%22+label%3A%22breaking+change%22).

See [issue #868](https://github.com/Microsoft/TypeScript/issues/868) for more details about breaking changes related to Union Types

#### Multiple Best Common Type Candidates
Given multiple viable candidates from a Best Common Type computation we now choose an item (depending on the compiler's implementation) rather than the first item.

```ts
var a: { x: number; y?: number };
var b: { x: number; z?: number };

// was { x: number; z?: number; }[]
// now { x: number; y?: number; }[]
var bs = [b, a]; 
```

This can happen in a variety of circumstances. A shared set of required properties and a disjoint set of other properties (optional or otherwise), empty types, compatible signature types (including generic and non-generic signatures when type parameters are stamped out with ```any```).

**Recommendation**
Provide a type annotation if you need a specific type to be chosen
```ts
var bs: { x: number; y?: number; z?: number }[] = [b, a];
```

#### Generic Type Inference
Using different types for multiple arguments of type T is now an error, even with constraints involved:

```ts
declare function foo<T>(x: T, y:T): T;
var r = foo(1, ""); // r used to be {}, now this is an error
```
With constraints:

```ts
interface Animal { x }
interface Giraffe extends Animal { y }
interface Elephant extends Animal { z }
function f<T extends Animal>(x: T, y: T): T { return undefined; }
var g: Giraffe;
var e: Elephant;
f(g, e);
```

See https://github.com/Microsoft/TypeScript/pull/824#discussion_r18665727 for explanation.

**Recommendations**
Specify an explicit type parameter if the mismatch was intentional:
```ts
var r = foo<{}>(1, ""); // Emulates 1.0 behavior
var r = foo<string|number>(1, ""); // Most useful
var r = foo<any>(1, ""); // Easiest
f<Animal>(g, e);
```
*or* rewrite the function definition to specify that mismatches are OK:
```ts
declare function foo<T,U>(x: T, y:U): T|U;
function f<T extends Animal, U extends Animal>(x: T, y: U): T|U { return undefined; }
```

#### Generic Rest Parameters 
You cannot use heterogeneous argument types anymore:

```ts
function makeArray<T>(...items: T[]): T[] { return items; }
var r = makeArray(1, ""); // used to return {}[], now an error
```
Likewise for `new Array(...)`

**Recommendations**
Declare a back-compat signature if the 1.0 behavior was desired:
```ts
function makeArray<T>(...items: T[]): T[];
function makeArray(...items: {}[]): {}[];
function makeArray<T>(...items: T[]): T[] { return items; }
```

#### Overload Resolution with Type Argument Inference

```ts
var f10: <T>(x: T, b: () => (a: T) => void, y: T) => T;
var r9 = f10('', () => (a => a.foo), 1); // r9 was any, now this is an error
```

**Recommendations**
Manually specify a type parameter
```ts
var r9 = f10<any>('', () => (a => a.foo), 1);
```

#### Strict Mode Parsing for Class Declarations and Class Expressions
ECMAScript 2015 Language Specification (ECMA-262 6<sup>th</sup> Edition) specifies that *ClassDeclaration* and *ClassExpression* are strict mode productions. 
Thus, additional restrictions will be applied when parsing a class declaration or class expression. 

Examples:

```ts
class implements {}  // Invalid: implements is a reserved word in strict mode
class C {
    foo(arguments: any) {   // Invalid: "arguments" is not allow as a function argument
        var eval = 10;      // Invalid: "eval" is not allowed as the left-hand-side expression
        arguments = [];     // Invalid: arguments object is immutable
	}
}
```
For complete list of strict mode restrictions, please see Annex C - The Strict Mode of ECMAScript of ECMA-262 6<sup>th</sup> Edition.


# TypeScript 1.1

For full list of breaking changes see the [breaking change issues](https://github.com/Microsoft/TypeScript/issues?q=is%3Aissue+milestone%3A%22TypeScript+1.1%22+label%3A%22breaking+change%22+).

## Working with null and undefined in ways that are observably incorrect is now an error

Examples: 

```TypeScript
var ResultIsNumber17 = +(null + undefined);
// Operator '+' cannot be applied to types 'undefined' and 'undefined'.

var ResultIsNumber18 = +(null + null);
// Operator '+' cannot be applied to types 'null' and 'null'.

var ResultIsNumber19 = +(undefined + undefined);
// Operator '+' cannot be applied to types 'undefined' and 'undefined'.
```

Similarly, using null and undefined directly as objects that have methods now is an error

Examples:

```TypeScript
null.toBAZ();

undefined.toBAZ();
```