# React + TypeScript Syntax Reference

A single-page lookup for React 19 with TypeScript: component and prop typing, every hook, events, forms, and the type-level idioms that come up daily.

This page assumes the [TypeScript reference](./typescript.md) for the language itself — `keyof`, `satisfies`, mapped types and the rest are used here without re-explanation.

**How to use this page.** One long file, so browser find (`Ctrl+F` / `Cmd+F`) is the search tool. Search for the API name — `useActionState`, `ComponentProps`, `ReactNode`, `forwardRef` — rather than a description.

| Mark | Meaning |
|---|---|
| `// →` | the value the expression produces at runtime |
| `// ^?` | the type TypeScript infers |
| `// ✗` | a compile error, followed by the message |
| `// ≤18` | how this was written before React 19 |

Checked against **React 19.3** with **@types/react 19.3** and **TypeScript 6.0** under `strict: true`.

---

## Contents

[Setup](#setup) · [Components](#components) · [JSX types](#jsx-types) · [Props](#props) · [Children](#children) · [Deriving prop types](#deriving-prop-types) · [Discriminated props](#discriminated-props) · [Polymorphic components](#polymorphic-components) · [useState](#usestate) · [useReducer](#usereducer) · [useRef](#useref) · [ref as a prop](#ref-as-a-prop) · [useEffect](#useeffect) · [useEffectEvent](#useeffectevent) · [useMemo and useCallback](#usememo-and-usecallback) · [Context](#context) · [use](#use) · [Events](#events) · [Forms and Actions](#forms-and-actions) · [useActionState](#useactionstate) · [useFormStatus](#useformstatus) · [useOptimistic](#useoptimistic) · [Transitions](#transitions) · [Suspense](#suspense) · [Error boundaries](#error-boundaries) · [Custom hooks](#custom-hooks) · [Generic components](#generic-components) · [memo](#memo) · [Portals and fragments](#portals-and-fragments) · [Activity](#activity) · [ViewTransition](#viewtransition) · [Server components](#server-components) · [Idioms](#general-idioms) · [Gotchas](#gotchas) · [React 18 → 19](#react-18--19) · [Versions](#version-notes)

---

## Setup

```jsonc
{
  "compilerOptions": {
    "target": "es2023",
    "lib": ["es2023", "dom", "dom.iterable"],
    "module": "esnext",
    "moduleResolution": "bundler",

    "jsx": "react-jsx",          // the modern transform — no React import needed
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "skipLibCheck": true,
    "noEmit": true
  },
  "include": ["src"]
}
```

| `jsx` value | Emits |
|---|---|
| `react-jsx` | `_jsx(...)` from `react/jsx-runtime` — the default for React 17+ |
| `react-jsxdev` | same, with source locations; use in development |
| `preserve` | leaves JSX alone for a bundler to handle |
| `react` | legacy `React.createElement` — requires `import React` in every file |

React 19 **requires** the modern transform. The old one triggers a console warning and blocks `ref` as a prop.

```ts
// React 19 ships its own types. Remove these if they are in package.json:
//   "@types/react": pinned to 18.x
//   "@types/react-dom": pinned to 18.x
// and make sure nothing in node_modules pulls an 18 copy back in:
//   npm ls @types/react
```

Files containing JSX must be `.tsx`, not `.ts`. In a `.tsx` file the angle-bracket assertion `<T>x` is illegal because it parses as JSX — use `x as T`.

```tsx
const a = "x" as string;            // ok everywhere
// ✗ const b = <string>"x";
//   in .tsx this parses as a JSX tag and fails.
// A lone generic arrow also needs a nudge:
const id = <T,>(v: T) => v;         // note the trailing comma
id(1);                              // → 1
```

---

## Components

A component is a function returning `ReactNode`. That is the whole contract.

```tsx
// the default form — annotate props, let the return type be inferred
function Greeting({ name }: { name: string }) {
  return <h1>Hello {name}</h1>;
}

// with a named prop type, which is what you want for anything reused
type ButtonProps = {
  label: string;
  onClick: () => void;
  disabled?: boolean;
};

function Button({ label, onClick, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}

// arrow form, identical
const Button2 = ({ label, onClick }: ButtonProps) => (
  <button onClick={onClick}>{label}</button>
);

// explicit return type, if you want the check
function Typed({ name }: { name: string }): React.ReactNode {
  return <span>{name}</span>;
}
```

### Do not use `React.FC`

```tsx
// ≤18 this was the common style, and in React 17 and earlier it also
// silently added an implicit `children` prop
const Old: React.FC<ButtonProps> = ({ label, onClick }) => (
  <button onClick={onClick}>{label}</button>
);

// it still works in 19, but it buys nothing and costs you:
//   - it cannot be generic without extra ceremony
//   - it constrains the return type more tightly than a plain function
//   - `children` must now be declared explicitly anyway (since @types/react 18)
//
// prefer the plain function. It is shorter, generic-friendly, and the props
// type is right where you read it.
```

### Naming and exports

```tsx
// a component must start with a capital letter or JSX treats it as a DOM tag
function widget() { return <div />; }
// <widget /> renders a literal <widget> element — almost never what you want

// ✗ <widget />
//   Property 'widget' does not exist on type 'JSX.IntrinsicElements'.

// default vs named — named exports survive refactors and autocomplete better
export function Card() { return <div />; }
export default Card;
```

---

## JSX types

The types that describe "a thing React can render". Choosing the wrong one is the single most common typing mistake in React.

| Type | Means | Use for |
|---|---|---|
| `ReactNode` | anything renderable: element, string, number, `null`, `undefined`, `boolean`, array | **props, return types — the default choice** |
| `ReactElement` | specifically a JSX element object | when you need `.props` or `.type` |
| `JSX.Element` | `ReactElement<any, any>` | legacy alias; prefer `ReactElement` |
| `ComponentType<P>` | a function *or* class component taking `P` | passing a component as a value |
| `ElementType` | any valid JSX tag, including `"div"` | polymorphic `as` props |
| `ReactPortal` | a portal | rare |

```tsx
import type { ReactNode, ReactElement, ComponentType, ElementType } from "react";

// props that accept content → ReactNode, always
type PanelProps = {
  header: ReactNode;
  children: ReactNode;
};

function Panel({ header, children }: PanelProps) {
  return <section><h2>{header}</h2>{children}</section>;
}

<Panel header="Title">body</Panel>;
<Panel header={<em>Title</em>}>{null}</Panel>;
<Panel header={42}>{["a", "b"]}</Panel>;

// ReactNode accepts all of these; ReactElement accepts none but the first
// ✗ type Bad = { header: ReactElement };  <Panel header="Title" />
//   Type 'string' is not assignable to type 'ReactElement'.

// passing a component itself, not rendered content
type SlotProps = {
  icon: ComponentType<{ size: number }>;
  as?: ElementType;
};

function Slot({ icon: Icon, as: Tag = "div" }: SlotProps) {
  return <Tag><Icon size={16} /></Tag>;
}
// note the rename: lowercase JSX names are DOM tags, so `icon` must be
// destructured into a capitalised binding before it can be used as <Icon />
```

### What actually renders

```tsx
// these all render, and are all ReactNode
<div>{"text"}</div>;
<div>{42}</div>;
<div>{null}</div>;
<div>{undefined}</div>;
<div>{false}</div>;                 // renders nothing — the && idiom relies on this
<div>{[<span key="a" />, <span key="b" />]}</div>;
<div>{0}</div>;                     // renders "0" — the && trap, see Gotchas

// bigint renders; symbols and plain objects throw at runtime
// ✗ <div>{{ a: 1 }}</div>
//   Type '{ a: number; }' is not assignable to type 'ReactNode'.
```

---

## Props

```tsx
type Props = {
  // required
  id: string;
  // optional
  title?: string;
  // default via destructuring, not via the type
  variant?: "primary" | "ghost";
  // a union, so typos are caught
  size: "sm" | "md" | "lg";
  // functions: name the parameters, return void unless you consume the result
  onSelect: (id: string) => void;
  // content
  children?: React.ReactNode;
  // readonly arrays cost callers nothing and prevent accidental mutation
  items: readonly string[];
  // an index signature if you really must pass arbitrary attrs
  data?: Record<string, string>;
};

function Thing({ id, title = "untitled", variant = "primary", size, onSelect, items }: Props) {
  return (
    <ul data-variant={variant} data-size={size} aria-label={title}>
      {items.map((i) => (
        <li key={i} onClick={() => onSelect(id)}>{i}</li>
      ))}
    </ul>
  );
}

<Thing id="a" size="md" items={["x"]} onSelect={() => {}} />;
// ✗ <Thing id="a" size="huge" items={[]} onSelect={() => {}} />
//   Type '"huge"' is not assignable to type '"sm" | "md" | "lg"'.
// ✗ <Thing size="md" items={[]} onSelect={() => {}} />
//   Property 'id' is missing.
```

### Defaults belong in the destructuring

```tsx
type P = { count?: number; label?: string };

// right — the default is visible at the point of use
function Ok({ count = 0, label = "none" }: P) {
  return <span>{label}: {count}</span>;
}

// defaultProps was removed for function components in React 19
// ≤18 this worked:
//   function Old(p: P) { return <span>{p.count}</span>; }
//   Old.defaultProps = { count: 0 };
// In React 19 it is ignored and warns in development.
```

### `readonly` props

```tsx
// props are frozen in development and must never be mutated. The type does
// not stop you by default:
function Bad(props: { items: string[] }) {
  // props.items.push("x");        // compiles; corrupts the parent's state
  return <>{props.items.length}</>;
}

// declare them readonly and the compiler helps
function Good(props: { readonly items: readonly string[] }) {
  // ✗ props.items.push("x");
  //   Property 'push' does not exist on type 'readonly string[]'.
  return <>{props.items.length}</>;
}
```

---

## Children

```tsx
import type { ReactNode, PropsWithChildren } from "react";

// explicit — the clearest form
type A = { title: string; children: ReactNode };

// optional children
type B = { title: string; children?: ReactNode };

// the helper, which just adds `children?: ReactNode`
type C = PropsWithChildren<{ title: string }>;

function Layout({ title, children }: C) {
  return <div><h1>{title}</h1>{children}</div>;
}
<Layout title="t">hello</Layout>;

// a render-prop child: a FUNCTION, not a node
type RenderProps = {
  children: (value: number) => ReactNode;
};
function Counter({ children }: RenderProps) {
  return <>{children(42)}</>;
}
<Counter>{(n) => <b>{n}</b>}</Counter>;

// require exactly one element child
type Single = { children: React.ReactElement };
function Wrap({ children }: Single) { return children; }
<Wrap><span /></Wrap>;
// ✗ <Wrap>text</Wrap>
//   Type 'string' is not assignable to type 'ReactElement'.
```

`children` is not implicit. Since `@types/react` 18 you must declare it; a component whose props type omits `children` rejects them.

```tsx
function NoKids({ title }: { title: string }) { return <h1>{title}</h1>; }
// ✗ <NoKids title="t">oops</NoKids>
//   Type '{ title: string; children: string; }' is not assignable to
//   type 'IntrinsicAttributes & { title: string; }'.
```

### `Children` utilities

```tsx
import { Children, isValidElement, cloneElement } from "react";

function Count({ children }: { children: ReactNode }) {
  Children.count(children);                     // counts top-level nodes
  Children.toArray(children);                   // flattens, adds keys, drops null
  const only = Children.only(<div />);          // throws unless exactly one
  return <>{Children.map(children, (c) => (isValidElement(c) ? c : null))}</>;
}

// these are legacy escape hatches. Prefer an explicit prop or a render prop —
// walking children couples a parent to its children's internals.
```

---

## Deriving prop types

Never retype what React already knows. This is the highest-value section on the page.

| Want | Use |
|---|---|
| all props of a DOM tag | `ComponentProps<"button">` |
| all props of a component | `ComponentProps<typeof MyButton>` |
| props minus `ref` | `ComponentPropsWithoutRef<"input">` |
| props including `ref` | `ComponentPropsWithRef<"input">` |
| the ref's element type | `ComponentRef<"input">` |
| one event handler's type | `ComponentProps<"form">["onSubmit"]` |
| a CSS style object | `React.CSSProperties` |

```tsx
import type { ComponentProps, ComponentPropsWithoutRef, ComponentRef } from "react";

// extend a native button: every native attribute comes for free
type ButtonProps = ComponentProps<"button"> & {
  variant?: "primary" | "ghost";
};

function Button({ variant = "primary", className, ...rest }: ButtonProps) {
  return <button className={`btn-${variant} ${className ?? ""}`} {...rest} />;
}

// all of these now type-check, with no extra declarations
<Button type="submit" aria-label="Save" onClick={() => {}} autoFocus />;
<Button variant="ghost" disabled form="f" />;
// ✗ <Button type="sbmit" />
//   Type '"sbmit"' is not assignable to type '"button" | "submit" | "reset"'.

// overriding a native prop: Omit it first, or the intersection becomes never
type LinkProps = Omit<ComponentProps<"a">, "href"> & {
  href: URL | string;
};

// borrowing another component's props
type MyProps = ComponentProps<typeof Button>;
//   ^? ButtonProps

// one handler's type, pulled out by name
type OnSubmit = ComponentProps<"form">["onSubmit"];
//   ^? React.FormEventHandler<HTMLFormElement> | undefined

// the element a ref points at
type InputEl = ComponentRef<"input">;
//   ^? HTMLInputElement

// style objects
const style: React.CSSProperties = {
  display: "flex",
  marginTop: 8,                     // numbers get "px" appended
  ["--brand" as string]: "#09f",    // custom properties need the cast
};
<div style={style} />;
// ✗ const bad: React.CSSProperties = { colour: "red" };
//   Object literal may only specify known properties, and 'colour' does not
//   exist in type 'Properties<string | number, string & {}>'.
```

### `ComponentProps` vs the older names

```tsx
// ≤18 you often needed the ref-aware variants because `ref` was not a prop:
//   ComponentPropsWithoutRef<"input">   ← props as a caller passes them
//   ComponentPropsWithRef<"input">      ← plus the ref
//   ElementRef<"input">                 ← the element type
//
// In React 19, `ref` IS a regular prop, so ComponentProps<"input"> already
// includes it and the three collapse to one in most code.
// `ElementRef` is deprecated in favour of `ComponentRef`.

type Now = ComponentProps<"input">;
type Explicit = ComponentPropsWithoutRef<"input">;   // still valid, still useful
                                                     // when you re-declare ref
```

### Native element interfaces

```tsx
// the underlying DOM interfaces, which you need for refs and event targets
// HTMLInputElement, HTMLButtonElement, HTMLFormElement, HTMLDivElement,
// HTMLAnchorElement, HTMLCanvasElement, HTMLSelectElement,
// HTMLTextAreaElement, HTMLDialogElement, SVGSVGElement

// when you do not know it, ask ComponentRef
type El = ComponentRef<"dialog">;
//   ^? HTMLDialogElement
```

---

## Discriminated props

Model "these props go together" in the type, so impossible combinations do not compile.

```tsx
// the mistake: everything optional, all combinations allowed
type Loose = {
  status: "loading" | "error" | "done";
  data?: string;
  error?: Error;
};
// nothing stops { status: "loading", error: new Error() }

// the fix: a union keyed on the discriminant
type State =
  | { status: "loading" }
  | { status: "error"; error: Error }
  | { status: "done"; data: string };

function View(props: State) {
  switch (props.status) {
    case "loading": return <p>…</p>;
    case "error":   return <p>{props.error.message}</p>;
    case "done":    return <p>{props.data}</p>;
  }
}

<View status="loading" />;
<View status="done" data="x" />;
// ✗ <View status="loading" data="x" />
//   Property 'data' does not exist on type '{ status: "loading"; }'.
// ✗ <View status="done" />
//   Property 'data' is missing.
```

### Mutually exclusive props without a discriminant

```tsx
// "either href or onClick, never both"
type Never<T> = { [K in keyof T]?: never };

type AsLink = { href: string } & Never<{ onClick: unknown }>;
type AsButton = { onClick: () => void } & Never<{ href: unknown }>;

function Action(props: (AsLink | AsButton) & { label: string }) {
  return "href" in props && props.href
    ? <a href={props.href}>{props.label}</a>
    : <button onClick={props.onClick}>{props.label}</button>;
}

<Action label="go" href="/x" />;
<Action label="go" onClick={() => {}} />;
// ✗ <Action label="go" href="/x" onClick={() => {}} />
//   Type '() => void' is not assignable to type 'undefined'.
```

### Conditionally required props

```tsx
// truncate requires maxLines; without truncate, maxLines is forbidden
type TextProps = { children: ReactNode } & (
  | { truncate: true; maxLines: number }
  | { truncate?: false; maxLines?: never }
);

function Text(props: TextProps) {
  return <p>{props.children}</p>;
}

<Text>plain</Text>;
<Text truncate maxLines={2}>clipped</Text>;
// ✗ <Text truncate>clipped</Text>
//   Property 'maxLines' is missing.
// ✗ <Text maxLines={2}>plain</Text>
//   Type 'number' is not assignable to type 'undefined'.
```

---

## Polymorphic components

A component that renders as a different tag depending on an `as` prop. Genuinely hard to type; here is the working recipe.

```tsx
import type { ElementType, ComponentPropsWithoutRef, ReactNode } from "react";

type BoxProps<T extends ElementType> = {
  as?: T;
  children?: ReactNode;
} & Omit<ComponentPropsWithoutRef<T>, "as" | "children">;

function Box<T extends ElementType = "div">({
  as,
  children,
  ...rest
}: BoxProps<T>) {
  const Tag = (as ?? "div") as ElementType;
  return <Tag {...rest}>{children}</Tag>;
}

<Box>default div</Box>;
<Box as="a" href="/x">link</Box>;
<Box as="button" type="submit" onClick={() => {}}>press</Box>;
// ✗ <Box as="a" type="submit">bad</Box>
//   Property 'type' does not exist on type ... AnchorHTMLAttributes

// the cast on `Tag` is unavoidable: TypeScript cannot prove that a generic
// ElementType is constructible with the spread props it was given. Keep the
// cast internal — callers still get full checking.
```

Weigh this against just writing two components. Polymorphism costs a generic, a cast, and slower editor feedback; `<Button>` plus `<LinkButton>` costs neither.

---

## useState

```tsx
import { useState } from "react";

// inferred from the initial value — the normal case
const [count, setCount] = useState(0);
//     ^? number

const [name, setName] = useState("");
//     ^? string

// annotate when the initial value is narrower than the state
const [user, setUser] = useState<User | null>(null);
//     ^? User | null
// without the annotation this would be `null` forever:
//   const [u, setU] = useState(null);   // ^? null
//   ✗ setU({ id: "a" })  →  not assignable to parameter of type 'null'

const [items, setItems] = useState<string[]>([]);
//     ^? string[]        an unannotated [] infers never[]

const [status, setStatus] = useState<"idle" | "busy">("idle");
//     ^? "idle" | "busy"    the annotation stops widening to string

type User = { id: string; name: string };

// the setter accepts a value or an updater
setCount(1);
setCount((c) => c + 1);             // use this whenever the next value
                                    // depends on the current one
// ✗ setCount("1");
//   Argument of type 'string' is not assignable to parameter of type
//   'SetStateAction<number>'.

// the setter's own type, for passing it down
type SetCount = React.Dispatch<React.SetStateAction<number>>;
//              ^? (value: number | ((prev: number) => number)) => void
```

### Lazy initial state

```tsx
// the initializer runs on EVERY render; the function form runs once
const [eager, setEager] = useState(expensiveParse(localStorage.getItem("k")));   // bad
const [once, setOnce] = useState(() => expensiveParse(localStorage.getItem("k"))); // good

function expensiveParse(s: string | null): Record<string, unknown> {
  return s ? JSON.parse(s) : {};
}

// careful: to store a FUNCTION in state you must wrap it, or React calls it
const [fn, setFn] = useState<() => number>(() => () => 42);
fn();                               // → 42
setFn(() => () => 7);               // the outer arrow is the updater
```

### Updating objects and arrays

```tsx
type Form = { name: string; tags: string[] };
const [form, setForm] = useState<Form>({ name: "", tags: [] });

// never mutate — React compares by reference
setForm((f) => ({ ...f, name: "a" }));
setForm((f) => ({ ...f, tags: [...f.tags, "new"] }));
setForm((f) => ({ ...f, tags: f.tags.filter((t) => t !== "new") }));
setForm((f) => ({ ...f, tags: f.tags.map((t) => (t === "a" ? "b" : t)) }));

// ✗ setForm((f) => { f.name = "a"; return f; });
//   compiles, but the reference is unchanged so React skips the re-render

// declaring state readonly makes the mistake a compile error
const [ro, setRo] = useState<Readonly<Form>>({ name: "", tags: [] });
// ✗ ro.name = "x";
//   Cannot assign to 'name' because it is a read-only property.
```

### Derived state is not state

```tsx
const [first, setFirst] = useState("Ada");
const [last, setLast] = useState("L");

// right — compute during render
const full = `${first} ${last}`;
full;                               // → 'Ada L'

// wrong — a second source of truth that drifts
// const [full, setFull] = useState(`${first} ${last}`);
// useEffect(() => setFull(`${first} ${last}`), [first, last]);

// if it can be computed from props or other state, compute it.
// Only reach for useMemo when the computation is genuinely expensive.
```

---

## useReducer

```tsx
import { useReducer } from "react";

type State = { count: number; step: number };

type Action =
  | { type: "inc" }
  | { type: "dec" }
  | { type: "setStep"; step: number }
  | { type: "reset" };

const initial: State = { count: 0, step: 1 };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "inc":     return { ...state, count: state.count + state.step };
    case "dec":     return { ...state, count: state.count - state.step };
    case "setStep": return { ...state, step: action.step };
    case "reset":   return initial;
    default: {
      const _never: never = action;      // exhaustiveness check
      return _never;
    }
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initial);
  //     ^? State          ^? React.Dispatch<Action>

  return (
    <>
      <output>{state.count}</output>
      <button onClick={() => dispatch({ type: "inc" })}>+</button>
      <button onClick={() => dispatch({ type: "setStep", step: 5 })}>step 5</button>
    </>
  );
}

reducer(initial, { type: "inc" });          // → { count: 1, step: 1 }
reducer({ count: 4, step: 2 }, { type: "dec" });   // → { count: 2, step: 2 }
// ✗ dispatch({ type: "increment" })
//   Type '"increment"' is not assignable to type
//   '"inc" | "dec" | "setStep" | "reset"'.
// ✗ dispatch({ type: "setStep" })
//   Property 'step' is missing.
```

Annotate the reducer's *return* type as `State`. Without it, a branch that forgets a field infers a narrower object and the error surfaces far from the cause.

### Lazy init

```tsx
function init(count: number): State {
  return { count, step: 1 };
}
// third argument: React calls init(arg) once
const [s, d] = useReducer(reducer, 10, init);
//     ^? State
```

`useReducer` over `useState` when: several fields change together, the next state depends on the previous in non-trivial ways, or you want the transitions testable as a pure function. `reducer` is just a function — test it directly, no React needed.

---

## useRef

Three distinct uses, three different types. Getting these wrong is a common source of confusion.

| Use | Declaration | `.current` type |
|---|---|---|
| DOM element | `useRef<HTMLInputElement \| null>(null)` | `HTMLInputElement \| null` |
| mutable value box | `useRef(0)` | `number` |
| value box, not yet set | `useRef<T \| null>(null)` | `T \| null` |

```tsx
import { useRef, useEffect } from "react";

function Input() {
  const ref = useRef<HTMLInputElement | null>(null);

  useEffect(() => {
    ref.current?.focus();           // always optional-chain: null until mounted
  }, []);

  return <input ref={ref} />;
}

// a mutable box that does not trigger re-renders
function Timer() {
  const renders = useRef(0);
  const timer = useRef<ReturnType<typeof setTimeout> | null>(null);

  renders.current += 1;             // fine; does NOT re-render

  useEffect(() => {
    timer.current = setTimeout(() => {}, 1000);
    return () => { if (timer.current) clearTimeout(timer.current); };
  }, []);

  return <span>{renders.current}</span>;
}
```

### The React 19 signature change

```tsx
// ≤18 there were three overloads, and this one produced a READONLY ref
//   const ref = useRef<HTMLInputElement>(null);
//   ^? React.RefObject<HTMLInputElement>   with `readonly current`
//   ...which meant you could pass it to <input ref={ref}> but never assign to it.
//   For a mutable box you had to write useRef<T | null>(null) explicitly.
//
// In React 19 `useRef` requires an argument and `RefObject<T>.current` is
// mutable. The practical effect: one spelling now works for both uses.

const a = useRef<HTMLInputElement | null>(null);
a.current = null;                   // allowed in 19; was an error in 18

// ✗ useRef();
//   Expected 1 arguments, but got 0.
//   (≤18 this was legal and gave MutableRefObject<undefined>)
const b = useRef<number | undefined>(undefined);   // the 19 spelling
```

### Ref callbacks and cleanup

```tsx
// a ref callback receives the element, or null on unmount
function Measured() {
  return (
    <div
      ref={(el) => {
        if (!el) return;
        el.scrollIntoView();
        // React 19: RETURN a cleanup function, like an effect
        return () => { /* teardown */ };
      }}
    />
  );
}

// ≤18 a ref callback returning anything was ignored, and cleanup was signalled
// by a second call with null:
//   ref={(el) => { if (el) attach(el); else detach(); }}
//
// In React 19 returning a function is the supported form. Returning a
// non-function value is now an error:
// ✗ ref={(el) => el && el.focus()}
//   the arrow implicitly returns a value; write a block body instead.

// correct concise form
<input ref={(el) => { el?.focus(); }} />;

// the callback ref's type
type RefCb = React.RefCallback<HTMLDivElement>;
//   ^? (instance: HTMLDivElement | null) => void | (() => void)
```

### `useImperativeHandle`

```tsx
import { useImperativeHandle } from "react";

type FieldHandle = {
  focus: () => void;
  clear: () => void;
};

function Field({ ref }: { ref?: React.Ref<FieldHandle> }) {
  const input = useRef<HTMLInputElement | null>(null);

  useImperativeHandle(ref, () => ({
    focus: () => input.current?.focus(),
    clear: () => { if (input.current) input.current.value = ""; },
  }), []);

  return <input ref={input} />;
}

function Parent() {
  const handle = useRef<FieldHandle | null>(null);
  return (
    <>
      <Field ref={handle} />
      <button onClick={() => handle.current?.clear()}>clear</button>
    </>
  );
}
```

---

## ref as a prop

The headline React 19 change for component authors: `forwardRef` is no longer needed.

```tsx
// React 19 — ref is an ordinary prop
type InputProps = ComponentProps<"input"> & { label: string };

function LabeledInput({ label, ref, ...rest }: InputProps) {
  return (
    <label>
      {label}
      <input ref={ref} {...rest} />
    </label>
  );
}

function Uses() {
  const r = useRef<HTMLInputElement | null>(null);
  return <LabeledInput label="Name" ref={r} />;
}

// ComponentProps<"input"> already contains `ref`, typed
// React.Ref<HTMLInputElement>. For a custom prop type, declare it:
type OwnProps = {
  label: string;
  ref?: React.Ref<HTMLInputElement>;
};
```

```tsx
// ≤18 the same component required forwardRef, and the generic order is
// <RefType, PropsType> — backwards from what everyone expects:
//
//   const Old = forwardRef<HTMLInputElement, { label: string }>(
//     ({ label }, ref) => (
//       <label>{label}<input ref={ref} /></label>
//     ),
//   );
//   Old.displayName = "Old";     // needed, or devtools shows "ForwardRef"
//
// forwardRef still works in 19 and is deprecated. There is an official codemod:
//   npx codemod@latest react/19/replace-use-form-state
//   npx types-react-codemod@latest preset-19 ./src
```

| | ≤18 | 19 |
|---|---|---|
| receive a ref | `forwardRef((props, ref) => …)` | `function C({ ref })` |
| ref type | `ForwardedRef<T>` | `Ref<T>` |
| generic order | `forwardRef<Element, Props>` | normal props generic |
| displayName | required for devtools | function name is used |
| ref cleanup | second call with `null` | return a cleanup function |
| `useRef()` no args | allowed | error — pass an argument |

---

## useEffect

```tsx
import { useEffect, useLayoutEffect } from "react";

function Sync({ id }: { id: string }) {
  const [data, setData] = useState<string | null>(null);

  // runs after paint, after every render where a dep changed
  useEffect(() => {
    let cancelled = false;

    fetch(`/api/${id}`)
      .then((r) => r.text())
      .then((t) => { if (!cancelled) setData(t); });

    // cleanup: runs before the next effect and on unmount
    return () => { cancelled = true; };
  }, [id]);

  return <>{data}</>;
}
```

| Deps | Runs |
|---|---|
| omitted | after every render |
| `[]` | once on mount (twice in StrictMode dev) |
| `[a, b]` | on mount and whenever `a` or `b` change by `Object.is` |

```tsx
// the cleanup must return void or a function — returning a promise is an error
// ✗ useEffect(async () => { await load(); }, []);
//   Argument of type '() => Promise<void>' is not assignable to parameter of
//   type 'EffectCallback'.
//     Type 'Promise<void>' is not assignable to type 'void | Destructor'.

// declare the async function inside
useEffect(() => {
  void (async () => { await load(); })();
}, []);

async function load() {}

// subscriptions
useEffect(() => {
  const onResize = () => {};
  window.addEventListener("resize", onResize);
  return () => window.removeEventListener("resize", onResize);
}, []);

// useLayoutEffect: same signature, runs synchronously BEFORE paint.
// Use only for measuring the DOM and preventing a visible flash.
useLayoutEffect(() => {}, []);
```

### You often do not need an effect

```tsx
// ✗ syncing derived state
// const [full, setFull] = useState("");
// useEffect(() => setFull(`${first} ${last}`), [first, last]);
// → just compute it during render

// ✗ resetting state when a prop changes
// useEffect(() => setSelected(null), [listId]);
// → give the component a key instead: <List key={listId} />

// ✗ handling a user event
// useEffect(() => { if (submitted) post(); }, [submitted]);
// → call post() in the onSubmit handler

// Effects are for synchronising with systems OUTSIDE React: the DOM, the
// network, timers, subscriptions, browser APIs. Everything else belongs in
// render or in an event handler.
```

---

## useEffectEvent

New in **19.2**. Extracts non-reactive logic from an effect, so a value can be read without becoming a dependency.

```tsx
import { useEffect, useEffectEvent, useState } from "react";

function Chat({ roomId, theme }: { roomId: string; theme: string }) {
  // onConnected always sees the latest `theme`, but does not make the effect
  // depend on it
  const onConnected = useEffectEvent(() => {
    showToast(`Connected to ${roomId}`, theme);
  });

  useEffect(() => {
    const conn = connect(roomId);
    conn.on("open", () => onConnected());
    return () => conn.close();
  }, [roomId]);            // theme is deliberately absent, and correctly so

  return null;
}

declare function connect(id: string): { on(e: string, f: () => void): void; close(): void };
declare function showToast(msg: string, theme: string): void;
```

```tsx
// ≤18 the workaround was a ref, kept in sync by a second effect:
//
//   const themeRef = useRef(theme);
//   useEffect(() => { themeRef.current = theme; });
//   useEffect(() => {
//     const conn = connect(roomId);
//     conn.on("open", () => showToast("...", themeRef.current));
//     return () => conn.close();
//   }, [roomId]);
//
// ...or, worse, omitting `theme` from the deps and silencing the lint rule.
```

Rules: an effect event may only be called from inside an effect, never passed to another component or hook, and never listed in a dependency array. The signature is `useEffectEvent<T extends Function>(callback: T): T` — the type is passed straight through.

---

## useMemo and useCallback

```tsx
import { useMemo, useCallback } from "react";

function List({ items, query }: { items: readonly string[]; query: string }) {
  // memoise an expensive computation
  const filtered = useMemo(
    () => items.filter((i) => i.includes(query)),
    [items, query],
  );
  //  ^? string[]

  // memoise a function identity, so a memo'd child does not re-render
  const onPick = useCallback((v: string) => {
    console.log(v);
  }, []);
  //  ^? (v: string) => void

  // annotate when the inferred type is too narrow
  const empty = useMemo<string[]>(() => [], []);

  return <Child items={filtered} onPick={onPick} />;
}

const Child = memo(function Child(p: {
  items: readonly string[];
  onPick: (v: string) => void;
}) {
  return <ul>{p.items.map((i) => <li key={i} onClick={() => p.onPick(i)}>{i}</li>)}</ul>;
});
```

```tsx
// useCallback(fn, deps) is exactly useMemo(() => fn, deps)
const a = useCallback((n: number) => n * 2, []);
const b = useMemo(() => (n: number) => n * 2, []);
// a and b have the same type and the same behaviour
```

### The React Compiler makes most of these unnecessary

```tsx
// React Compiler 1.0 (stable, 2025) memoises automatically at build time.
// With it enabled, hand-written useMemo/useCallback/memo are mostly redundant:
//
//   // babel.config.js
//   plugins: [["babel-plugin-react-compiler", {}]]
//
// Keep them only where you have measured a problem the compiler did not solve,
// or where a stable identity is required for correctness rather than speed —
// e.g. a value in a dependency array, or a key in a Map.
//
// Without the compiler, the rule of thumb is: memoise when the child is
// memo()'d, or when the computation is genuinely expensive. Everything else
// costs more than it saves.
```

### `useMemo` does not guarantee anything

```tsx
// React may discard memo caches to free memory. Never rely on useMemo for
// correctness — only for speed.
// ✗ const id = useMemo(() => crypto.randomUUID(), []);   // may change!
// → use useState(() => crypto.randomUUID()) or useId()

import { useId } from "react";
function Field() {
  const id = useId();               // stable, SSR-safe, unique per component
  return (
    <>
      <label htmlFor={id}>Name</label>
      <input id={id} />
    </>
  );
}
// useId is for accessibility attributes, not for list keys.
```

---

## Context

```tsx
import { createContext, useContext, useState } from "react";

type Theme = "light" | "dark";
type ThemeCtx = {
  theme: Theme;
  setTheme: (t: Theme) => void;
};

// null default + a guard hook is the pattern that gives you real types
const ThemeContext = createContext<ThemeCtx | null>(null);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>("light");
  // React 19: the context itself is the provider
  return (
    <ThemeContext value={{ theme, setTheme }}>
      {children}
    </ThemeContext>
  );
}

export function useTheme(): ThemeCtx {
  const ctx = useContext(ThemeContext);
  if (ctx === null) throw new Error("useTheme must be used inside ThemeProvider");
  return ctx;                       // ^? ThemeCtx — never null for consumers
}

function Toggle() {
  const { theme, setTheme } = useTheme();
  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      {theme}
    </button>
  );
}
```

```tsx
// ≤18 the provider needed the .Provider property:
//   <ThemeContext.Provider value={...}>{children}</ThemeContext.Provider>
//
// In React 19 <Context> and <Context.Provider> are both valid; .Provider is
// deprecated. <Context.Consumer> is deprecated too — use useContext.
```

The `| null` default plus a throwing hook beats a fake default object. A fake default silently renders wrong data when a provider is missing; the guard fails loudly, and the consumer's type has no `null` in it.

```tsx
// the lazy alternative, if you truly have a sensible default
const CountContext = createContext(0);
const count = useContext(CountContext);
//    ^? number                     no narrowing needed, but no missing-provider check

// splitting value and setter avoids re-rendering readers when only the
// setter identity is needed
const ValueCtx = createContext<number | null>(null);
const SetterCtx = createContext<((n: number) => void) | null>(null);
```

---

## use

`use` reads a promise or a context **conditionally** — the only hook-like API allowed inside a condition or loop.

```tsx
import { use, Suspense } from "react";

// reading a promise: suspends until it resolves
function Message({ promise }: { promise: Promise<string> }) {
  const text = use(promise);
  //    ^? string
  return <p>{text}</p>;
}

function Page() {
  const promise = getText();        // created OUTSIDE, in a parent or cache
  return (
    <Suspense fallback={<p>loading…</p>}>
      <Message promise={promise} />
    </Suspense>
  );
}
declare function getText(): Promise<string>;

// reading context conditionally — impossible with useContext
const ThemeCtx = createContext<{ theme: string } | null>(null);

function Row({ compact }: { compact: boolean }) {
  if (compact) {
    const t = use(ThemeCtx);                  // legal inside an if
    //    ^? { theme: string } | null
    return <span>{t?.theme}</span>;
  }
  return <div />;
}
```

```tsx
// the trap: creating the promise during render restarts it every render
function Bad() {
  const text = use(fetch("/api").then((r) => r.text()));   // new promise each render
  return <p>{text}</p>;
}
// → infinite suspend loop. The promise must be stable: create it in a
//   Server Component, in an event handler, or behind a cache.

// use() is NOT a data-fetching library. With no framework, reach for one:
//   TanStack Query, SWR, or a router loader.
```

`use` may be called conditionally, but it still may only be called from a component or a hook. It is not usable in an event handler or a plain function.

---

## Events

Every handler has a typed event. The pattern is `React.<Kind>Event<TElement>`.

| Handler prop | Event type | Handler type |
|---|---|---|
| `onClick` | `MouseEvent<HTMLButtonElement>` | `MouseEventHandler<T>` |
| `onChange` | `ChangeEvent<HTMLInputElement>` | `ChangeEventHandler<T>` |
| `onSubmit` | `FormEvent<HTMLFormElement>` | `FormEventHandler<T>` |
| `onKeyDown` | `KeyboardEvent<HTMLInputElement>` | `KeyboardEventHandler<T>` |
| `onFocus` / `onBlur` | `FocusEvent<T>` | `FocusEventHandler<T>` |
| `onPointerDown` | `PointerEvent<T>` | `PointerEventHandler<T>` |
| `onWheel` | `WheelEvent<T>` | `WheelEventHandler<T>` |
| `onDrop` | `DragEvent<T>` | `DragEventHandler<T>` |
| `onScroll` | `UIEvent<T>` | `UIEventHandler<T>` |
| `onAnimationEnd` | `AnimationEvent<T>` | `AnimationEventHandler<T>` |
| `onTransitionEnd` | `TransitionEvent<T>` | `TransitionEventHandler<T>` |

```tsx
function Form() {
  // inline handlers are contextually typed — no annotation needed
  return (
    <form onSubmit={(e) => { e.preventDefault(); }}>
      {/*      ^? e is React.FormEvent<HTMLFormElement> */}
      <input onChange={(e) => console.log(e.target.value)} />
      {/*          ^? e is React.ChangeEvent<HTMLInputElement>
                      e.target is HTMLInputElement, so .value is typed */}
      <button onClick={(e) => e.currentTarget.disabled = true}>go</button>
    </form>
  );
}

// a standalone handler must be annotated
function Standalone() {
  const onChange = (e: React.ChangeEvent<HTMLInputElement>) => e.target.value;
  const onClick: React.MouseEventHandler<HTMLButtonElement> = (e) => {
    e.currentTarget.blur();
  };
  return <><input onChange={onChange} /><button onClick={onClick} /></>;
}
```

### `target` vs `currentTarget`

```tsx
// currentTarget is the element the handler is ATTACHED to — always correctly typed
// target is where the event ORIGINATED — typed only as EventTarget for most events

<div onClick={(e) => {
  e.currentTarget;                  // ^? HTMLDivElement — safe
  e.target;                         // ^? EventTarget — could be any descendant
}} />;

// ChangeEvent is the exception: it narrows target for you
<input onChange={(e) => {
  e.target.value;                   // ^? string — ChangeEvent<T> types target as T
}} />;

// reading a form on submit
<form onSubmit={(e) => {
  e.preventDefault();
  const data = new FormData(e.currentTarget);
  //                        ^? HTMLFormElement
  data.get("email");                // ^? FormDataEntryValue | null
  String(data.get("email") ?? "");
}} />;

// when you must use target, narrow it
<div onClick={(e) => {
  if (e.target instanceof HTMLAnchorElement) {
    e.target.href;                  // ^? string
  }
}} />;
```

### Native vs synthetic

```tsx
// React handlers receive a SyntheticEvent. addEventListener receives a native one.
useEffect(() => {
  const onKey = (e: KeyboardEvent) => {       // the DOM lib type, not React's
    if (e.key === "Escape") close();
  };
  window.addEventListener("keydown", onKey);
  return () => window.removeEventListener("keydown", onKey);
}, []);
declare function close(): void;

// inside a React handler, the native event is on .nativeEvent
<input onKeyDown={(e: React.KeyboardEvent<HTMLInputElement>) => {
  e.nativeEvent;                    // ^? KeyboardEvent (the DOM one)
  e.key;                            // ^? string
}} />;

// React 19 no longer pools events — you can read them asynchronously.
// ≤16 required e.persist(); that method is now a no-op.
```

---

## Forms and Actions

React 19's form story: pass a function to `action`, and React handles pending state, submission, and resetting.

```tsx
function Simple() {
  async function save(formData: FormData) {
    const name = String(formData.get("name") ?? "");
    await fetch("/api", { method: "POST", body: JSON.stringify({ name }) });
  }

  return (
    <form action={save}>
      <input name="name" />
      <button type="submit">Save</button>
    </form>
  );
}

// the action prop's type
type FormAction = ComponentProps<"form">["action"];
//   ^? string | ((formData: FormData) => void | Promise<void>) | undefined

// React resets an uncontrolled form automatically after a successful action.
// It also wraps the call in a transition, so useFormStatus and useTransition
// see it as pending.
```

```tsx
// ≤18 all of this was manual:
//
//   const [pending, setPending] = useState(false);
//   const [error, setError] = useState<string | null>(null);
//   async function onSubmit(e: React.FormEvent<HTMLFormElement>) {
//     e.preventDefault();
//     setPending(true);
//     setError(null);
//     try {
//       await fetch(...);
//       e.currentTarget.reset();
//     } catch (err) {
//       setError(err instanceof Error ? err.message : "failed");
//     } finally {
//       setPending(false);
//     }
//   }
//   <form onSubmit={onSubmit}> ... </form>
//
// `action` replaces the whole block, and works without JavaScript when the
// action is a Server Function.
```

`formAction` does the same on a `<button>` or `<input type="submit">`, letting one form have several actions.

```tsx
<form action={saveDraft}>
  <input name="body" />
  <button type="submit">Save draft</button>
  <button formAction={publish}>Publish</button>
</form>;
declare function saveDraft(fd: FormData): Promise<void>;
declare function publish(fd: FormData): Promise<void>;
```

---

## useActionState

Wraps an action so you also get its return value and a pending flag.

```tsx
import { useActionState } from "react";

type State = { ok: boolean; message: string };

const initial: State = { ok: true, message: "" };

async function submit(prev: State, formData: FormData): Promise<State> {
  const email = String(formData.get("email") ?? "");
  if (!email.includes("@")) return { ok: false, message: "invalid email" };
  await fetch("/api/subscribe", { method: "POST", body: formData });
  return { ok: true, message: `subscribed ${email}` };
}

function Subscribe() {
  const [state, formAction, isPending] = useActionState(submit, initial);
  //     ^? State  ^? (payload: FormData) => void   ^? boolean

  return (
    <form action={formAction}>
      <input name="email" type="email" />
      <button disabled={isPending}>{isPending ? "…" : "Subscribe"}</button>
      {state.message && <p role={state.ok ? "status" : "alert"}>{state.message}</p>}
    </form>
  );
}
```

The signature, from `@types/react`:

```ts
function useActionState<State, Payload>(
  action: (state: Awaited<State>, payload: Payload) => State | Promise<State>,
  initialState: Awaited<State>,
  permalink?: string,
): [state: Awaited<State>, dispatch: (payload: Payload) => void, isPending: boolean];
```

| Point | Detail |
|---|---|
| first parameter | the *previous state*, not the form data — a frequent mix-up |
| `Payload` | inferred from the action's second parameter; `FormData` when used with `<form action>` |
| return state | `Awaited<State>` — the promise is unwrapped for you |
| `permalink` | a URL for progressive enhancement before hydration |
| not form-only | `Payload` can be anything if you call `dispatch` yourself |

```tsx
// a non-form action: Payload is whatever you declare
const [count, bump, busy] = useActionState(
  async (prev: number, by: number) => prev + by,
  0,
);
//     ^? number     ^? (payload: number) => void    ^? boolean
<button onClick={() => bump(5)} disabled={busy}>+5</button>;
```

```tsx
// ≤18 this was the canary `useFormState` from react-dom, with no pending flag:
//   import { useFormState } from "react-dom";
//   const [state, action] = useFormState(submit, initial);
//
// Renamed to useActionState, moved to `react`, and given the third tuple
// element in React 19. Codemod:
//   npx codemod@latest react/19/replace-use-form-state
```

---

## useFormStatus

Reads the pending state of the **parent** form. It must be called from a component rendered *inside* the `<form>` — not the one that renders the form.

```tsx
import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();
  //      ^? boolean  ^? FormData | null  ^? "get" | "post" | null

  return (
    <button type="submit" disabled={pending}>
      {pending ? "Saving…" : "Save"}
    </button>
  );
}

function Wrapper() {
  return (
    <form action={save}>
      <input name="x" />
      <SubmitButton />              {/* inside the form — sees the status */}
    </form>
  );
}
declare function save(fd: FormData): Promise<void>;

// ✗ calling useFormStatus in Wrapper returns { pending: false } always,
//   because Wrapper is not inside its own <form>.
```

Note the import is from `react-dom`, not `react`. `data` lets a button show what is being submitted while it is in flight.

---

## useOptimistic

Show the result immediately, roll back automatically if the action fails.

```tsx
import { useOptimistic, useState } from "react";

type Message = { id: string; text: string; sending?: boolean };

function Thread({ initial }: { initial: Message[] }) {
  const [messages, setMessages] = useState(initial);

  const [optimistic, addOptimistic] = useOptimistic(
    messages,
    (state: Message[], text: string): Message[] => [
      ...state,
      { id: "temp", text, sending: true },
    ],
  );
  //  ^? Message[]        ^? (action: string) => void

  async function send(formData: FormData) {
    const text = String(formData.get("text") ?? "");
    addOptimistic(text);                  // renders instantly
    const saved = await post(text);       // if this throws, the optimistic
    setMessages((m) => [...m, saved]);    // entry disappears on its own
  }

  return (
    <>
      <ul>
        {optimistic.map((m, i) => (
          <li key={m.id + i} style={{ opacity: m.sending ? 0.5 : 1 }}>{m.text}</li>
        ))}
      </ul>
      <form action={send}><input name="text" /><button>Send</button></form>
    </>
  );
}
declare function post(text: string): Promise<Message>;
```

The signatures:

```ts
function useOptimistic<State>(
  passthrough: State,
): [State, (action: State | ((pendingState: State) => State)) => void];

function useOptimistic<State, Action>(
  passthrough: State,
  reducer: (state: State, action: Action) => State,
): [State, (action: Action) => void];
```

| Point | Detail |
|---|---|
| annotate the reducer's parameters | `State` is inferred from `passthrough`, but `Action` is only inferred from the reducer |
| the optimistic value only exists during a transition | outside one it is exactly `passthrough` |
| rollback is automatic | when the action settles, React reverts to the real state |
| must be called inside an action | or the optimistic update is discarded at once |

---

## Transitions

```tsx
import { useTransition, startTransition, useDeferredValue } from "react";

function Tabs() {
  const [isPending, startT] = useTransition();
  //     ^? boolean  ^? (scope: () => void | Promise<void>) => void
  const [tab, setTab] = useState("home");

  return (
    <>
      <button onClick={() => startT(() => setTab("slow"))}>slow tab</button>
      <div style={{ opacity: isPending ? 0.6 : 1 }}>{tab}</div>
    </>
  );
}

// React 19: the scope function may be async — these are "Actions"
function Save() {
  const [isPending, startT] = useTransition();
  return (
    <button
      disabled={isPending}
      onClick={() => {
        startT(async () => {
          await fetch("/api", { method: "POST" });
          setDone(true);
        });
      }}
    >
      Save
    </button>
  );
}
declare function setDone(v: boolean): void;

// ≤18 startTransition only accepted a synchronous function; awaiting inside
// it silently ended the transition at the first await.
```

```tsx
// useDeferredValue: let an expensive subtree lag behind an input
function Search() {
  const [query, setQuery] = useState("");
  const deferred = useDeferredValue(query);
  //     ^? string
  const stale = query !== deferred;

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <div style={{ opacity: stale ? 0.5 : 1 }}>
        <Results query={deferred} />
      </div>
    </>
  );
}
declare function Results(p: { query: string }): React.ReactNode;

// React 19 adds an initial-value overload
const d = useDeferredValue(someValue, "initial");
declare const someValue: string;
```

In **19.3**, transitions render independently rather than being entangled into one render, so a slow transition no longer blocks unrelated ones.

---

## Suspense

```tsx
import { Suspense, lazy } from "react";

const Heavy = lazy(() => import("./Heavy.js"));
//    ^? React.LazyExoticComponent<...>

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Heavy />
    </Suspense>
  );
}
declare function Spinner(): React.ReactNode;

// lazy needs a DEFAULT export. For a named export, remap it:
const Named = lazy(async () => {
  const m = await import("./widgets.js");
  return { default: m.Widget };
});
```

```tsx
// nesting controls granularity: the nearest boundary above a suspending
// component wins
<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Suspense fallback={<ListSkeleton />}>
    <SlowList />
  </Suspense>
</Suspense>;
declare function PageSkeleton(): React.ReactNode;
declare function ListSkeleton(): React.ReactNode;
declare function Header(): React.ReactNode;
declare function SlowList(): React.ReactNode;

// what suspends: lazy(), use(promise), and framework data APIs.
// A plain fetch in useEffect does NOT suspend.
```

---

## Error boundaries

Still class-only. There is no hook equivalent, and `use`/Suspense do not catch render errors.

```tsx
import { Component, type ErrorInfo, type ReactNode } from "react";

type Props = { children: ReactNode; fallback: (e: Error) => ReactNode };
type State = { error: Error | null };

class ErrorBoundary extends Component<Props, State> {
  state: State = { error: null };

  static getDerivedStateFromError(error: Error): State {
    return { error };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error(error, info.componentStack);
  }

  render() {
    return this.state.error
      ? this.props.fallback(this.state.error)
      : this.props.children;
  }
}

<ErrorBoundary fallback={(e) => <p role="alert">{e.message}</p>}>
  <App />
</ErrorBoundary>;
declare function App(): React.ReactNode;
```

| Catches | Does not catch |
|---|---|
| errors during render | event handlers |
| errors in lifecycle methods | `setTimeout` / async callbacks |
| errors in constructors below it | server-side rendering |
| | errors thrown in the boundary itself |

React 19 adds root-level options so uncaught errors can be reported once rather than logged twice:

```tsx
// createRoot(el, { onUncaughtError, onCaughtError, onRecoverableError })
// In practice use react-error-boundary rather than hand-rolling the class.
```

---

## Custom hooks

A custom hook is a function whose name starts with `use` and which calls other hooks. TypeScript cares about the return type, and nothing else.

```tsx
// return a tuple with `as const`, or the type widens to an array union
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  const toggle = useCallback(() => setOn((v) => !v), []);
  return [on, toggle] as const;
  //     ^? readonly [boolean, () => void]
}

const [open, toggleOpen] = useToggle();
open;                               // ^? boolean
toggleOpen;                         // ^? () => void

// without `as const`:
//   return [on, toggle];
//   ^? (boolean | (() => void))[]     — destructuring gives you a useless union

// an object return is self-documenting and order-independent — prefer it
// for anything with more than two members
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    fetch(url)
      .then((r) => r.json() as Promise<T>)
      .then((d) => { if (!cancelled) { setData(d); setError(null); } })
      .catch((e: unknown) => {
        if (!cancelled) setError(e instanceof Error ? e : new Error(String(e)));
      })
      .finally(() => { if (!cancelled) setLoading(false); });
    return () => { cancelled = true; };
  }, [url]);

  return { data, error, loading };
}

const { data } = useFetch<{ id: string }>("/api/user");
data?.id;                           // ^? string | undefined
```

### Returning a discriminated union from a hook

```tsx
type Async<T> =
  | { status: "loading"; data: null; error: null }
  | { status: "error"; data: null; error: Error }
  | { status: "success"; data: T; error: null };

function useAsync<T>(url: string): Async<T> {
  const { data, error, loading } = useFetch<T>(url);
  if (loading) return { status: "loading", data: null, error: null };
  if (error) return { status: "error", data: null, error };
  return { status: "success", data: data as T, error: null };
}

function Show() {
  const r = useAsync<{ name: string }>("/api");
  if (r.status === "loading") return <p>…</p>;
  if (r.status === "error") return <p>{r.error.message}</p>;
  return <p>{r.data.name}</p>;      // narrowed: data is non-null here
}
```

This is worth the extra lines. With three independent fields the caller must remember that `data` is non-null only when `loading` is false and `error` is null; with a union the compiler remembers for them.

### Rules, enforced by lint not by types

```tsx
// hooks must be called unconditionally, in the same order, every render.
// TypeScript cannot check this — install eslint-plugin-react-hooks.
//
// ✗ if (x) { const [a] = useState(0); }
// ✗ for (const i of xs) { useEffect(...); }
// ✗ hooks inside callbacks, conditions, or after an early return
//
// `use` is the single exception — it may be called conditionally.
```

---

## Generic components

```tsx
// a generic list, inferring the item type from the data prop
type ListProps<T> = {
  items: readonly T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyOf: (item: T) => string;
};

function List<T>({ items, renderItem, keyOf }: ListProps<T>) {
  return <ul>{items.map((it, i) => <li key={keyOf(it)}>{renderItem(it, i)}</li>)}</ul>;
}

type User = { id: string; name: string };
const users: User[] = [{ id: "1", name: "Ada" }];

<List
  items={users}
  keyOf={(u) => u.id}               // ^? u is User — inferred
  renderItem={(u) => <b>{u.name}</b>}
/>;
// ✗ renderItem={(u) => <b>{u.nmae}</b>}
//   Property 'nmae' does not exist on type 'User'.

// explicit type argument, when inference is not enough
<List<User> items={[]} keyOf={(u) => u.id} renderItem={(u) => u.name} />;
```

### Constraining the item type

```tsx
// require an id, and derive the key automatically
function Keyed<T extends { id: string }>({ items }: { items: readonly T[] }) {
  return <ul>{items.map((it) => <li key={it.id}>{String(it.id)}</li>)}</ul>;
}
<Keyed items={users} />;
// ✗ <Keyed items={[{ name: "x" }]} />
//   Property 'id' is missing in type '{ name: string; }'.

// a generic key-of-object prop
function Column<T, K extends keyof T>({ row, field }: { row: T; field: K }) {
  return <td>{String(row[field])}</td>;
}
<Column row={users[0]!} field="name" />;
// ✗ <Column row={users[0]!} field="nope" />
//   Type '"nope"' is not assignable to type 'keyof User'.
```

### `memo` erases generics

```tsx
// memo() returns a non-generic component — the type parameter is lost
const MemoList = memo(List) as typeof List;
// the cast restores callers' inference. Without it:
//   const M = memo(List);  <M items={users} .../>  → T is inferred as unknown
```

---

## memo

```tsx
import { memo } from "react";

// memo skips a re-render when props are shallowly equal
const Row = memo(function Row({ label, onPick }: {
  label: string;
  onPick: (l: string) => void;
}) {
  return <li onClick={() => onPick(label)}>{label}</li>;
});
//    ^? React.MemoExoticComponent<...>

// a custom comparator — note it returns true to SKIP the render, the
// opposite of shouldComponentUpdate's name
const Deep = memo(
  function Deep({ data }: { data: { id: string } }) { return <p>{data.id}</p>; },
  (prev, next) => prev.data.id === next.data.id,
);

// memo is defeated by any prop that is a fresh reference each render
// ✗ <Row label="a" onPick={(l) => pick(l)} />       new function every render
//   <Row label="a" onPick={stableOnPick} />         useCallback, or the compiler
declare function pick(l: string): void;
declare const stableOnPick: (l: string) => void;
```

With React Compiler enabled, `memo` is usually redundant — the compiler inserts equivalent checks. Without it, `memo` only pays off when the component is expensive *and* its props are stable.

---

## Portals and fragments

```tsx
import { createPortal } from "react-dom";
import { Fragment } from "react";

function Modal({ children }: { children: React.ReactNode }) {
  return createPortal(
    <div role="dialog">{children}</div>,
    document.body,
  );
  //  ^? React.ReactPortal
}

// fragments: <>…</> is shorthand, but a keyed fragment needs the long form
const rows = [{ id: "a", v: 1 }];
<>
  {rows.map((r) => (
    <Fragment key={r.id}>
      <dt>{r.id}</dt>
      <dd>{r.v}</dd>
    </Fragment>
  ))}
</>;
// ✗ <key={r.id}>  — the shorthand accepts no props at all
```

### Fragment refs — new in 19.3

Attach DOM behaviour to a group of siblings without adding a wrapper element.

```tsx
// pass a ref to a Fragment and it exposes the group's DOM nodes, so you can
// observe or focus them without a wrapper <div> distorting your layout
function Group({ children }: { children: React.ReactNode }) {
  const ref = useRef<React.FragmentInstance | null>(null);

  useEffect(() => {
    ref.current?.focus();
    // also: observeUsing / unobserveUsing for Intersection and Resize observers
  }, []);

  return <Fragment ref={ref}>{children}</Fragment>;
}

// ≤19.2 the only option was a wrapper element, which could break a grid or
// flex layout, or cloneElement gymnastics over Children.
```

---

## Activity

New in **19.2**. Keeps a subtree mounted but hidden, preserving its state while unmounting its effects.

```tsx
import { Activity } from "react";

function Tabs({ active }: { active: "a" | "b" }) {
  return (
    <>
      <Activity mode={active === "a" ? "visible" : "hidden"}>
        <TabA />
      </Activity>
      <Activity mode={active === "b" ? "visible" : "hidden"}>
        <TabB />
      </Activity>
    </>
  );
}
declare function TabA(): React.ReactNode;
declare function TabB(): React.ReactNode;

// mode: "visible" | "hidden"
// hidden  → children stay mounted, state is preserved, effects are cleaned up,
//           and updates render at low priority
// visible → children reappear with their state intact and effects re-created
```

| | conditional render | `display: none` | `<Activity mode="hidden">` |
|---|---|---|---|
| state preserved | no | yes | yes |
| effects run while hidden | n/a | yes | no |
| in the layout | no | no | no |
| pre-renders before shown | no | yes | yes, at low priority |

Use it for tabs, sidebars and multi-step forms where remounting would lose scroll position, form input or expension state. It is not a replacement for conditional rendering when the subtree is genuinely done with.

---

## ViewTransition

New in **19.3**. Animates elements entering, exiting, moving or resizing using the browser's View Transition API.

```tsx
import { ViewTransition, startTransition, addTransitionType } from "react";

function Gallery({ id }: { id: string | null }) {
  return (
    <ViewTransition>
      {id ? <Detail id={id} /> : <Grid />}
    </ViewTransition>
  );
}
declare function Detail(p: { id: string }): React.ReactNode;
declare function Grid(): React.ReactNode;

// name a transition so React can pair elements across the change
<ViewTransition name="hero"><img src="/a.jpg" /></ViewTransition>;

// tag a transition to drive different animations from CSS
function navigate(to: string) {
  startTransition(() => {
    addTransitionType(to === "/" ? "nav-back" : "nav-forward");
    setRoute(to);
  });
}
declare function setRoute(to: string): void;
```

Only updates inside a transition (`startTransition`, an Action, a Suspense reveal) are animated. Support degrades gracefully: browsers without the View Transition API simply render without animation.

---

## Server components

Covered properly in the Next.js page; here is the type-level surface.

```tsx
// a Server Component may be async — a Client Component may not
async function Page({ id }: { id: string }) {
  const user = await db.user(id);
  //    ^? User
  return <Profile user={user} />;
}
type User = { id: string; name: string };
declare const db: { user(id: string): Promise<User> };
declare function Profile(p: { user: User }): React.ReactNode;

// ✗ marking a Client Component async
//   "use client";
//   async function Bad() { return <div />; }
//   → React throws at runtime: async Client Components are not supported.

// directives are string literals at the top of a file, not imports
// "use client"  → this module and its imports run in the browser
// "use server"  → the exported functions are Server Functions, callable
//                 from the client as if they were local

// a Server Function's type is just its signature; the network hop is invisible
// export async function save(fd: FormData): Promise<void> { "use server"; ... }
```

Props crossing the server → client boundary must be serialisable: primitives, plain objects, arrays, `Date`, `Map`, `Set`, `FormData`, promises, and Server Functions. Class instances and ordinary functions are not.

```tsx
// ✗ <ClientThing onDone={() => {}} />  from a Server Component
//   → runtime error: Functions cannot be passed directly to Client Components.
//   Pass a Server Function, or move the handler into the client component.
```

---

## General idioms

### Conditional rendering

```tsx
declare const items: string[];
declare const user: { name: string } | null;
// (written with `declare`: `const user: T | null = null` narrows to `null`,
//  which makes the truthy branch below `never` and its property access fail)

// ternary for either/or
<div>{user ? <b>{user.name}</b> : <i>anonymous</i>}</div>;

// && for "render or nothing" — but guard against 0
<div>{items.length > 0 && <ul>{items.length}</ul>}</div>;
// ✗ {items.length && <ul>…</ul>}   renders a literal "0" when empty

// null renders nothing, and is the right early return
function Maybe({ show }: { show: boolean }) {
  if (!show) return null;
  return <div />;
}

// ?? for a fallback value, not a fallback element
<span>{user?.name ?? "anonymous"}</span>;
```

### Keys

```tsx
const rows = [{ id: "a" }, { id: "b" }];

rows.map((r) => <li key={r.id} />);            // right: stable and unique
rows.map((r, i) => <li key={i} />);            // wrong when the list reorders
                                               // or items are inserted/removed
rows.map((r) => <li key={crypto.randomUUID()} />);  // wrong: remounts every render

// key is not a prop — it is never visible inside the component
function Row({ key }: { key?: string }) { return <li />; }
// ✗ reading props.key gives undefined and warns in development

// key also forces a remount, which is the idiomatic way to reset state
<Editor key={documentId} initial={text} />;
declare function Editor(p: { initial: string }): React.ReactNode;
declare const documentId: string;
declare const text: string;
```

### Controlled inputs

```tsx
function Controlled() {
  const [value, setValue] = useState("");
  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}

// value={undefined} makes an input uncontrolled; value={null} warns.
// Use "" as the empty value, never null.
const [v, setV] = useState<string>("");   // not <string | null>

// checkboxes use `checked`, selects use `value`, textareas use `value`
<input type="checkbox" checked={true} onChange={() => {}} />;
<select value="a" onChange={(e) => e.target.value}><option value="a">A</option></select>;
<textarea value="x" onChange={(e) => e.target.value} />;

// uncontrolled with defaultValue, read via a ref or FormData
<input defaultValue="x" name="field" />;
```

### Typing a props spread through a wrapper

```tsx
// forward everything you do not consume, and keep className composable
type CardProps = ComponentProps<"div"> & { tone?: "info" | "warn" };

function Card({ tone = "info", className, children, ...rest }: CardProps) {
  return (
    <div className={[`card-${tone}`, className].filter(Boolean).join(" ")} {...rest}>
      {children}
    </div>
  );
}

<Card tone="warn" id="c" onClick={() => {}} role="note">body</Card>;
```

### Narrowing `ReactNode` is not possible

```tsx
// there is no reliable way to inspect what a ReactNode "is"
// ✗ if (children.type === MyComponent) ...
//   works only for a single element and couples you to internals.
// → pass separate props instead of inspecting children:
type SplitProps = { header: ReactNode; body: ReactNode };
```

### Exhaustive component maps

```tsx
type Kind = "info" | "warn" | "error";

const ICONS: Record<Kind, ComponentType<{ size: number }>> = {
  info: InfoIcon,
  warn: WarnIcon,
  error: ErrorIcon,
};
declare function InfoIcon(p: { size: number }): React.ReactNode;
declare function WarnIcon(p: { size: number }): React.ReactNode;
declare function ErrorIcon(p: { size: number }): React.ReactNode;

function Alert({ kind }: { kind: Kind }) {
  const Icon = ICONS[kind];
  return <Icon size={16} />;
}
// add a Kind and the build fails until ICONS is updated.
```

### Document metadata and resources

React 19 hoists these from anywhere in the tree into `<head>`, so no Helmet-style library is needed.

```tsx
function Article({ title }: { title: string }) {
  return (
    <article>
      <title>{title}</title>
      <meta name="description" content="…" />
      <link rel="canonical" href="/a" />
      <link rel="stylesheet" href="/a.css" precedence="default" />
      <script async src="/a.js" />
      <p>body</p>
    </article>
  );
}

// `precedence` on a stylesheet controls insertion order and enables
// deduplication. Preloading APIs live in react-dom:
//   import { preload, preinit, prefetchDNS, preconnect } from "react-dom";
```

---

## Gotchas

| Trap | Reality |
|---|---|
| `{items.length && <X/>}` | renders `0` when empty; use `> 0 &&` |
| `key={index}` | breaks on reorder, insert and delete |
| `useRef()` with no argument | an error in React 19 |
| `useState(null)` unannotated | infers `null`, and nothing else is assignable |
| `useState([])` unannotated | infers `never[]` |
| `useState(() => fn)` | the function is treated as a lazy initializer, not stored |
| `useEffect(async () => …)` | returns a promise where a cleanup is expected |
| mutating state then `setState(same)` | same reference, so no re-render |
| `useMemo` for identity | caches may be dropped; use `useState` or `useId` |
| `e.target` | `EventTarget`, not your element — use `e.currentTarget` |
| `value={null}` on an input | warns; use `""` |
| `React.FC` | no longer implies children, and blocks generics |
| `forwardRef` | deprecated in 19; `ref` is a prop |
| `<Context.Provider>` | deprecated in 19; render `<Context>` |
| `propTypes` / `defaultProps` | removed for function components in 19 |
| `memo(GenericComponent)` | loses the type parameter |
| async Client Component | runtime error — only Server Components may be async |
| passing a function from a Server to a Client Component | runtime serialisation error |

```tsx
// the && trap, in full. These are renderToStaticMarkup outputs:
<div>{0 && <span>hi</span>}</div>;           // → <div>0</div>       the bug
<div>{0 > 0 && <span>hi</span>}</div>;       // → <div></div>        the fix
<div>{Boolean(0) && <span>hi</span>}</div>;  // → <div></div>        also fine

// which falsy values actually render is not intuitive:
<div>{""}</div>;                             // → <div></div>        safe
<div>{null}</div>;                           // → <div></div>        safe
<div>{undefined}</div>;                      // → <div></div>        safe
<div>{false}</div>;                          // → <div></div>        safe
<div>{0}</div>;                              // → <div>0</div>       renders!
<div>{NaN}</div>;                            // → <div>NaN</div>     renders!

// so: empty strings, null, undefined and false are all safe on the left of &&.
// Numbers are not — and that includes NaN, which is falsy yet still printed.
// Always compare explicitly: `xs.length > 0 &&`, never `xs.length &&`.

// StrictMode double-invokes render, effects and state updaters in development
// to surface impure code. An effect that runs twice on mount is StrictMode
// doing its job, not a bug. Write effects that tolerate it.

// stale closure
function Stale() {
  const [n, setN] = useState(0);
  useEffect(() => {
    const t = setInterval(() => setN(n + 1), 1000);   // n is captured at 0
    return () => clearInterval(t);
  }, []);                                            // forever 1
  return <>{n}</>;
}
function Fixed() {
  const [n, setN] = useState(0);
  useEffect(() => {
    const t = setInterval(() => setN((c) => c + 1), 1000);  // updater form
    return () => clearInterval(t);
  }, []);
  return <>{n}</>;
}
```

---

## React 18 → 19

| Area | React 18 | React 19 |
|---|---|---|
| refs into components | `forwardRef((p, ref) => …)` | `ref` is a normal prop |
| `useRef` | optional argument; `RefObject.current` readonly | argument required; `current` mutable |
| ref cleanup | second call with `null` | return a cleanup function |
| context provider | `<Ctx.Provider value>` | `<Ctx value>` |
| context consumer | `<Ctx.Consumer>` | `useContext` or `use` |
| form submission | `onSubmit` + manual pending/error | `<form action={fn}>` |
| form state | `useFormState` from `react-dom` | `useActionState` from `react`, plus `isPending` |
| optimistic UI | hand-rolled with `useState` | `useOptimistic` |
| async in a transition | not supported | `startTransition(async () => …)` |
| reading a promise in render | not supported | `use(promise)` |
| non-reactive effect logic | a ref synced by a second effect | `useEffectEvent` (19.2) |
| hiding UI without losing state | `display: none` | `<Activity mode="hidden">` (19.2) |
| animating route changes | a third-party library | `<ViewTransition>` (19.3) |
| grouping refs without a wrapper | wrapper `<div>` | Fragment refs (19.3) |
| `<head>` tags | `react-helmet` | render `<title>` / `<meta>` / `<link>` anywhere |
| `defaultProps` (function components) | supported | removed |
| `propTypes` | supported | removed |
| string refs, legacy context, module factories | deprecated | removed |
| `ReactDOM.render` | deprecated | removed — use `createRoot` |
| `react-test-renderer` | available | deprecated |
| memoisation | manual `useMemo` / `memo` | React Compiler 1.0 |

```bash
# upgrading the types
npx types-react-codemod@latest preset-19 ./src

# the individual codemods
npx codemod@latest react/19/replace-reactdom-render
npx codemod@latest react/19/replace-use-form-state
npx codemod@latest react/19/remove-forward-ref
```

---

## Version notes

| Feature | Requires |
|---|---|
| hooks | 16.8 |
| `Suspense` for lazy | 16.6 |
| new JSX transform (no `React` import) | 17 |
| `createRoot`, automatic batching | 18 |
| `useId`, `useSyncExternalStore`, `useDeferredValue`, `useTransition` | 18 |
| `children` no longer implicit in `React.FC` | `@types/react` 18 |
| `use` | 19 |
| `ref` as a prop; `forwardRef` deprecated | 19 |
| `useActionState`, `useOptimistic`, `useFormStatus` | 19 |
| `<form action>`, `formAction` | 19 |
| async functions in `startTransition` | 19 |
| `<Context>` as provider | 19 |
| document metadata hoisting, `precedence` on stylesheets | 19 |
| ref callback cleanup functions | 19 |
| `useRef` requires an argument | 19 |
| `<Activity>` | 19.2 |
| `useEffectEvent` | 19.2 |
| `cacheSignal` (RSC) | 19.2 |
| partial pre-rendering / resume APIs | 19.2 |
| React Compiler 1.0 stable | 2025 |
| `<ViewTransition>`, `addTransitionType` | 19.3 |
| Fragment refs | 19.3 |
| `browser()` in `react-dom` | 19.3 |
| Trusted Types integration | 19.3 |
| independent (non-entangled) transitions | 19.3 |

React 19.0 shipped December 2024; 19.2 in October 2025; 19.3 in September 2026.

Check with `npm ls react @types/react`.
