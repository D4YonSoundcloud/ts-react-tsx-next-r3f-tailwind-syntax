# Data Structures and Algorithms in TypeScript

A single-page reference for the built-in collections, the classic data structures implemented from scratch, and the algorithms that operate on them — all in TypeScript, all with the complexity and a realistic use for each.

This page assumes the [TypeScript reference](./typescript.md) for the language itself.

**How to use this page.** One long file, so browser find (`Ctrl+F` / `Cmd+F`) is the search tool. Search for the structure or method — `LinkedList`, `Dijkstra`, `splice`, `topological` — rather than a description.

| Mark | Meaning |
|---|---|
| `// →` | the value the expression produces — **executed and checked**, not asserted |
| `// ^?` | the type TypeScript infers |
| `// ✗` | an error, followed by the message |
| `// O(...)` | time complexity, amortised unless stated |

Every `// →` on this page was extracted, compiled and run; the value shown is the value the code produced.

---

## Contents

[Complexity](#complexity) · [Arrays](#arrays) · [Array algorithms](#array-algorithms) · [Strings](#strings) · [Objects as records](#objects-as-records) · [Map](#map) · [Set](#set) · [WeakMap and WeakSet](#weakmap-and-weakset) · [Hash maps from scratch](#hash-maps-from-scratch) · [Typed arrays](#typed-arrays) · [Stack](#stack) · [Queue](#queue) · [Deque](#deque) · [Singly linked list](#singly-linked-list) · [Doubly linked list](#doubly-linked-list) · [Circular linked list](#circular-linked-list) · [LRU cache](#lru-cache) · [Binary heap](#binary-heap) · [Priority queue](#priority-queue) · [Binary trees](#binary-trees) · [Tree traversals](#tree-traversals) · [Binary search tree](#binary-search-tree) · [AVL tree](#avl-tree) · [Trie](#trie) · [Union-Find](#union-find) · [Segment tree](#segment-tree) · [Fenwick tree](#fenwick-tree) · [Graphs](#graphs) · [BFS and DFS](#bfs-and-dfs) · [Topological sort](#topological-sort) · [Shortest paths](#shortest-paths) · [Minimum spanning tree](#minimum-spanning-tree) · [Sorting](#sorting) · [Binary search](#binary-search) · [Two pointers](#two-pointers) · [Sliding window](#sliding-window) · [Backtracking](#backtracking) · [Dynamic programming](#dynamic-programming) · [Gotchas](#gotchas) · [Cheat sheet](#cheat-sheet)

---

## Complexity

| Notation | Name | At n = 1,000,000 |
|---|---|---|
| `O(1)` | constant | 1 |
| `O(log n)` | logarithmic | ~20 |
| `O(n)` | linear | 1,000,000 |
| `O(n log n)` | linearithmic | ~20,000,000 |
| `O(n²)` | quadratic | 10¹² — too slow |
| `O(2ⁿ)` | exponential | hopeless past n ≈ 30 |
| `O(n!)` | factorial | hopeless past n ≈ 12 |

Rules of thumb for a rough budget of 10⁸ simple operations per second:

| n up to | What fits |
|---|---|
| 10 | `O(n!)` |
| 25 | `O(2ⁿ)` |
| 5,000 | `O(n²)` |
| 10⁶ | `O(n log n)` |
| 10⁸ | `O(n)` |

Space complexity counts extra memory beyond the input. A recursive function's call stack counts: a recursion of depth `n` is `O(n)` space even with no arrays.

---

## Arrays

Contiguous, indexed, growable. The default choice unless you have a reason otherwise.

| Operation | Method | Time |
|---|---|---|
| read / write by index | `a[i]` | O(1) |
| append | `push` | O(1) amortised |
| remove last | `pop` | O(1) |
| prepend | `unshift` | O(n) |
| remove first | `shift` | O(n) |
| insert / delete middle | `splice` | O(n) |
| search unsorted | `indexOf`, `includes`, `find` | O(n) |
| search sorted | binary search | O(log n) |
| sort | `sort` | O(n log n) |
| copy | `slice`, `[...a]` | O(n) |
| concatenate | `concat`, `[...a, ...b]` | O(n + m) |

```ts
const a = [3, 1, 4, 1, 5];

// reading
a[0];                                   // → 3
a.at(-1);                               // → 5
a.length;                               // → 5

// mutating — these change `a` in place and are why `const` does not mean frozen
const b = [1, 2, 3];
b.push(4);                              // → 4          the new length
b.pop();                                // → 4          the removed element
b.unshift(0);                           // → 4          O(n): everything shifts
b.shift();                              // → 0          O(n)
b;                                      // → [1, 2, 3]

// splice(start, deleteCount, ...insert) — the swiss army knife, all O(n)
const c = [1, 2, 3, 4, 5];
c.splice(1, 2);                         // → [2, 3]     returns what it removed
c;                                      // → [1, 4, 5]
c.splice(1, 0, 9, 9);                   // → []         pure insert
c;                                      // → [1, 9, 9, 4, 5]

// non-mutating equivalents (ES2023) — prefer these in React and reducers
const d = [3, 1, 2];
d.toSorted();                           // → [1, 2, 3]
d.toReversed();                         // → [2, 1, 3]
d.with(0, 99);                          // → [99, 1, 2]
d.toSpliced(1, 1);                      // → [3, 2]
d;                                      // → [3, 1, 2]  unchanged
```

### Searching and transforming

```ts
const xs = [3, 1, 4, 1, 5, 9];

xs.indexOf(1);                          // → 1          first match, O(n)
xs.lastIndexOf(1);                      // → 3
xs.includes(4);                         // → true
xs.find((v) => v > 3);                  // → 4
xs.findIndex((v) => v > 3);             // → 2
xs.findLast((v) => v > 3);              // → 9
xs.findLastIndex((v) => v > 3);         // → 5

xs.filter((v) => v > 2);                // → [3, 4, 5, 9]
xs.map((v) => v * 2);                   // → [6, 2, 8, 2, 10, 18]
xs.reduce((acc, v) => acc + v, 0);      // → 23
xs.some((v) => v > 8);                  // → true
xs.every((v) => v > 0);                 // → true
xs.flat();                              // → [3, 1, 4, 1, 5, 9]
[[1, 2], [3]].flat();                   // → [1, 2, 3]
[1, 2].flatMap((v) => [v, v]);          // → [1, 1, 2, 2]

xs.slice(1, 3);                         // → [1, 4]     O(n), non-mutating
xs.join("-");                           // → '3-1-4-1-5-9'
```

### Creating

```ts
Array.from({ length: 4 }, (_, i) => i);         // → [0, 1, 2, 3]
Array.from("abc");                              // → ['a', 'b', 'c']
Array.from(new Set([1, 1, 2]));                 // → [1, 2]
new Array(3).fill(0);                           // → [0, 0, 0]
Array.of(1, 2);                                 // → [1, 2]

// a 2D grid — each row MUST be its own array
const grid = Array.from({ length: 2 }, () => new Array(3).fill(0));
grid[0]![0] = 9;
grid;                                           // → [[9, 0, 0], [0, 0, 0]]

// ✗ the classic bug: every row is the SAME array
const broken = new Array(2).fill(new Array(3).fill(0));
broken[0]![0] = 9;
broken;                                         // → [[9, 0, 0], [9, 0, 0]]
```

### Sorting

```ts
// sort() compares as STRINGS by default — always pass a comparator for numbers
[10, 9, 1].sort();                              // → [1, 10, 9]
[10, 9, 1].sort((x, y) => x - y);               // → [1, 9, 10]
[10, 9, 1].sort((x, y) => y - x);               // → [10, 9, 1]

["b", "a"].sort();                              // → ['a', 'b']
["b", "a"].sort((x, y) => x.localeCompare(y));  // → ['a', 'b']

// multi-key: compare in priority order, fall through on ties
type Row = { name: string; age: number };
const rows: Row[] = [
  { name: "b", age: 30 },
  { name: "a", age: 30 },
  { name: "c", age: 20 },
];
const byAgeThenName = rows.toSorted((p, q) => p.age - q.age || p.name.localeCompare(q.name));
byAgeThenName.map((r) => r.name);               // → ['c', 'a', 'b']

// sort is stable (guaranteed since ES2019), so equal elements keep their order
```

**Common uses.** Arrays are the right default for: an ordered list you mostly append to and iterate; a fixed grid; anything you will sort or binary-search. Switch away when you need O(1) membership tests (`Set`), O(1) keyed lookup (`Map`), or frequent insertion at the front (`Deque`, linked list).

---

## Array algorithms

```ts
// deduplicate, order preserved
const dedupe = <T>(xs: readonly T[]): T[] => [...new Set(xs)];
dedupe([3, 1, 3, 2]);                           // → [3, 1, 2]

// group by a key — Object.groupBy is ES2024
function groupBy<T, K extends PropertyKey>(xs: readonly T[], key: (v: T) => K) {
  const out = new Map<K, T[]>();
  for (const v of xs) {
    const k = key(v);
    const bucket = out.get(k);
    if (bucket) bucket.push(v);
    else out.set(k, [v]);
  }
  return out;
}
[...groupBy(["ant", "bee", "ape"], (w) => w[0]!)];
// → [['a', ['ant', 'ape']], ['b', ['bee']]]

// count occurrences
function counter<T>(xs: Iterable<T>): Map<T, number> {
  const m = new Map<T, number>();
  for (const v of xs) m.set(v, (m.get(v) ?? 0) + 1);
  return m;
}
// Iterable, not readonly T[], so it also accepts a string or a Set
[...counter("abracadabra")].slice(0, 2);        // → [['a', 5], ['b', 2]]

// chunk into fixed-size pieces
function chunk<T>(xs: readonly T[], n: number): T[][] {
  const out: T[][] = [];
  for (let i = 0; i < xs.length; i += n) out.push(xs.slice(i, i + n));
  return out;
}
chunk([1, 2, 3, 4, 5], 2);                      // → [[1, 2], [3, 4], [5]]

// zip
function zip<A, B>(a: readonly A[], b: readonly B[]): [A, B][] {
  const n = Math.min(a.length, b.length);
  return Array.from({ length: n }, (_, i) => [a[i]!, b[i]!]);
}
zip([1, 2, 3], ["a", "b"]);                     // → [[1, 'a'], [2, 'b']]

// rotate left by k, O(n)
function rotate<T>(xs: readonly T[], k: number): T[] {
  const n = xs.length;
  if (n === 0) return [];
  const s = ((k % n) + n) % n;
  return [...xs.slice(s), ...xs.slice(0, s)];
}
rotate([1, 2, 3, 4, 5], 2);                     // → [3, 4, 5, 1, 2]
rotate([1, 2, 3], -1);                          // → [3, 1, 2]

// in-place reverse, O(n) time O(1) space
function reverseInPlace<T>(xs: T[]): T[] {
  for (let i = 0, j = xs.length - 1; i < j; i++, j--) {
    [xs[i], xs[j]] = [xs[j]!, xs[i]!];
  }
  return xs;
}
reverseInPlace([1, 2, 3, 4]);                   // → [4, 3, 2, 1]

// prefix sums — turns any range-sum query into O(1)
function prefixSums(xs: readonly number[]): number[] {
  const out = [0];
  for (const v of xs) out.push(out[out.length - 1]! + v);
  return out;
}
const ps = prefixSums([1, 2, 3, 4]);
ps;                                             // → [0, 1, 3, 6, 10]
ps[3]! - ps[1]!;                                // → 5    sum of indices 1..2

// Kadane: maximum subarray sum, O(n)
function maxSubarray(xs: readonly number[]): number {
  let best = xs[0] ?? 0, cur = best;
  for (let i = 1; i < xs.length; i++) {
    cur = Math.max(xs[i]!, cur + xs[i]!);
    best = Math.max(best, cur);
  }
  return best;
}
maxSubarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]);   // → 6
```

---

## Strings

Immutable. Every method returns a new string, which makes repeated concatenation in a loop O(n²).

| Task | Code | Time |
|---|---|---|
| length | `s.length` | O(1) |
| char at | `s[i]`, `s.at(-1)` | O(1) |
| substring | `s.slice(a, b)` | O(n) |
| search | `s.indexOf`, `s.includes` | O(n·m) |
| replace | `s.replace`, `s.replaceAll` | O(n) |
| split | `s.split(sep)` | O(n) |
| join | `arr.join(sep)` | O(n) |
| compare | `a.localeCompare(b)` | O(n) |

```ts
const s = "  Hello, World  ";

s.trim();                                       // → 'Hello, World'
s.trim().toLowerCase();                         // → 'hello, world'
"a-b-c".split("-");                             // → ['a', 'b', 'c']
"a b  c".split(/\s+/);                          // → ['a', 'b', 'c']
["a", "b"].join("-");                           // → 'a-b'
"abc".startsWith("ab");                         // → true
"abc".endsWith("bc");                           // → true
"hello".indexOf("zz");                          // → -1
"hello".replace("l", "L");                      // → 'heLlo'
"hello".replaceAll("l", "L");                   // → 'heLLo'
"7".padStart(3, "0");                           // → '007'
"ab".padEnd(4, ".");                            // → 'ab..'
"ab".repeat(3);                                 // → 'ababab'
[..."abc"].reverse().join("");                  // → 'cba'
"abc".charCodeAt(0);                            // → 97
String.fromCharCode(97);                        // → 'a'
"abc".localeCompare("abd");                     // → -1
```

### Building strings efficiently

```ts
// ✗ O(n²) — each += allocates a whole new string
function slow(n: number): string {
  let out = "";
  for (let i = 0; i < n; i++) out += "x";
  return out;
}
slow(3);                                        // → 'xxx'

// ✓ O(n) — collect, then join once
function fast(n: number): string {
  const parts: string[] = [];
  for (let i = 0; i < n; i++) parts.push("x");
  return parts.join("");
}
fast(3);                                        // → 'xxx'
```

Modern engines optimise `+=` with ropes, so the difference is smaller than the theory suggests — but `join` is never slower and always clearer at scale.

### Unicode

```ts
// .length counts UTF-16 code units, not characters
"café".length;                                  // → 4
"👋".length;                                    // → 2    a surrogate pair!
"👋"[0] === "👋";                               // → false

// spread and for..of iterate code POINTS, which is usually what you want
[..."a👋b"].length;                             // → 3
Array.from("a👋b");                             // → ['a', '👋', 'b']

// ...but still not grapheme clusters
"👨‍👩‍👧".length;                                  // → 8
[..."👨‍👩‍👧"].length;                             // → 5    ZWJ-joined family

// for real text segmentation use Intl.Segmenter
const seg = new Intl.Segmenter("en", { granularity: "grapheme" });
[...seg.segment("👨‍👩‍👧")].length;                  // → 1
```

### String algorithms

```ts
// palindrome check, O(n) time O(1) space
function isPalindrome(s: string): boolean {
  for (let i = 0, j = s.length - 1; i < j; i++, j--) {
    if (s[i] !== s[j]) return false;
  }
  return true;
}
isPalindrome("racecar");                        // → true
isPalindrome("abca");                           // → false

// anagram check via character counts, O(n)
function isAnagram(a: string, b: string): boolean {
  if (a.length !== b.length) return false;
  const counts = new Map<string, number>();
  for (const ch of a) counts.set(ch, (counts.get(ch) ?? 0) + 1);
  for (const ch of b) {
    const n = counts.get(ch);
    if (n === undefined || n === 0) return false;
    counts.set(ch, n - 1);
  }
  return true;
}
isAnagram("listen", "silent");                  // → true
isAnagram("ab", "cd");                          // → false

// longest common prefix, O(n·m)
function commonPrefix(strs: readonly string[]): string {
  if (strs.length === 0) return "";
  let p = strs[0]!;
  for (const s of strs) {
    while (!s.startsWith(p)) p = p.slice(0, -1);
    if (p === "") return "";
  }
  return p;
}
commonPrefix(["flower", "flow", "flight"]);     // → 'fl'
```

---

## Objects as records

A plain object is a string-keyed hash map with awkward edges. Use it for fixed, known shapes; use `Map` for dynamic collections.

```ts
const o: Record<string, number> = { a: 1, b: 2 };

o["a"];                                         // → 1
o["z"];                                         // → undefined
"a" in o;                                       // → true
Object.keys(o);                                 // → ['a', 'b']
Object.values(o);                               // → [1, 2]
Object.entries(o);                              // → [['a', 1], ['b', 2]]
Object.fromEntries([["x", 9]]);                 // → { x: 9 }
Object.assign({}, o, { c: 3 });                 // → { a: 1, b: 2, c: 3 }
({ ...o, b: 9 });                               // → { a: 1, b: 9 }

delete o["a"];
o;                                              // → { b: 2 }
```

### Why `Map` usually wins

| | object | `Map` |
|---|---|---|
| key types | string, symbol only | anything, including objects |
| numeric keys | coerced to strings | kept as numbers |
| size | `Object.keys(o).length` — O(n) | `m.size` — O(1) |
| iteration order | integer-like keys first, then insertion | strictly insertion |
| inherited keys | `toString`, `constructor`, … | none |
| iterate | `Object.entries(o)` allocates | `for (const [k, v] of m)` |
| JSON | direct | needs conversion |

```ts
// numeric keys become strings
const num: Record<number, string> = { 1: "a" };
Object.keys(num);                               // → ['1']   a string!

// integer-like keys sort ahead of the rest, regardless of insertion order
const ordered: Record<string, number> = {};
ordered["b"] = 1;
ordered["2"] = 2;
ordered["a"] = 3;
Object.keys(ordered);                           // → ['2', 'b', 'a']

// a Map keeps insertion order and key types exactly
const m = new Map<unknown, number>();
m.set("b", 1); m.set(2, 2); m.set("a", 3);
[...m.keys()];                                  // → ['b', 2, 'a']

// prototype pollution: an object inherits keys you did not set
const plain: Record<string, unknown> = {};
"toString" in plain;                            // → true    !
plain["toString"] === undefined;                // → false

// a null-prototype object avoids it
const bare = Object.create(null) as Record<string, unknown>;
"toString" in bare;                             // → false
```

**Common uses.** Objects for configuration, API payloads, and fixed-key records where the key set is known at compile time and you want `keyof` to mean something. `Map` for caches, indexes, counters, adjacency lists — anything where keys arrive at runtime.

---

## Map

Insertion-ordered, any key type, O(1) average for all core operations.

| Operation | Time |
|---|---|
| `get`, `set`, `has`, `delete` | O(1) average |
| `size` | O(1) |
| iteration | O(n), insertion order |

```ts
const m = new Map<string, number>([
  ["a", 1],
  ["b", 2],
]);

m.get("a");                                     // → 1
m.get("z");                                     // → undefined
m.has("a");                                     // → true
m.size;                                         // → 2
m.set("c", 3).set("d", 4);                      // chainable
m.size;                                         // → 4
m.delete("d");                                  // → true
m.delete("zz");                                 // → false

[...m.keys()];                                  // → ['a', 'b', 'c']
[...m.values()];                                // → [1, 2, 3]
[...m.entries()];                               // → [['a', 1], ['b', 2], ['c', 3]]
[...m];                                         // → [['a', 1], ['b', 2], ['c', 3]]

// the counter idiom
const counts = new Map<string, number>();
for (const ch of "aab") counts.set(ch, (counts.get(ch) ?? 0) + 1);
[...counts];                                    // → [['a', 2], ['b', 1]]

// the grouping idiom
const groups = new Map<string, string[]>();
for (const w of ["ant", "ape", "bee"]) {
  const k = w[0]!;
  groups.set(k, [...(groups.get(k) ?? []), w]);
}
[...groups];                                    // → [['a', ['ant', 'ape']], ['b', ['bee']]]

// object keys work, compared by IDENTITY not value
const k1 = { id: 1 };
const k2 = { id: 1 };
const byRef = new Map<object, string>();
byRef.set(k1, "first");
byRef.get(k1);                                  // → 'first'
byRef.get(k2);                                  // → undefined   different object

// conversions
Object.fromEntries(m);                          // → { a: 1, b: 2, c: 3 }
new Map(Object.entries({ x: 1 })).get("x");     // → 1

// JSON needs an explicit conversion — a Map serialises to {}
JSON.stringify(m);                              // → '{}'
JSON.stringify([...m]);                         // → '[["a",1],["b",2],["c",3]]'
```

**Common uses.** Frequency counters, adjacency lists, memoisation caches, index-by-id lookups, and any time you would reach for an object but the keys are data rather than schema.

---

## Set

Unique values, insertion-ordered, O(1) membership.

```ts
const s = new Set([3, 1, 3, 2]);

s.size;                                         // → 3
s.has(3);                                       // → true
s.add(4);
s.delete(1);                                    // → true
[...s];                                         // → [3, 2, 4]

// dedupe an array, order preserved
[...new Set([3, 1, 3, 2])];                     // → [3, 1, 2]

// set algebra (ES2025 methods)
const A = new Set([1, 2, 3]);
const B = new Set([3, 4]);

[...A.union(B)];                                // → [1, 2, 3, 4]
[...A.intersection(B)];                         // → [3]
[...A.difference(B)];                           // → [1, 2]
[...A.symmetricDifference(B)];                  // → [1, 2, 4]
A.isSubsetOf(new Set([1, 2, 3, 4]));            // → true
A.isSupersetOf(new Set([1]));                   // → true
A.isDisjointFrom(new Set([9]));                 // → true

// the portable equivalents, if you cannot rely on ES2025
const union = new Set([...A, ...B]);
const inter = new Set([...A].filter((v) => B.has(v)));
const diff = new Set([...A].filter((v) => !B.has(v)));
[...union];                                     // → [1, 2, 3, 4]
[...inter];                                     // → [3]
[...diff];                                      // → [1, 2]

// membership: O(1) on a Set, O(n) on an array
const big = new Set(Array.from({ length: 1000 }, (_, i) => i));
big.has(999);                                   // → true
```

**Common uses.** Visited-node tracking in graph traversal, deduplication, fast "is this allowed" checks against a whitelist, and detecting duplicates (`xs.length !== new Set(xs).size`).

```ts
const hasDupes = <T>(xs: readonly T[]) => xs.length !== new Set(xs).size;
hasDupes([1, 2, 1]);                            // → true
hasDupes([1, 2, 3]);                            // → false
```

---

## WeakMap and WeakSet

Keys must be objects, and are held *weakly* — an entry does not stop its key being garbage collected.

```ts
const cache = new WeakMap<object, number>();
const key = { id: 1 };

cache.set(key, 42);
cache.get(key);                                 // → 42
cache.has(key);                                 // → true
cache.delete(key);                              // → true

// no size, no iteration, no clear — you cannot observe what is still alive
// ✗ cache.size
//   Property 'size' does not exist on type 'WeakMap<object, number>'.
// ✗ [...cache]
//   Type 'WeakMap<object, number>' must have a '[Symbol.iterator]()' method.

// ✗ cache.set("string", 1)
//   Argument of type 'string' is not assignable to parameter of type 'object'.
```

**Common uses.** Attaching metadata to objects you do not own (DOM nodes, framework instances) without leaking them; per-instance private state; memoising a function keyed on an object argument. If you used a `Map` for any of these, every key would be retained forever.

---

## Hash maps from scratch

You will not write one in production — `Map` is faster and correct. You will be asked how one works.

```ts
class HashMap<K, V> {
  #buckets: Array<Array<[K, V]>>;
  #size = 0;
  #capacity: number;

  constructor(capacity = 8) {
    this.#capacity = capacity;
    this.#buckets = Array.from({ length: capacity }, () => []);
  }

  // FNV-1a over the key's string form
  #hash(key: K): number {
    const s = String(key);
    let h = 2166136261;
    for (let i = 0; i < s.length; i++) {
      h ^= s.charCodeAt(i);
      h = Math.imul(h, 16777619);
    }
    return (h >>> 0) % this.#capacity;
  }

  set(key: K, value: V): this {
    const bucket = this.#buckets[this.#hash(key)]!;
    const found = bucket.find((e) => e[0] === key);
    if (found) { found[1] = value; return this; }
    bucket.push([key, value]);
    this.#size++;
    if (this.#size / this.#capacity > 0.75) this.#resize();
    return this;
  }

  get(key: K): V | undefined {
    return this.#buckets[this.#hash(key)]!.find((e) => e[0] === key)?.[1];
  }

  has(key: K): boolean {
    return this.#buckets[this.#hash(key)]!.some((e) => e[0] === key);
  }

  delete(key: K): boolean {
    const bucket = this.#buckets[this.#hash(key)]!;
    const i = bucket.findIndex((e) => e[0] === key);
    if (i === -1) return false;
    bucket.splice(i, 1);
    this.#size--;
    return true;
  }

  #resize(): void {
    const old = this.#buckets;
    this.#capacity *= 2;
    this.#buckets = Array.from({ length: this.#capacity }, () => []);
    this.#size = 0;
    for (const bucket of old) for (const [k, v] of bucket) this.set(k, v);
  }

  get size(): number { return this.#size; }

  *entries(): IterableIterator<[K, V]> {
    for (const bucket of this.#buckets) yield* bucket;
  }
}

const hm = new HashMap<string, number>();
hm.set("a", 1).set("b", 2);
hm.get("a");                                    // → 1
hm.get("zz");                                   // → undefined
hm.has("b");                                    // → true
hm.size;                                        // → 2
hm.delete("a");                                 // → true
hm.size;                                        // → 1

// growth triggers a rehash, and everything survives
const big = new HashMap<number, number>(2);
for (let i = 0; i < 50; i++) big.set(i, i * i);
big.size;                                       // → 50
big.get(49);                                    // → 2401
```

| Concept | Detail |
|---|---|
| hash function | maps a key to a bucket index; must be fast and well-distributed |
| collision | two keys landing in one bucket |
| separate chaining | each bucket is a list — what this implementation does |
| open addressing | probe the next free slot instead; better cache locality |
| load factor | `size / capacity`; resize past ~0.75 or lookups degrade |
| resize cost | O(n), but amortised O(1) per insert |
| worst case | O(n) when every key collides |

---

## Typed arrays

Fixed-length, fixed-type, contiguous binary buffers. Meaningfully faster and far more compact than `number[]` for large numeric data.

```ts
const i32 = new Int32Array(4);
i32[0] = 42;
i32[0];                                         // → 42
i32.length;                                     // → 4
[...i32];                                       // → [42, 0, 0, 0]

Array.from(Int32Array.from([3, 1, 2]).sort());  // → [1, 2, 3]

// out of range wraps rather than throwing
const u8 = new Uint8Array(1);
u8[0] = 256;
u8[0];                                          // → 0
u8[0] = -1;
u8[0];                                          // → 255

// clamped variants saturate instead
const clamped = new Uint8ClampedArray(1);
clamped[0] = 300;
clamped[0];                                     // → 255

// out-of-bounds writes are silently dropped, not appended
const fixed = new Int32Array(2);
fixed[5] = 1;
fixed.length;                                   // → 2

// several views over one buffer
const buf = new ArrayBuffer(8);
const asI32 = new Int32Array(buf);
const asU8 = new Uint8Array(buf);
asI32[0] = 1;
asU8[0];                                        // → 1
asU8.length;                                    // → 8
```

| Type | Bytes | Range |
|---|---|---|
| `Int8Array` / `Uint8Array` | 1 | −128..127 / 0..255 |
| `Int16Array` / `Uint16Array` | 2 | ±32k / 0..65535 |
| `Int32Array` / `Uint32Array` | 4 | ±2.1b / 0..4.29b |
| `Float32Array` / `Float64Array` | 4 / 8 | IEEE 754 |
| `BigInt64Array` / `BigUint64Array` | 8 | 64-bit integers |

**Common uses.** Pixel buffers, audio samples, geometry data for WebGL, large bitsets, and any hot numeric loop where `number[]` boxing shows up in a profile.

---

## Stack

LIFO. The last thing in is the first thing out.

```ts
// an array IS a stack — push/pop are both O(1) at the end
const stack: number[] = [];
stack.push(1);
stack.push(2);
stack[stack.length - 1];                        // → 2      peek
stack.pop();                                    // → 2
stack.length;                                   // → 1

// a typed wrapper, when you want the API to forbid random access
class Stack<T> {
  #items: T[] = [];

  push(v: T): this { this.#items.push(v); return this; }         // O(1)
  pop(): T | undefined { return this.#items.pop(); }             // O(1)
  peek(): T | undefined { return this.#items[this.#items.length - 1]; }
  get size(): number { return this.#items.length; }
  get isEmpty(): boolean { return this.#items.length === 0; }
  *[Symbol.iterator](): Iterator<T> { for (let i = this.#items.length - 1; i >= 0; i--) yield this.#items[i]!; }
}

const s = new Stack<string>();
s.push("a").push("b");
s.peek();                                       // → 'b'
s.size;                                         // → 2
s.pop();                                        // → 'b'
s.isEmpty;                                      // → false
[...s];                                         // → ['a']
new Stack<number>().pop();                      // → undefined
```

### Common uses

```ts
// 1. balanced brackets
function isBalanced(s: string): boolean {
  const pairs: Record<string, string> = { ")": "(", "]": "[", "}": "{" };
  const st: string[] = [];
  for (const ch of s) {
    if (ch === "(" || ch === "[" || ch === "{") st.push(ch);
    else if (ch in pairs) {
      if (st.pop() !== pairs[ch]) return false;
    }
  }
  return st.length === 0;
}
isBalanced("{[()]}");                           // → true
isBalanced("{[(])}");                           // → false
isBalanced("(");                                // → false

// 2. undo history
class History<T> {
  #undo: T[] = [];
  #redo: T[] = [];
  constructor(private state: T) {}
  commit(next: T): void { this.#undo.push(this.state); this.#redo.length = 0; this.state = next; }
  undo(): T { const p = this.#undo.pop(); if (p !== undefined) { this.#redo.push(this.state); this.state = p; } return this.state; }
  redo(): T { const n = this.#redo.pop(); if (n !== undefined) { this.#undo.push(this.state); this.state = n; } return this.state; }
  get current(): T { return this.state; }
}
const h = new History("v1");
h.commit("v2"); h.commit("v3");
h.current;                                      // → 'v3'
h.undo();                                       // → 'v2'
h.undo();                                       // → 'v1'
h.redo();                                       // → 'v2'

// 3. converting recursion to iteration — see the iterative DFS below

// 4. evaluating reverse Polish notation
function evalRPN(tokens: readonly string[]): number {
  const st: number[] = [];
  for (const t of tokens) {
    if (/^-?\d+$/.test(t)) { st.push(Number(t)); continue; }
    const b = st.pop()!, a = st.pop()!;
    st.push(t === "+" ? a + b : t === "-" ? a - b : t === "*" ? a * b : Math.trunc(a / b));
  }
  return st[0]!;
}
evalRPN(["2", "3", "+", "4", "*"]);             // → 20

// 5. monotonic stack: next greater element, O(n)
function nextGreater(xs: readonly number[]): number[] {
  const out = new Array<number>(xs.length).fill(-1);
  const st: number[] = [];                       // holds INDICES
  for (let i = 0; i < xs.length; i++) {
    while (st.length && xs[st[st.length - 1]!]! < xs[i]!) out[st.pop()!] = xs[i]!;
    st.push(i);
  }
  return out;
}
nextGreater([2, 1, 2, 4, 3]);                   // → [4, 2, 4, -1, -1]
```

---

## Queue

FIFO. The first thing in is the first thing out.

```ts
// ✗ the naive version: shift() is O(n), so draining n items is O(n²)
const bad: number[] = [1, 2, 3];
bad.shift();                                    // → 1     but O(n)

// ✓ a head pointer makes dequeue O(1), at the cost of holding the array
class Queue<T> {
  #items: T[] = [];
  #head = 0;

  enqueue(v: T): this { this.#items.push(v); return this; }      // O(1)

  dequeue(): T | undefined {                                     // O(1)
    if (this.#head >= this.#items.length) return undefined;
    const v = this.#items[this.#head];
    this.#items[this.#head] = undefined as T;                    // release the reference
    this.#head++;
    // compact once the dead prefix dominates, amortised O(1)
    if (this.#head > 32 && this.#head * 2 >= this.#items.length) {
      this.#items = this.#items.slice(this.#head);
      this.#head = 0;
    }
    return v;
  }

  peek(): T | undefined { return this.#items[this.#head]; }
  get size(): number { return this.#items.length - this.#head; }
  get isEmpty(): boolean { return this.size === 0; }
  *[Symbol.iterator](): Iterator<T> { for (let i = this.#head; i < this.#items.length; i++) yield this.#items[i]!; }
}

const q = new Queue<string>();
q.enqueue("a").enqueue("b").enqueue("c");
q.peek();                                       // → 'a'
q.dequeue();                                    // → 'a'
q.size;                                         // → 2
[...q];                                         // → ['b', 'c']
new Queue<number>().dequeue();                  // → undefined

// it stays O(1) under load
const load = new Queue<number>();
for (let i = 0; i < 10000; i++) load.enqueue(i);
for (let i = 0; i < 9999; i++) load.dequeue();
load.dequeue();                                 // → 9999
load.isEmpty;                                   // → true
```

### Circular buffer (ring buffer)

Fixed capacity, O(1) both ends, no allocation after construction.

```ts
class RingBuffer<T> {
  #buf: (T | undefined)[];
  #head = 0;
  #tail = 0;
  #size = 0;

  constructor(readonly capacity: number) {
    this.#buf = new Array<T | undefined>(capacity);
  }

  push(v: T): boolean {                          // O(1), false when full
    if (this.#size === this.capacity) return false;
    this.#buf[this.#tail] = v;
    this.#tail = (this.#tail + 1) % this.capacity;
    this.#size++;
    return true;
  }

  overwrite(v: T): void {                        // push, dropping the oldest
    if (this.#size === this.capacity) this.shift();
    this.push(v);
  }

  shift(): T | undefined {                       // O(1)
    if (this.#size === 0) return undefined;
    const v = this.#buf[this.#head];
    this.#buf[this.#head] = undefined;
    this.#head = (this.#head + 1) % this.capacity;
    this.#size--;
    return v;
  }

  get size(): number { return this.#size; }
  *[Symbol.iterator](): Iterator<T> {
    for (let i = 0; i < this.#size; i++) yield this.#buf[(this.#head + i) % this.capacity]!;
  }
}

const rb = new RingBuffer<number>(3);
rb.push(1); rb.push(2); rb.push(3);
rb.push(4);                                     // → false   full
[...rb];                                        // → [1, 2, 3]
rb.shift();                                     // → 1
rb.push(4);                                     // → true    wraps around
[...rb];                                        // → [2, 3, 4]

rb.overwrite(5);
[...rb];                                        // → [3, 4, 5]
```

**Common uses.** `Queue` for BFS, task scheduling, and rate-limited work. `RingBuffer` for the last N log lines, audio/streaming buffers, and moving-window statistics where the capacity is known.

---

## Deque

Double-ended queue: O(1) at both ends.

```ts
class Deque<T> {
  #front: T[] = [];       // stored reversed
  #back: T[] = [];

  pushFront(v: T): this { this.#front.push(v); return this; }
  pushBack(v: T): this { this.#back.push(v); return this; }

  popFront(): T | undefined {
    if (this.#front.length) return this.#front.pop();
    if (!this.#back.length) return undefined;
    // move the back into the front, reversed, amortised O(1)
    this.#front = this.#back.reverse();
    this.#back = [];
    return this.#front.pop();
  }

  popBack(): T | undefined {
    if (this.#back.length) return this.#back.pop();
    if (!this.#front.length) return undefined;
    this.#back = this.#front.reverse();
    this.#front = [];
    return this.#back.pop();
  }

  get size(): number { return this.#front.length + this.#back.length; }
  *[Symbol.iterator](): Iterator<T> {
    for (let i = this.#front.length - 1; i >= 0; i--) yield this.#front[i]!;
    yield* this.#back;
  }
}

const dq = new Deque<number>();
dq.pushBack(2).pushBack(3).pushFront(1);
[...dq];                                        // → [1, 2, 3]
dq.popFront();                                  // → 1
dq.popBack();                                   // → 3
dq.size;                                        // → 1
```

```ts
// the headline use: sliding window maximum in O(n)
function windowMax(xs: readonly number[], k: number): number[] {
  const out: number[] = [];
  const idx: number[] = [];                      // indices, values decreasing
  for (let i = 0; i < xs.length; i++) {
    if (idx.length && idx[0]! <= i - k) idx.shift();
    while (idx.length && xs[idx[idx.length - 1]!]! <= xs[i]!) idx.pop();
    idx.push(i);
    if (i >= k - 1) out.push(xs[idx[0]!]!);
  }
  return out;
}
windowMax([1, 3, -1, -3, 5, 3, 6, 7], 3);       // → [3, 3, 5, 5, 6, 7]
```

---

## Singly linked list

A chain of nodes, each holding a value and a pointer to the next.

| Operation | Time |
|---|---|
| prepend | O(1) |
| append (with a tail pointer) | O(1) |
| access / search by index | O(n) |
| insert / delete after a known node | O(1) |
| insert / delete at index | O(n) |

```ts
class ListNode<T> {
  next: ListNode<T> | null = null;
  constructor(public value: T) {}
}

class LinkedList<T> {
  #head: ListNode<T> | null = null;
  #tail: ListNode<T> | null = null;
  #size = 0;

  get size(): number { return this.#size; }
  get head(): ListNode<T> | null { return this.#head; }

  prepend(v: T): this {                                        // O(1)
    const node = new ListNode(v);
    node.next = this.#head;
    this.#head = node;
    this.#tail ??= node;
    this.#size++;
    return this;
  }

  append(v: T): this {                                         // O(1)
    const node = new ListNode(v);
    if (this.#tail) { this.#tail.next = node; this.#tail = node; }
    else { this.#head = this.#tail = node; }
    this.#size++;
    return this;
  }

  get(i: number): T | undefined {                              // O(n)
    if (i < 0 || i >= this.#size) return undefined;
    let cur = this.#head;
    for (let k = 0; k < i; k++) cur = cur!.next;
    return cur!.value;
  }

  insertAt(i: number, v: T): boolean {                         // O(n)
    if (i < 0 || i > this.#size) return false;
    if (i === 0) { this.prepend(v); return true; }
    if (i === this.#size) { this.append(v); return true; }
    let prev = this.#head!;
    for (let k = 0; k < i - 1; k++) prev = prev.next!;
    const node = new ListNode(v);
    node.next = prev.next;
    prev.next = node;
    this.#size++;
    return true;
  }

  removeAt(i: number): T | undefined {                         // O(n)
    if (i < 0 || i >= this.#size) return undefined;
    if (i === 0) {
      const node = this.#head!;
      this.#head = node.next;
      if (!this.#head) this.#tail = null;
      this.#size--;
      return node.value;
    }
    let prev = this.#head!;
    for (let k = 0; k < i - 1; k++) prev = prev.next!;
    const node = prev.next!;
    prev.next = node.next;
    if (node === this.#tail) this.#tail = prev;
    this.#size--;
    return node.value;
  }

  indexOf(v: T): number {                                      // O(n)
    let i = 0;
    for (let cur = this.#head; cur; cur = cur.next, i++) if (cur.value === v) return i;
    return -1;
  }

  reverse(): this {                                            // O(n), O(1) space
    let prev: ListNode<T> | null = null;
    let cur = this.#head;
    this.#tail = this.#head;
    while (cur) {
      const next: ListNode<T> | null = cur.next;
      cur.next = prev;
      prev = cur;
      cur = next;
    }
    this.#head = prev;
    return this;
  }

  toArray(): T[] { return [...this]; }

  *[Symbol.iterator](): Iterator<T> {
    for (let cur = this.#head; cur; cur = cur.next) yield cur.value;
  }
}

const ll = new LinkedList<number>();
ll.append(1).append(2).append(3);
ll.toArray();                                   // → [1, 2, 3]
ll.size;                                        // → 3
ll.prepend(0);
ll.toArray();                                   // → [0, 1, 2, 3]
ll.get(2);                                      // → 2
ll.get(99);                                     // → undefined
ll.indexOf(3);                                  // → 3
ll.indexOf(99);                                 // → -1
ll.insertAt(1, 9);                              // → true
ll.toArray();                                   // → [0, 9, 1, 2, 3]
ll.removeAt(1);                                 // → 9
ll.toArray();                                   // → [0, 1, 2, 3]
ll.reverse().toArray();                         // → [3, 2, 1, 0]
```

### Classic problems

```ts
// build a bare chain for the algorithms below
function chain(xs: readonly number[]): ListNode<number> | null {
  let head: ListNode<number> | null = null;
  for (let i = xs.length - 1; i >= 0; i--) {
    const n = new ListNode(xs[i]!);
    n.next = head;
    head = n;
  }
  return head;
}
function toArr(head: ListNode<number> | null): number[] {
  const out: number[] = [];
  for (let c = head; c; c = c.next) out.push(c.value);
  return out;
}

// reverse, iteratively
function reverseList(head: ListNode<number> | null): ListNode<number> | null {
  let prev: ListNode<number> | null = null;
  while (head) { const nx: ListNode<number> | null = head.next; head.next = prev; prev = head; head = nx; }
  return prev;
}
toArr(reverseList(chain([1, 2, 3])));           // → [3, 2, 1]

// middle node — the two-pointer (tortoise and hare) trick
function middle(head: ListNode<number> | null): number | undefined {
  let slow = head, fast = head;
  while (fast?.next) { slow = slow!.next; fast = fast.next.next; }
  return slow?.value;
}
middle(chain([1, 2, 3, 4, 5]));                 // → 3
middle(chain([1, 2, 3, 4]));                    // → 3    the second middle

// cycle detection — Floyd's algorithm, O(n) time O(1) space
function hasCycle(head: ListNode<number> | null): boolean {
  let slow = head, fast = head;
  while (fast?.next) {
    slow = slow!.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }
  return false;
}
hasCycle(chain([1, 2, 3]));                     // → false

const cyc = chain([1, 2, 3])!;
cyc.next!.next!.next = cyc;                     // tie the tail to the head
hasCycle(cyc);                                  // → true

// nth from the end, one pass
function nthFromEnd(head: ListNode<number> | null, n: number): number | undefined {
  let lead = head;
  for (let i = 0; i < n; i++) { if (!lead) return undefined; lead = lead.next; }
  let trail = head;
  while (lead) { lead = lead.next; trail = trail!.next; }
  return trail?.value;
}
nthFromEnd(chain([1, 2, 3, 4, 5]), 2);          // → 4

// merge two sorted lists
function mergeSorted(a: ListNode<number> | null, b: ListNode<number> | null): ListNode<number> | null {
  const dummy = new ListNode(0);
  let tail = dummy;
  while (a && b) {
    if (a.value <= b.value) { tail.next = a; a = a.next; }
    else { tail.next = b; b = b.next; }
    tail = tail.next;
  }
  tail.next = a ?? b;
  return dummy.next;
}
toArr(mergeSorted(chain([1, 3, 5]), chain([2, 4])));   // → [1, 2, 3, 4, 5]
```

The `dummy` head is the standard trick: it removes the special case for "the result list is still empty", so there is one code path instead of two.

---

## Doubly linked list

Each node points both ways, so deletion given a node is O(1) — the property that makes an LRU cache work.

```ts
class DNode<T> {
  prev: DNode<T> | null = null;
  next: DNode<T> | null = null;
  constructor(public value: T) {}
}

class DoublyLinkedList<T> {
  #head: DNode<T> | null = null;
  #tail: DNode<T> | null = null;
  #size = 0;

  get size(): number { return this.#size; }

  pushBack(v: T): DNode<T> {                                   // O(1)
    const node = new DNode(v);
    if (this.#tail) { node.prev = this.#tail; this.#tail.next = node; this.#tail = node; }
    else { this.#head = this.#tail = node; }
    this.#size++;
    return node;
  }

  pushFront(v: T): DNode<T> {                                  // O(1)
    const node = new DNode(v);
    if (this.#head) { node.next = this.#head; this.#head.prev = node; this.#head = node; }
    else { this.#head = this.#tail = node; }
    this.#size++;
    return node;
  }

  popFront(): T | undefined {                                  // O(1)
    if (!this.#head) return undefined;
    const node = this.#head;
    this.remove(node);
    return node.value;
  }

  popBack(): T | undefined {                                   // O(1)
    if (!this.#tail) return undefined;
    const node = this.#tail;
    this.remove(node);
    return node.value;
  }

  // the payoff: removing a KNOWN node costs O(1), no search
  remove(node: DNode<T>): void {
    if (node.prev) node.prev.next = node.next; else this.#head = node.next;
    if (node.next) node.next.prev = node.prev; else this.#tail = node.prev;
    node.prev = node.next = null;
    this.#size--;
  }

  toArray(): T[] { return [...this]; }
  toArrayReversed(): T[] {
    const out: T[] = [];
    for (let c = this.#tail; c; c = c.prev) out.push(c.value);
    return out;
  }

  *[Symbol.iterator](): Iterator<T> {
    for (let c = this.#head; c; c = c.next) yield c.value;
  }
}

const dll = new DoublyLinkedList<string>();
dll.pushBack("b");
const nodeC = dll.pushBack("c");
dll.pushFront("a");
dll.toArray();                                  // → ['a', 'b', 'c']
dll.toArrayReversed();                          // → ['c', 'b', 'a']
dll.remove(nodeC);                              // O(1) — no traversal
dll.toArray();                                  // → ['a', 'b']
dll.popFront();                                 // → 'a'
dll.popBack();                                  // → 'b'
dll.size;                                       // → 0
dll.popBack();                                  // → undefined
```

| | singly | doubly |
|---|---|---|
| memory per node | 1 pointer | 2 pointers |
| delete given a node | O(n) — need the predecessor | **O(1)** |
| reverse iteration | no | yes |
| use for | simple stacks/queues, memory-tight | LRU caches, editors, browser history |

---

## Circular linked list

The tail points back to the head, so iteration wraps forever. Useful whenever "next" must never run out.

```ts
class CircularList<T> {
  #tail: DNode<T> | null = null;   // keeping the TAIL gives O(1) at both ends
  #size = 0;

  get size(): number { return this.#size; }

  push(v: T): this {                                           // O(1)
    const node = new DNode(v);
    if (!this.#tail) { node.next = node; node.prev = node; this.#tail = node; }
    else {
      const head = this.#tail.next!;
      node.prev = this.#tail; node.next = head;
      this.#tail.next = node; head.prev = node;
      this.#tail = node;
    }
    this.#size++;
    return this;
  }

  // walk n steps forward from the head, wrapping
  at(n: number): T | undefined {
    if (!this.#tail) return undefined;
    let cur = this.#tail.next!;
    const steps = ((n % this.#size) + this.#size) % this.#size;
    for (let i = 0; i < steps; i++) cur = cur.next!;
    return cur.value;
  }

  toArray(): T[] {
    if (!this.#tail) return [];
    const out: T[] = [];
    let cur = this.#tail.next!;
    for (let i = 0; i < this.#size; i++) { out.push(cur.value); cur = cur.next!; }
    return out;
  }
}

const cl = new CircularList<string>();
cl.push("a").push("b").push("c");
cl.toArray();                                   // → ['a', 'b', 'c']
cl.at(0);                                       // → 'a'
cl.at(3);                                       // → 'a'    wrapped
cl.at(4);                                       // → 'b'
cl.at(-1);                                      // → 'c'
cl.size;                                        // → 3
```

```ts
// the Josephus problem: every k-th person is eliminated until one remains
function josephus(n: number, k: number): number {
  let survivor = 0;                              // 0-indexed
  for (let i = 2; i <= n; i++) survivor = (survivor + k) % i;
  return survivor + 1;
}
josephus(7, 3);                                 // → 4
josephus(5, 2);                                 // → 3
```

**Common uses.** Round-robin scheduling, turn order in a game, a repeating carousel, and any fixed rotation where you want `next` without a bounds check.

---

## LRU cache

Bounded by recency: `get` and `put` are both O(1). The classic combination of a hash map (for lookup) and a doubly linked list (for ordering).

```ts
class LRUCache<K, V> {
  #map = new Map<K, V>();
  constructor(readonly capacity: number) {}

  get(key: K): V | undefined {                                 // O(1)
    if (!this.#map.has(key)) return undefined;
    const v = this.#map.get(key)!;
    this.#map.delete(key);                       // re-insert to mark as newest
    this.#map.set(key, v);
    return v;
  }

  put(key: K, value: V): void {                                // O(1)
    if (this.#map.has(key)) this.#map.delete(key);
    else if (this.#map.size >= this.capacity) {
      const oldest = this.#map.keys().next().value as K;
      this.#map.delete(oldest);
    }
    this.#map.set(key, value);
  }

  get size(): number { return this.#map.size; }
  keys(): K[] { return [...this.#map.keys()]; }                 // oldest first
}

const lru = new LRUCache<string, number>(3);
lru.put("a", 1); lru.put("b", 2); lru.put("c", 3);
lru.keys();                                     // → ['a', 'b', 'c']
lru.get("a");                                   // → 1      'a' becomes newest
lru.keys();                                     // → ['b', 'c', 'a']
lru.put("d", 4);                                // evicts 'b', the oldest
lru.keys();                                     // → ['c', 'a', 'd']
lru.get("b");                                   // → undefined
lru.size;                                       // → 3
```

This version leans on `Map`'s guaranteed insertion order, which makes it about fifteen lines. The interview answer usually wants the explicit version:

```ts
class LRUExplicit<K, V> {
  #map = new Map<K, DNode<[K, V]>>();
  #list = new DoublyLinkedList<[K, V]>();

  constructor(readonly capacity: number) {}

  get(key: K): V | undefined {
    const node = this.#map.get(key);
    if (!node) return undefined;
    this.#list.remove(node);                     // O(1), thanks to prev pointers
    this.#map.set(key, this.#list.pushBack(node.value));
    return node.value[1];
  }

  put(key: K, value: V): void {
    const existing = this.#map.get(key);
    if (existing) this.#list.remove(existing);
    else if (this.#map.size >= this.capacity) {
      const evicted = this.#list.popFront();
      if (evicted) this.#map.delete(evicted[0]);
    }
    this.#map.set(key, this.#list.pushBack([key, value]));
  }

  keys(): K[] { return this.#list.toArray().map(([k]) => k); }
}

const lru2 = new LRUExplicit<string, number>(2);
lru2.put("a", 1); lru2.put("b", 2);
lru2.get("a");                                  // → 1
lru2.put("c", 3);                               // evicts 'b'
lru2.keys();                                    // → ['a', 'c']
lru2.get("b");                                  // → undefined
```

The doubly linked list is what makes eviction O(1): the map gives you the node, and the node's `prev`/`next` let you unlink it without a search.

---

## Binary heap

A complete binary tree stored in an array, where every parent beats its children. Gives you the minimum (or maximum) in O(1) and insert/extract in O(log n).

```ts
// index arithmetic — no pointers needed
//   parent(i) = (i - 1) >> 1
//   left(i)   = 2i + 1
//   right(i)  = 2i + 2

class MinHeap<T> {
  #items: T[] = [];
  constructor(private readonly compare: (a: T, b: T) => number = (a, b) => (a < b ? -1 : a > b ? 1 : 0)) {}

  get size(): number { return this.#items.length; }
  peek(): T | undefined { return this.#items[0]; }              // O(1)

  push(v: T): this {                                            // O(log n)
    this.#items.push(v);
    this.#up(this.#items.length - 1);
    return this;
  }

  pop(): T | undefined {                                        // O(log n)
    if (this.#items.length === 0) return undefined;
    const top = this.#items[0]!;
    const last = this.#items.pop()!;
    if (this.#items.length) { this.#items[0] = last; this.#down(0); }
    return top;
  }

  #up(i: number): void {
    while (i > 0) {
      const p = (i - 1) >> 1;
      if (this.compare(this.#items[i]!, this.#items[p]!) >= 0) break;
      [this.#items[i], this.#items[p]] = [this.#items[p]!, this.#items[i]!];
      i = p;
    }
  }

  #down(i: number): void {
    const n = this.#items.length;
    for (;;) {
      const l = 2 * i + 1, r = l + 1;
      let best = i;
      if (l < n && this.compare(this.#items[l]!, this.#items[best]!) < 0) best = l;
      if (r < n && this.compare(this.#items[r]!, this.#items[best]!) < 0) best = r;
      if (best === i) break;
      [this.#items[i], this.#items[best]] = [this.#items[best]!, this.#items[i]!];
      i = best;
    }
  }

  // build from an array in O(n), not O(n log n)
  static from<T>(xs: readonly T[], compare?: (a: T, b: T) => number): MinHeap<T> {
    const h = new MinHeap<T>(compare);
    h.#items = [...xs];
    for (let i = (h.#items.length >> 1) - 1; i >= 0; i--) h.#down(i);
    return h;
  }

  drain(): T[] {
    const out: T[] = [];
    for (let v = this.pop(); v !== undefined; v = this.pop()) out.push(v);
    return out;
  }
}

const h = new MinHeap<number>();
h.push(5).push(1).push(3);
h.peek();                                       // → 1
h.size;                                         // → 3
h.pop();                                        // → 1
h.pop();                                        // → 3
h.pop();                                        // → 5
h.pop();                                        // → undefined

MinHeap.from([5, 2, 8, 1, 9]).drain();          // → [1, 2, 5, 8, 9]

// a max-heap is the same structure with the comparator inverted
const maxH = new MinHeap<number>((a, b) => b - a);
maxH.push(5).push(1).push(9);
maxH.pop();                                     // → 9

// heaps of objects: compare on a field
type Task = { name: string; priority: number };
const tasks = new MinHeap<Task>((a, b) => a.priority - b.priority);
tasks.push({ name: "low", priority: 5 });
tasks.push({ name: "urgent", priority: 1 });
tasks.pop()?.name;                              // → 'urgent'
```

| Operation | Time |
|---|---|
| `peek` | O(1) |
| `push` | O(log n) |
| `pop` | O(log n) |
| build from array | O(n) |
| search for an arbitrary value | O(n) — a heap is not searchable |

### Heap applications

```ts
// k largest elements, O(n log k) with a size-k min-heap
function kLargest(xs: readonly number[], k: number): number[] {
  const heap = new MinHeap<number>((a, b) => a - b);
  for (const v of xs) {
    heap.push(v);
    if (heap.size > k) heap.pop();               // drop the smallest
  }
  return heap.drain();
}
kLargest([3, 1, 5, 12, 2, 11], 3);              // → [5, 11, 12]

// heapsort, O(n log n) time O(1) extra if done in place
function heapsort(xs: readonly number[]): number[] {
  return MinHeap.from(xs, (a, b) => a - b).drain();
}
heapsort([4, 2, 7, 1]);                         // → [1, 2, 4, 7]

// merge k sorted lists, O(N log k)
function mergeK(lists: readonly (readonly number[])[]): number[] {
  type Entry = { value: number; list: number; idx: number };
  const heap = new MinHeap<Entry>((a, b) => a.value - b.value);
  lists.forEach((l, i) => { if (l.length) heap.push({ value: l[0]!, list: i, idx: 0 }); });
  const out: number[] = [];
  while (heap.size) {
    const e = heap.pop()!;
    out.push(e.value);
    const next = lists[e.list]![e.idx + 1];
    if (next !== undefined) heap.push({ value: next, list: e.list, idx: e.idx + 1 });
  }
  return out;
}
mergeK([[1, 4, 7], [2, 5], [3, 6, 8]]);         // → [1, 2, 3, 4, 5, 6, 7, 8]

// running median with two heaps
class MedianFinder {
  #lo = new MinHeap<number>((a, b) => b - a);    // max-heap, lower half
  #hi = new MinHeap<number>((a, b) => a - b);    // min-heap, upper half

  add(v: number): void {
    this.#lo.push(v);
    this.#hi.push(this.#lo.pop()!);              // funnel through to keep order
    if (this.#hi.size > this.#lo.size) this.#lo.push(this.#hi.pop()!);
  }

  median(): number {
    if (this.#lo.size === 0) return NaN;
    return this.#lo.size > this.#hi.size
      ? this.#lo.peek()!
      : (this.#lo.peek()! + this.#hi.peek()!) / 2;
  }
}
const mf = new MedianFinder();
mf.add(1);
mf.median();                                    // → 1
mf.add(3);
mf.median();                                    // → 2
mf.add(2);
mf.median();                                    // → 2
```

---

## Priority queue

A heap with a task-shaped API. Same complexity, clearer intent.

```ts
class PriorityQueue<T> {
  #heap: MinHeap<{ value: T; priority: number; seq: number }>;
  #seq = 0;

  constructor() {
    // ties broken by insertion order, which makes the queue STABLE
    this.#heap = new MinHeap((a, b) => a.priority - b.priority || a.seq - b.seq);
  }

  enqueue(value: T, priority: number): this {
    this.#heap.push({ value, priority, seq: this.#seq++ });
    return this;
  }

  dequeue(): T | undefined { return this.#heap.pop()?.value; }
  peek(): T | undefined { return this.#heap.peek()?.value; }
  get size(): number { return this.#heap.size; }
}

const pq = new PriorityQueue<string>();
pq.enqueue("write tests", 2)
  .enqueue("fix outage", 0)
  .enqueue("reply to email", 2)
  .enqueue("review PR", 1);

pq.dequeue();                                   // → 'fix outage'
pq.dequeue();                                   // → 'review PR'
pq.dequeue();                                   // → 'write tests'    inserted first
pq.dequeue();                                   // → 'reply to email'
pq.size;                                        // → 0
```

Without the `seq` tiebreaker a heap is **not** stable: equal-priority items come out in arbitrary order. Adding a monotonic counter costs nothing and removes a whole class of confusing behaviour.

**Common uses.** Dijkstra and A*, task schedulers, event simulation, bandwidth shaping, and any "do the most important thing next" loop.

---

## Binary trees

```ts
class TreeNode<T> {
  left: TreeNode<T> | null = null;
  right: TreeNode<T> | null = null;
  constructor(public value: T) {}
}

// build a small tree:
//        1
//       / \
//      2   3
//     / \
//    4   5
function sample(): TreeNode<number> {
  const root = new TreeNode(1);
  root.left = new TreeNode(2);
  root.right = new TreeNode(3);
  root.left.left = new TreeNode(4);
  root.left.right = new TreeNode(5);
  return root;
}

// build from a level-order array, null for a missing child
function fromLevelOrder(xs: readonly (number | null)[]): TreeNode<number> | null {
  if (!xs.length || xs[0] === null) return null;
  const root = new TreeNode(xs[0]!);
  const q: TreeNode<number>[] = [root];
  let i = 1;
  while (q.length && i < xs.length) {
    const node = q.shift()!;
    const l = xs[i++];
    if (l !== null && l !== undefined) { node.left = new TreeNode(l); q.push(node.left); }
    const r = xs[i++];
    if (r !== null && r !== undefined) { node.right = new TreeNode(r); q.push(node.right); }
  }
  return root;
}
```

### Measurements

```ts
function height(node: TreeNode<number> | null): number {        // O(n)
  return node ? 1 + Math.max(height(node.left), height(node.right)) : 0;
}
height(sample());                               // → 3
height(null);                                   // → 0

function countNodes(node: TreeNode<number> | null): number {
  return node ? 1 + countNodes(node.left) + countNodes(node.right) : 0;
}
countNodes(sample());                           // → 5

function countLeaves(node: TreeNode<number> | null): number {
  if (!node) return 0;
  if (!node.left && !node.right) return 1;
  return countLeaves(node.left) + countLeaves(node.right);
}
countLeaves(sample());                          // → 3

// balanced: every subtree's children differ in height by at most 1
function isBalanced(node: TreeNode<number> | null): boolean {
  function check(n: TreeNode<number> | null): number {
    if (!n) return 0;
    const l = check(n.left); if (l === -1) return -1;
    const r = check(n.right); if (r === -1) return -1;
    return Math.abs(l - r) > 1 ? -1 : 1 + Math.max(l, r);
  }
  return check(node) !== -1;
}
isBalanced(sample());                           // → true
isBalanced(fromLevelOrder([1, 2, null, 3, null, 4]));   // → false

function invert(node: TreeNode<number> | null): TreeNode<number> | null {
  if (!node) return null;
  [node.left, node.right] = [invert(node.right), invert(node.left)];
  return node;
}

function isSymmetric(root: TreeNode<number> | null): boolean {
  function mirror(a: TreeNode<number> | null, b: TreeNode<number> | null): boolean {
    if (!a && !b) return true;
    if (!a || !b || a.value !== b.value) return false;
    return mirror(a.left, b.right) && mirror(a.right, b.left);
  }
  return mirror(root?.left ?? null, root?.right ?? null);
}
isSymmetric(fromLevelOrder([1, 2, 2, 3, 4, 4, 3]));     // → true
isSymmetric(sample());                                  // → false

// lowest common ancestor, in a plain binary tree
function lca(root: TreeNode<number> | null, p: number, q: number): number | null {
  if (!root || root.value === p || root.value === q) return root?.value ?? null;
  const l = lca(root.left, p, q);
  const r = lca(root.right, p, q);
  if (l !== null && r !== null) return root.value;
  return l ?? r;
}
lca(sample(), 4, 5);                            // → 2
lca(sample(), 4, 3);                            // → 1

// maximum path sum through any two nodes
function maxPathSum(root: TreeNode<number> | null): number {
  let best = -Infinity;
  function gain(n: TreeNode<number> | null): number {
    if (!n) return 0;
    const l = Math.max(gain(n.left), 0);
    const r = Math.max(gain(n.right), 0);
    best = Math.max(best, n.value + l + r);
    return n.value + Math.max(l, r);
  }
  gain(root);
  return best;
}
maxPathSum(sample());                           // → 11
```

---

## Tree traversals

Four orders, each with a distinct purpose.

```ts
// depth-first, recursive
function inorder(n: TreeNode<number> | null, out: number[] = []): number[] {
  if (!n) return out;
  inorder(n.left, out);
  out.push(n.value);
  inorder(n.right, out);
  return out;
}

function preorder(n: TreeNode<number> | null, out: number[] = []): number[] {
  if (!n) return out;
  out.push(n.value);
  preorder(n.left, out);
  preorder(n.right, out);
  return out;
}

function postorder(n: TreeNode<number> | null, out: number[] = []): number[] {
  if (!n) return out;
  postorder(n.left, out);
  postorder(n.right, out);
  out.push(n.value);
  return out;
}

inorder(sample());                              // → [4, 2, 5, 1, 3]
preorder(sample());                             // → [1, 2, 4, 5, 3]
postorder(sample());                            // → [4, 5, 2, 3, 1]
```

| Order | Visits | Use for |
|---|---|---|
| **in**order | left, node, right | a BST in sorted order |
| **pre**order | node, left, right | copying/serialising a tree |
| **post**order | left, right, node | deleting; evaluating an expression tree |
| **level**order | breadth-first by depth | shortest path; printing by row |

```ts
// level-order (BFS), with the rows kept separate
function levelOrder(root: TreeNode<number> | null): number[][] {
  if (!root) return [];
  const out: number[][] = [];
  let level: TreeNode<number>[] = [root];
  while (level.length) {
    out.push(level.map((n) => n.value));
    const next: TreeNode<number>[] = [];
    for (const n of level) {
      if (n.left) next.push(n.left);
      if (n.right) next.push(n.right);
    }
    level = next;
  }
  return out;
}
levelOrder(sample());                           // → [[1], [2, 3], [4, 5]]

// zigzag, alternating direction per row
function zigzag(root: TreeNode<number> | null): number[][] {
  return levelOrder(root).map((row, i) => (i % 2 ? [...row].reverse() : row));
}
zigzag(sample());                               // → [[1], [3, 2], [4, 5]]

// iterative inorder with an explicit stack — no recursion, no stack overflow
function inorderIterative(root: TreeNode<number> | null): number[] {
  const out: number[] = [];
  const st: TreeNode<number>[] = [];
  let cur = root;
  while (cur || st.length) {
    while (cur) { st.push(cur); cur = cur.left; }
    const node = st.pop()!;
    out.push(node.value);
    cur = node.right;
  }
  return out;
}
inorderIterative(sample());                     // → [4, 2, 5, 1, 3]

// a generator traversal — lazy, and composes with any iterator helper
function* walk(n: TreeNode<number> | null): Generator<number> {
  if (!n) return;
  yield* walk(n.left);
  yield n.value;
  yield* walk(n.right);
}
[...walk(sample())];                            // → [4, 2, 5, 1, 3]

// take just the first two, without traversing the rest
const it = walk(sample());
[it.next().value, it.next().value];              // → [4, 2]
```

Recursion depth is the practical limit: a degenerate tree of 100,000 nodes will overflow the call stack. Use the iterative form when the input is untrusted or deep.

---

## Binary search tree

Left subtree < node < right subtree. Gives O(log n) search **when balanced**, O(n) when not.

```ts
class BST {
  #root: TreeNode<number> | null = null;
  #size = 0;

  get size(): number { return this.#size; }
  get root(): TreeNode<number> | null { return this.#root; }

  insert(v: number): this {                                     // O(h)
    const node = new TreeNode(v);
    if (!this.#root) { this.#root = node; this.#size++; return this; }
    let cur = this.#root;
    for (;;) {
      if (v === cur.value) return this;                          // no duplicates
      if (v < cur.value) {
        if (!cur.left) { cur.left = node; this.#size++; return this; }
        cur = cur.left;
      } else {
        if (!cur.right) { cur.right = node; this.#size++; return this; }
        cur = cur.right;
      }
    }
  }

  has(v: number): boolean {                                      // O(h)
    let cur = this.#root;
    while (cur) {
      if (v === cur.value) return true;
      cur = v < cur.value ? cur.left : cur.right;
    }
    return false;
  }

  min(): number | undefined {
    let cur = this.#root;
    while (cur?.left) cur = cur.left;
    return cur?.value;
  }

  max(): number | undefined {
    let cur = this.#root;
    while (cur?.right) cur = cur.right;
    return cur?.value;
  }

  delete(v: number): boolean {                                   // O(h)
    const before = this.#size;
    this.#root = this.#deleteFrom(this.#root, v);
    return this.#size < before;
  }

  #deleteFrom(node: TreeNode<number> | null, v: number): TreeNode<number> | null {
    if (!node) return null;
    if (v < node.value) { node.left = this.#deleteFrom(node.left, v); return node; }
    if (v > node.value) { node.right = this.#deleteFrom(node.right, v); return node; }

    // found it — three cases
    this.#size--;
    if (!node.left) return node.right;                           // 0 or 1 child
    if (!node.right) return node.left;
    // 2 children: replace with the in-order successor
    let succ = node.right;
    while (succ.left) succ = succ.left;
    node.value = succ.value;
    this.#size++;                                                // the recursive call re-decrements
    node.right = this.#deleteFrom(node.right, succ.value);
    return node;
  }

  toSortedArray(): number[] { return inorder(this.#root); }      // O(n)
}

const bst = new BST();
for (const v of [50, 30, 70, 20, 40, 60, 80]) bst.insert(v);

bst.toSortedArray();                            // → [20, 30, 40, 50, 60, 70, 80]
bst.has(40);                                    // → true
bst.has(45);                                    // → false
bst.min();                                      // → 20
bst.max();                                      // → 80
bst.size;                                       // → 7

bst.delete(20);                                 // → true   leaf
bst.toSortedArray();                            // → [30, 40, 50, 60, 70, 80]
bst.delete(30);                                 // → true   one child
bst.toSortedArray();                            // → [40, 50, 60, 70, 80]
bst.delete(50);                                 // → true   two children — root
bst.toSortedArray();                            // → [40, 60, 70, 80]
bst.delete(999);                                // → false
bst.size;                                       // → 4
```

### Validation, and why balance matters

```ts
// a tree is a BST only if EVERY node is within its inherited bounds —
// checking just parent-vs-child is the classic wrong answer
function isBST(node: TreeNode<number> | null, lo = -Infinity, hi = Infinity): boolean {
  if (!node) return true;
  if (node.value <= lo || node.value >= hi) return false;
  return isBST(node.left, lo, node.value) && isBST(node.right, node.value, hi);
}
isBST(fromLevelOrder([2, 1, 3]));               // → true
isBST(fromLevelOrder([5, 1, 6, null, null, 4, 7]));     // → false   4 < 5, on the right

// inserting sorted data degenerates a BST into a linked list
const degenerate = new BST();
for (let i = 1; i <= 8; i++) degenerate.insert(i);
height(degenerate.root);                        // → 8    should be ~4

const balanced = new BST();
for (const v of [4, 2, 6, 1, 3, 5, 7]) balanced.insert(v);
height(balanced.root);                          // → 3
```

That degenerate case is why self-balancing trees exist — and why, in practice, you reach for a `Map` unless you specifically need ordering.

---

## AVL tree

A BST that rebalances on every insert, guaranteeing height O(log n). The balance factor of every node stays within [−1, 1].

```ts
class AVLNode {
  left: AVLNode | null = null;
  right: AVLNode | null = null;
  height = 1;
  constructor(public value: number) {}
}

class AVLTree {
  #root: AVLNode | null = null;
  #size = 0;

  get size(): number { return this.#size; }
  get height(): number { return this.#h(this.#root); }

  #h(n: AVLNode | null): number { return n?.height ?? 0; }
  #balance(n: AVLNode): number { return this.#h(n.left) - this.#h(n.right); }
  #update(n: AVLNode): void { n.height = 1 + Math.max(this.#h(n.left), this.#h(n.right)); }

  #rotateRight(y: AVLNode): AVLNode {
    const x = y.left!;
    y.left = x.right;
    x.right = y;
    this.#update(y); this.#update(x);
    return x;
  }

  #rotateLeft(x: AVLNode): AVLNode {
    const y = x.right!;
    x.right = y.left;
    y.left = x;
    this.#update(x); this.#update(y);
    return y;
  }

  #rebalance(n: AVLNode): AVLNode {
    this.#update(n);
    const b = this.#balance(n);
    if (b > 1) {
      if (this.#balance(n.left!) < 0) n.left = this.#rotateLeft(n.left!);   // left-right
      return this.#rotateRight(n);                                          // left-left
    }
    if (b < -1) {
      if (this.#balance(n.right!) > 0) n.right = this.#rotateRight(n.right!); // right-left
      return this.#rotateLeft(n);                                            // right-right
    }
    return n;
  }

  insert(v: number): this {
    const add = (n: AVLNode | null): AVLNode => {
      if (!n) { this.#size++; return new AVLNode(v); }
      if (v < n.value) n.left = add(n.left);
      else if (v > n.value) n.right = add(n.right);
      else return n;
      return this.#rebalance(n);
    };
    this.#root = add(this.#root);
    return this;
  }

  has(v: number): boolean {
    let cur = this.#root;
    while (cur) {
      if (v === cur.value) return true;
      cur = v < cur.value ? cur.left : cur.right;
    }
    return false;
  }

  toSortedArray(): number[] {
    const out: number[] = [];
    const walk = (n: AVLNode | null): void => {
      if (!n) return;
      walk(n.left); out.push(n.value); walk(n.right);
    };
    walk(this.#root);
    return out;
  }
}

const avl = new AVLTree();
for (let i = 1; i <= 8; i++) avl.insert(i);     // sorted input — the BST killer

avl.toSortedArray();                            // → [1, 2, 3, 4, 5, 6, 7, 8]
avl.height;                                     // → 4    stays logarithmic
avl.size;                                       // → 8
avl.has(7);                                     // → true
avl.has(99);                                    // → false

// the same input left a plain BST with height 8
const big = new AVLTree();
for (let i = 0; i < 1000; i++) big.insert(i);
big.height;                                     // → 10
big.size;                                       // → 1000
```

| Rotation case | Condition | Fix |
|---|---|---|
| left-left | balance > 1, left child balance ≥ 0 | rotate right |
| left-right | balance > 1, left child balance < 0 | rotate left on the child, then right |
| right-right | balance < −1, right child balance ≤ 0 | rotate left |
| right-left | balance < −1, right child balance > 0 | rotate right on the child, then left |

| Tree | Balance guarantee | Insert cost | Notes |
|---|---|---|---|
| BST | none | O(h) | degenerates on sorted input |
| AVL | strict (±1) | O(log n), more rotations | fastest lookups |
| Red-black | loose (≤2x) | O(log n), fewer rotations | what most libraries use |
| B-tree | by node fan-out | O(log n) | databases, filesystems |

**Common uses.** Ordered maps and sets where you need range queries, predecessor/successor, or in-order iteration. If you only need key-value lookup, a hash map is faster and simpler.

---

## Trie

A prefix tree. Each edge is a character, so lookup costs O(m) in the key length — independent of how many keys are stored.

```ts
class TrieNode {
  children = new Map<string, TrieNode>();
  isWord = false;
}

class Trie {
  #root = new TrieNode();
  #size = 0;

  get size(): number { return this.#size; }

  insert(word: string): this {                                  // O(m)
    let cur = this.#root;
    for (const ch of word) {
      let next = cur.children.get(ch);
      if (!next) { next = new TrieNode(); cur.children.set(ch, next); }
      cur = next;
    }
    if (!cur.isWord) { cur.isWord = true; this.#size++; }
    return this;
  }

  #find(prefix: string): TrieNode | null {
    let cur = this.#root;
    for (const ch of prefix) {
      const next = cur.children.get(ch);
      if (!next) return null;
      cur = next;
    }
    return cur;
  }

  has(word: string): boolean {                                  // O(m)
    return this.#find(word)?.isWord ?? false;
  }

  startsWith(prefix: string): boolean {                         // O(m)
    return this.#find(prefix) !== null;
  }

  // every word under a prefix — the autocomplete primitive
  withPrefix(prefix: string, limit = Infinity): string[] {      // O(m + k)
    const node = this.#find(prefix);
    if (!node) return [];
    const out: string[] = [];
    const walk = (n: TrieNode, acc: string): void => {
      if (out.length >= limit) return;
      if (n.isWord) out.push(prefix + acc);
      for (const [ch, child] of n.children) walk(child, acc + ch);
    };
    walk(node, "");
    return out;
  }

  delete(word: string): boolean {
    const prune = (n: TrieNode, i: number): boolean => {
      if (i === word.length) {
        if (!n.isWord) return false;
        n.isWord = false;
        this.#size--;
        return n.children.size === 0;            // safe to remove this node
      }
      const ch = word[i]!;
      const child = n.children.get(ch);
      if (!child) return false;
      if (prune(child, i + 1)) n.children.delete(ch);
      return !n.isWord && n.children.size === 0;
    };
    const before = this.#size;
    prune(this.#root, 0);
    return this.#size < before;
  }
}

const trie = new Trie();
for (const w of ["cat", "car", "card", "care", "dog"]) trie.insert(w);

trie.size;                                      // → 5
trie.has("car");                                // → true
trie.has("ca");                                 // → false    a prefix, not a word
trie.startsWith("ca");                          // → true
trie.startsWith("z");                           // → false
trie.withPrefix("car");                         // → ['car', 'card', 'care']
trie.withPrefix("ca");                          // → ['cat', 'car', 'card', 'care']
trie.withPrefix("ca", 2);                       // → ['cat', 'car']
trie.withPrefix("zz");                          // → []

trie.delete("car");                             // → true
trie.has("car");                                // → false
trie.has("card");                               // → true     still intact
trie.delete("nope");                            // → false
trie.size;                                      // → 4
```

| Operation | Trie | Hash set |
|---|---|---|
| exact lookup | O(m) | O(m) to hash, O(1) after |
| prefix search | **O(m + k)** | O(n) — scan everything |
| sorted iteration | free | needs a sort |
| memory | higher, one node per prefix char | lower |

**Common uses.** Autocomplete and typeahead, spell-check dictionaries, IP routing tables, word games, and any prefix-matching problem. If you never query by prefix, use a `Set`.

```ts
// word-break: can a string be split entirely into dictionary words?
function wordBreak(s: string, words: readonly string[]): boolean {
  const dict = new Trie();
  for (const w of words) dict.insert(w);
  const ok = new Array<boolean>(s.length + 1).fill(false);
  ok[0] = true;
  for (let i = 0; i < s.length; i++) {
    if (!ok[i]) continue;
    for (let j = i + 1; j <= s.length; j++) {
      if (dict.has(s.slice(i, j))) ok[j] = true;
    }
  }
  return ok[s.length]!;
}
wordBreak("applepen", ["apple", "pen"]);        // → true
wordBreak("applepenx", ["apple", "pen"]);       // → false
```

---

## Union-Find

Disjoint-set union. Tracks a partition into groups, answering "are these two in the same group?" in near-constant time.

```ts
class UnionFind {
  #parent: number[];
  #rank: number[];
  #count: number;

  constructor(n: number) {
    this.#parent = Array.from({ length: n }, (_, i) => i);
    this.#rank = new Array<number>(n).fill(0);
    this.#count = n;
  }

  // path compression: point every node on the path straight at the root
  find(x: number): number {                                     // ~O(1)
    let root = x;
    while (this.#parent[root] !== root) root = this.#parent[root]!;
    while (this.#parent[x] !== root) {
      const next = this.#parent[x]!;
      this.#parent[x] = root;
      x = next;
    }
    return root;
  }

  // union by rank: hang the shorter tree off the taller
  union(a: number, b: number): boolean {                        // ~O(1)
    const ra = this.find(a), rb = this.find(b);
    if (ra === rb) return false;                                 // already joined
    if (this.#rank[ra]! < this.#rank[rb]!) this.#parent[ra] = rb;
    else if (this.#rank[ra]! > this.#rank[rb]!) this.#parent[rb] = ra;
    else { this.#parent[rb] = ra; this.#rank[ra]!++; }
    this.#count--;
    return true;
  }

  connected(a: number, b: number): boolean { return this.find(a) === this.find(b); }
  get count(): number { return this.#count; }                    // number of groups

  groups(): number[][] {
    const m = new Map<number, number[]>();
    for (let i = 0; i < this.#parent.length; i++) {
      const r = this.find(i);
      m.set(r, [...(m.get(r) ?? []), i]);
    }
    return [...m.values()];
  }
}

const uf = new UnionFind(6);
uf.count;                                       // → 6
uf.union(0, 1);                                 // → true
uf.union(1, 2);                                 // → true
uf.union(3, 4);                                 // → true
uf.union(0, 2);                                 // → false   already connected
uf.connected(0, 2);                             // → true
uf.connected(0, 3);                             // → false
uf.count;                                       // → 3
uf.groups();                                    // → [[0, 1, 2], [3, 4], [5]]
```

With both path compression and union by rank, each operation is O(α(n)) — the inverse Ackermann function, which is below 5 for any n you will ever see.

```ts
// counting connected components in an undirected graph
function componentCount(n: number, edges: readonly [number, number][]): number {
  const dsu = new UnionFind(n);
  for (const [a, b] of edges) dsu.union(a, b);
  return dsu.count;
}
componentCount(5, [[0, 1], [1, 2], [3, 4]]);    // → 2

// cycle detection in an undirected graph: a union that fails means a cycle
function hasUndirectedCycle(n: number, edges: readonly [number, number][]): boolean {
  const dsu = new UnionFind(n);
  return edges.some(([a, b]) => !dsu.union(a, b));
}
hasUndirectedCycle(3, [[0, 1], [1, 2]]);        // → false
hasUndirectedCycle(3, [[0, 1], [1, 2], [2, 0]]); // → true
```

**Common uses.** Kruskal's MST, connected components, cycle detection, "friend circles", percolation, and incremental connectivity where edges only ever get added.

---

## Segment tree

Range queries plus point updates, both in O(log n). The general version: swap the combine function to change the query.

```ts
class SegmentTree {
  #n: number;
  #tree: number[];

  constructor(
    values: readonly number[],
    private readonly combine: (a: number, b: number) => number = (a, b) => a + b,
    private readonly identity = 0,
  ) {
    this.#n = values.length;
    this.#tree = new Array<number>(2 * this.#n).fill(identity);
    for (let i = 0; i < this.#n; i++) this.#tree[this.#n + i] = values[i]!;
    for (let i = this.#n - 1; i > 0; i--) {
      this.#tree[i] = combine(this.#tree[2 * i]!, this.#tree[2 * i + 1]!);
    }
  }

  update(i: number, value: number): void {                      // O(log n)
    let k = i + this.#n;
    this.#tree[k] = value;
    for (k >>= 1; k >= 1; k >>= 1) {
      this.#tree[k] = this.combine(this.#tree[2 * k]!, this.#tree[2 * k + 1]!);
    }
  }

  // half-open [lo, hi)
  query(lo: number, hi: number): number {                       // O(log n)
    let resL = this.identity, resR = this.identity;
    let l = lo + this.#n, r = hi + this.#n;
    while (l < r) {
      if (l & 1) resL = this.combine(resL, this.#tree[l++]!);
      if (r & 1) resR = this.combine(this.#tree[--r]!, resR);
      l >>= 1; r >>= 1;
    }
    return this.combine(resL, resR);
  }
}

const sum = new SegmentTree([1, 2, 3, 4, 5]);
sum.query(0, 5);                                // → 15
sum.query(1, 3);                                // → 5      indices 1..2
sum.update(1, 10);
sum.query(0, 5);                                // → 23
sum.query(1, 3);                                // → 13

// the same structure, different combine
const min = new SegmentTree([5, 2, 8, 1], (a, b) => Math.min(a, b), Infinity);
min.query(0, 4);                                // → 1
min.query(0, 2);                                // → 2
min.update(3, 99);
min.query(0, 4);                                // → 2

const max = new SegmentTree([5, 2, 8, 1], (a, b) => Math.max(a, b), -Infinity);
max.query(0, 4);                                // → 8
max.query(2, 4);                                // → 8
```

The combine function must be **associative**, and `identity` must be its neutral element — `sum`/`0`, `min`/`Infinity`, `max`/`-Infinity`, `gcd`/`0`.

---

## Fenwick tree

Binary indexed tree. Does prefix sums with point updates in O(log n), like a segment tree but roughly a third of the code and memory.

```ts
class FenwickTree {
  #tree: number[];

  constructor(size: number) {
    this.#tree = new Array<number>(size + 1).fill(0);           // 1-indexed
  }

  add(i: number, delta: number): void {                         // O(log n)
    for (let k = i + 1; k < this.#tree.length; k += k & -k) this.#tree[k]! += delta;
  }

  // sum of [0, i]
  prefixSum(i: number): number {                                // O(log n)
    let s = 0;
    for (let k = i + 1; k > 0; k -= k & -k) s += this.#tree[k]!;
    return s;
  }

  // sum of [lo, hi]
  rangeSum(lo: number, hi: number): number {
    return this.prefixSum(hi) - (lo > 0 ? this.prefixSum(lo - 1) : 0);
  }

  static from(values: readonly number[]): FenwickTree {
    const f = new FenwickTree(values.length);
    values.forEach((v, i) => f.add(i, v));
    return f;
  }
}

const fen = FenwickTree.from([1, 2, 3, 4, 5]);
fen.prefixSum(4);                               // → 15
fen.prefixSum(2);                               // → 6
fen.rangeSum(1, 3);                             // → 9
fen.add(1, 10);                                 // element 1 becomes 12
fen.rangeSum(1, 3);                             // → 19
fen.prefixSum(4);                               // → 25
```

`k & -k` isolates the lowest set bit, which is how the tree walks its implicit structure.

| | Prefix sums array | Fenwick | Segment tree |
|---|---|---|---|
| build | O(n) | O(n log n) | O(n) |
| range query | O(1) | O(log n) | O(log n) |
| point update | **O(n)** | O(log n) | O(log n) |
| range update | O(n) | O(log n) with tricks | O(log n) with lazy propagation |
| min/max queries | no | no | yes |

Use a prefix-sum array when the data never changes, Fenwick when it does and you only need sums, a segment tree when you need min/max or range updates.

---

## Graphs

Three representations, each with different trade-offs.

```ts
// 1. adjacency list — the default. O(V + E) space.
type Graph = Map<string, string[]>;

const g: Graph = new Map([
  ["A", ["B", "C"]],
  ["B", ["D"]],
  ["C", ["D"]],
  ["D", []],
]);
g.get("A");                                     // → ['B', 'C']

// 2. adjacency matrix — O(V²) space, O(1) edge lookup. Good for dense graphs.
const matrix = [
  [0, 1, 1, 0],
  [0, 0, 0, 1],
  [0, 0, 0, 1],
  [0, 0, 0, 0],
];
matrix[0]![1];                                  // → 1    A → B exists

// 3. edge list — simplest; what Kruskal's algorithm wants.
const edges: [string, string, number][] = [
  ["A", "B", 4], ["A", "C", 2], ["B", "D", 5], ["C", "D", 8],
];
edges.length;                                   // → 4
```

A reusable weighted graph:

```ts
class WeightedGraph<T> {
  #adj = new Map<T, Map<T, number>>();

  addNode(v: T): this {
    if (!this.#adj.has(v)) this.#adj.set(v, new Map());
    return this;
  }

  addEdge(a: T, b: T, weight = 1, directed = false): this {
    this.addNode(a).addNode(b);
    this.#adj.get(a)!.set(b, weight);
    if (!directed) this.#adj.get(b)!.set(a, weight);
    return this;
  }

  neighbors(v: T): [T, number][] { return [...(this.#adj.get(v) ?? [])]; }
  nodes(): T[] { return [...this.#adj.keys()]; }
  get order(): number { return this.#adj.size; }
  hasEdge(a: T, b: T): boolean { return this.#adj.get(a)?.has(b) ?? false; }
  weight(a: T, b: T): number | undefined { return this.#adj.get(a)?.get(b); }
}

const wg = new WeightedGraph<string>();
wg.addEdge("A", "B", 4).addEdge("A", "C", 2).addEdge("B", "D", 5).addEdge("C", "D", 8);
wg.order;                                       // → 4
wg.nodes();                                     // → ['A', 'B', 'C', 'D']
wg.neighbors("A");                              // → [['B', 4], ['C', 2]]
wg.hasEdge("A", "B");                           // → true
wg.hasEdge("B", "A");                           // → true    undirected
wg.weight("C", "D");                            // → 8
```

| | Adjacency list | Adjacency matrix |
|---|---|---|
| space | O(V + E) | O(V²) |
| "is there an edge a→b?" | O(deg(a)) | O(1) |
| iterate a node's neighbours | O(deg(a)) | O(V) |
| best for | sparse (most real graphs) | dense, or edge-lookup-heavy |

---

## BFS and DFS

The two ways to explore. BFS finds the shortest path in an **unweighted** graph; DFS goes deep and suits cycle detection and topological ordering.

```ts
const sampleGraph: Graph = new Map([
  ["A", ["B", "C"]],
  ["B", ["D", "E"]],
  ["C", ["F"]],
  ["D", []],
  ["E", ["F"]],
  ["F", []],
]);

// BFS — a queue, level by level
function bfs(graph: Graph, start: string): string[] {           // O(V + E)
  const visited = new Set<string>([start]);
  const order: string[] = [];
  const queue = [start];
  let head = 0;
  while (head < queue.length) {
    const node = queue[head++]!;
    order.push(node);
    for (const next of graph.get(node) ?? []) {
      if (visited.has(next)) continue;
      visited.add(next);                         // mark on ENQUEUE, not dequeue
      queue.push(next);
    }
  }
  return order;
}
bfs(sampleGraph, "A");                          // → ['A', 'B', 'C', 'D', 'E', 'F']

// DFS — recursive
function dfs(graph: Graph, start: string, visited = new Set<string>()): string[] {
  if (visited.has(start)) return [];
  visited.add(start);
  const order = [start];
  for (const next of graph.get(start) ?? []) order.push(...dfs(graph, next, visited));
  return order;
}
dfs(sampleGraph, "A");                          // → ['A', 'B', 'D', 'E', 'F', 'C']

// DFS — iterative, with an explicit stack
function dfsIterative(graph: Graph, start: string): string[] {
  const visited = new Set<string>();
  const order: string[] = [];
  const stack = [start];
  while (stack.length) {
    const node = stack.pop()!;
    if (visited.has(node)) continue;
    visited.add(node);
    order.push(node);
    // push in reverse so the first neighbour is explored first
    const next = graph.get(node) ?? [];
    for (let i = next.length - 1; i >= 0; i--) stack.push(next[i]!);
  }
  return order;
}
dfsIterative(sampleGraph, "A");                 // → ['A', 'B', 'D', 'E', 'F', 'C']
```

Marking visited on **enqueue** rather than dequeue is what keeps BFS O(V + E); marking on dequeue lets a node enter the queue many times.

```ts
// shortest path in an unweighted graph, reconstructed from a parent map
function shortestPath(graph: Graph, start: string, goal: string): string[] | null {
  if (start === goal) return [start];
  const parent = new Map<string, string | null>([[start, null]]);
  const queue = [start];
  let head = 0;
  while (head < queue.length) {
    const node = queue[head++]!;
    for (const next of graph.get(node) ?? []) {
      if (parent.has(next)) continue;
      parent.set(next, node);
      if (next === goal) {
        const path: string[] = [];
        for (let c: string | null = goal; c !== null; c = parent.get(c) ?? null) path.push(c);
        return path.reverse();
      }
      queue.push(next);
    }
  }
  return null;
}
shortestPath(sampleGraph, "A", "F");            // → ['A', 'C', 'F']
shortestPath(sampleGraph, "D", "A");            // → null

// distance to every node
function distances(graph: Graph, start: string): Map<string, number> {
  const dist = new Map<string, number>([[start, 0]]);
  const queue = [start];
  let head = 0;
  while (head < queue.length) {
    const node = queue[head++]!;
    for (const next of graph.get(node) ?? []) {
      if (dist.has(next)) continue;
      dist.set(next, dist.get(node)! + 1);
      queue.push(next);
    }
  }
  return dist;
}
[...distances(sampleGraph, "A")];
// → [['A', 0], ['B', 1], ['C', 1], ['D', 2], ['E', 2], ['F', 2]]
```

### Grid traversal

A grid is a graph whose edges are implied by adjacency — no adjacency list needed.

```ts
const DIRS: readonly [number, number][] = [[-1, 0], [1, 0], [0, -1], [0, 1]];

// count islands of 1s, 4-directionally connected
function countIslands(grid: readonly (readonly number[])[]): number {
  const rows = grid.length, cols = grid[0]?.length ?? 0;
  const seen = new Set<number>();
  let count = 0;

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r]![c] !== 1 || seen.has(r * cols + c)) continue;
      count++;
      const stack: [number, number][] = [[r, c]];
      seen.add(r * cols + c);
      while (stack.length) {
        const [cr, cc] = stack.pop()!;
        for (const [dr, dc] of DIRS) {
          const nr = cr + dr, nc = cc + dc;
          if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
          if (grid[nr]![nc] !== 1 || seen.has(nr * cols + nc)) continue;
          seen.add(nr * cols + nc);
          stack.push([nr, nc]);
        }
      }
    }
  }
  return count;
}
const islandGrid = [
  [1, 1, 0, 0],
  [1, 0, 0, 1],
  [0, 0, 1, 0],
];
countIslands(islandGrid);                       // → 3

// shortest path through a maze, BFS
function mazeShortest(grid: readonly (readonly number[])[]): number {
  const rows = grid.length, cols = grid[0]?.length ?? 0;
  if (grid[0]![0] === 1) return -1;
  const dist = new Map<number, number>([[0, 0]]);
  const queue: [number, number][] = [[0, 0]];
  let head = 0;
  while (head < queue.length) {
    const [r, c] = queue[head++]!;
    const d = dist.get(r * cols + c)!;
    if (r === rows - 1 && c === cols - 1) return d;
    for (const [dr, dc] of DIRS) {
      const nr = r + dr, nc = c + dc;
      if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
      if (grid[nr]![nc] === 1 || dist.has(nr * cols + nc)) continue;
      dist.set(nr * cols + nc, d + 1);
      queue.push([nr, nc]);
    }
  }
  return -1;
}
const maze = [
  [0, 0, 1],
  [1, 0, 0],
  [0, 0, 0],
];
mazeShortest(maze);                             // → 4
```

---

## Topological sort

Orders a directed acyclic graph so every edge points forward. Returns `null` when a cycle makes that impossible.

```ts
type DiGraph = Map<string, string[]>;

// Kahn's algorithm — BFS on in-degrees
function topoSort(graph: DiGraph): string[] | null {            // O(V + E)
  const indegree = new Map<string, number>();
  for (const node of graph.keys()) indegree.set(node, 0);
  for (const next of graph.values()) {
    for (const n of next) indegree.set(n, (indegree.get(n) ?? 0) + 1);
  }

  const queue = [...indegree].filter(([, d]) => d === 0).map(([n]) => n);
  const order: string[] = [];
  let head = 0;
  while (head < queue.length) {
    const node = queue[head++]!;
    order.push(node);
    for (const next of graph.get(node) ?? []) {
      const d = indegree.get(next)! - 1;
      indegree.set(next, d);
      if (d === 0) queue.push(next);
    }
  }
  return order.length === indegree.size ? order : null;          // short = cycle
}

const deps: DiGraph = new Map([
  ["shirt", ["tie"]],
  ["tie", ["jacket"]],
  ["trousers", ["shoes", "belt"]],
  ["belt", ["jacket"]],
  ["shoes", []],
  ["jacket", []],
]);
topoSort(deps);
// → ['shirt', 'trousers', 'tie', 'shoes', 'belt', 'jacket']

// a cycle is detected, not silently mis-ordered
const cyclic: DiGraph = new Map([["a", ["b"]], ["b", ["c"]], ["c", ["a"]]]);
topoSort(cyclic);                               // → null

// DFS version — push on the way OUT, then reverse
function topoDFS(graph: DiGraph): string[] | null {
  const state = new Map<string, 0 | 1 | 2>();    // 0 unseen, 1 in progress, 2 done
  const out: string[] = [];
  let ok = true;

  const visit = (node: string): void => {
    const s = state.get(node) ?? 0;
    if (s === 1) { ok = false; return; }          // back edge → cycle
    if (s === 2) return;
    state.set(node, 1);
    for (const next of graph.get(node) ?? []) visit(next);
    state.set(node, 2);
    out.push(node);
  };

  for (const node of graph.keys()) visit(node);
  return ok ? out.reverse() : null;
}
topoDFS(cyclic);                                // → null
topoDFS(new Map([["a", ["b"]], ["b", []]]));    // → ['a', 'b']
```

The three-colour scheme is the key idea: an edge back into a node that is still *in progress* is a cycle, whereas an edge into a *finished* node is fine.

**Common uses.** Build systems and task runners, module bundlers resolving imports, course prerequisites, spreadsheet formula recalculation, and dependency installation order.

---

## Shortest paths

| Algorithm | Handles | Time |
|---|---|---|
| BFS | unweighted | O(V + E) |
| Dijkstra | non-negative weights | O((V + E) log V) |
| Bellman-Ford | negative weights, detects negative cycles | O(V·E) |
| Floyd-Warshall | all pairs | O(V³) |
| A* | with a heuristic | depends |

### Dijkstra

```ts
function dijkstra<T>(graph: WeightedGraph<T>, start: T): Map<T, number> {
  const dist = new Map<T, number>([[start, 0]]);
  const done = new Set<T>();
  const pq = new MinHeap<[T, number]>((a, b) => a[1] - b[1]);
  pq.push([start, 0]);

  while (pq.size) {
    const [node, d] = pq.pop()!;
    if (done.has(node)) continue;                // a stale duplicate entry
    done.add(node);
    for (const [next, w] of graph.neighbors(node)) {
      const nd = d + w;
      if (nd < (dist.get(next) ?? Infinity)) {
        dist.set(next, nd);
        pq.push([next, nd]);                     // lazy deletion: just push again
      }
    }
  }
  return dist;
}

const road = new WeightedGraph<string>();
road.addEdge("A", "B", 4).addEdge("A", "C", 2)
    .addEdge("C", "B", 1).addEdge("B", "D", 5)
    .addEdge("C", "D", 8);

[...dijkstra(road, "A")].sort();
// → [['A', 0], ['B', 3], ['C', 2], ['D', 8]]
```

The `done` set plus re-pushing is "lazy deletion" — simpler than a decrease-key operation, and the standard approach when your heap does not support one.

```ts
// with the path reconstructed
function dijkstraPath<T>(graph: WeightedGraph<T>, start: T, goal: T): { cost: number; path: T[] } | null {
  const dist = new Map<T, number>([[start, 0]]);
  const prev = new Map<T, T>();
  const done = new Set<T>();
  const pq = new MinHeap<[T, number]>((a, b) => a[1] - b[1]);
  pq.push([start, 0]);

  while (pq.size) {
    const [node, d] = pq.pop()!;
    if (done.has(node)) continue;
    done.add(node);
    if (node === goal) break;
    for (const [next, w] of graph.neighbors(node)) {
      const nd = d + w;
      if (nd < (dist.get(next) ?? Infinity)) {
        dist.set(next, nd);
        prev.set(next, node);
        pq.push([next, nd]);
      }
    }
  }

  if (!dist.has(goal)) return null;
  const path: T[] = [];
  for (let c: T | undefined = goal; c !== undefined; c = prev.get(c)) path.push(c);
  return { cost: dist.get(goal)!, path: path.reverse() };
}

dijkstraPath(road, "A", "D");                   // → { cost: 8, path: ['A', 'C', 'B', 'D'] }
dijkstraPath(road, "A", "Z");                   // → null
```

**Dijkstra breaks on negative weights** — once a node is marked done, a later negative edge cannot reopen it.

### Bellman-Ford

```ts
function bellmanFord(
  nodes: readonly string[],
  edges: readonly [string, string, number][],
  start: string,
): Map<string, number> | null {
  const dist = new Map<string, number>(nodes.map((n) => [n, Infinity]));
  dist.set(start, 0);

  // relax every edge V-1 times
  for (let i = 0; i < nodes.length - 1; i++) {
    let changed = false;
    for (const [a, b, w] of edges) {
      const da = dist.get(a)!;
      if (da !== Infinity && da + w < dist.get(b)!) { dist.set(b, da + w); changed = true; }
    }
    if (!changed) break;
  }

  // one more pass: anything still improving sits on a negative cycle
  for (const [a, b, w] of edges) {
    const da = dist.get(a)!;
    if (da !== Infinity && da + w < dist.get(b)!) return null;
  }
  return dist;
}

const withNeg: [string, string, number][] = [
  ["A", "B", 4], ["A", "C", 2], ["B", "D", 3], ["C", "B", -2], ["C", "D", 6],
];
[...bellmanFord(["A", "B", "C", "D"], withNeg, "A")!].sort();
// → [['A', 0], ['B', 0], ['C', 2], ['D', 3]]

const negCycle: [string, string, number][] = [["A", "B", 1], ["B", "C", -3], ["C", "A", 1]];
bellmanFord(["A", "B", "C"], negCycle, "A");    // → null
```

### Floyd-Warshall

```ts
function floydWarshall(n: number, edges: readonly [number, number, number][]): number[][] {
  const d = Array.from({ length: n }, (_, i) =>
    Array.from({ length: n }, (_, j) => (i === j ? 0 : Infinity)),
  );
  for (const [a, b, w] of edges) d[a]![b] = Math.min(d[a]![b]!, w);

  for (let k = 0; k < n; k++)
    for (let i = 0; i < n; i++)
      for (let j = 0; j < n; j++)
        if (d[i]![k]! + d[k]![j]! < d[i]![j]!) d[i]![j] = d[i]![k]! + d[k]![j]!;

  return d;
}

floydWarshall(4, [[0, 1, 5], [0, 3, 10], [1, 2, 3], [2, 3, 1]]);
// → [[0, 5, 8, 9], [Infinity, 0, 3, 4], [Infinity, Infinity, 0, 1], [Infinity, Infinity, Infinity, 0]]
```

`k` must be the **outermost** loop — it represents "paths allowed to route through the first k nodes". Any other nesting silently produces wrong answers.

---

## Minimum spanning tree

The cheapest set of edges connecting every node, with no cycles.

```ts
// Kruskal: sort edges, add any that does not create a cycle. O(E log E)
function kruskal(n: number, edges: readonly [number, number, number][]): { weight: number; edges: [number, number, number][] } {
  const dsu = new UnionFind(n);
  const chosen: [number, number, number][] = [];
  let weight = 0;
  for (const e of [...edges].sort((a, b) => a[2] - b[2])) {
    if (dsu.union(e[0], e[1])) { chosen.push(e); weight += e[2]; }
  }
  return { weight, edges: chosen };
}

const mstEdges: [number, number, number][] = [
  [0, 1, 4], [0, 2, 3], [1, 2, 1], [1, 3, 2], [2, 3, 4], [3, 4, 2],
];
kruskal(5, mstEdges).weight;                    // → 8
kruskal(5, mstEdges).edges;                     // → [[1, 2, 1], [1, 3, 2], [3, 4, 2], [0, 2, 3]]

// Prim: grow one tree, always taking the cheapest edge leaving it. O(E log V)
function prim(graph: WeightedGraph<string>, start: string): number {
  const inTree = new Set<string>();
  const pq = new MinHeap<[string, number]>((a, b) => a[1] - b[1]);
  pq.push([start, 0]);
  let total = 0;
  while (pq.size) {
    const [node, w] = pq.pop()!;
    if (inTree.has(node)) continue;
    inTree.add(node);
    total += w;
    for (const [next, nw] of graph.neighbors(node)) {
      if (!inTree.has(next)) pq.push([next, nw]);
    }
  }
  return total;
}

const pg = new WeightedGraph<string>();
pg.addEdge("a", "b", 4).addEdge("a", "c", 3).addEdge("b", "c", 1)
  .addEdge("b", "d", 2).addEdge("c", "d", 4).addEdge("d", "e", 2);
prim(pg, "a");                                  // → 8
```

| | Kruskal | Prim |
|---|---|---|
| approach | globally cheapest edge first | grow from one node |
| needs | Union-Find + a sort | a priority queue |
| best for | sparse graphs, edge lists | dense graphs |

**Common uses.** Network design (laying cable, roads), clustering, approximation algorithms for the travelling salesman problem, and maze generation.

---

## Sorting

| Algorithm | Best | Average | Worst | Space | Stable |
|---|---|---|---|---|---|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | yes |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | yes |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | no |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | yes |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | no |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | no |
| Counting | O(n + k) | O(n + k) | O(n + k) | O(k) | yes |
| Radix | O(nk) | O(nk) | O(nk) | O(n + k) | yes |

In production, call `toSorted`. These exist to be understood, and because the ideas (partitioning, merging) recur elsewhere.

```ts
// insertion — genuinely the fastest for small or nearly-sorted arrays
function insertionSort(input: readonly number[]): number[] {
  const a = [...input];
  for (let i = 1; i < a.length; i++) {
    const key = a[i]!;
    let j = i - 1;
    while (j >= 0 && a[j]! > key) { a[j + 1] = a[j]!; j--; }
    a[j + 1] = key;
  }
  return a;
}
insertionSort([5, 2, 4, 6, 1, 3]);              // → [1, 2, 3, 4, 5, 6]

// selection — minimum number of swaps, O(n²) comparisons regardless
function selectionSort(input: readonly number[]): number[] {
  const a = [...input];
  for (let i = 0; i < a.length - 1; i++) {
    let min = i;
    for (let j = i + 1; j < a.length; j++) if (a[j]! < a[min]!) min = j;
    if (min !== i) [a[i], a[min]] = [a[min]!, a[i]!];
  }
  return a;
}
selectionSort([64, 25, 12, 22]);                // → [12, 22, 25, 64]

// bubble, with the early-exit that makes it O(n) on sorted input
function bubbleSort(input: readonly number[]): number[] {
  const a = [...input];
  for (let i = 0; i < a.length - 1; i++) {
    let swapped = false;
    for (let j = 0; j < a.length - 1 - i; j++) {
      if (a[j]! > a[j + 1]!) { [a[j], a[j + 1]] = [a[j + 1]!, a[j]!]; swapped = true; }
    }
    if (!swapped) break;
  }
  return a;
}
bubbleSort([3, 1, 2]);                          // → [1, 2, 3]

// merge sort — stable, predictable, O(n) extra space
function mergeSort(a: readonly number[]): number[] {
  if (a.length <= 1) return [...a];
  const mid = a.length >> 1;
  const left = mergeSort(a.slice(0, mid));
  const right = mergeSort(a.slice(mid));
  const out: number[] = [];
  let i = 0, j = 0;
  while (i < left.length && j < right.length) {
    out.push(left[i]! <= right[j]! ? left[i++]! : right[j++]!);   // <= keeps it stable
  }
  while (i < left.length) out.push(left[i++]!);
  while (j < right.length) out.push(right[j++]!);
  return out;
}
mergeSort([38, 27, 43, 3, 9, 82, 10]);          // → [3, 9, 10, 27, 38, 43, 82]

// quicksort — Lomuto partition, in place
function quickSort(input: readonly number[]): number[] {
  const a = [...input];
  const partition = (lo: number, hi: number): number => {
    // median-of-three pivot avoids the O(n²) case on sorted input
    const mid = (lo + hi) >> 1;
    const cand = [a[lo]!, a[mid]!, a[hi]!].sort((x, y) => x - y)[1]!;
    const pi = a.indexOf(cand, lo);
    [a[pi], a[hi]] = [a[hi]!, a[pi]!];
    const pivot = a[hi]!;
    let i = lo;
    for (let j = lo; j < hi; j++) {
      if (a[j]! < pivot) { [a[i], a[j]] = [a[j]!, a[i]!]; i++; }
    }
    [a[i], a[hi]] = [a[hi]!, a[i]!];
    return i;
  };
  const sort = (lo: number, hi: number): void => {
    if (lo >= hi) return;
    const p = partition(lo, hi);
    sort(lo, p - 1);
    sort(p + 1, hi);
  };
  sort(0, a.length - 1);
  return a;
}
quickSort([10, 7, 8, 9, 1, 5]);                 // → [1, 5, 7, 8, 9, 10]
quickSort([1, 2, 3, 4, 5]);                     // → [1, 2, 3, 4, 5]

// counting sort — O(n + k), beats the O(n log n) bound for small integer ranges
function countingSort(a: readonly number[]): number[] {
  if (!a.length) return [];
  const min = Math.min(...a), max = Math.max(...a);
  const counts = new Array<number>(max - min + 1).fill(0);
  for (const v of a) counts[v - min]!++;
  const out: number[] = [];
  counts.forEach((c, i) => { for (let k = 0; k < c; k++) out.push(i + min); });
  return out;
}
countingSort([4, 2, 2, 8, 3, 3, 1]);            // → [1, 2, 2, 3, 3, 4, 8]

// radix sort — digit by digit, using a stable counting sort per pass
function radixSort(a: readonly number[]): number[] {
  let out = [...a];
  const max = Math.max(0, ...out);
  for (let exp = 1; Math.floor(max / exp) > 0; exp *= 10) {
    const buckets: number[][] = Array.from({ length: 10 }, () => []);
    for (const v of out) buckets[Math.floor(v / exp) % 10]!.push(v);
    out = buckets.flat();
  }
  return out;
}
radixSort([170, 45, 75, 90, 2, 802, 24, 66]);   // → [2, 24, 45, 66, 75, 90, 170, 802]

// quickselect — the k-th smallest in O(n) average, without a full sort
function quickselect(input: readonly number[], k: number): number | undefined {
  const a = [...input];
  if (k < 0 || k >= a.length) return undefined;
  let lo = 0, hi = a.length - 1;
  while (lo <= hi) {
    const pivot = a[hi]!;
    let i = lo;
    for (let j = lo; j < hi; j++) if (a[j]! < pivot) { [a[i], a[j]] = [a[j]!, a[i]!]; i++; }
    [a[i], a[hi]] = [a[hi]!, a[i]!];
    if (i === k) return a[i];
    if (i < k) lo = i + 1; else hi = i - 1;
  }
  return undefined;
}
quickselect([7, 10, 4, 3, 20, 15], 2);          // → 7    the 3rd smallest
quickselect([7, 10, 4, 3, 20, 15], 0);          // → 3
```

`Array.prototype.sort` in V8 is TimSort — a merge/insertion hybrid that is stable and near-linear on partly-ordered data. You will not beat it in JavaScript.

---

## Binary search

O(log n) on sorted data. Easy to describe, famously easy to get subtly wrong.

```ts
function binarySearch(a: readonly number[], target: number): number {
  let lo = 0, hi = a.length - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);            // avoids overflow in other languages
    if (a[mid] === target) return mid;
    if (a[mid]! < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}
const sorted = [1, 3, 5, 7, 9, 11];
binarySearch(sorted, 7);                        // → 3
binarySearch(sorted, 1);                        // → 0
binarySearch(sorted, 4);                        // → -1
```

### The bounds variants

These matter more than the plain search, because they handle duplicates.

```ts
// lowerBound: the first index where a[i] >= target (the insertion point)
function lowerBound(a: readonly number[], target: number): number {
  let lo = 0, hi = a.length;                     // note: hi = length, half-open
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (a[mid]! < target) lo = mid + 1; else hi = mid;
  }
  return lo;
}

// upperBound: the first index where a[i] > target
function upperBound(a: readonly number[], target: number): number {
  let lo = 0, hi = a.length;
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (a[mid]! <= target) lo = mid + 1; else hi = mid;
  }
  return lo;
}

const dupes = [1, 2, 2, 2, 3, 5];
lowerBound(dupes, 2);                           // → 1
upperBound(dupes, 2);                           // → 4
upperBound(dupes, 2) - lowerBound(dupes, 2);    // → 3    how many 2s
lowerBound(dupes, 4);                           // → 5    where 4 would go
lowerBound(dupes, 0);                           // → 0
lowerBound(dupes, 9);                           // → 6

// keeping an array sorted as you insert
function insertSorted(a: number[], v: number): number[] {
  a.splice(lowerBound(a, v), 0, v);
  return a;
}
insertSorted([1, 3, 5], 4);                     // → [1, 3, 4, 5]
```

### Binary search on the answer

The most useful form in practice: when the answer is monotonic, search the *value* space rather than an array.

```ts
// smallest capacity that ships all packages within `days`
function shipCapacity(weights: readonly number[], days: number): number {
  const feasible = (cap: number): boolean => {
    let needed = 1, load = 0;
    for (const w of weights) {
      if (load + w > cap) { needed++; load = 0; }
      load += w;
    }
    return needed <= days;
  };

  let lo = Math.max(...weights);                 // must fit the heaviest item
  let hi = weights.reduce((s, w) => s + w, 0);   // one day
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (feasible(mid)) hi = mid; else lo = mid + 1;
  }
  return lo;
}
shipCapacity([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 5);   // → 15

// integer square root
function isqrt(n: number): number {
  let lo = 0, hi = n;
  while (lo < hi) {
    const mid = lo + ((hi - lo + 1) >> 1);       // round UP, or this loops forever
    if (mid * mid <= n) lo = mid; else hi = mid - 1;
  }
  return lo;
}
isqrt(16);                                      // → 4
isqrt(17);                                      // → 4
isqrt(0);                                       // → 0

// search a rotated sorted array
function searchRotated(a: readonly number[], target: number): number {
  let lo = 0, hi = a.length - 1;
  while (lo <= hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (a[mid] === target) return mid;
    if (a[lo]! <= a[mid]!) {                     // left half is sorted
      if (a[lo]! <= target && target < a[mid]!) hi = mid - 1; else lo = mid + 1;
    } else {                                      // right half is sorted
      if (a[mid]! < target && target <= a[hi]!) lo = mid + 1; else hi = mid - 1;
    }
  }
  return -1;
}
searchRotated([4, 5, 6, 7, 0, 1, 2], 0);        // → 4
searchRotated([4, 5, 6, 7, 0, 1, 2], 3);        // → -1
```

The `lo < hi` + `hi = mid` form and the `lo <= hi` + `hi = mid - 1` form are different templates. Mixing them is the usual cause of an infinite loop — pick one and keep the invariant straight.

---

## Two pointers

Two indices moving through a sequence, turning an O(n²) scan into O(n).

```ts
// pair summing to a target, in a SORTED array
function twoSumSorted(a: readonly number[], target: number): [number, number] | null {
  let lo = 0, hi = a.length - 1;
  while (lo < hi) {
    const sum = a[lo]! + a[hi]!;
    if (sum === target) return [lo, hi];
    if (sum < target) lo++; else hi--;
  }
  return null;
}
twoSumSorted([2, 7, 11, 15], 9);                // → [0, 1]
twoSumSorted([2, 7, 11, 15], 26);               // → [2, 3]
twoSumSorted([2, 7, 11, 15], 100);              // → null

// unsorted: a hash map is the O(n) answer
function twoSum(a: readonly number[], target: number): [number, number] | null {
  const seen = new Map<number, number>();
  for (let i = 0; i < a.length; i++) {
    const need = target - a[i]!;
    const j = seen.get(need);
    if (j !== undefined) return [j, i];
    seen.set(a[i]!, i);
  }
  return null;
}
twoSum([3, 2, 4], 6);                           // → [1, 2]

// three-sum: sort, then fix one and two-pointer the rest, O(n²)
function threeSum(input: readonly number[]): number[][] {
  const a = [...input].sort((x, y) => x - y);
  const out: number[][] = [];
  for (let i = 0; i < a.length - 2; i++) {
    if (i > 0 && a[i] === a[i - 1]) continue;     // skip duplicate anchors
    let lo = i + 1, hi = a.length - 1;
    while (lo < hi) {
      const sum = a[i]! + a[lo]! + a[hi]!;
      if (sum === 0) {
        out.push([a[i]!, a[lo]!, a[hi]!]);
        while (lo < hi && a[lo] === a[lo + 1]) lo++;
        while (lo < hi && a[hi] === a[hi - 1]) hi--;
        lo++; hi--;
      } else if (sum < 0) lo++;
      else hi--;
    }
  }
  return out;
}
threeSum([-1, 0, 1, 2, -1, -4]);                // → [[-1, -1, 2], [-1, 0, 1]]

// remove duplicates in place from a sorted array, returning the new length
function dedupeSorted(a: number[]): number {
  if (!a.length) return 0;
  let write = 1;
  for (let read = 1; read < a.length; read++) {
    if (a[read] !== a[write - 1]) a[write++] = a[read]!;
  }
  return write;
}
const dd = [1, 1, 2, 2, 3];
dedupeSorted(dd);                               // → 3
dd.slice(0, 3);                                 // → [1, 2, 3]

// container with most water
function maxArea(h: readonly number[]): number {
  let lo = 0, hi = h.length - 1, best = 0;
  while (lo < hi) {
    best = Math.max(best, Math.min(h[lo]!, h[hi]!) * (hi - lo));
    if (h[lo]! < h[hi]!) lo++; else hi--;         // always move the shorter side
  }
  return best;
}
maxArea([1, 8, 6, 2, 5, 4, 8, 3, 7]);           // → 49
```

---

## Sliding window

A contiguous range that grows on the right and shrinks on the left. The standard answer to "longest/shortest subarray satisfying X".

```ts
// fixed window: maximum sum of k consecutive elements
function maxSumWindow(a: readonly number[], k: number): number {
  let sum = 0;
  for (let i = 0; i < k; i++) sum += a[i]!;
  let best = sum;
  for (let i = k; i < a.length; i++) {
    sum += a[i]! - a[i - k]!;                     // add one, drop one
    best = Math.max(best, sum);
  }
  return best;
}
maxSumWindow([2, 1, 5, 1, 3, 2], 3);            // → 9

// variable window: longest substring with no repeated character
function longestUnique(s: string): number {
  const lastSeen = new Map<string, number>();
  let start = 0, best = 0;
  for (let i = 0; i < s.length; i++) {
    const ch = s[i]!;
    const prev = lastSeen.get(ch);
    if (prev !== undefined && prev >= start) start = prev + 1;
    lastSeen.set(ch, i);
    best = Math.max(best, i - start + 1);
  }
  return best;
}
longestUnique("abcabcbb");                      // → 3
longestUnique("bbbbb");                         // → 1
longestUnique("pwwkew");                        // → 3
longestUnique("");                              // → 0

// shortest subarray with a sum at least `target`
function minWindowSum(a: readonly number[], target: number): number {
  let start = 0, sum = 0, best = Infinity;
  for (let end = 0; end < a.length; end++) {
    sum += a[end]!;
    while (sum >= target) {
      best = Math.min(best, end - start + 1);
      sum -= a[start++]!;
    }
  }
  return best === Infinity ? 0 : best;
}
minWindowSum([2, 3, 1, 2, 4, 3], 7);            // → 2
minWindowSum([1, 1, 1], 100);                   // → 0

// longest substring with at most k distinct characters
function atMostKDistinct(s: string, k: number): number {
  const counts = new Map<string, number>();
  let start = 0, best = 0;
  for (let end = 0; end < s.length; end++) {
    const ch = s[end]!;
    counts.set(ch, (counts.get(ch) ?? 0) + 1);
    while (counts.size > k) {
      const left = s[start++]!;
      const n = counts.get(left)! - 1;
      if (n === 0) counts.delete(left); else counts.set(left, n);
    }
    best = Math.max(best, end - start + 1);
  }
  return best;
}
atMostKDistinct("eceba", 2);                    // → 3
atMostKDistinct("aa", 1);                       // → 2

// all anagram start-indices of p in s
function findAnagrams(s: string, p: string): number[] {
  if (p.length > s.length) return [];
  const need = new Map<string, number>();
  for (const ch of p) need.set(ch, (need.get(ch) ?? 0) + 1);
  const window = new Map<string, number>();
  const out: number[] = [];
  for (let i = 0; i < s.length; i++) {
    const add = s[i]!;
    window.set(add, (window.get(add) ?? 0) + 1);
    if (i >= p.length) {
      const drop = s[i - p.length]!;
      const n = window.get(drop)! - 1;
      if (n === 0) window.delete(drop); else window.set(drop, n);
    }
    if (i >= p.length - 1 &&
        window.size === need.size &&
        [...need].every(([ch, c]) => window.get(ch) === c)) {
      out.push(i - p.length + 1);
    }
  }
  return out;
}
findAnagrams("cbaebabacd", "abc");              // → [0, 6]
```

---

## Backtracking

Build a candidate incrementally, abandon it the moment it cannot work, undo, try the next. The shape is always: choose → explore → un-choose.

```ts
// all subsets (the power set), O(2ⁿ)
function subsets<T>(a: readonly T[]): T[][] {
  const out: T[][] = [];
  const cur: T[] = [];
  const go = (i: number): void => {
    if (i === a.length) { out.push([...cur]); return; }
    go(i + 1);                                    // exclude a[i]
    cur.push(a[i]!);
    go(i + 1);                                    // include a[i]
    cur.pop();                                    // un-choose
  };
  go(0);
  return out;
}
subsets([1, 2]);                                // → [[], [2], [1], [1, 2]]
subsets([1, 2, 3]).length;                      // → 8

// all permutations, O(n!)
function permutations<T>(a: readonly T[]): T[][] {
  const out: T[][] = [];
  const cur: T[] = [];
  const used = new Array<boolean>(a.length).fill(false);
  const go = (): void => {
    if (cur.length === a.length) { out.push([...cur]); return; }
    for (let i = 0; i < a.length; i++) {
      if (used[i]) continue;
      used[i] = true; cur.push(a[i]!);
      go();
      cur.pop(); used[i] = false;
    }
  };
  go();
  return out;
}
permutations([1, 2, 3]);
// → [[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]

// combinations of size k
function combinations(n: number, k: number): number[][] {
  const out: number[][] = [];
  const cur: number[] = [];
  const go = (start: number): void => {
    if (cur.length === k) { out.push([...cur]); return; }
    // prune: not enough numbers left to reach length k
    for (let i = start; i <= n - (k - cur.length) + 1; i++) {
      cur.push(i);
      go(i + 1);
      cur.pop();
    }
  };
  go(1);
  return out;
}
combinations(4, 2);
// → [[1, 2], [1, 3], [1, 4], [2, 3], [2, 4], [3, 4]]

// combination sum with unlimited reuse
function combinationSum(candidates: readonly number[], target: number): number[][] {
  const out: number[][] = [];
  const cur: number[] = [];
  const go = (start: number, remaining: number): void => {
    if (remaining === 0) { out.push([...cur]); return; }
    for (let i = start; i < candidates.length; i++) {
      const c = candidates[i]!;
      if (c > remaining) continue;
      cur.push(c);
      go(i, remaining - c);                       // i, not i+1 — reuse allowed
      cur.pop();
    }
  };
  go(0, target);
  return out;
}
combinationSum([2, 3, 6, 7], 7);                // → [[2, 2, 3], [7]]

// N-queens, counting solutions
function nQueens(n: number): number {
  const cols = new Set<number>();
  const diag = new Set<number>();                 // r - c
  const anti = new Set<number>();                 // r + c
  let count = 0;
  const place = (r: number): void => {
    if (r === n) { count++; return; }
    for (let c = 0; c < n; c++) {
      if (cols.has(c) || diag.has(r - c) || anti.has(r + c)) continue;
      cols.add(c); diag.add(r - c); anti.add(r + c);
      place(r + 1);
      cols.delete(c); diag.delete(r - c); anti.delete(r + c);
    }
  };
  place(0);
  return count;
}
nQueens(4);                                     // → 2
nQueens(6);                                     // → 4
nQueens(8);                                     // → 92

// generate valid parenthesis strings
function parens(n: number): string[] {
  const out: string[] = [];
  const go = (acc: string, open: number, close: number): void => {
    if (acc.length === 2 * n) { out.push(acc); return; }
    if (open < n) go(acc + "(", open + 1, close);
    if (close < open) go(acc + ")", open, close + 1);
  };
  go("", 0, 0);
  return out;
}
parens(3);                                      // → ['((()))', '(()())', '(())()', '()(())', '()()()']
```

The pruning conditions (`close < open`, the length check in `combinations`) are what separate a working solution from one that enumerates the whole space and filters — often the difference between milliseconds and minutes.

---

## Dynamic programming

Overlapping subproblems plus optimal substructure. Two directions: memoised recursion (top-down) or a table (bottom-up).

```ts
// naive Fibonacci: O(2ⁿ), recomputes the same values exponentially often
function fibSlow(n: number): number {
  return n <= 1 ? n : fibSlow(n - 1) + fibSlow(n - 2);
}
fibSlow(20);                                    // → 6765

// top-down with memoisation: O(n)
function fibMemo(n: number, memo = new Map<number, number>()): number {
  if (n <= 1) return n;
  const hit = memo.get(n);
  if (hit !== undefined) return hit;
  const v = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  memo.set(n, v);
  return v;
}
fibMemo(50);                                    // → 12586269025

// bottom-up, O(n) time O(1) space
function fib(n: number): number {
  let a = 0, b = 1;
  for (let i = 0; i < n; i++) [a, b] = [b, a + b];
  return a;
}
fib(50);                                        // → 12586269025

// a reusable memoiser
function memoize<A extends unknown[], R>(fn: (...a: A) => R): (...a: A) => R {
  const cache = new Map<string, R>();
  return (...args: A): R => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key)!;
    const v = fn(...args);
    cache.set(key, v);
    return v;
  };
}
const slowSquare = memoize((n: number) => n * n);
slowSquare(9);                                  // → 81
slowSquare(9);                                  // → 81    cached
```

### Classic problems

```ts
// climbing stairs — Fibonacci in disguise
function climbStairs(n: number): number {
  let a = 1, b = 1;
  for (let i = 0; i < n - 1; i++) [a, b] = [b, a + b];
  return b;
}
climbStairs(5);                                 // → 8

// coin change: fewest coins making `amount`
function coinChange(coins: readonly number[], amount: number): number {
  const dp = new Array<number>(amount + 1).fill(Infinity);
  dp[0] = 0;
  for (let a = 1; a <= amount; a++) {
    for (const c of coins) {
      if (c <= a && dp[a - c]! + 1 < dp[a]!) dp[a] = dp[a - c]! + 1;
    }
  }
  return dp[amount] === Infinity ? -1 : dp[amount]!;
}
coinChange([1, 5, 10, 25], 30);                 // → 2
coinChange([2], 3);                             // → -1
coinChange([1, 3, 4], 6);                       // → 2

// number of ways to make `amount` — note the loop order
function coinWays(coins: readonly number[], amount: number): number {
  const dp = new Array<number>(amount + 1).fill(0);
  dp[0] = 1;
  for (const c of coins) {                        // coins OUTSIDE: combinations
    for (let a = c; a <= amount; a++) dp[a]! += dp[a - c]!;
  }
  return dp[amount]!;
}
coinWays([1, 2, 5], 5);                         // → 4

// 0/1 knapsack
function knapsack(weights: readonly number[], values: readonly number[], cap: number): number {
  const dp = new Array<number>(cap + 1).fill(0);
  for (let i = 0; i < weights.length; i++) {
    for (let c = cap; c >= weights[i]!; c--) {    // BACKWARDS: each item once
      dp[c] = Math.max(dp[c]!, dp[c - weights[i]!]! + values[i]!);
    }
  }
  return dp[cap]!;
}
knapsack([1, 3, 4, 5], [1, 4, 5, 7], 7);        // → 9

// longest common subsequence
function lcs(a: string, b: string): number {
  const dp = Array.from({ length: a.length + 1 }, () => new Array<number>(b.length + 1).fill(0));
  for (let i = 1; i <= a.length; i++) {
    for (let j = 1; j <= b.length; j++) {
      dp[i]![j] = a[i - 1] === b[j - 1]
        ? dp[i - 1]![j - 1]! + 1
        : Math.max(dp[i - 1]![j]!, dp[i]![j - 1]!);
    }
  }
  return dp[a.length]![b.length]!;
}
lcs("abcde", "ace");                            // → 3
lcs("abc", "xyz");                              // → 0

// edit distance (Levenshtein)
function editDistance(a: string, b: string): number {
  let prev = Array.from({ length: b.length + 1 }, (_, j) => j);
  for (let i = 1; i <= a.length; i++) {
    const cur = [i, ...new Array<number>(b.length).fill(0)];
    for (let j = 1; j <= b.length; j++) {
      cur[j] = a[i - 1] === b[j - 1]
        ? prev[j - 1]!
        : 1 + Math.min(prev[j]!, cur[j - 1]!, prev[j - 1]!);
    }
    prev = cur;
  }
  return prev[b.length]!;
}
editDistance("kitten", "sitting");              // → 3
editDistance("", "abc");                        // → 3

// longest increasing subsequence, O(n log n) via patience sorting
function lis(a: readonly number[]): number {
  const tails: number[] = [];
  for (const v of a) {
    const i = lowerBound(tails, v);
    if (i === tails.length) tails.push(v); else tails[i] = v;
  }
  return tails.length;
}
lis([10, 9, 2, 5, 3, 7, 101, 18]);              // → 4
lis([7, 7, 7]);                                 // → 1

// house robber — no two adjacent
function rob(houses: readonly number[]): number {
  let prev = 0, cur = 0;
  for (const v of houses) [prev, cur] = [cur, Math.max(cur, prev + v)];
  return cur;
}
rob([2, 7, 9, 3, 1]);                           // → 12

// unique paths through a grid
function uniquePaths(rows: number, cols: number): number {
  const dp = new Array<number>(cols).fill(1);
  for (let r = 1; r < rows; r++) {
    for (let c = 1; c < cols; c++) dp[c]! += dp[c - 1]!;
  }
  return dp[cols - 1]!;
}
uniquePaths(3, 7);                              // → 28
```

| Signal | Reach for |
|---|---|
| "how many ways" | DP counting |
| "minimum / maximum cost" | DP optimisation |
| "is it possible" | DP boolean table |
| overlapping subcalls in a recursion | memoisation |
| each state depends only on the previous row | roll the table into one array |
| "all solutions", not just the count | backtracking, not DP |

The loop direction carries meaning. In `knapsack`, iterating capacity **backwards** is what makes each item usable once; forwards would make it unbounded. In `coinWays`, putting coins in the outer loop counts combinations, while swapping them counts permutations.

---

## Gotchas

| Trap | Reality |
|---|---|
| `[10, 9, 1].sort()` | lexicographic — `[1, 10, 9]`. Always pass a comparator |
| `arr.shift()` in a loop | O(n) each, O(n²) total — use a head index |
| `new Array(2).fill([])` | every slot is the *same* array |
| `arr[99]` on a short array | `undefined`, but typed `T` without `noUncheckedIndexedAccess` |
| `delete arr[0]` | leaves a hole and keeps `length`; use `splice` |
| `Object.keys` order | integer-like keys sort first, whatever the insertion order |
| object keys | always strings — `{1: "a"}` has key `"1"` |
| `map.get(objKey)` | compares by reference, not by value |
| `JSON.stringify(map)` | `'{}'` — convert with `[...map]` first |
| `"👋".length` | 2 — UTF-16 units, not characters |
| `+=` in a string loop | O(n²) in theory; collect and `join` |
| recursion depth | ~10,000 frames before a stack overflow |
| `0.1 + 0.2 === 0.3` | `false` — floating point |
| `NaN === NaN` | `false`; use `Number.isNaN` or `Object.is` |
| comparing arrays with `===` | reference identity, never contents |
| Dijkstra with negative weights | silently wrong; use Bellman-Ford |
| BFS marking visited on dequeue | duplicates in the queue, worse than O(V+E) |
| Floyd-Warshall loop order | `k` must be outermost |

```ts
// holes vs undefined
const holey = [1, 2, 3];
delete holey[1];
holey.length;                                   // → 3
holey[1];                                       // → undefined
// but a hole is not a value: map/filter SKIP it
holey.filter(() => true).length;                // → 2

// floating point
0.1 + 0.2;                                      // → 0.30000000000000004
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON;     // → true

// NaN
[NaN].indexOf(NaN);                             // → -1     uses ===
[NaN].includes(NaN);                            // → true   uses SameValueZero
new Set([NaN, NaN]).size;                       // → 1

// -0
Object.is(-0, 0);                               // → false
new Set([-0, 0]).size;                          // → 1      SameValueZero again

// array equality is reference identity
const arrA: number[] = [1, 2];
const arrB: number[] = [1, 2];
arrA === arrB;                                  // → false
JSON.stringify(arrA) === JSON.stringify(arrB);  // → true   cheap, order-sensitive
// (comparing two literals directly is a compile error in TS:
//  "This condition will always return 'false' since JavaScript compares
//   objects by reference, not value.")

// integer limits
Number.MAX_SAFE_INTEGER;                        // → 9007199254740991
Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2;   // → true
(2n ** 64n).toString().length;                  // → 20     use BigInt past 2^53

// sort mutates
const orig = [3, 1, 2];
const alsoOrig = orig.sort((a, b) => a - b);
orig;                                           // → [1, 2, 3]
alsoOrig === orig;                              // → true   same array!
// toSorted returns a copy
const src = [3, 1, 2];
src.toSorted((a, b) => a - b) === src;          // → false
```

---

## Cheat sheet

### Pick a structure

| Need | Use |
|---|---|
| ordered list, index access | `Array` |
| O(1) membership | `Set` |
| O(1) keyed lookup | `Map` |
| keys that may be garbage collected | `WeakMap` |
| LIFO | `Array` as a stack |
| FIFO | `Queue` with a head index |
| O(1) at both ends | `Deque` |
| fixed-size recent history | `RingBuffer` |
| always the min/max | `MinHeap` |
| "most important next" | `PriorityQueue` |
| sorted with range queries | balanced BST / AVL |
| prefix matching | `Trie` |
| grouping / connectivity | `UnionFind` |
| range sums, mutable data | `FenwickTree` |
| range min/max, mutable | `SegmentTree` |
| bounded cache | `LRUCache` |
| large numeric buffers | typed arrays |

### Operation costs

| Structure | Access | Search | Insert | Delete |
|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) |
| Array (end) | O(1) | — | O(1) | O(1) |
| Sorted array | O(1) | O(log n) | O(n) | O(n) |
| Singly linked list | O(n) | O(n) | O(1)* | O(1)* |
| Doubly linked list | O(n) | O(n) | O(1)* | O(1)* |
| Stack / Queue | — | — | O(1) | O(1) |
| Hash map (`Map`) | — | O(1) | O(1) | O(1) |
| Binary heap | O(1) peek | O(n) | O(log n) | O(log n) |
| BST (balanced) | O(log n) | O(log n) | O(log n) | O(log n) |
| BST (degenerate) | O(n) | O(n) | O(n) | O(n) |
| Trie | O(m) | O(m) | O(m) | O(m) |
| Union-Find | — | O(α(n)) | O(α(n)) | — |

