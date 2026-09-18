# TypeScript Syntax Reference

A single-page lookup for TypeScript syntax, type operators, utility types, and general-purpose idioms.

Everything here is language reference material — nothing is specific to any framework or domain. React, Next.js, Tailwind and react-three-fiber have their own pages.

**How to use this page.** It is deliberately one long file so browser find (`Ctrl+F` / `Cmd+F`) is the search tool. Search for the operator or keyword — `satisfies`, `infer`, `asserts`, `NoInfer` — rather than a description.

Three annotations appear in the code:

| Mark | Meaning |
|---|---|
| `// →` | the value the expression produces at runtime |
| `// ^?` | the type TypeScript infers for the expression |
| `// ✗` | a compile error, followed by the message |

Everything is checked against **TypeScript 7.0** (and re-checked against 6.0) with `strict: true`. The two agree on every example here — 7.0's type checker is a port of 6.0's, not a redesign. Under non-strict settings roughly a third of this page behaves differently; see [tsconfig](#tsconfig).

---

## Contents

[Annotating values](#annotating-values) · [Inference and widening](#inference-and-widening) · [Unions](#unions) · [Narrowing](#narrowing) · [Objects](#objects) · [Interfaces vs type aliases](#interfaces-vs-type-aliases) · [Arrays](#arrays) · [Tuples](#tuples) · [Functions](#functions) · [Overloads](#overloads) · [Null and undefined](#null-and-undefined) · [Type operators](#type-operators) · [Conditional types](#conditional-types) · [Mapped types](#mapped-types) · [Template literal types](#template-literal-types) · [Generics](#generics) · [Utility types](#utility-types) · [Classes](#classes) · [Enums](#enums) · [Assertions](#assertions-and-satisfies) · [The top and bottom types](#unknown-any-never-void) · [Async](#async-and-promises) · [Errors](#errors-and-catch) · [Modules](#modules) · [Declaration files](#declaration-files) · [Idioms](#general-idioms) · [Gotchas](#gotchas) · [tsconfig](#tsconfig) · [Versions](#version-notes)

---

## Annotating values

| Syntax | Meaning |
|---|---|
| `const x: number = 1` | annotate a variable |
| `function f(a: string): void` | annotate parameters and the return |
| `(a: number) => string` | a function *type* |
| `let x: string \| null` | a union |
| `type Name = string` | a type alias |
| `x!` | non-null assertion |
| `x as T` | type assertion |
| `x satisfies T` | check against `T`, keep the narrower type |

The primitive types are lowercase. `String`, `Number` and `Boolean` are the wrapper-object types and are almost never what you want.

```ts
const n: number = 42;
const s: string = "hi";
const b: boolean = true;
const big: bigint = 10n;
const sym: symbol = Symbol("k");
const u: undefined = undefined;
const nul: null = null;

// arrays — two spellings, identical meaning
const xs: number[] = [1, 2, 3];
const ys: Array<number> = [1, 2, 3];

// functions
function greet(name: string): string {
  return `hi ${name}`;
}
const greet2: (name: string) => string = (name) => `hi ${name}`;
const greet3 = (name: string): string => `hi ${name}`;

greet("ada");                       // → 'hi ada'

// objects
const pt: { x: number; y: number } = { x: 1, y: 2 };

// aliases — a name for any type, not just objects
type ID = string | number;
type Point = { x: number; y: number };
type Handler = (e: string) => void;

const id: ID = 7;                   // → 7

// ✗ const bad: number = "1";
//   Type 'string' is not assignable to type 'number'.
```

The annotation is a *constraint*, not a conversion. Nothing about the emitted JavaScript changes; every type in this document is erased at compile time.

```ts
// this file compiles to exactly: const v = "42"; const bumped = v + 1;
const v = "42" as unknown as number;
const bumped = v + 1;               // → '421'    string concatenation, at runtime
//    ^? number                     the type says otherwise, and is simply wrong
```

---

## Inference and widening

You rarely need to annotate a variable. You almost always need to annotate a parameter.

| Declaration | Inferred |
|---|---|
| `let s = "a"` | `string` — widened |
| `const s = "a"` | `"a"` — the literal |
| `let n = 1` | `number` |
| `const o = { a: 1 }` | `{ a: number }` — properties widen even under `const` |
| `const o = { a: 1 } as const` | `{ readonly a: 1 }` |
| `const xs = [1, 2]` | `number[]` |
| `const xs = [1, 2] as const` | `readonly [1, 2]` |
| `let x` | `any` — implicitly, an *evolving* any |
| `let x: unknown` | `unknown` |

```ts
let mutable = "a";
//  ^? string                       reassignable, so the literal is widened

const fixed = "a";
//    ^? "a"                        can never change, so the literal survives

const obj = { kind: "circle", r: 2 };
//    ^? { kind: string; r: number }    obj.kind is reassignable

const frozen = { kind: "circle", r: 2 } as const;
//    ^? { readonly kind: "circle"; readonly r: 2 }

const list = [1, 2, 3];
//    ^? number[]
const pair = [1, "a"];
//    ^? (string | number)[]        NOT a tuple
const tup = [1, "a"] as const;
//    ^? readonly [1, "a"]

// return types are inferred; annotate them anyway on exported functions
function add(a: number, b: number) {
  return a + b;
}
//       ^? (a: number, b: number) => number
add(1, 2);                          // → 3

// ✗ function bad(a) { return a; }
//   Parameter 'a' implicitly has an 'any' type.
```

### Where widening bites

```ts
type Method = "GET" | "POST";

const cfg = { method: "GET" };
// ✗ fetchWith(cfg.method)
//   Argument of type 'string' is not assignable to parameter of type 'Method'.
function fetchWith(m: Method) { return m; }

// three fixes, in order of preference
const cfg1 = { method: "GET" } as const;                    // freeze everything
const cfg2 = { method: "GET" } satisfies { method: Method } // check, keep literal
const cfg3: { method: Method } = { method: "GET" };

// note that `satisfies` must wrap the whole literal. Applied to the inner
// value it checks, then the property widens anyway:
// ✗ const wrong = { method: "GET" satisfies Method };
//   fetchWith(wrong.method) → 'string' is not assignable to 'Method'.

fetchWith(cfg1.method);             // → 'GET'
fetchWith(cfg2.method);             // → 'GET'
fetchWith(cfg3.method);             // → 'GET'
```

### Contextual typing

Inference runs backwards too: a parameter's type can come from the position the function is used in.

```ts
const nums = [1, 2, 3];

nums.map((n) => n * 2);             // → [2, 4, 6]
//        ^? n is number, inferred from the array

window.addEventListener("click", (e) => e.clientX);
//                                ^? e is MouseEvent

type Cb = (a: string, b: number) => void;
const cb: Cb = (a, b) => {};        // no annotations needed on a or b

// but a standalone function gets nothing
// ✗ const loose = (n) => n * 2;
//   Parameter 'n' implicitly has an 'any' type.
```

---

## Unions

`A | B` is "one of". `A & B` is "all of at once".

| Syntax | Meaning |
|---|---|
| `A \| B` | union — either |
| `A & B` | intersection — both |
| `"a" \| "b"` | literal union, the workhorse of TS |
| `keyof T` | the union of `T`'s keys |
| `T[keyof T]` | the union of `T`'s value types |

Before narrowing, only the members *common to every branch* are reachable.

```ts
type Res = { ok: true; data: string } | { ok: false; error: string };

function handle(r: Res) {
  r.ok;                             // fine — present on both
  // ✗ r.data
  //   Property 'data' does not exist on type 'Res'.
  //     Property 'data' does not exist on type '{ ok: false; error: string; }'.
  return r.ok ? r.data : r.error;   // narrowed by the check
}

handle({ ok: true, data: "x" });    // → 'x'
handle({ ok: false, error: "e" });  // → 'e'

// a union of strings is an enum you do not have to import
type Dir = "up" | "down";
const d: Dir = "up";                // → 'up'
// ✗ const bad: Dir = "left";
//   Type '"left"' is not assignable to type 'Dir'.

// intersections merge object types
type WithId = { id: string };
type WithTime = { at: number };
type Row = WithId & WithTime;
const row: Row = { id: "a", at: 1 };
row.id;                             // → 'a'

// intersecting incompatible primitives yields never
type Nope = string & number;
//   ^? never
```

### Unions distribute over arrays and functions in different directions

```ts
type A = { a: 1 };
type B = { b: 2 };

// an array of either vs either array — not the same
const mixed: (A | B)[] = [{ a: 1 }, { b: 2 }];        // ok
// ✗ const split: A[] | B[] = [{ a: 1 }, { b: 2 }];
//   Type '{ b: 2; }' is not assignable to type 'A'.

// a union of function types only accepts the intersection of their parameters
type F = ((n: number) => void) | ((s: string) => void);
declare const f: F;
// ✗ f(1)
//   Argument of type 'number' is not assignable to parameter of type 'never'.
```

---

## Narrowing

Narrowing is how a union becomes a single member. TypeScript follows control flow; it does not evaluate your logic, it pattern-matches on a fixed set of forms.

| Guard | Narrows |
|---|---|
| `typeof x === "string"` | primitives |
| `Array.isArray(x)` | arrays |
| `x instanceof Cls` | class instances |
| `"key" in x` | object shapes |
| `x.kind === "a"` | discriminated unions |
| `x === null`, `x != null` | nullish |
| `if (x)` | truthiness — **beware `0` and `""`** |
| `function isT(x): x is T` | anything, via a user-defined predicate |
| `function assertT(x): asserts x is T` | anything, by throwing |

```ts
function fmt(v: string | number | boolean | null) {
  if (v === null) return "null";
  if (typeof v === "string") return v.toUpperCase();
  //                              ^? v is string
  if (typeof v === "number") return v.toFixed(1);
  return v ? "yes" : "no";
  //     ^? v is boolean — everything else eliminated
}

fmt("ab");                          // → 'AB'
fmt(3);                             // → '3.0'
fmt(true);                          // → 'yes'
fmt(null);                          // → 'null'

// `typeof` strings TypeScript understands
// "string" "number" "bigint" "boolean" "symbol" "undefined" "object" "function"
// note: typeof null === "object", so it does NOT narrow null away
function t(v: string | null) {
  if (typeof v === "object") return v;
  //                                ^? null
  return v.length;
}
t(null);                            // → null
t("abc");                           // → 3
```

### Discriminated unions

The single most useful pattern in TypeScript. Give every member a shared literal-typed field.

```ts
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "rect"; w: number; h: number }
  | { kind: "square"; side: number };

function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.r ** 2;
    case "rect":   return s.w * s.h;
    case "square": return s.side ** 2;
  }
}

area({ kind: "rect", w: 2, h: 3 });         // → 6
area({ kind: "square", side: 4 });          // → 16

// the discriminant must be a literal type, so `as const` or an explicit
// annotation is required when the value is built separately
const c = { kind: "circle", r: 1 } as const;
area(c);                                    // → 3.141592653589793
```

### Exhaustiveness with `never`

```ts
type Shape2 = { kind: "a" } | { kind: "b" };

function check(s: Shape2): string {
  switch (s.kind) {
    case "a": return "A";
    case "b": return "B";
    default: {
      const _exhaustive: never = s;         // ✗ here if a case is added
      return _exhaustive;
    }
  }
}
check({ kind: "b" });                       // → 'B'

// add `| { kind: "c" }` to Shape2 and the default branch fails with:
//   Type '{ kind: "c"; }' is not assignable to type 'never'.
// which is exactly the compile error you want.
```

### Type predicates

`x is T` tells the compiler what a boolean return *means*. It is unchecked — you are asserting, and a wrong predicate silently lies.

```ts
type Cat = { meow(): void };
type Dog = { bark(): void };

function isCat(a: Cat | Dog): a is Cat {
  return "meow" in a;
}

function speak(a: Cat | Dog) {
  if (isCat(a)) return "meow";
  //     ^? a is Cat
  return "bark";
}
speak({ bark() {} });                       // → 'bark'

// the classic: filtering nulls out of an array
const raw: (string | null)[] = ["a", null, "b"];

raw.filter((v) => v !== null);
//  ^? (string | null)[]                    the inline check is NOT carried out

raw.filter((v): v is string => v !== null);
//  ^? string[]
// → ['a', 'b']

// TS 5.5+ infers the predicate for simple one-expression callbacks
const clean = raw.filter((v) => v !== null);
//    ^? string[]                           inferred since 5.5
clean;                                      // → ['a', 'b']
```

### Assertion functions

```ts
function assertString(v: unknown): asserts v is string {
  if (typeof v !== "string") throw new Error("not a string");
}

function useIt(v: unknown) {
  assertString(v);
  return v.length;                          // v is string from here on
}
useIt("abcd");                              // → 4

// an assertion function must be called through an explicitly typed binding
const assertNum: (v: unknown) => asserts v is number = (v) => {
  if (typeof v !== "number") throw new Error("nope");
};
// ✗ const inferred = (v: unknown): asserts v is number => {...}
//   Assertions require every name in the call target to be declared with an
//   explicit type annotation.
```

### Where narrowing is lost

```ts
type Box = { v?: string };

function lost(b: Box) {
  if (b.v) {
    // narrowing survives straight-line code
    b.v.length;                             // ok
    doSomething();
    // ✗ b.v.length
    //   nothing is invalidated by a call — but a closure below IS
    return b.v.length;
  }
  return 0;
}
function doSomething() {}
lost({ v: "abc" });                         // → 3

// mutation inside a callback defeats it
function inCallback(b: Box) {
  if (b.v !== undefined) {
    return [1].map(() => b.v!.length);      // ! needed: TS cannot prove b.v survives
  }
  return [];
}
inCallback({ v: "ab" });                    // → [2]

// copy to a const and the narrowing sticks
function viaConst(b: Box) {
  const v = b.v;
  if (v !== undefined) {
    return [1].map(() => v.length);         // no ! needed
  }
  return [];
}
viaConst({ v: "ab" });                      // → [2]

// `let` reassigned in a branch re-widens
let val: string | number = "a";
val.length;                                 // ok, narrowed to string
val = 1;
val.toFixed();                              // now narrowed to number
// → '1'
```

### Truthiness narrowing is a trap

```ts
function len(s: string | undefined) {
  if (s) return s.length;                   // "" is falsy — treated as missing
  return -1;
}
len("");                                    // → -1        probably a bug
len("ab");                                  // → 2

function len2(s: string | undefined) {
  if (s !== undefined) return s.length;     // test for the thing you mean
  return -1;
}
len2("");                                   // → 0
```

---

## Objects

| Syntax | Meaning |
|---|---|
| `{ a: string }` | required property |
| `{ a?: string }` | optional — type becomes `string \| undefined` |
| `{ readonly a: string }` | no reassignment (compile-time only) |
| `{ [k: string]: number }` | index signature |
| `{ a: string } & { b: number }` | both |
| `Record<string, number>` | the index signature, spelled shorter |
| `{ f(): void }` | method shorthand |
| `{ f: () => void }` | property holding a function |

```ts
type User = {
  readonly id: string;
  name: string;
  age?: number;                     // string | undefined
  tags: string[];
  greet(): string;                  // method syntax
  onSave?: (u: User) => void;
};

const u: User = {
  id: "u1",
  name: "Ada",
  tags: [],
  greet() { return `hi ${this.name}`; },
};

u.greet();                          // → 'hi Ada'
u.age;                              // → undefined
u.name = "Grace";                   // ok
// ✗ u.id = "u2";
//   Cannot assign to 'id' because it is a read-only property.

// readonly is shallow and erased at runtime
type Frozen = { readonly xs: number[] };
const f: Frozen = { xs: [1] };
f.xs.push(2);                       // allowed — the array itself is mutable
f.xs;                               // → [1, 2]
```

### Excess property checks

An *object literal* assigned directly to a typed slot may not carry unknown keys. Going through a variable skips the check.

```ts
type Opts = { url: string; retries?: number };

// ✗ const a: Opts = { url: "/x", timeout: 5 };
//   Object literal may only specify known properties, and 'timeout' does not
//   exist in type 'Opts'.

const loose = { url: "/x", timeout: 5 };
const b: Opts = loose;              // fine — not a fresh literal
b.url;                              // → '/x'

// this is a *freshness* check, not a structural rule. Structural typing says
// "more properties is fine"; the literal check exists to catch typos.
```

### Index signatures

```ts
type Counts = { [key: string]: number };

const c: Counts = { a: 1, b: 2 };
c.z;                                // → undefined   but typed `number`, not `number | undefined`
c["q"] = 3;
c;                                  // → { a: 1, b: 2, q: 3 }

// noUncheckedIndexedAccess: true fixes the lie above, at the cost of ! or ??
// then c.z is `number | undefined`

// every declared property must conform to the index signature
// ✗ type Bad = { [k: string]: number; name: string };
//   Property 'name' of type 'string' is not assignable to 'string' index
//   type 'number'.

type Ok = { [k: string]: number | string; name: string };
const ok: Ok = { name: "a", n: 1 };
ok.name;                            // → 'a'

// a number index must be assignable to the string index
type Both = { [k: string]: string; [n: number]: string };

// Record is the same thing with a closed key set
type Flags = Record<"read" | "write", boolean>;
const fl: Flags = { read: true, write: false };
fl.read;                            // → true
// ✗ const missing: Flags = { read: true };
//   Property 'write' is missing in type '{ read: boolean; }'.
```

### Reading and combining

```ts
const base = { a: 1, b: 2 };

const spread = { ...base, b: 9 };
//    ^? { a: number; b: number }
spread;                             // → { a: 1, b: 9 }

const { a, ...rest } = base;
a;                                  // → 1
rest;                               // → { b: 2 }
//^? { b: number }

// destructuring with types and defaults
function draw({ x, y = 0 }: { x: number; y?: number }) {
  return [x, y];
}
draw({ x: 1 });                     // → [1, 0]

// renaming: the left side is a NEW NAME, not a type
const { a: alpha } = base;
alpha;                              // → 1

// optional chaining and key checks
const maybe: { a?: { b?: number } } = {};
maybe.a?.b;                         // → undefined
"a" in maybe;                       // → false
Object.keys(base);                  // → ['a', 'b']
//^? string[]                       NOT ('a' | 'b')[] — see Gotchas
```

---

## Interfaces vs type aliases

Both describe object shapes. They are interchangeable in most code.

| | `interface` | `type` |
|---|---|---|
| object shapes | yes | yes |
| unions, primitives, tuples | no | yes |
| mapped / conditional types | no | yes |
| extends | `extends A, B` | `A & B` |
| declaration merging | yes | no |
| implemented by a class | yes | yes (object shapes) |
| error messages | often shorter, keeps the name | often expanded inline |
| recursion | fine | fine |

```ts
interface Animal {
  name: string;
}
interface Dog extends Animal {
  breed: string;
}
const d: Dog = { name: "Rex", breed: "lab" };
d.name;                             // → 'Rex'

type AnimalT = { name: string };
type DogT = AnimalT & { breed: string };

// only `type` can name these
type Id = string | number;
type Pair = [number, number];
type Keys<T> = keyof T;
type Maybe<T> = T | null;

// only `interface` merges — two declarations become one
interface Win { a: number }
interface Win { b: number }
const w: Win = { a: 1, b: 2 };
w.b;                                // → 2

// ✗ type T1 = { a: number };
//   type T1 = { b: number };
//   Duplicate identifier 'T1'.

// interfaces are open, which is why they are used for global augmentation,
// and why a type alias is the safer default for your own data shapes.
```

Rule of thumb: `type` by default; `interface` when you publish a shape that consumers may need to augment, or when you want the name preserved in errors.

---

## Arrays

| Task | Code | Result type |
|---|---|---|
| annotate | `number[]` / `Array<number>` | |
| readonly | `readonly number[]` / `ReadonlyArray<number>` | |
| element type | `T[number]` | |
| empty literal | `const xs: string[] = []` | otherwise `never[]` |
| `map` | `xs.map(f)` | `U[]` |
| `filter` | `xs.filter(p)` | same type, unless `p` is a predicate |
| `find` | `xs.find(p)` | `T \| undefined` |
| `at` | `xs.at(-1)` | `T \| undefined` |
| `includes` | `xs.includes(v)` | `boolean`, `v` must be assignable to `T` |
| `flat` | `xs.flat()` | one level by default |
| `reduce` | `xs.reduce(f, init)` | annotate the accumulator |

```ts
const xs = [1, 2, 3];

xs.map((n) => n * 2);               // → [2, 4, 6]
//^? number[]
xs.map(String);                     // → ['1', '2', '3']
//^? string[]
xs.filter((n) => n > 1);            // → [2, 3]
xs.find((n) => n > 1);              // → 2
//^? number | undefined
xs.at(-1);                          // → 3
//^? number | undefined
xs.includes(2);                     // → true
xs.indexOf(9);                      // → -1
[...xs].reverse();                  // → [3, 2, 1]
[[1], [2, 3]].flat();               // → [1, 2, 3]
xs.slice(0, 2);                     // → [1, 2]
xs.join("-");                       // → '1-2-3'

// reduce almost always needs the accumulator annotated
xs.reduce((acc, n) => acc + n, 0);          // → 6
xs.reduce<Record<string, number>>(
  (acc, n) => ({ ...acc, [n]: n }), {});    // → { '1': 1, '2': 2, '3': 3 }

// an empty literal with no annotation infers never[]
const empty = [];
//    ^? any[]                      an evolving any under noImplicitAny
const typed: string[] = [];
typed.push("a");
typed;                              // → ['a']

// indexing is unchecked by default — this is the single biggest hole in strict mode
const first = xs[99];
//    ^? number                     but the value is undefined
first;                              // → undefined
// noUncheckedIndexedAccess: true makes it `number | undefined`
```

### readonly arrays

```ts
const ro: readonly number[] = [1, 2, 3];

ro[0];                              // → 1
ro.length;                          // → 3
[...ro].sort();                     // → [1, 2, 3]   copy first
// ✗ ro.push(4);
//   Property 'push' does not exist on type 'readonly number[]'.
// ✗ ro[0] = 9;
//   Index signature in type 'readonly number[]' only permits reading.

// assignability goes one way
const mut: number[] = [1];
const asRo: readonly number[] = mut;        // ok — widening to readonly
// ✗ const back: number[] = ro;
//   The type 'readonly number[]' is 'readonly' and cannot be assigned to the
//   mutable type 'number[]'.

// accept readonly in parameters; it costs callers nothing and documents intent
function sum(ns: readonly number[]) {
  return ns.reduce((a, b) => a + b, 0);
}
sum([1, 2, 3]);                     // → 6
sum(ro);                            // → 6
```

---

## Tuples

Fixed-length, position-typed arrays.

| Syntax | Meaning |
|---|---|
| `[string, number]` | exactly two, in that order |
| `[string, number?]` | optional trailing element |
| `[string, ...number[]]` | rest element |
| `[name: string, age: number]` | named members (labels only) |
| `readonly [1, 2]` | frozen, from `as const` |
| `T[0]`, `T[number]` | index into it |

```ts
type Entry = [key: string, value: number];

const e: Entry = ["a", 1];
e[0];                               // → 'a'
//^? string
e.length;                           // → 2
//^? 2                              a literal, not number

const [k, v] = e;
k;                                  // → 'a'
v;                                  // → 1

// ✗ const bad: Entry = ["a"];
//   Source has 1 element(s) but target requires 2.

type Opt = [string, number?];
const o1: Opt = ["a"];              // → ['a']
const o2: Opt = ["a", 1];           // → ['a', 1]

type Rest = [string, ...number[]];
const r: Rest = ["a", 1, 2, 3];     // → ['a', 1, 2, 3]

// indexed access
type K = Entry[0];
//   ^? string
type Any = Entry[number];
//   ^? string | number

// as const produces a readonly tuple, which is how you get literal positions
const cfg = ["GET", 200] as const;
//    ^? readonly ["GET", 200]
cfg[0];                             // → 'GET'

// tuples are how you type variadic function parameters
type Args = Parameters<(a: string, b: number) => void>;
//   ^? [a: string, b: number]

function call<A extends unknown[], R>(f: (...a: A) => R, ...args: A): R {
  return f(...args);
}
call((a: number, b: number) => a + b, 1, 2);        // → 3
// ✗ call((a: number, b: number) => a + b, 1, "x");
//   Argument of type 'string' is not assignable to parameter of type 'number'.
```

### Building tuples from unions

```ts
// swap a pair
function swap<A, B>([a, b]: [A, B]): [B, A] {
  return [b, a];
}
swap([1, "x"]);                     // → ['x', 1]
//^? [string, number]

// Object.entries loses literal keys; a manual tuple keeps them
const pairs = [["a", 1], ["b", 2]] as const;
//    ^? readonly [readonly ["a", 1], readonly ["b", 2]]
pairs[0][0];                        // → 'a'
```

---

## Functions

| Syntax | Meaning |
|---|---|
| `(a: T) => R` | function type |
| `(a?: T) => R` | optional parameter — must be trailing |
| `(a: T = d) => R` | default; makes the parameter optional for callers |
| `(...rest: T[]) => R` | rest parameter |
| `(this: T, a: U) => R` | typed `this`, erased at runtime |
| `function f(): asserts x is T` | assertion function |
| `function f(): x is T` | type predicate |
| `(): never` | never returns normally |
| `new (a: T) => R` | construct signature |

```ts
function f(a: number, b = 1, ...rest: string[]): string {
  return `${a}${b}${rest.join("")}`;
}
f(1);                               // → '11'
f(1, 2, "a", "b");                  // → '12ab'

// optional vs undefined-able — NOT the same
function opt(a?: string) { return a ?? "d"; }
function und(a: string | undefined) { return a ?? "d"; }
opt();                              // → 'd'
und(undefined);                     // → 'd'
// ✗ und();
//   Expected 1 arguments, but got 0.

// parameters are checked bivariantly for methods, contravariantly for
// function-typed properties under strictFunctionTypes
type Fn = (a: string | number) => void;
const narrow = (a: string) => {};
// ✗ const g: Fn = narrow;
//   Type '(a: string) => void' is not assignable to type 'Fn'.
//     Type 'string | number' is not assignable to type 'string'.

// but FEWER parameters is always fine — this is why callbacks work
[1, 2].map(() => 0);                // → [0, 0]
["a"].forEach((s) => s);            // → undefined

// void return means "I ignore the result", not "you must return nothing"
type Void = () => void;
const returnsNum: Void = () => 1;   // allowed
returnsNum();                       // → 1, but typed void

// this is why this compiles and silently misbehaves:
const out: number[] = [];
[1, 2].forEach((n) => out.push(n)); // push returns number; forEach wants void
out;                                // → [1, 2]

// typed `this`
function withThis(this: { n: number }, add: number) {
  return this.n + add;
}
withThis.call({ n: 1 }, 2);         // → 3

// never — a function that always throws or loops
function fail(msg: string): never {
  throw new Error(msg);
}
function branch(v: string | number) {
  if (typeof v === "string") return v;
  if (typeof v === "number") return String(v);
  return fail("unreachable");       // keeps the return type string
}
branch(2);                          // → '2'
```

### Generic functions

```ts
function first<T>(xs: readonly T[]): T | undefined {
  return xs[0];
}
first([1, 2]);                      // → 1
//^? number | undefined
first(["a"]);                       // → 'a'
//^? string | undefined

// constrain to get at properties
function byLength<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}
byLength("abc", "d");               // → 'abc'
byLength([1, 2], [1]);              // → [1, 2]
// ✗ byLength(1, 2);
//   Argument of type 'number' is not assignable to parameter of type
//   '{ length: number; }'.

// keyof for safe property access
function pluck<T, K extends keyof T>(o: T, k: K): T[K] {
  return o[k];
}
pluck({ a: 1, b: "x" }, "b");       // → 'x'
//^? string
// ✗ pluck({ a: 1 }, "z");
//   Argument of type '"z"' is not assignable to parameter of type '"a"'.

// const type parameters (5.0+) preserve literals without `as const`
function route<const T extends string[]>(parts: T): T {
  return parts;
}
route(["a", "b"]);                  // → ['a', 'b']
//^? readonly ["a", "b"]

// NoInfer (5.4+) stops a parameter from being an inference site
function pick<T>(xs: T[], fallback: NoInfer<T>): T {
  return xs[0] ?? fallback;
}
pick(["a", "b"], "c");              // → 'a'
// ✗ pick(["a", "b"], 1);
//   Argument of type 'number' is not assignable to parameter of type 'string'.
// without NoInfer, T would widen to string | number and the mistake would pass
```

---

## Overloads

Several call signatures, one implementation. The implementation signature is not callable from outside.

```ts
function parse(v: string): string[];
function parse(v: number): number[];
function parse(v: string | number): string[] | number[] {
  return typeof v === "string" ? v.split("") : [v];
}

parse("ab");                        // → ['a', 'b']
//^? string[]                       not string[] | number[]
parse(1);                           // → [1]
//^? number[]
// ✗ parse(true);
//   No overload matches this call.

// overloads are tried top to bottom — put the most specific first
// ✗ ordering trap: a broad signature first swallows the narrow one

// union parameters are usually better than overloads
function better(v: string | number): (string | number)[] {
  return [v];
}

// generics are better still when input and output are linked
function best<T>(v: T): T[] {
  return [v];
}
best("a");                          // → ['a']
//^? string[]

// object methods can be overloaded in a type
type Api = {
  get(path: "/users"): Promise<{ id: string }[]>;
  get(path: "/config"): Promise<{ debug: boolean }>;
};
```

---

## Null and undefined

Under `strictNullChecks` (part of `strict`) neither is assignable to anything else.

| Task | Code | Note |
|---|---|---|
| allow null | `T \| null` | |
| optional property | `a?: T` | adds `undefined`, not `null` |
| default when nullish | `x ?? d` | only `null` / `undefined` trigger it |
| default when falsy | `x \|\| d` | `0` and `""` trigger it too |
| assign if nullish | `x ??= d` | |
| read through possibly-missing | `a?.b?.c` | short-circuits to `undefined` |
| call if present | `f?.()` | |
| index if present | `a?.[i]` | |
| assert non-null | `x!` | unchecked; erased |
| test for both at once | `x != null` | `!=` not `!==`, deliberately |

```ts
type Cfg = { host?: string; port: number | null };
const c: Cfg = { port: null };

c.host;                             // → undefined
c.host ?? "localhost";              // → 'localhost'
c.port ?? 80;                       // → 80

const zero: number | undefined = 0;
zero ?? 99;                         // → 0     only nullish falls through
zero || 99;                         // → 99    the classic bug

let name: string | null = null;
name ??= "anon";
name;                               // → 'anon'

// optional chaining
const deep: { a?: { b?: { c?: number } } } = {};
deep.a?.b?.c;                       // → undefined
deep.a?.b?.c ?? -1;                 // → -1

declare const fn: (() => number) | undefined;
fn?.();                             // → undefined
// (written with `declare` on purpose: `const fn = undefined` narrows fn to
//  `undefined`, and fn?.() then fails with "Type 'never' has no call signatures")

const arr: number[] | undefined = undefined;
arr?.[0];                           // → undefined

// != null covers both, and is the one place loose equality is idiomatic
function present(v: string | null | undefined) {
  return v != null ? v.length : -1;
}
present(null);                      // → -1
present(undefined);                 // → -1
present("ab");                      // → 2

// the non-null assertion — a promise to the compiler, unchecked at runtime
const el = { q: () => null as string | null };
el.q()!.length;                     // throws at runtime, compiles clean

// prefer a guard, or a throwing helper
function must<T>(v: T | null | undefined, msg = "missing"): T {
  if (v == null) throw new Error(msg);
  return v;
}
must("x").length;                   // → 1
```

### `?.` does not protect the call site

```ts
const maybeObj: { f?: () => number } = {};

maybeObj.f?.();                     // → undefined   safe
// ✗ maybeObj.f();
//   Cannot invoke an object which is possibly 'undefined'.

// and it short-circuits the WHOLE chain, including calls further right
const o: { a?: { b(): number } } = {};
o.a?.b();                           // → undefined   b() is never reached
```

### `exactOptionalPropertyTypes`

Off by default even in `strict`. Without it, `a?: string` accepts an explicit `undefined`.

```ts
type T = { a?: string };

const t1: T = {};                   // ok either way
const t2: T = { a: undefined };     // ok by default; ✗ under exactOptionalPropertyTypes
//   Type '{ a: undefined; }' is not assignable to type 'T' with
//   'exactOptionalPropertyTypes: true'.

// with the flag on, spell it out when you mean it
type T2 = { a?: string | undefined };
const t3: T2 = { a: undefined };    // ok
```

---

## Type operators

Operations that compute one type from another. These run at compile time only.

| Operator | Meaning |
|---|---|
| `keyof T` | union of `T`'s keys |
| `typeof v` | the type of a *value* (in type position) |
| `T[K]` | indexed access — the type at key `K` |
| `T[number]` | element type of an array or tuple |
| `T extends U ? X : Y` | conditional |
| `infer R` | capture a type inside a conditional |
| `{ [K in U]: T }` | mapped type |
| `` `a-${T}` `` | template literal type |
| `readonly`, `?`, `-readonly`, `-?` | modifiers in mapped types |

```ts
type User = { id: string; age: number; tags: string[] };

type UserKeys = keyof User;
//   ^? "id" | "age" | "tags"

type AgeType = User["age"];
//   ^? number

type Values = User[keyof User];
//   ^? string | number | string[]

type Tag = User["tags"][number];
//   ^? string

// keyof on an index signature
type AnyKey = keyof { [k: string]: number };
//   ^? string | number             number, because obj[1] and obj["1"] are the same

type NumKey = keyof { [k: number]: boolean };
//   ^? number

// keyof on an array
type ArrKeys = keyof string[];
//   ^? number | "length" | "toString" | ... every Array member
```

### `typeof` in type position

The single most useful tool for not repeating yourself: derive types from values you already wrote.

```ts
const defaults = {
  host: "localhost",
  port: 8080,
  tls: false,
};

type Config = typeof defaults;
//   ^? { host: string; port: number; tls: boolean }

const override: Config = { host: "x", port: 1, tls: true };
override.port;                      // → 1

// with `as const`, the keys and values become literals
const ROUTES = {
  home: "/",
  user: "/u/:id",
} as const;

type RouteName = keyof typeof ROUTES;
//   ^? "home" | "user"
type RoutePath = (typeof ROUTES)[RouteName];
//   ^? "/" | "/u/:id"

ROUTES.home;                        // → '/'

// typeof on a function gives its signature
function send(url: string, body: object): Promise<void> {
  return Promise.resolve();
}
type Send = typeof send;
//   ^? (url: string, body: object) => Promise<void>

// typeof on a class gives the CONSTRUCTOR, not the instance
class Box { v = 1 }
type BoxCtor = typeof Box;
//   ^? new () => Box
type BoxInst = Box;
//   ^? Box
type AlsoInst = InstanceType<typeof Box>;
//   ^? Box
```

The value `typeof` and the type `typeof` are different operators that share a spelling. In an expression it produces a string at runtime; in a type position it never emits anything.

---

## Conditional types

`T extends U ? X : Y`. `extends` here means "is assignable to", not "inherits from".

```ts
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<"a">;             // → "yes"
type B = IsString<number>;          // → "no"

// they DISTRIBUTE over naked union type parameters
type C = IsString<string | number>;
//   ^? "yes" | "no"                checked member by member

// wrap in a tuple to stop distribution
type NoDist<T> = [T] extends [string] ? "yes" : "no";
type D = NoDist<string | number>;
//   ^? "no"                        checked as a whole

// this is how the built-in Exclude works
type MyExclude<T, U> = T extends U ? never : T;
type E = MyExclude<"a" | "b" | "c", "b">;
//   ^? "a" | "c"                   the "b" branch produced never and vanished

// never as an input short-circuits distribution entirely
type F = MyExclude<never, "a">;
//   ^? never                       the body never runs
```

### `infer`

Capture a type from inside a shape you are matching.

```ts
type ElementOf<T> = T extends (infer U)[] ? U : never;
type G = ElementOf<string[]>;
//   ^? string
type H = ElementOf<number>;
//   ^? never

type Returned<T> = T extends (...a: any[]) => infer R ? R : never;
type I = Returned<() => Promise<number>>;
//   ^? Promise<number>

type Unwrap<T> = T extends Promise<infer U> ? U : T;
type J = Unwrap<Promise<string>>;
//   ^? string
type K = Unwrap<number>;
//   ^? number

// recursive unwrapping — this is essentially Awaited
type DeepUnwrap<T> = T extends Promise<infer U> ? DeepUnwrap<U> : T;
type L = DeepUnwrap<Promise<Promise<boolean>>>;
//   ^? boolean

// infer with a constraint (4.7+)
type FirstChar<T> = T extends `${infer C extends string}${string}` ? C : never;

// several infers in one pattern
type Split<S extends string, D extends string> =
  S extends `${infer Head}${D}${infer Tail}`
    ? [Head, ...Split<Tail, D>]
    : [S];

type M = Split<"a.b.c", ".">;
//   ^? ["a", "b", "c"]

// head and tail of a tuple
type Head<T extends unknown[]> = T extends [infer H, ...unknown[]] ? H : never;
type Tail<T extends unknown[]> = T extends [unknown, ...infer R] ? R : never;
type N = Head<[1, 2, 3]>;           // → 1
type O = Tail<[1, 2, 3]>;           // → [2, 3]
```

---

## Mapped types

Build an object type by walking a union of keys.

| Syntax | Effect |
|---|---|
| `{ [K in Keys]: T }` | one property per key |
| `{ [K in keyof T]: T[K] }` | identity — the starting point |
| `{ readonly [K in keyof T]: T[K] }` | add readonly |
| `{ [K in keyof T]?: T[K] }` | add optional |
| `{ -readonly [K in keyof T]: T[K] }` | strip readonly |
| `{ [K in keyof T]-?: T[K] }` | strip optional |
| `{ [K in keyof T as NewK]: T[K] }` | rename or filter keys (4.1+) |

```ts
type User = { id: string; age: number };

type MyPartial<T> = { [K in keyof T]?: T[K] };
type P = MyPartial<User>;
//   ^? { id?: string; age?: number }

type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
type Mutable<T> = { -readonly [K in keyof T]: T[K] };
type Required2<T> = { [K in keyof T]-?: T[K] };

type Nullable<T> = { [K in keyof T]: T[K] | null };
type Q = Nullable<User>;
//   ^? { id: string | null; age: number | null }

// key remapping with `as`
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};
type R = Getters<User>;
//   ^? { getId: () => string; getAge: () => number }

// filtering: map a key to never and it disappears
type OnlyStrings<T> = {
  [K in keyof T as T[K] extends string ? K : never]: T[K]
};
type S = OnlyStrings<User>;
//   ^? { id: string }

// mapping over a plain union of keys
type Flags = { [K in "a" | "b"]: boolean };
//   ^? { a: boolean; b: boolean }

const fl: Flags = { a: true, b: false };
fl.a;                               // → true

// homomorphic mapped types preserve readonly/optional automatically —
// this only happens with the exact form `[K in keyof T]`
type Src = { readonly a?: string };
type Copy = { [K in keyof Src]: Src[K] };
//   ^? { readonly a?: string }      modifiers carried over
```

### Practical mapped types

```ts
type User2 = { id: string; name: string; age: number };

// pick by value type
type StringKeys<T> = { [K in keyof T]: T[K] extends string ? K : never }[keyof T];
type T1 = StringKeys<User2>;
//   ^? "id" | "name"

// deep readonly
type DeepReadonly<T> = T extends (infer U)[]
  ? readonly DeepReadonly<U>[]
  : T extends object
    ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
    : T;

type T2 = DeepReadonly<{ a: { b: number[] } }>;
//   ^? { readonly a: { readonly b: readonly number[] } }

// make some keys optional, keep the rest
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
type T3 = PartialBy<User2, "age">;
//   ^? { id: string; name: string } & { age?: number }

const u: T3 = { id: "a", name: "b" };
u.id;                               // → 'a'

// flatten an intersection so hovers and errors read cleanly
type Prettify<T> = { [K in keyof T]: T[K] } & {};
type T4 = Prettify<T3>;
//   ^? { id: string; name: string; age?: number }
```

`Prettify` is worth keeping in every project. It changes nothing semantically and makes error messages dramatically more readable.

---

## Template literal types

String manipulation at the type level.

| Helper | Effect |
|---|---|
| `Uppercase<S>` | `"ab"` → `"AB"` |
| `Lowercase<S>` | `"AB"` → `"ab"` |
| `Capitalize<S>` | `"ab"` → `"Ab"` |
| `Uncapitalize<S>` | `"Ab"` → `"ab"` |

```ts
type Ev = "click" | "focus";
type Handler = `on${Capitalize<Ev>}`;
//   ^? "onClick" | "onFocus"

// unions multiply across every slot
type Size = "sm" | "lg";
type Color = "red" | "blue";
type Cls = `${Size}-${Color}`;
//   ^? "sm-red" | "sm-blue" | "lg-red" | "lg-blue"

// parsing with infer
type Method<S> = S extends `${infer M} ${string}` ? M : never;
type T5 = Method<"GET /users">;
//   ^? "GET"

type Params<S> = S extends `${string}:${infer P}/${infer Rest}`
  ? P | Params<`/${Rest}`>
  : S extends `${string}:${infer P}`
    ? P
    : never;
type T6 = Params<"/u/:id/p/:postId">;
//   ^? "id" | "postId"

// constrain a string argument to a shape
function nav<S extends `/${string}`>(path: S) { return path; }
nav("/home");                       // → '/home'
// ✗ nav("home");
//   Argument of type '"home"' is not assignable to parameter of type
//   `/${string}`.

// dotted paths into a nested object
type Paths<T> = T extends object
  ? { [K in keyof T & string]: K | `${K}.${Paths<T[K]>}` }[keyof T & string]
  : never;
type T7 = Paths<{ a: { b: number }; c: string }>;
//   ^? "a" | "c" | "a.b"
```

These compose fast, and they blow up fast — a four-way union across four slots is 256 members. TypeScript caps a union at 100,000 members and will error before that in practice.

---

## Generics

| Syntax | Meaning |
|---|---|
| `<T>` | a type parameter |
| `<T extends U>` | constrained |
| `<T = D>` | with a default |
| `<const T>` | infer literals without `as const` (5.0+) |
| `<in T>`, `<out T>` | explicit variance (4.7+) |
| `NoInfer<T>` | block inference at this position (5.4+) |

```ts
function identity<T>(v: T): T { return v; }
identity(1);                        // → 1
//^? number
identity<string>("a");              // → 'a'

// constraints
interface HasId { id: string }
function index<T extends HasId>(xs: T[]): Record<string, T> {
  return Object.fromEntries(xs.map((x) => [x.id, x]));
}
index([{ id: "a", n: 1 }]);         // → { a: { id: 'a', n: 1 } }

// defaults
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

const r: Result<number> = { ok: true, value: 1 };
r.ok && r.value;                    // → 1

// generic interfaces and classes
interface Store<T> {
  get(k: string): T | undefined;
  set(k: string, v: T): void;
}

class MapStore<T> implements Store<T> {
  #data = new Map<string, T>();
  get(k: string) { return this.#data.get(k); }
  set(k: string, v: T) { this.#data.set(k, v); }
}

const st = new MapStore<number>();
st.set("a", 1);
st.get("a");                        // → 1
st.get("z");                        // → undefined

// multiple parameters, one inferred from the other
function mapValues<T extends object, R>(
  o: T,
  f: (v: T[keyof T]) => R,
): Record<keyof T, R> {
  return Object.fromEntries(
    Object.entries(o).map(([k, v]) => [k, f(v as T[keyof T])]),
  ) as Record<keyof T, R>;
}
mapValues({ a: 1, b: 2 }, (n) => n * 10);   // → { a: 10, b: 20 }
```

### Inference rules worth knowing

```ts
// one candidate is collected per argument position, then the best common
// supertype wins. Repeating T does NOT silently union them:
function pair<T>(a: T, b: T): T[] { return [a, b]; }
pair(1, 2);                         // → [1, 2]
//^? number[]
// ✗ pair(1, "a");
//   Argument of type 'string' is not assignable to parameter of type 'number'.
pair<string | number>(1, "a");      // → [1, 'a']    explicit, so both fit

// where inference DOES silently widen: a parameter meant only to be validated
// against T, which instead contributes a candidate to it
function fsmLoose<S extends string>(c: { states: S[]; initial: S }) {
  return c.initial;
}
fsmLoose({ states: ["on", "off"], initial: "nope" });   // → 'nope'   no error!
//         S is inferred as "on" | "off" | "nope"

// NoInfer removes that position from inference, so S comes from `states` alone
function fsm<S extends string>(c: { states: S[]; initial: NoInfer<S> }) {
  return c.initial;
}
fsm({ states: ["on", "off"], initial: "on" });          // → 'on'
// ✗ fsm({ states: ["on", "off"], initial: "nope" });
//   Type '"nope"' is not assignable to type '"on" | "off"'.

// a return-only type parameter is almost always a mistake — it becomes
// whatever the caller says, with no checking
function unsafe<T>(json: string): T { return JSON.parse(json); }
const bad = unsafe<{ a: number }>("{}");
bad.a;                              // → undefined, typed number

// return unknown and make the caller narrow
function safe(json: string): unknown { return JSON.parse(json); }
```

### Variance annotations

```ts
// in = contravariant (input), out = covariant (output)
interface Producer<out T> { get(): T }
interface Consumer<in T> { set(v: T): void }
interface Both<in out T> { get(): T; set(v: T): void }

// these are assertions the compiler checks; they also speed up large unions.
// You rarely need them, but they document intent in library code.
```

---

## Utility types

Built in; no import.

| Type | Produces |
|---|---|
| `Partial<T>` | every property optional |
| `Required<T>` | every property required |
| `Readonly<T>` | every property readonly |
| `Pick<T, K>` | keep keys `K` |
| `Omit<T, K>` | drop keys `K` |
| `Record<K, V>` | object with keys `K`, values `V` |
| `Exclude<T, U>` | remove union members assignable to `U` |
| `Extract<T, U>` | keep union members assignable to `U` |
| `NonNullable<T>` | drop `null` and `undefined` |
| `Parameters<F>` | parameter tuple |
| `ReturnType<F>` | return type |
| `ConstructorParameters<C>` | constructor parameter tuple |
| `InstanceType<C>` | instance type of a class |
| `Awaited<T>` | unwrap promises, recursively |
| `ThisParameterType<F>` | the `this` parameter |
| `OmitThisParameter<F>` | drop it |
| `NoInfer<T>` | block inference (5.4+) |
| `Uppercase` / `Lowercase` / `Capitalize` / `Uncapitalize` | string transforms |

```ts
type User = { id: string; name: string; age?: number };

type U1 = Partial<User>;
//   ^? { id?: string; name?: string; age?: number }
type U2 = Required<User>;
//   ^? { id: string; name: string; age: number }
type U3 = Readonly<User>;
type U4 = Pick<User, "id" | "name">;
//   ^? { id: string; name: string }
type U5 = Omit<User, "age">;
//   ^? { id: string; name: string }
type U6 = Record<"a" | "b", number>;
//   ^? { a: number; b: number }

type U7 = Exclude<"a" | "b" | "c", "a">;
//   ^? "b" | "c"
type U8 = Extract<"a" | 1 | true, string | number>;
//   ^? "a" | 1
type U9 = NonNullable<string | null | undefined>;
//   ^? string

declare function send(url: string, n: number): Promise<boolean>;
type U10 = Parameters<typeof send>;
//   ^? [url: string, n: number]
type U11 = ReturnType<typeof send>;
//   ^? Promise<boolean>
type U12 = Awaited<ReturnType<typeof send>>;
//   ^? boolean
type U13 = Awaited<Promise<Promise<number>>>;
//   ^? number

class Conn { constructor(public host: string, public port: number) {} }
type U14 = ConstructorParameters<typeof Conn>;
//   ^? [host: string, port: number]
type U15 = InstanceType<typeof Conn>;
//   ^? Conn

new Conn("h", 1).host;              // → 'h'
```

### The `Omit` caveat

`Omit` does not check that the keys exist, and it collapses unions.

```ts
type User2 = { id: string; name: string };

type Typo = Omit<User2, "nmae">;    // no error, no effect
//   ^? { id: string; name: string }

// a strict version
type StrictOmit<T, K extends keyof T> = Omit<T, K>;
// ✗ type Bad = StrictOmit<User2, "nmae">;
//   Type '"nmae"' does not satisfy the constraint 'keyof User2'.

// Omit on a union merges it into one object — usually not what you want
type Shape = { kind: "a"; x: number } | { kind: "b"; y: number };
type Flat = Omit<Shape, "kind">;
//   ^? { x?: undefined; y: number } | ... collapsed, discriminant gone

// distribute manually
type DistOmit<T, K extends PropertyKey> =
  T extends unknown ? Omit<T, K> : never;
type Kept = DistOmit<Shape, "kind">;
//   ^? { x: number } | { y: number }
```

### Useful types you have to write yourself

```ts
type Prettify<T> = { [K in keyof T]: T[K] } & {};

type ValueOf<T> = T[keyof T];

type Entries<T> = { [K in keyof T]: [K, T[K]] }[keyof T][];

type Maybe<T> = T | null | undefined;

type AsyncReturn<F extends (...a: any[]) => any> = Awaited<ReturnType<F>>;

type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;

type RequireAtLeastOne<T, K extends keyof T = keyof T> =
  Omit<T, K> & { [P in K]-?: Required<Pick<T, P>> & Partial<Omit<T, P>> }[K];

type X = ValueOf<{ a: 1; b: 2 }>;
//   ^? 1 | 2
type Y = Entries<{ a: number }>;
//   ^? ["a", number][]
type Z = DeepPartial<{ a: { b: number } }>;
//   ^? { a?: { b?: number } }
```

---

## Classes

| Syntax | Meaning |
|---|---|
| `class C { x = 1 }` | field with an initializer |
| `x: number;` | declared field, must be assigned in the constructor |
| `public` / `protected` / `private` | compile-time visibility, erased |
| `#x` | true runtime private, enforced by the engine |
| `readonly x` | assignable only in the constructor |
| `constructor(private x: number)` | parameter property — declares and assigns |
| `static x` | on the constructor, not instances |
| `abstract class` / `abstract m()` | cannot be instantiated |
| `implements I` | checked, but adds no types |
| `get x()` / `set x(v)` | accessors |
| `override m()` | checked under `noImplicitOverride` |
| `declare x: T` | type-only field, emits nothing |

```ts
class Point {
  static origin = new Point(0, 0);
  readonly created = Date.now();
  #secret = 42;

  constructor(
    public x: number,
    public y: number = 0,
  ) {}

  get length(): number {
    return Math.hypot(this.x, this.y);
  }

  set length(v: number) {
    const s = v / this.length;
    this.x *= s;
    this.y *= s;
  }

  move(dx: number): this {
    this.x += dx;
    return this;                    // `this` return type enables chaining
  }

  static from([x, y]: [number, number]): Point {
    return new Point(x, y);
  }

  reveal() { return this.#secret; }
}

const p = new Point(3, 4);
p.x;                                // → 3
p.length;                           // → 5
p.move(1).move(1).x;                // → 5
Point.from([1, 2]).y;               // → 2
Point.origin.x;                     // → 0
p.reveal();                         // → 42
// ✗ p.#secret
//   Property '#secret' is not accessible outside class 'Point'.
```

### Visibility

```ts
class Base {
  public a = 1;                     // default
  protected b = 2;                  // this class and subclasses
  private c = 3;                    // this class only
  #d = 4;                           // this class only, at RUNTIME too
}

class Sub extends Base {
  read() { return this.b; }         // ok
  // ✗ readC() { return this.c; }
  //   Property 'c' is private and only accessible within class 'Base'.
}

const base = new Base();
base.a;                             // → 1
// ✗ base.b  →  Property 'b' is protected ...
// ✗ base.c  →  Property 'c' is private ...

// private is erased; the property is still there at runtime
JSON.stringify(new Base());         // → '{"a":1,"b":2,"c":3}'   #d is absent
Object.keys(new Base());            // → ['a', 'b', 'c']
```

`private` is a lint rule. `#` is a guarantee. Use `#` for anything that must not leak into `JSON.stringify`, `Object.keys`, or a debugger.

### Inheritance and abstract

```ts
abstract class Shape {
  abstract area(): number;
  describe(): string {
    return `area ${this.area()}`;
  }
}

class Circle extends Shape {
  constructor(private r: number) { super(); }
  override area(): number { return Math.PI * this.r ** 2; }
}

new Circle(1).describe();           // → 'area 3.141592653589793'
// ✗ new Shape();
//   Cannot create an instance of an abstract class.

// super() must run before any `this` access in a derived constructor
class A { constructor(public n = 1) {} }
class B extends A {
  m = this.n * 2;                   // field initializers run after super()
}
new B().m;                          // → 2
```

### `implements` does not infer

```ts
interface Serializer {
  encode(v: unknown): string;
}

class JsonSerializer implements Serializer {
  encode(v: unknown) { return JSON.stringify(v); }
  //     ^ still needs its own types — implements only CHECKS
}
new JsonSerializer().encode({ a: 1 });      // → '{"a":1}'

// ✗ class Broken implements Serializer {}
//   Class 'Broken' incorrectly implements interface 'Serializer'.
//     Property 'encode' is missing.
```

### Classes are types and values

```ts
class Widget { id = "w" }

function make(C: new () => Widget): Widget { return new C(); }
make(Widget).id;                    // → 'w'

// the class name used as a type means the INSTANCE type
const w: Widget = new Widget();

// structural typing applies to classes too — a plain object matches
const fake: Widget = { id: "x" };
fake.id;                            // → 'x'

// unless there is a private or # member, which makes it nominal
class Token { private brand = 1 }
// ✗ const fakeToken: Token = { brand: 1 };
//   Property 'brand' is private in type 'Token' but not in type '{ brand: number; }'.
```

### Branded types — nominal typing without classes

```ts
type UserId = string & { readonly __brand: "UserId" };
type PostId = string & { readonly __brand: "PostId" };

const asUserId = (s: string) => s as UserId;

function getUser(id: UserId) { return id; }

getUser(asUserId("u1"));            // → 'u1'
// ✗ getUser("u1");
//   Argument of type 'string' is not assignable to parameter of type 'UserId'.
// ✗ getUser(asPostId("p1"));       same error — the brands differ
const asPostId = (s: string) => s as PostId;
```

---

## Enums

TypeScript's enums are the one feature that emits real JavaScript and changes runtime behaviour. Most codebases now avoid them.

```ts
enum Dir { Up, Down }
Dir.Up;                             // → 0
Dir[0];                             // → 'Up'          reverse mapping, numeric only

enum Status { Active = "ACTIVE", Idle = "IDLE" }
Status.Active;                      // → 'ACTIVE'
// ✗ Status["ACTIVE"]  — string enums have no reverse mapping

// const enums are inlined and emit nothing, but break under isolatedModules
// and are disallowed by many bundlers
const enum Fast { A = 1 }
Fast.A;                             // → 1

// numeric enums accept any number, which defeats the point
enum Level { Low = 1, High = 2 }
const l: Level = 99 as Level;       // no error at the assignment site
```

### The preferred alternative

```ts
const Dir2 = {
  Up: "up",
  Down: "down",
} as const;

type Dir2 = (typeof Dir2)[keyof typeof Dir2];
//   ^? "up" | "down"

const d: Dir2 = "up";               // → 'up'
Dir2.Up;                            // → 'up'
// ✗ const bad: Dir2 = "left";
//   Type '"left"' is not assignable to type '"up" | "down"'.

// or skip the object entirely when you do not need a namespace
type Method = "GET" | "POST" | "PUT";
const m: Method = "GET";            // → 'GET'
```

The union of string literals is serialisable, tree-shakes to nothing, needs no import at runtime, and narrows properly in a `switch`. Reach for an `enum` only when you need reverse mapping or you are matching an existing API.

---

## Assertions and `satisfies`

| Form | Meaning |
|---|---|
| `x as T` | trust me, it is `T` |
| `<T>x` | same, illegal in `.tsx` |
| `x!` | trust me, it is not null |
| `x as const` | freeze to literals, recursively |
| `x satisfies T` | check `x` against `T`, keep `x`'s narrow type |
| `x as unknown as T` | force an unrelated cast |

```ts
const raw: unknown = "abc";
(raw as string).length;             // → 3

// an assertion may only move along the assignability chain
// ✗ const n = "a" as number;
//   Conversion of type 'string' to type 'number' may be a mistake because
//   neither type sufficiently overlaps with the other. If this was
//   intentional, convert the expression to 'unknown' first.
const forced = "a" as unknown as number;    // compiles, lies

// `as const` on literals, arrays, objects
const arr = [1, 2] as const;
//    ^? readonly [1, 2]
const obj = { a: 1 } as const;
//    ^? { readonly a: 1 }
const s = "GET" as const;
//    ^? "GET"
```

### `satisfies` — the one to reach for

`as` overrides the compiler. `satisfies` asks it to check, and then gets out of the way.

```ts
type Palette = Record<string, [number, number, number] | string>;

// with an annotation, the values are widened
const p1: Palette = { red: [255, 0, 0], green: "#0f0" };
// ✗ p1.red.length
//   Property 'length' does not exist on type 'string | [number, number, number]'.

// with `satisfies`, the check happens but the literal types survive
const p2 = {
  red: [255, 0, 0],
  green: "#0f0",
} satisfies Palette;

p2.red.length;                      // → 3     known to be the tuple
p2.green.toUpperCase();             // → '#0F0'
// ✗ const p3 = { red: 0 } satisfies Palette;
//   Type 'number' is not assignable to type 'string | [number, number, number]'.

// keys stay literal too
type Keys = keyof typeof p2;
//   ^? "red" | "green"

// the common config case
const config = {
  port: 8080,
  mode: "production",
} satisfies { port: number; mode: "production" | "development" };

config.mode;                        // → 'production'
//     ^? "production"              not the full union
```

Use `satisfies` wherever you were reaching for `as const` plus an annotation. It catches typos in the keys *and* keeps the narrow types.

---

## `unknown`, `any`, `never`, `void`

| Type | Accepts | Assignable to | Use for |
|---|---|---|---|
| `any` | everything | everything | escaping the type system; avoid |
| `unknown` | everything | only `unknown` / `any` | untrusted input |
| `never` | nothing | everything | impossible states, exhaustiveness |
| `void` | `undefined` (and ignored returns) | `any`, `unknown` | "I ignore this return" |
| `object` | any non-primitive | | rarely useful |
| `{}` | anything except `null` / `undefined` | | rarely what you meant |

```ts
let a: any = 1;
a.whatever.deeply.nested;           // no error, no safety
a = "s";

let un: unknown = 1;
// ✗ un.toFixed()
//   'un' is of type 'unknown'.
if (typeof un === "number") un.toFixed();   // → '1'
// ✗ const n: number = un;
//   Type 'unknown' is not assignable to type 'number'.

// never is the empty type
function loop(): never { while (true) {} }
type Impossible = string & number;
//   ^? never

// never vanishes from unions, and absorbs intersections
type U = string | never;
//   ^? string
type I = string & never;
//   ^? never

// never[] is the type of an array that can never receive anything
const none: never[] = [];
// ✗ none.push(1);
//   Argument of type 'number' is not assignable to parameter of type 'never'.

// void vs undefined
function v(): void {}
function u(): undefined { return undefined; }
v();                                // → undefined
// ✗ const x: undefined = v();
//   Type 'void' is not assignable to type 'undefined'.

// {} means "not null or undefined" — almost never the intent
type NotNullish = {};
const ok1: NotNullish = 1;          // → 1      a number is assignable to {}
const ok2: NotNullish = "a";        // → 'a'
// ✗ const bad: NotNullish = null;
//   Type 'null' is not assignable to type '{}'.
```

When you are tempted by `any`, try `unknown` first. It forces exactly one narrowing at the boundary, which is where the check belonged anyway.

---

## Async and promises

```ts
async function load(id: string): Promise<{ id: string }> {
  return { id };                    // a plain value is wrapped automatically
}
//       ^? (id: string) => Promise<{ id: string }>

// an async function ALWAYS returns a promise; annotate the inner type
// ✗ async function bad(): number { return 1; }
//   The return type of an async function must be the global Promise<T> type.

await load("a");                    // → { id: 'a' }

// Awaited unwraps nesting and non-promises alike
type A = Awaited<Promise<Promise<number>>>;
//   ^? number
type B = Awaited<number>;
//   ^? number

// Promise combinators
const ps = [Promise.resolve(1), Promise.resolve("a")] as const;

await Promise.all(ps);              // → [1, 'a']
//    ^? [number, string]           tuples are preserved

await Promise.all([load("a"), load("b")]);
//    ^? { id: string }[]

await Promise.allSettled([Promise.resolve(1)]);
//    ^? PromiseSettledResult<number>[]
// → [{ status: 'fulfilled', value: 1 }]

await Promise.race([Promise.resolve(1)]);        // → 1
await Promise.any([Promise.resolve(1)]);         // → 1

// allSettled needs narrowing on `status`
const settled = await Promise.allSettled([load("a")]);
const first = settled[0];
if (first.status === "fulfilled") first.value.id;       // → 'a'
else first.reason;

// for await over an async iterable
async function* gen(): AsyncGenerator<number> {
  yield 1;
  yield 2;
}
const out: number[] = [];
for await (const n of gen()) out.push(n);
out;                                // → [1, 2]

// a sync generator
function* count(n: number): Generator<number, string, undefined> {
  for (let i = 0; i < n; i++) yield i;
  return "done";
}
[...count(3)];                      // → [0, 1, 2]
```

### Typing the failure case

Promises are typed only on success. `Promise<T>` says nothing about rejection, so model failure in the value.

```ts
declare function load(id: string): Promise<{ id: string }>;

type Result<T, E = string> =
  | { ok: true; value: T }
  | { ok: false; error: E };

async function safeLoad(id: string): Promise<Result<{ id: string }>> {
  try {
    return { ok: true, value: await load(id) };
  } catch (e) {
    return { ok: false, error: e instanceof Error ? e.message : String(e) };
  }
}

const r = await safeLoad("a");
r.ok ? r.value.id : r.error;        // → 'a'
```

---

## Errors and catch

Under `strict`, a caught value is `unknown` — because anything can be thrown.

```ts
try {
  throw new Error("boom");
} catch (e) {
  //     ^? unknown
  // ✗ e.message
  //   'e' is of type 'unknown'.
  if (e instanceof Error) e.message;        // → 'boom'
}

// an explicit annotation is allowed only as unknown or any
try {} catch (e: unknown) {}
// ✗ catch (e: Error) {}
//   Catch clause variable type annotation must be 'any' or 'unknown' if specified.

// a reusable normaliser
function toError(e: unknown): Error {
  if (e instanceof Error) return e;
  return new Error(typeof e === "string" ? e : JSON.stringify(e));
}
toError("oops").message;            // → 'oops'

// custom errors need the prototype fixed when targeting ES5; on ES2015+
// targets extends works normally
class HttpError extends Error {
  constructor(public status: number, message: string) {
    super(message);
    this.name = "HttpError";
  }
}
const he = new HttpError(404, "not found");
he.status;                          // → 404
he instanceof Error;                // → true
he.message;                         // → 'not found'

// discriminate several error classes
function describe(e: unknown): string {
  if (e instanceof HttpError) return `http ${e.status}`;
  if (e instanceof Error) return e.message;
  return "unknown";
}
describe(he);                       // → 'http 404'
describe("x");                      // → 'unknown'
```

`useUnknownInCatchVariables` is what turns this on; it is included in `strict`. Setting it to `false` restores the old `any` behaviour.

---

## Modules

| Syntax | Meaning |
|---|---|
| `import { a } from "m"` | named import |
| `import type { T } from "m"` | type-only — erased entirely |
| `import { type T, a } from "m"` | inline type specifier |
| `export type { T }` | type-only re-export |
| `import a from "m"` | default |
| `import * as ns from "m"` | namespace |
| `export default` | one per module |
| `export * from "m"` | re-export everything |
| `declare module "m"` | ambient module declaration |
| `declare global` | augment global scope |

```ts
// types.ts
export type User = { id: string };
export const VERSION = "1.0";
export default function main() {}

// app.ts
import main, { VERSION, type User } from "./types.js";
import type { User as U2 } from "./types.js";

// `import type` guarantees the import is erased — required under
// verbatimModuleSyntax, and the reason a type-only import never causes
// a runtime cycle or a side effect

VERSION;                            // → '1.0'
```

Note the `.js` extension on relative imports. Under `module: "nodenext"` you write the *output* path, even from a `.ts` file. Under `bundler` resolution you omit it.

### Declaration merging and augmentation

```ts
// adding to a third-party module's types
declare module "some-lib" {
  interface Options {
    extra?: boolean;
  }
}

// adding to the global scope — the file must be a module (have an import
// or export) for `declare global` to be legal
export {};

declare global {
  interface Window {
    __APP_VERSION__: string;
  }
  var featureFlag: boolean;
}

// namespace + function merging, for attaching properties
function api() {}
namespace api {
  export const version = "1";
}
api.version;                        // → '1'
```

### `import` in type positions

```ts
// pull a type out of a module without a top-level import
type Cfg = import("./types.js").User;

// useful in .d.ts files and in JSDoc-typed JavaScript
```

---

## Declaration files

`.d.ts` files describe types with no implementation. They emit nothing.

```ts
// global.d.ts — no imports/exports, so everything here is global
declare const BUILD_ID: string;
declare function track(event: string): void;

interface Window {
  analytics?: { track(e: string): void };
}

// describing an untyped module
declare module "untyped-lib" {
  export function parse(s: string): unknown;
  export default function main(): void;
}

// wildcard modules, for non-JS imports handled by a bundler
declare module "*.svg" {
  const src: string;
  export default src;
}
declare module "*.css";

// ambient enums and classes
declare class External {
  constructor(n: number);
  run(): void;
}
```

| Directive | Effect |
|---|---|
| `declare` | this exists elsewhere; emit nothing |
| `export {}` | makes the file a module, so declarations stop being global |
| `/// <reference types="node" />` | pull in another package's types |
| `declare global { }` | reach back into global scope from a module |

For a package you publish, point `types` (or `exports["."].types`) in `package.json` at the `.d.ts`, and turn on `declaration: true` so `tsc` generates it for you.

---

## General idioms

### Deriving instead of duplicating

| Want | Code |
|---|---|
| a type from a value | `typeof value` |
| a type from a const object's keys | `keyof typeof obj` |
| a type from a const object's values | `(typeof obj)[keyof typeof obj]` |
| a type from an array literal | `(typeof arr)[number]` |
| a function's return | `ReturnType<typeof f>` |
| an async function's resolved value | `Awaited<ReturnType<typeof f>>` |
| a promise's value | `Awaited<T>` |
| one field's type | `User["email"]` |
| an array's element | `Users[number]` |

```ts
const ROLES = ["admin", "editor", "viewer"] as const;
type Role = (typeof ROLES)[number];
//   ^? "admin" | "editor" | "viewer"

ROLES.includes("admin");            // → true
// ✗ ROLES.includes("nope");
//   Argument of type '"nope"' is not assignable to parameter of type
//   '"admin" | "editor" | "viewer"'.

const HANDLERS = {
  onOpen: () => 1,
  onClose: () => "x",
} as const;

type HandlerName = keyof typeof HANDLERS;
//   ^? "onOpen" | "onClose"
type HandlerReturn = ReturnType<(typeof HANDLERS)[HandlerName]>;
//   ^? 1 | "x"

declare function fetchUser(id: string): Promise<{ id: string; email: string }>;
type User = Awaited<ReturnType<typeof fetchUser>>;
//   ^? { id: string; email: string }
type Email = User["email"];
//   ^? string
```

Write the value once, derive the type from it. The alternative — a type and a matching constant, maintained separately — drifts apart on the first refactor.

### Exhaustive records

```ts
type Role = "admin" | "editor";

// Record forces every member to be handled; adding a role breaks the build
const LABELS: Record<Role, string> = {
  admin: "Administrator",
  editor: "Editor",
};
LABELS.admin;                       // → 'Administrator'

// ✗ add "viewer" to Role and this becomes:
//   Property 'viewer' is missing in type '{ admin: string; editor: string; }'.

// the same idea for behaviour
const HANDLE: Record<Role, (n: number) => number> = {
  admin: (n) => n * 2,
  editor: (n) => n,
};
HANDLE.admin(3);                    // → 6
```

### Validating at the boundary

```ts
// everything crossing a network or storage boundary is unknown
async function getJson(url: string): Promise<unknown> {
  const res = await fetch(url);
  return res.json();
}

type Post = { id: number; title: string };

function isPost(v: unknown): v is Post {
  return (
    typeof v === "object" && v !== null &&
    "id" in v && typeof (v as Post).id === "number" &&
    "title" in v && typeof (v as Post).title === "string"
  );
}

isPost({ id: 1, title: "a" });      // → true
isPost({ id: "1" });                // → false

// hand-written guards do not scale; a schema library derives the type from
// the validator, so there is exactly one source of truth:
//   const Post = z.object({ id: z.number(), title: z.string() });
//   type Post = z.infer<typeof Post>;
```

### Option and result shapes

```ts
type Option<T> = { some: true; value: T } | { some: false };
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

function tryParse(s: string): Result<number> {
  const n = Number(s);
  return Number.isNaN(n)
    ? { ok: false, error: new Error(`bad number: ${s}`) }
    : { ok: true, value: n };
}

const r = tryParse("12");
r.ok ? r.value : r.error.message;   // → 12
tryParse("x").ok;                   // → false
```

### Builder and fluent chains

```ts
class Query<T extends object = {}> {
  #parts: string[] = [];

  where<K extends string, V>(k: K, v: V): Query<T & Record<K, V>> {
    this.#parts.push(`${k}=${String(v)}`);
    return this as unknown as Query<T & Record<K, V>>;
  }

  build(): string { return this.#parts.join("&"); }
}

new Query().where("id", 1).where("q", "x").build();      // → 'id=1&q=x'
```

### Function factories that keep literals

```ts
function createStore<S extends object>(initial: S) {
  let state = initial;
  return {
    get: () => state,
    set: <K extends keyof S>(k: K, v: S[K]) => { state = { ...state, [k]: v }; },
  };
}

const store = createStore({ count: 0, name: "a" });
store.set("count", 1);
store.get().count;                  // → 1
// ✗ store.set("count", "x");
//   Argument of type 'string' is not assignable to parameter of type 'number'.
// ✗ store.set("nope", 1);
//   Argument of type '"nope"' is not assignable to parameter of type
//   '"count" | "name"'.
```

### Typing object iteration

The built-in signatures are deliberately loose, because an object may have more keys than its type admits.

```ts
const obj = { a: 1, b: 2 };

Object.keys(obj);                   // → ['a', 'b']
//^? string[]                       NOT ('a' | 'b')[]
Object.entries(obj);                // → [['a', 1], ['b', 2]]
//^? [string, number][]
Object.values(obj);                 // → [1, 2]
//^? number[]

// narrow it yourself, knowing you are asserting
const keys = Object.keys(obj) as (keyof typeof obj)[];
keys[0];                            // → 'a'
//^? "a" | "b"

function typedEntries<T extends object>(o: T): [keyof T, T[keyof T]][] {
  return Object.entries(o) as [keyof T, T[keyof T]][];
}
typedEntries(obj)[0];               // → ['a', 1]

// `for...in` also gives string, for the same reason
for (const k in obj) {
  k;                                // ^? string
  obj[k as keyof typeof obj];
}
```

This looseness is correct, not a bug: a `{ a: number }` slot can hold `{ a: 1, b: 2 }`, so the runtime keys are genuinely unknown. The assertion is safe only when you built the object yourself.

### Immutable updates

```ts
type State = { count: number; user: { name: string }; tags: string[] };

const s: State = { count: 0, user: { name: "a" }, tags: ["x"] };

const s1 = { ...s, count: s.count + 1 };
s1.count;                           // → 1

const s2 = { ...s, user: { ...s.user, name: "b" } };
s2.user.name;                       // → 'b'

const s3 = { ...s, tags: [...s.tags, "y"] };
s3.tags;                            // → ['x', 'y']

const s4 = { ...s, tags: s.tags.filter((t) => t !== "x") };
s4.tags;                            // → []

const s5 = { ...s, tags: s.tags.map((t) => (t === "x" ? "z" : t)) };
s5.tags;                            // → ['z']

// spread is shallow — the nested objects in s1 are the SAME references as in s
s1.user === s.user;                 // → true

// structuredClone for a real deep copy (functions and symbols do not survive)
const deep = structuredClone(s);
deep.user === s.user;               // → false
```

### Narrowing helper functions

```ts
const isDefined = <T>(v: T | null | undefined): v is T => v != null;
const isString = (v: unknown): v is string => typeof v === "string";
const isRecord = (v: unknown): v is Record<string, unknown> =>
  typeof v === "object" && v !== null && !Array.isArray(v);

[1, null, 2].filter(isDefined);     // → [1, 2]
//^? number[]
["a", 1].filter(isString);          // → ['a']
//^? string[]
isRecord({ a: 1 });                 // → true
isRecord([1]);                      // → false
isRecord(null);                     // → false
```

### `as const` in argument position

```ts
// without it, the array widens and the tuple shape is lost
function useAxis(a: [number, number]) { return a[0]; }
// ✗ useAxis([1, 2] as number[]);
//   Argument of type 'number[]' is not assignable to parameter of type
//   '[number, number]'.
useAxis([1, 2]);                    // → 1     a fresh literal is contextually typed

const pos = [1, 2];
// ✗ useAxis(pos);
//   Target requires 2 element(s) but source may have fewer.
const pos2 = [1, 2] as const;
useAxis([...pos2]);                 // → 1
```

---

## Gotchas

| Trap | Reality |
|---|---|
| `xs[0]` on an empty array | typed `T`, actually `undefined` — turn on `noUncheckedIndexedAccess` |
| `Object.keys(o)` | `string[]`, never `(keyof T)[]` |
| `as` | silences the compiler; it does not convert anything |
| `x!` | erased; if `x` is null you get a runtime crash with no warning |
| `if (x)` on `number \| undefined` | `0` takes the else branch |
| `a \|\| b` on `number` | `0` falls through; use `??` |
| `enum` | emits runtime code and numeric enums accept any number |
| `T extends U ? ... : ...` on a union | distributes, member by member |
| `Omit<T, K>` | does not check `K`, and flattens unions |
| `() => void` as a callback type | any return value is accepted and discarded |
| `private` | a compile-time convention; `#` is the real one |
| `interface` | merges silently across declarations; `type` errors |
| `readonly` | shallow, and erased at runtime |
| `string & number` | `never`, not an error |
| optional `a?: T` | means `T \| undefined`, never `T \| null` |
| `catch (e)` | `unknown` under `strict` |
| `any` anywhere in a chain | disables checking for everything downstream |

```ts
// indexing lies by default
const xs: number[] = [];
const first = xs[0];
//    ^? number
first;                              // → undefined
first.toFixed();                    // runtime TypeError, compiles clean

// noUncheckedIndexedAccess: true → number | undefined, and this is caught

// as does nothing at runtime
const n = "5" as unknown as number;
n + 1;                              // → '51'    string concatenation

// void swallows returns
const results: number[] = [];
[1, 2].forEach((v) => results.push(v));     // push returns number, ignored
results;                            // → [1, 2]

// any is contagious
const data: any = { a: 1 };
const out = data.a.b.c;             // no error
//    ^? any

// unions in a conditional distribute
type Arr<T> = T extends unknown ? T[] : never;
type A1 = Arr<string | number>;
//   ^? string[] | number[]         NOT (string | number)[]

// excess property checks only fire on fresh literals
type O = { a: number };
const tmp = { a: 1, b: 2 };
const o: O = tmp;                   // allowed
// ✗ const o2: O = { a: 1, b: 2 };  Object literal may only specify known properties

// declaration merging can silently widen a type you thought you controlled
interface Cfg { a: number }
interface Cfg { b: string }         // no error anywhere
const cfg: Cfg = { a: 1, b: "x" };
cfg.b;                              // → 'x'

// this in a callback
class Counter {
  n = 0;
  badInc() { return function () { /* this is undefined here */ }; }
  goodInc = () => { this.n++; };    // arrow field captures the instance
}
const c = new Counter();
const inc = c.goodInc;
inc();
c.n;                                // → 1

// comparing unrelated literal types is an error, not `false`
// ✗ "a" === "b"
//   This comparison appears to be unintentional because the types '"a"' and
//   '"b"' have no overlap.
```

---

## tsconfig

The settings that change what this page means.

| Option | Effect |
|---|---|
| `strict` | turns on the eight flags below |
| `noImplicitAny` | an un-inferable parameter is an error |
| `strictNullChecks` | `null` / `undefined` are not assignable to `T` |
| `strictFunctionTypes` | parameters are checked contravariantly |
| `strictBindCallApply` | `bind` / `call` / `apply` are type-checked |
| `strictPropertyInitialization` | class fields must be assigned |
| `noImplicitThis` | an untyped `this` is an error |
| `useUnknownInCatchVariables` | `catch (e)` is `unknown` |
| `alwaysStrict` | emits `"use strict"` |

Not in `strict`, but worth enabling:

| Option | Effect |
|---|---|
| `noUncheckedIndexedAccess` | `xs[i]` is `T \| undefined` — the biggest real safety win |
| `exactOptionalPropertyTypes` | `a?: T` rejects an explicit `undefined` |
| `noImplicitOverride` | overriding a method requires `override` |
| `noFallthroughCasesInSwitch` | a non-empty case must break or return |
| `noUnusedLocals` / `noUnusedParameters` | dead-code errors |
| `verbatimModuleSyntax` | imports are emitted exactly as written; forces `import type` |
| `isolatedModules` | each file must compile alone — required by esbuild/swc |
| `erasableSyntaxOnly` | bans enums, namespaces and parameter properties (5.8+) |

```jsonc
{
  "compilerOptions": {
    "target": "es2022",
    "lib": ["es2023", "dom", "dom.iterable"],
    "module": "nodenext",
    "moduleResolution": "nodenext",

    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true,

    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,

    "declaration": true,
    "sourceMap": true,
    "outDir": "dist",
    "noEmit": false
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

| `moduleResolution` | Use when |
|---|---|
| `bundler` | Vite, webpack, Next.js — extensionless relative imports |
| `nodenext` | plain Node — relative imports need the `.js` extension |
| `node10` | legacy, deprecated in 6.0 |

`skipLibCheck: true` is close to mandatory on a real project; without it one badly-typed dependency in `node_modules` fails your build.

---

## Version notes

| Feature | Requires |
|---|---|
| `unknown` | 3.0 |
| optional chaining `?.`, nullish `??` | 3.7 |
| assertion functions `asserts x is T` | 3.7 |
| `import type` / `export type` | 3.8 |
| `#private` fields | 3.8 |
| `??=`, `\|\|=`, `&&=` | 4.0 |
| variadic tuple types | 4.0 |
| named tuple members | 4.0 |
| template literal types | 4.1 |
| key remapping `as` in mapped types | 4.1 |
| recursive conditional types | 4.1 |
| abstract construct signatures | 4.2 |
| `override` / `noImplicitOverride` | 4.3 |
| `unknown` in `catch` | 4.4 |
| `in`/`out` variance annotations | 4.7 |
| `infer X extends T` | 4.7 |
| `satisfies` | 4.9 |
| `const` type parameters | 5.0 |
| ES decorators (standard) | 5.0 |
| `verbatimModuleSyntax` | 5.0 |
| `using` / `await using` (explicit resource management) | 5.2 |
| `NoInfer<T>` | 5.4 |
| inferred type predicates on `filter` | 5.5 |
| `erasableSyntaxOnly` | 5.8 |
| `import defer` | 5.9 |
| `es2025` target and lib, Temporal types | 6.0 |
| `#/` subpath imports | 6.0 |
| ES5 target removed, `node10` resolution deprecated | 6.0 |
| native Go compiler, 8–12x faster builds | 7.0 |
| stable programmatic compiler API | 7.1 (expected) |

### The 6.0 / 7.0 split

TypeScript 6.0 (March 2026) was the last release built on the JavaScript
compiler codebase. TypeScript 7.0 (July 2026) is the Go port — the first stable
release on the new foundation, and what `npm i typescript` now installs.

| | 6.0 | 7.0 |
|---|---|---|
| implementation | TypeScript, on V8 | Go, native, multithreaded |
| full-build speed | baseline | typically 8–12x faster |
| memory | baseline | roughly 18% lower |
| type-checking semantics | — | **identical**; a port, not a rewrite |
| command name | `tsc` | `tsc` |
| programmatic API | stable | **not yet — 7.1 at the earliest** |

The language surface does not change between them, which is why nothing on this
page is version-specific to either. Code that compiles cleanly under 6.0 with
`stableTypeOrdering` on and no `ignoreDeprecations` should compile identically
under 7.0.

The one thing that will hold a project back is the missing programmatic API.
Anything that drives the compiler as a library — `typescript-eslint`, and the
framework tooling for Vue, Svelte, Astro, MDX and Angular — cannot run on 7.0
until 7.1 lands. Until then a common arrangement is 7.0 for `tsc --noEmit` in
CI and the editor, with 6.0 retained for the lint and framework toolchain.

```bash
npx tsc -v          # → Version 7.0.2
```

The old `tsgo` binary and the `@typescript/native-preview` package are
superseded; nightlies are published under `typescript` again.