\* given a reference to the node; O(n) if you must find it first.

### Pick an approach

| The problem says | Try |
|---|---|
| "sorted array" | binary search, two pointers |
| "contiguous subarray / substring" | sliding window, prefix sums |
| "top k" / "k-th largest" | heap, quickselect |
| "shortest path, unweighted" | BFS |
| "shortest path, weighted" | Dijkstra |
| "negative weights" | Bellman-Ford |
| "all pairs" | Floyd-Warshall |
| "ordering with dependencies" | topological sort |
| "connected / grouped" | Union-Find, BFS/DFS |
| "prefix" / "autocomplete" | Trie |
| "all combinations / permutations" | backtracking |
| "count the ways" / "min cost" | dynamic programming |
| "seen this before?" | `Set` or `Map` |
| "matching brackets / undo / nesting" | stack |
| "in-order / sorted traversal" | BST inorder |
| "maximise over a window" | monotonic deque |
| "overlapping intervals" | sort by start, then sweep |

```ts
// merge overlapping intervals — the sweep in full
function mergeIntervals(input: readonly [number, number][]): [number, number][] {
  if (!input.length) return [];
  const sorted = [...input].sort((a, b) => a[0] - b[0]);
  const out: [number, number][] = [sorted[0]!];
  for (const [start, end] of sorted.slice(1)) {
    const last = out[out.length - 1]!;
    if (start <= last[1]) last[1] = Math.max(last[1], end);
    else out.push([start, end]);
  }
  return out;
}
mergeIntervals([[1, 3], [2, 6], [8, 10], [15, 18]]);
// → [[1, 6], [8, 10], [15, 18]]
mergeIntervals([[1, 4], [4, 5]]);               // → [[1, 5]]
```
