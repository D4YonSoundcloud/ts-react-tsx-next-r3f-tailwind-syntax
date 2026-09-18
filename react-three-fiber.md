# React Three Fiber Syntax Reference

A single-page lookup for react-three-fiber v9 and drei with TypeScript: the JSX-to-three.js mapping, hooks, events, loaders, and the drei helpers worth knowing.

This page assumes the [TypeScript](./typescript.md) and [React](./react.md) references. It does not teach three.js concepts — it shows how they are spelled in JSX and how they are typed.

**How to use this page.** One long file, so browser find (`Ctrl+F` / `Cmd+F`) is the search tool. Search for the element or hook — `useFrame`, `args`, `attach`, `OrbitControls` — rather than a description.

| Mark | Meaning |
|---|---|
| `// →` | the value the expression produces at runtime |
| `// ^?` | the type TypeScript infers |
| `// ✗` | a compile or runtime error, followed by the message |
| `// ≤v8` | how this was written before R3F v9 |
| `// three:` | the equivalent in plain three.js |

Checked against **@react-three/fiber 9.7**, **@react-three/drei 10.7**, **three 0.186**, **@types/three 0.186**, **React 19.2** and **TypeScript** under `strict: true`.

---

## Contents

[Setup](#setup) · [The JSX mapping](#the-jsx-mapping) · [args](#args) · [Dashed props](#dashed-props) · [attach](#attach) · [Canvas](#canvas) · [Meshes](#meshes) · [Geometries](#geometries) · [Materials](#materials) · [Lights](#lights) · [Cameras](#cameras) · [Groups and transforms](#groups-and-transforms) · [useFrame](#useframe) · [useThree](#usethree) · [Refs](#refs) · [Events](#events) · [extend](#extend) · [Custom elements](#custom-elements) · [primitive](#primitive) · [Loading models](#loading-models) · [Textures](#textures) · [Suspense](#suspense-and-loading) · [Instancing](#instancing) · [Performance](#performance) · [drei: controls](#drei-controls) · [drei: staging](#drei-staging) · [drei: abstractions](#drei-abstractions) · [drei: text and HTML](#drei-text-and-html) · [drei: helpers](#drei-helpers) · [Animation](#animation) · [Shaders](#shaders) · [State management](#state-management) · [Idioms](#general-idioms) · [Gotchas](#gotchas) · [v8 → v9](#v8--v9) · [Versions](#version-notes)

---

## Setup

```bash
npm i three @react-three/fiber @react-three/drei
npm i -D @types/three
```

```jsonc
{
  "compilerOptions": {
    "target": "es2023",
    "lib": ["es2023", "dom", "dom.iterable"],
    "module": "esnext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true
  }
}
```

### Version pinning matters here

```jsonc
// @react-three/fiber 9.7 declares:
//   "peerDependencies": { "react": ">=19 <19.3", "react-dom": ">=19 <19.3", "three": ">=0.156" }
//
// R3F v9 requires React 19 — and, as of 9.7, does NOT yet accept React 19.3.
// Installing react@latest alongside it gives:
//   npm error Could not resolve dependency:
//   npm error peer react@">=19 <19.3" from @react-three/fiber@9.7.0
//
// Pin React to 19.2 for now, rather than reaching for --legacy-peer-deps.

// @react-three/drei 10.7 declares:
//   "peerDependencies": { "@react-three/fiber": "^9.0.0", "react": "^19",
//                         "react-dom": "^19", "three": ">=0.159" }
```

`@types/three` must match your `three` version — the three.js types change shape between releases, and a mismatch shows up as confusing errors on constructor arguments.

```bash
npm ls three @types/three @react-three/fiber @react-three/drei
```

### Next.js

```tsx
// R3F must run in the browser. In the App Router, mark the file:
"use client";

// and keep <Canvas> out of a Server Component's direct render path. The usual
// arrangement is a thin client wrapper imported by the server page.
```

---

## The JSX mapping

R3F is a React renderer for three.js. **Every three.js class is available as a JSX element in camelCase.** There is no component library to learn — if it exists on the `THREE` namespace, it is an element.

```tsx
// three:
//   const geometry = new THREE.BoxGeometry(1, 1, 1);
//   const material = new THREE.MeshStandardMaterial({ color: "orange" });
//   const mesh = new THREE.Mesh(geometry, material);
//   mesh.position.set(0, 1, 0);
//   scene.add(mesh);

// R3F:
<mesh position={[0, 1, 0]}>
  <boxGeometry args={[1, 1, 1]} />
  <meshStandardMaterial color="orange" />
</mesh>;
```

| three.js | JSX element |
|---|---|
| `THREE.Mesh` | `<mesh>` |
| `THREE.Group` | `<group>` |
| `THREE.BoxGeometry` | `<boxGeometry>` |
| `THREE.MeshStandardMaterial` | `<meshStandardMaterial>` |
| `THREE.DirectionalLight` | `<directionalLight>` |
| `THREE.PerspectiveCamera` | `<perspectiveCamera>` |
| `THREE.Points` | `<points>` |
| `THREE.InstancedMesh` | `<instancedMesh>` |
| `THREE.Vector3` | `<vector3>` (rare — usually a prop value) |

The rule: lowercase the first letter. `MeshPhysicalMaterial` → `<meshPhysicalMaterial>`.

```tsx
// the four elements R3F renames, because they clash with HTML/SVG tags:
// THREE.Audio, THREE.Line, THREE.Source and SVG's <path> are omitted from
// the intrinsic set. Reach them with <primitive> or extend():
import * as THREE from "three";
import { extend } from "@react-three/fiber";

extend({ ThreeLine: THREE.Line });
// then <threeLine />
```

### Where the types come from

```tsx
// v9 augments React's JSX namespace globally, so intrinsics work with no
// imports and no setup:
<mesh />;                           // just works

// the exported types, when you need to name one:
import type { ThreeElement, ThreeElements } from "@react-three/fiber";

type MeshProps = ThreeElements["mesh"];
type BoxProps = ThreeElements["boxGeometry"];

// ThreeElement<T> builds the prop type for any constructor
import { Mesh } from "three";
type M = ThreeElement<typeof Mesh>;
```

```tsx
// ≤v8 the props types were named differently and lived on a namespace:
//   import type { MeshProps, GroupProps } from "@react-three/fiber";
//   function Thing(props: MeshProps) { return <mesh {...props} /> }
//
// v9 replaces the per-element aliases with the ThreeElements lookup:
//   function Thing(props: ThreeElements["mesh"]) { return <mesh {...props} /> }
// The old names still exist for common elements, but the lookup is the
// one that works for every class, including ones you extend() yourself.
```

---

## args

`args` is the **constructor arguments**. It is spread into `new Klass(...args)` when the element mounts.

```tsx
// three:  new THREE.BoxGeometry(2, 1, 3)
<boxGeometry args={[2, 1, 3]} />;

// three:  new THREE.MeshStandardMaterial({ color: "red", roughness: 0.4 })
<meshStandardMaterial args={[{ color: "red", roughness: 0.4 }]} />;
// ...but constructor options are almost always settable as props instead:
<meshStandardMaterial color="red" roughness={0.4} />;

<perspectiveCamera args={[75, 1, 0.1, 1000]} />;
<fog args={["#000", 1, 100]} />;
<color args={["#222"]} attach="background" />;

// args is typed from the constructor, so a wrong arity or type is caught
// ✗ <boxGeometry args={["a", 1, 1]} />
//   Type 'string' is not assignable to type 'number | undefined'.
```

**Changing `args` remounts the object.** R3F cannot call a constructor again on an existing instance, so a new `args` array disposes and rebuilds it.

```tsx
// ✗ this rebuilds the geometry on every render — a new array each time
function Bad({ w }: { w: number }) {
  return <boxGeometry args={[w, 1, 1]} />;
}
// it is only rebuilt when the VALUES differ, so the above is fine in practice;
// the real trap is an inline object:
// ✗ <meshStandardMaterial args={[{ color: "red" }]} />   new object every render
// ✓ <meshStandardMaterial color="red" />
```

---

## Dashed props

A dash reaches into a nested property. `foo-bar-baz={v}` sets `instance.foo.bar.baz = v`.

```tsx
// three:  mesh.rotation.x = Math.PI / 2
<mesh rotation-x={Math.PI / 2} />;

// three:  material.color.set("red")
<meshStandardMaterial color-r={1} />;

<mesh position-y={2} scale-x={3} />;

// equivalent to the whole-value form
<mesh rotation={[Math.PI / 2, 0, 0]} />;

// dashed props are useful when you want to animate ONE axis without
// reconstructing the tuple every render
function Spin({ angle }: { angle: number }) {
  return <mesh rotation-y={angle}><boxGeometry /><meshNormalMaterial /></mesh>;
}
```

### Shorthand values

R3F converts arrays and scalars into three.js math objects for you.

| Prop type | Accepts |
|---|---|
| `Vector3` | `[x, y, z]`, a scalar (`2` → `2,2,2` for `scale`), or a `THREE.Vector3` |
| `Euler` | `[x, y, z]` in radians |
| `Color` | `"red"`, `"#ff0000"`, `0xff0000`, `[r, g, b]` |
| `Quaternion` | `[x, y, z, w]` |
| `Matrix4` | a 16-number array |

```tsx
<mesh position={[0, 1, 0]} scale={2} rotation={[0, Math.PI, 0]} />;
//                          ^ uniform scale shorthand

<meshStandardMaterial color="hotpink" />;
<meshStandardMaterial color="#ff69b4" />;
<meshStandardMaterial color={0xff69b4} />;

import * as THREE from "three";
<mesh position={new THREE.Vector3(0, 1, 0)} />;
```

---

## attach

`attach` tells R3F which **property** of the parent to assign the child to, rather than calling `parent.add(child)`.

```tsx
// geometry and material attach automatically by convention
<mesh>
  <boxGeometry />                  {/* → mesh.geometry */}
  <meshStandardMaterial />         {/* → mesh.material */}
</mesh>;

// anything else needs it spelled out
declare const map: THREE.Texture;
<mesh>
  <boxGeometry />
  <meshStandardMaterial map={map} />
</mesh>;

// `attach` as a CHILD works for anything that is a property of its parent:
<scene>
  <fog attach="fog" args={["#111", 5, 20]} />
</scene>;

// textures are the exception: <texture>'s args tuple is three's full 9-argument
// constructor, so the child form is impractical. Load with useTexture and pass
// the result as a prop, as above.
// ✗ <texture attach="map" args={[image]} />
//   Type '[HTMLImageElement]' is not assignable to type
//   '[image: unknown, mapping: Mapping, wrapS: Wrapping, …]'.

// scene-level attachments
<color attach="background" args={["#111"]} />;
<fog attach="fog" args={["#111", 5, 20]} />;
<fogExp2 attach="fog" args={["#111", 0.05]} />;

// nested paths use dashes here too
<bufferGeometry>
  <bufferAttribute attach="attributes-position" args={[new Float32Array(9), 3]} />
</bufferGeometry>;

// an array slot, for multi-material meshes
<mesh>
  <boxGeometry />
  <meshStandardMaterial attach="material-0" color="red" />
  <meshStandardMaterial attach="material-1" color="blue" />
</mesh>;
```

Without `attach`, a child is added to the parent's object graph — which is correct for meshes and groups, and wrong for everything that is a property.

---

## Canvas

`<Canvas>` creates the renderer, scene, camera and render loop, and sets up the event layer.

```tsx
"use client";
import { Canvas } from "@react-three/fiber";

export function Scene() {
  return (
    <Canvas
      camera={{ position: [3, 3, 3], fov: 50, near: 0.1, far: 100 }}
      gl={{ antialias: true, alpha: false }}
      dpr={[1, 2]}                    // clamp device pixel ratio
      shadows                         // enable the shadow map
      frameloop="always"              // "always" | "demand" | "never"
      orthographic={false}
      flat={false}                    // true → no tone mapping
      linear={false}                  // true → no sRGB conversion
      style={{ width: "100%", height: "100vh" }}
      onCreated={(state) => {
        state.gl.setClearColor("#111");
      }}
    >
      <ambientLight intensity={0.4} />
      <directionalLight position={[5, 5, 5]} castShadow />
      <mesh castShadow receiveShadow>
        <boxGeometry />
        <meshStandardMaterial color="orange" />
      </mesh>
    </Canvas>
  );
}
```

| Prop | Meaning |
|---|---|
| `camera` | partial props merged onto the default camera |
| `gl` | `WebGLRenderer` options, or a factory function |
| `dpr` | pixel ratio; `[min, max]` clamps it |
| `shadows` | `true`, or a shadow-map type like `"soft"` |
| `frameloop` | `"always"` renders continuously; `"demand"` only on change |
| `orthographic` | swap the default camera type |
| `flat` | disable tone mapping |
| `linear` | disable sRGB colour conversion |
| `scene` | props for the default scene, or your own |
| `raycaster` | raycaster options |
| `onCreated` | runs once with the full `RootState` |
| `onPointerMissed` | fires when a click hits nothing |
| `eventSource` / `eventPrefix` | attach events to a different DOM element |

**The canvas fills its parent.** Give the parent a real size, or you get a zero-height canvas — the single most common first-run problem.

```tsx
// ✓ the parent has height
<div style={{ height: "100vh" }}>
  <Canvas>{/* … */}</Canvas>
</div>;

// `frameloop="demand"` is the cheapest performance win for a static scene:
// nothing renders until state changes or you call invalidate().
import { useThree } from "@react-three/fiber";
function Invalidator() {
  const invalidate = useThree((s) => s.invalidate);
  invalidate();
  return null;
}
```

Everything inside `<Canvas>` is three.js JSX. Everything outside is DOM. You cannot mix them — a `<div>` inside `<Canvas>` throws, which is what drei's `<Html>` exists to solve.

```tsx
// ✗ <Canvas><div>hi</div></Canvas>
//   R3F: Div is not part of the THREE namespace! Did you forget to extend?
```

---

## Meshes

```tsx
import * as THREE from "three";
import { useRef } from "react";

function Box() {
  const ref = useRef<THREE.Mesh>(null!);
  //                            ^ null! — see Refs below

  return (
    <mesh
      ref={ref}
      position={[0, 0.5, 0]}
      rotation={[0, Math.PI / 4, 0]}
      scale={1}
      castShadow
      receiveShadow
      visible
      renderOrder={0}
      frustumCulled
      userData={{ id: "box-1" }}
    >
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="orange" />
    </mesh>
  );
}
```

Every writable property of `THREE.Mesh` and its ancestors (`Object3D`) is a prop. `castShadow`, `visible` and `frustumCulled` are booleans, so the bare attribute form works.

---

## Geometries

```tsx
<boxGeometry args={[width, height, depth, widthSeg, heightSeg, depthSeg]} />;
<sphereGeometry args={[radius, widthSeg, heightSeg]} />;
<planeGeometry args={[width, height, widthSeg, heightSeg]} />;
<cylinderGeometry args={[radiusTop, radiusBottom, height, radialSeg]} />;
<coneGeometry args={[radius, height, radialSeg]} />;
<torusGeometry args={[radius, tube, radialSeg, tubularSeg]} />;
<torusKnotGeometry args={[radius, tube, tubularSeg, radialSeg]} />;
<circleGeometry args={[radius, segments]} />;
<ringGeometry args={[innerRadius, outerRadius, thetaSeg]} />;
<capsuleGeometry args={[radius, length, capSeg, radialSeg]} />;
<icosahedronGeometry args={[radius, detail]} />;
<dodecahedronGeometry args={[radius, detail]} />;
<tetrahedronGeometry args={[radius, detail]} />;
<octahedronGeometry args={[radius, detail]} />;

declare const width: number, height: number, depth: number, radius: number;
declare const widthSeg: number, heightSeg: number, depthSeg: number;
declare const radialSeg: number, tubularSeg: number, segments: number;
declare const radiusTop: number, radiusBottom: number, tube: number;
declare const innerRadius: number, outerRadius: number, thetaSeg: number;
declare const length: number, capSeg: number, detail: number;
```

### Custom geometry

```tsx
import { useMemo } from "react";
import * as THREE from "three";

function Triangle() {
  const positions = useMemo(
    () => new Float32Array([0, 1, 0, -1, -1, 0, 1, -1, 0]),
    [],
  );

  return (
    <mesh>
      <bufferGeometry>
        <bufferAttribute
          attach="attributes-position"
          args={[positions, 3]}
        />
      </bufferGeometry>
      <meshBasicMaterial color="red" side={THREE.DoubleSide} />
    </mesh>
  );
}
```

`useMemo` on the typed array matters: a fresh `Float32Array` each render changes `args`, which rebuilds the attribute.

---

## Materials

```tsx
import * as THREE from "three";

// unlit — ignores lights, cheapest
<meshBasicMaterial color="red" wireframe transparent opacity={0.5} />;

// the workhorse: physically based
<meshStandardMaterial
  color="#88ccff"
  roughness={0.3}
  metalness={0.8}
  emissive="#000"
  emissiveIntensity={1}
  flatShading={false}
  side={THREE.FrontSide}
  transparent
  opacity={1}
  depthWrite
  toneMapped
/>;

// standard plus clearcoat, transmission, iridescence
<meshPhysicalMaterial
  transmission={1}
  thickness={0.5}
  ior={1.5}
  clearcoat={1}
  clearcoatRoughness={0}
/>;

<meshNormalMaterial />;                   // debug: normals as colour
<meshDepthMaterial />;
<meshLambertMaterial color="red" />;      // cheap diffuse
<meshPhongMaterial shininess={100} />;    // cheap specular
<meshToonMaterial color="red" />;
<meshMatcapMaterial />;
<pointsMaterial size={0.05} sizeAttenuation />;
<lineBasicMaterial color="white" linewidth={1} />;
<spriteMaterial />;
<shadowMaterial opacity={0.3} />;
```

| Material | Lit | Use for |
|---|---|---|
| `meshBasicMaterial` | no | flat colour, wireframes, UI |
| `meshStandardMaterial` | yes | almost everything |
| `meshPhysicalMaterial` | yes | glass, car paint, coated surfaces |
| `meshNormalMaterial` | no | debugging orientation |
| `meshToonMaterial` | yes | cel shading |
| `meshLambertMaterial` / `meshPhongMaterial` | yes | legacy, cheaper than standard |

```tsx
// side, from the THREE namespace, not a string
<meshBasicMaterial side={THREE.DoubleSide} />;
// ✗ <meshBasicMaterial side="double" />
//   Type 'string' is not assignable to type 'Side'.

// transparency needs both flags
<meshStandardMaterial transparent opacity={0.4} />;
// opacity alone does nothing without transparent
```

---

## Lights

```tsx
<ambientLight intensity={0.3} />;                       {/* uniform, no direction */}
<hemisphereLight args={["#fff", "#444", 1]} />;         {/* sky / ground */}
<directionalLight
  position={[5, 10, 5]}
  intensity={1}
  castShadow
  shadow-mapSize-width={2048}
  shadow-mapSize-height={2048}
  shadow-camera-far={50}
  shadow-camera-left={-10}
  shadow-camera-right={10}
/>;
<pointLight position={[0, 2, 0]} intensity={10} distance={10} decay={2} />;
<spotLight position={[0, 5, 0]} angle={0.3} penumbra={0.5} intensity={20} castShadow />;
<rectAreaLight args={["#fff", 5, 4, 2]} position={[0, 2, 0]} />;
```

The dashed-prop syntax is how you configure shadow cameras, which are nested several levels deep — `shadow-camera-far` is `light.shadow.camera.far`.

```tsx
// three.js r155+ uses physically correct lighting units, so intensities are
// much larger than older tutorials suggest: a pointLight at intensity={1}
// is nearly invisible. Start around 10–50 and adjust.
```

---

## Cameras

```tsx
// the usual approach: configure the default camera via <Canvas>
<Canvas camera={{ position: [0, 0, 5], fov: 50 }} />;

import { Canvas } from "@react-three/fiber";

<Canvas orthographic camera={{ zoom: 50, position: [0, 0, 100] }} />;

// a declarative camera: use DREI's, not the raw intrinsic. `makeDefault` is a
// drei prop — the bare three.js element has no such property:
import { PerspectiveCamera, OrthographicCamera } from "@react-three/drei";
<PerspectiveCamera makeDefault position={[0, 0, 5]} fov={50} />;
<OrthographicCamera makeDefault zoom={50} />;

// ✗ <perspectiveCamera makeDefault position={[0, 0, 5]} />
//   Property 'makeDefault' does not exist on type
//   'Mutable<Overwrite<Partial<Overwrite<PerspectiveCamera, ...>>>>'.
// The raw <perspectiveCamera> constructs a camera but does not register it as
// the one the renderer uses.
```

```tsx
// ✗ changing <Canvas camera={{...}}> after mount does nothing — the prop is
//   read once. To move the camera at runtime, use a ref, useThree, or
//   drei's <PerspectiveCamera makeDefault>.
```

---

## Groups and transforms

```tsx
// <group> is THREE.Group — a transform node with no geometry
<group position={[0, 1, 0]} rotation={[0, Math.PI / 4, 0]} scale={2}>
  <mesh position={[-1, 0, 0]}><boxGeometry /><meshNormalMaterial /></mesh>
  <mesh position={[1, 0, 0]}><sphereGeometry /><meshNormalMaterial /></mesh>
</group>;

// transforms compose down the tree, exactly like the three.js scene graph
// three:  group.add(mesh); group.position.y = 1;
```

Use groups to rotate a rig around a pivot without touching the children, and to keep a component's own transform separate from its parent's.

---

## useFrame

Runs on every rendered frame, before the render. This is the animation loop.

```tsx
import { useFrame } from "@react-three/fiber";
import { useRef } from "react";
import * as THREE from "three";

function Spinner() {
  const ref = useRef<THREE.Mesh>(null!);

  useFrame((state, delta, xrFrame) => {
    //      ^? RootState  ^? number   ^? XRFrame | undefined
    ref.current.rotation.y += delta;               // delta = seconds since last frame
    ref.current.position.y = Math.sin(state.clock.elapsedTime) * 0.5;
  });

  return <mesh ref={ref}><boxGeometry /><meshNormalMaterial /></mesh>;
}

// the signature:
//   function useFrame(callback: RenderCallback, renderPriority?: number): null
```

| Rule | Detail |
|---|---|
| **never call `setState` here** | it runs 60+ times a second; mutate the object instead |
| use `delta`, not a fixed step | frame rate varies; `delta` keeps motion frame-rate independent |
| `state.clock.elapsedTime` | total seconds since start, for oscillation |
| `renderPriority > 0` | takes over the render loop — **you must call `gl.render` yourself** |
| it does not re-render React | nothing in the component tree updates |

```tsx
// ✗ the mistake that tanks performance
function Bad() {
  const [y, setY] = useState(0);
  useFrame((_, d) => setY((v) => v + d));    // a React render every frame
  return <mesh position-y={y} />;
}

// ✓ mutate the instance directly
function Good() {
  const ref = useRef<THREE.Mesh>(null!);
  useFrame((_, d) => { ref.current.position.y += d; });
  return <mesh ref={ref}><boxGeometry /><meshNormalMaterial /></mesh>;
}
```

```tsx
// taking over the render loop
import { useThree } from "@react-three/fiber";
function Manual() {
  const { gl, scene, camera } = useThree();
  useFrame(() => {
    gl.render(scene, camera);
  }, 1);                                     // priority ≥ 1 disables auto-render
  return null;
}

// ordering: lower priority runs first. Use it to update a rig before the
// camera reads it.
```

---

## useThree

Reads the root state. It is a zustand store, so pass a selector to subscribe narrowly.

```tsx
import { useThree } from "@react-three/fiber";

function Readout() {
  // whole state — re-renders on ANY state change
  const state = useThree();
  //    ^? RootState

  // selector — re-renders only when the selected value changes. Prefer this.
  const camera = useThree((s) => s.camera);
  const size = useThree((s) => s.size);
  const gl = useThree((s) => s.gl);

  return null;
}
```

| `RootState` key | Type |
|---|---|
| `gl` | `THREE.WebGLRenderer` |
| `scene` | `THREE.Scene` |
| `camera` | `THREE.PerspectiveCamera \| THREE.OrthographicCamera` |
| `raycaster` | `THREE.Raycaster` |
| `clock` | `THREE.Clock` |
| `pointer` | `THREE.Vector2` — normalised −1..1 (`mouse` is the deprecated alias) |
| `size` | `{ width, height, top, left }` in CSS pixels |
| `viewport` | `{ width, height, factor, distance, aspect }` in three units |
| `controls` | whatever set itself via `makeDefault` |
| `scene`, `events`, `xr` | the event layer and XR manager |
| `set` / `get` | write and read the store |
| `invalidate` | request a frame under `frameloop="demand"` |
| `advance` | step the loop manually under `frameloop="never"` |

```tsx
// viewport vs size: size is pixels, viewport is world units at z=0.
// Use viewport to make something fill the screen regardless of camera distance.
function FullscreenPlane() {
  const { viewport } = useThree();
  return (
    <mesh scale={[viewport.width, viewport.height, 1]}>
      <planeGeometry />
      <meshBasicMaterial color="black" />
    </mesh>
  );
}

// useThree only works INSIDE <Canvas>
// ✗ calling it in the component that renders <Canvas> throws:
//   R3F: Hooks can only be used within the Canvas component!
```

---

## Refs

```tsx
import { useRef, useEffect } from "react";
import * as THREE from "three";

function Thing() {
  // the R3F convention: null! asserts non-null, because the ref is always
  // populated before useFrame or useEffect run
  const mesh = useRef<THREE.Mesh>(null!);
  const mat = useRef<THREE.MeshStandardMaterial>(null!);
  const group = useRef<THREE.Group>(null!);

  useEffect(() => {
    mesh.current.geometry.computeBoundingBox();
  }, []);

  return (
    <group ref={group}>
      <mesh ref={mesh}>
        <boxGeometry />
        <meshStandardMaterial ref={mat} color="red" />
      </mesh>
    </group>
  );
}
```

```tsx
// null! is a deliberate lie, and it is the community norm here because the
// alternative is optional-chaining every line of an animation loop:
//   useFrame(() => { ref.current?.rotation && (ref.current.rotation.y += 0.01) })
//
// The honest version, if you prefer it:
import { useFrame } from "@react-three/fiber";

function Honest() {
  const ref = useRef<THREE.Mesh | null>(null);
  useFrame((_, d) => {
    if (!ref.current) return;
    ref.current.rotation.y += d;
  });
  return <mesh ref={ref}><boxGeometry /><meshNormalMaterial /></mesh>;
}

// ≤v8 useRef<THREE.Mesh>(null!) was needed because RefObject.current was
// readonly. React 19 made it mutable, so useRef<THREE.Mesh | null>(null) now
// works for both reading and assigning — but null! is still shorter.
```

### Typing a ref for a drei component

```tsx
import { useRef } from "react";
import { OrbitControls } from "@react-three/drei";
import type { OrbitControls as OrbitControlsImpl } from "three-stdlib";

function Controls() {
  const ref = useRef<OrbitControlsImpl>(null!);
  return <OrbitControls ref={ref} />;
}
// drei re-exports the underlying classes from three-stdlib; the component
// and the implementation share a name, so alias one of them.

// the generic escape hatch, when you cannot find the class:
type ControlsRef = React.ComponentRef<typeof OrbitControls>;
```

---

## Events

Pointer events work on any object with geometry. R3F raycasts for you.

```tsx
import type { ThreeEvent } from "@react-three/fiber";
import { useState } from "react";

function Clickable() {
  const [hovered, setHovered] = useState(false);

  return (
    <mesh
      onClick={(e: ThreeEvent<MouseEvent>) => {
        e.stopPropagation();          // stop objects BEHIND this one receiving it
        console.log(e.point);         // THREE.Vector3 — the world hit position
        console.log(e.distance);      // number — camera to hit
        console.log(e.object);        // THREE.Object3D — what was hit
        console.log(e.eventObject);   // the object the handler is attached to
        console.log(e.face);          // THREE.Face | null
        console.log(e.uv);            // THREE.Vector2 | undefined
      }}
      onPointerOver={() => setHovered(true)}
      onPointerOut={() => setHovered(false)}
    >
      <boxGeometry />
      <meshStandardMaterial color={hovered ? "hotpink" : "orange"} />
    </mesh>
  );
}
```

| Handler | Fires |
|---|---|
| `onClick` | a full click |
| `onContextMenu` | right click |
| `onDoubleClick` | double click |
| `onPointerDown` / `onPointerUp` | press and release |
| `onPointerOver` / `onPointerOut` | enter and leave |
| `onPointerEnter` / `onPointerLeave` | same, non-bubbling |
| `onPointerMove` | movement over the object |
| `onPointerMissed` | a click that hit nothing (also a `<Canvas>` prop) |
| `onWheel` | scroll over the object |
| `onPointerCancel` / `onLostPointerCapture` | pointer capture lifecycle |

```tsx
// ThreeEvent<T> = IntersectionEvent<T> & Properties<T>
// so it carries both the raycast result and the native DOM event fields
<mesh onPointerMove={(e: ThreeEvent<PointerEvent>) => {
  e.nativeEvent;                    // the underlying DOM event
  e.ray;                            // THREE.Ray
  e.intersections;                  // every hit, sorted by distance
}} />;

// the cursor pattern
function Cursor() {
  const [hovered, setHovered] = useState(false);
  useEffect(() => {
    document.body.style.cursor = hovered ? "pointer" : "auto";
    return () => { document.body.style.cursor = "auto"; };
  }, [hovered]);
  return (
    <mesh onPointerOver={() => setHovered(true)} onPointerOut={() => setHovered(false)}>
      <boxGeometry /><meshNormalMaterial />
    </mesh>
  );
}
```

**Without `stopPropagation`, a click passes through to every object along the ray.** This surprises people coming from the DOM, where only the topmost element is hit.

---

## extend

Makes a class available as a JSX element. Two forms in v9.

```tsx
import * as THREE from "three";
import { extend, type ThreeElement } from "@react-three/fiber";

// form 1 — v9: pass a single class, get a COMPONENT back
class WobbleMaterial extends THREE.MeshStandardMaterial {}
const Wobble = extend(WobbleMaterial);
//    ^? React.ExoticComponent<ThreeElement<typeof WobbleMaterial>>

function UsesComponent() {
  return <mesh><boxGeometry /><Wobble color="red" /></mesh>;
}

// form 2 — the catalogue form, which registers lowercase intrinsics
extend({ WobbleMaterial });
// makes <wobbleMaterial /> available, but TypeScript does not know about it
// until you augment the namespace:
declare module "@react-three/fiber" {
  interface ThreeElements {
    wobbleMaterial: ThreeElement<typeof WobbleMaterial>;
  }
}

function UsesIntrinsic() {
  return <mesh><boxGeometry /><wobbleMaterial color="red" /></mesh>;
}
```

```tsx
// ≤v8 only the catalogue form existed, and the augmentation target was
// the global JSX namespace:
//   extend({ WobbleMaterial })
//   declare global {
//     namespace JSX {
//       interface IntrinsicElements {
//         wobbleMaterial: ReactThreeFiber.Object3DNode<WobbleMaterial, typeof WobbleMaterial>
//       }
//     }
//   }
//
// v9 moves the augmentation to the `ThreeElements` interface on the module,
// and replaces Object3DNode/BufferGeometryNode/MaterialNode with the single
// ThreeElement<T> helper.
```

The single-class form is the one to prefer: no name collisions, no module augmentation, and the component is a real value you can export.

```tsx
// extending a library class — the common real use
import { EffectComposer } from "three-stdlib";
const Composer = extend(EffectComposer);
```

---

## Custom elements

```tsx
import * as THREE from "three";
import { extend, type ThreeElement } from "@react-three/fiber";
import { shaderMaterial } from "@react-three/drei";

// drei's shaderMaterial builds a class with typed uniforms
const ColorMaterial = shaderMaterial(
  { uTime: 0, uColor: new THREE.Color("red") },
  /* vertex */ `
    void main() {
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  /* fragment */ `
    uniform float uTime;
    uniform vec3 uColor;
    void main() {
      gl_FragColor = vec4(uColor * (0.5 + 0.5 * sin(uTime)), 1.0);
    }
  `,
);

const ColorMaterialEl = extend(ColorMaterial);

function Shaded() {
  return (
    <mesh>
      <boxGeometry />
      <ColorMaterialEl uTime={0} uColor={new THREE.Color("cyan")} />
    </mesh>
  );
}
```

---

## primitive

Renders an **existing instance** rather than constructing one. This is how loaded models get into the tree.

```tsx
import * as THREE from "three";

function Existing({ object }: { object: THREE.Object3D }) {
  return <primitive object={object} position={[0, 1, 0]} />;
}

// primitive takes no args — the object already exists
// ✗ <primitive object={obj} args={[1]} />
//   Property 'args' does not exist ... (Omit<ThreeElement<any>, 'args'>)

// it also needs a key when the object identity changes, or React reuses
// the old instance:
declare const loaded: THREE.Object3D;
<primitive key={loaded.uuid} object={loaded} />;
```

```tsx
// primitive does NOT dispose the object when unmounted — you own its lifetime.
// That is deliberate: the same loaded model is often reused in several places.
```

---

## Loading models

```tsx
import { useGLTF } from "@react-three/drei";

function Model() {
  const { scene, nodes, materials, animations } = useGLTF("/model.glb");
  return <primitive object={scene} />;
}

// preload outside the render, so the fetch starts early
useGLTF.preload("/model.glb");
```

```tsx
// useLoader is the generic form, from fiber
import { useLoader } from "@react-three/fiber";
import { GLTFLoader } from "three-stdlib";

function Generic() {
  const gltf = useLoader(GLTFLoader, "/model.glb");
  return <primitive object={gltf.scene} />;
}

// an array input gives an array result — the type follows
function Many() {
  const [a, b] = useLoader(GLTFLoader, ["/a.glb", "/b.glb"]);
  return <><primitive object={a.scene} /><primitive object={b.scene} /></>;
}

// the loader can be configured before it runs
import { DRACOLoader } from "three-stdlib";
function Compressed() {
  const gltf = useLoader(GLTFLoader, "/model.glb", (loader) => {
    const draco = new DRACOLoader();
    draco.setDecoderPath("/draco/");
    loader.setDRACOLoader(draco);
  });
  return <primitive object={gltf.scene} />;
}
```

`useLoader` suspends. Every component using it must sit under a `<Suspense>`.

### Typing a GLTF result

`useGLTF` returns loosely typed `nodes` and `materials` — string-keyed records. For a model you control, generate the types.

```bash
npx gltfjsx public/model.glb --types --transform
```

```tsx
// gltfjsx emits a component plus a type like this
import * as THREE from "three";
import type { GLTF } from "three-stdlib";
import { useGLTF } from "@react-three/drei";

type ModelGLTF = GLTF & {
  nodes: {
    Body: THREE.Mesh;
    Wheel: THREE.Mesh;
  };
  materials: {
    Paint: THREE.MeshStandardMaterial;
  };
};

function Car() {
  const { nodes, materials } = useGLTF("/car.glb") as unknown as ModelGLTF;

  return (
    <group>
      <mesh geometry={nodes.Body.geometry} material={materials.Paint} />
      <mesh geometry={nodes.Wheel.geometry} material={materials.Paint} />
    </group>
  );
}
```

The `as unknown as` cast is unavoidable — the loader cannot know your model's contents. Generate the type from the file so at least it is derived from reality rather than guessed.

```tsx
// rendering nodes individually, rather than <primitive object={scene} />,
// lets you attach refs, events and different materials per part. It is the
// reason gltfjsx exists.
```

---

## Textures

```tsx
import { useTexture } from "@react-three/drei";
import * as THREE from "three";

function Textured() {
  const map = useTexture("/color.jpg");
  //    ^? THREE.Texture
  return <mesh><boxGeometry /><meshStandardMaterial map={map} /></mesh>;
}

// several at once, as a record — the keys become the prop names
function PBR() {
  const props = useTexture({
    map: "/color.jpg",
    normalMap: "/normal.jpg",
    roughnessMap: "/roughness.jpg",
    aoMap: "/ao.jpg",
  });
  return <mesh><boxGeometry /><meshStandardMaterial {...props} /></mesh>;
}

// as an array
function Arr() {
  const [a, b] = useTexture(["/a.jpg", "/b.jpg"]);
  return <mesh><boxGeometry /><meshBasicMaterial map={a} /></mesh>;
}

useTexture.preload("/color.jpg");
```

```tsx
// tiling and colour space
function Tiled() {
  const map = useTexture("/tile.jpg");
  map.wrapS = map.wrapT = THREE.RepeatWrapping;
  map.repeat.set(4, 4);
  map.colorSpace = THREE.SRGBColorSpace;      // colour maps only
  return <mesh><planeGeometry /><meshStandardMaterial map={map} /></mesh>;
}

// colour maps are sRGB; data maps (normal, roughness, metalness, ao) are NOT.
// Setting colorSpace on a normal map washes out your lighting.
```

---

## Suspense and loading

```tsx
import { Suspense } from "react";
import { Canvas } from "@react-three/fiber";
import { useProgress, Html, Loader } from "@react-three/drei";

function Fallback() {
  const { progress, loaded, total, active } = useProgress();
  //      ^? number  ^? number ^? number ^? boolean
  return <Html center>{progress.toFixed(0)}%</Html>;
}

export function App() {
  return (
    <Canvas>
      <Suspense fallback={<Fallback />}>
        <Model />
      </Suspense>
    </Canvas>
  );
}
declare function Model(): React.ReactNode;
```

```tsx
// a DOM overlay instead — <Loader> lives OUTSIDE the canvas
export function WithLoader() {
  return (
    <>
      <Canvas>
        <Suspense fallback={null}>
          <Model />
        </Suspense>
      </Canvas>
      <Loader />
    </>
  );
}

// a 3D fallback must itself be valid three.js JSX. A <div> is not.
// ✗ <Suspense fallback={<div>loading</div>}>   inside <Canvas>
// ✓ fallback={null}, a <mesh>, or drei's <Html>
```

---

## Instancing

One draw call for many copies. The difference between 200 objects and 200,000.

```tsx
import { useRef, useLayoutEffect } from "react";
import * as THREE from "three";

const COUNT = 1000;

function Field() {
  const ref = useRef<THREE.InstancedMesh>(null!);

  useLayoutEffect(() => {
    const dummy = new THREE.Object3D();
    for (let i = 0; i < COUNT; i++) {
      dummy.position.set(
        (Math.random() - 0.5) * 20,
        (Math.random() - 0.5) * 20,
        (Math.random() - 0.5) * 20,
      );
      dummy.rotation.set(Math.random(), Math.random(), 0);
      dummy.updateMatrix();
      ref.current.setMatrixAt(i, dummy.matrix);
    }
    ref.current.instanceMatrix.needsUpdate = true;
  }, []);

  return (
    <instancedMesh ref={ref} args={[undefined, undefined, COUNT]}>
      <boxGeometry args={[0.2, 0.2, 0.2]} />
      <meshStandardMaterial color="orange" />
    </instancedMesh>
  );
}
```

`args={[geometry, material, count]}` — pass `undefined` for the first two when you supply them as children. `count` must be there, and changing it remounts.

```tsx
// per-instance colour
function Coloured() {
  const ref = useRef<THREE.InstancedMesh>(null!);
  useLayoutEffect(() => {
    const c = new THREE.Color();
    for (let i = 0; i < COUNT; i++) {
      ref.current.setColorAt(i, c.setHSL(i / COUNT, 0.8, 0.5));
    }
    if (ref.current.instanceColor) ref.current.instanceColor.needsUpdate = true;
  }, []);
  return (
    <instancedMesh ref={ref} args={[undefined, undefined, COUNT]}>
      <sphereGeometry args={[0.1]} />
      <meshStandardMaterial />
    </instancedMesh>
  );
}
```

```tsx
// drei's <Instances> is the declarative version, and much easier to read
import { Instances, Instance } from "@react-three/drei";

function Declarative() {
  return (
    <Instances limit={1000}>
      <boxGeometry args={[0.2, 0.2, 0.2]} />
      <meshStandardMaterial />
      {Array.from({ length: 1000 }, (_, i) => (
        <Instance key={i} position={[i % 10, Math.floor(i / 10), 0]} color="orange" />
      ))}
    </Instances>
  );
}
```

---

## Performance

| Technique | Effect |
|---|---|
| `frameloop="demand"` | render only when something changes |
| `<instancedMesh>` / `<Instances>` | one draw call for many copies |
| merge geometries | fewer draw calls for static scenes |
| reuse geometry and material instances | avoid rebuilding per component |
| `dpr={[1, 2]}` | cap pixel ratio on high-DPI screens |
| drei `<AdaptiveDpr>` / `<AdaptiveEvents>` | degrade under load |
| `<Detailed>` | level of detail by distance |
| `<BakeShadows>` | render shadows once |
| `useMemo` on geometries and arrays | stop `args` churn |
| `<Bvh>` | much faster raycasting on complex meshes |

```tsx
import { useMemo } from "react";
import * as THREE from "three";

// share one geometry and material across many meshes
function Shared({ positions }: { positions: [number, number, number][] }) {
  const geometry = useMemo(() => new THREE.BoxGeometry(1, 1, 1), []);
  const material = useMemo(() => new THREE.MeshStandardMaterial({ color: "red" }), []);

  return (
    <>
      {positions.map((p, i) => (
        <mesh key={i} position={p} geometry={geometry} material={material} />
      ))}
    </>
  );
}
// you now own disposal — see Gotchas.
```

```tsx
import { PerformanceMonitor, AdaptiveDpr, Stats } from "@react-three/drei";
import { useState } from "react";

function Adaptive() {
  const [dpr, setDpr] = useState(1.5);
  return (
    <PerformanceMonitor onIncline={() => setDpr(2)} onDecline={() => setDpr(1)}>
      <AdaptiveDpr pixelated />
      <Stats />
    </PerformanceMonitor>
  );
}
```

---

## drei: controls

```tsx
import {
  OrbitControls, MapControls, TrackballControls,
  FlyControls, FirstPersonControls, PointerLockControls,
  TransformControls, CameraControls, ScrollControls, PresentationControls,
} from "@react-three/drei";

<OrbitControls
  makeDefault                       // registers as state.controls
  enablePan
  enableZoom
  enableRotate
  enableDamping
  dampingFactor={0.05}
  minDistance={2}
  maxDistance={20}
  minPolarAngle={0}
  maxPolarAngle={Math.PI / 2}
  target={[0, 0, 0]}
  autoRotate={false}
  autoRotateSpeed={1}
/>;

<MapControls makeDefault />;          {/* pan-first, for top-down scenes */}
<PointerLockControls />;              {/* FPS mouse capture */}
<CameraControls makeDefault />;       {/* smooth, imperative API */}
```

```tsx
// makeDefault matters: it puts the controls on state.controls, so drei
// components that need to read or disable them (TransformControls, Gizmo,
// useCursor) can find them.
import { TransformControls, OrbitControls } from "@react-three/drei";
function Editor() {
  return (
    <>
      <TransformControls mode="translate">
        <mesh><boxGeometry /><meshNormalMaterial /></mesh>
      </TransformControls>
      <OrbitControls makeDefault />
      {/* without makeDefault, dragging the gizmo also orbits the camera */}
    </>
  );
}
```

```tsx
// PresentationControls: bounded, spring-damped rotation for product shots
import { PresentationControls } from "@react-three/drei";
<PresentationControls
  global
  polar={[-0.4, 0.2]}
  azimuth={[-1, 0.75]}
  damping={0.2}
  speed={1}
  snap
>
  {/* note: the prop is `damping`, not `config` — there is no spring-config
      object on this component. ✗ config={{ mass: 2, tension: 400 }} */}
  <mesh><boxGeometry /><meshNormalMaterial /></mesh>
</PresentationControls>;
```

---

## drei: staging

The fastest route from "a grey box" to "looks rendered".

```tsx
import {
  Environment, Stage, Center, Bounds, ContactShadows,
  AccumulativeShadows, RandomizedLight, SoftShadows, Sky, Stars, Cloud,
} from "@react-three/drei";

// image-based lighting from a built-in HDRI — usually the single biggest
// visual improvement available
<Environment preset="city" />;
<Environment preset="sunset" background blur={0.5} />;
<Environment files="/studio.hdr" />;

// presets: "apartment" | "city" | "dawn" | "forest" | "lobby" | "night"
//        | "park" | "studio" | "sunset" | "warehouse"

// Stage bundles environment, lighting, shadows and centring
<Stage environment="city" intensity={0.5} shadows="contact" adjustCamera>
  <mesh><torusKnotGeometry /><meshStandardMaterial /></mesh>
</Stage>;

// centre the contents on the origin, whatever their bounds
<Center>
  <mesh position={[5, 5, 5]}><boxGeometry /><meshNormalMaterial /></mesh>
</Center>;

// frame the camera to fit
<Bounds fit clip observe margin={1.2}>
  <mesh><boxGeometry /><meshNormalMaterial /></mesh>
</Bounds>;

// cheap grounded shadow — no shadow map needed
<ContactShadows position={[0, -1, 0]} opacity={0.6} blur={2} far={4} />;

// high-quality baked soft shadows for a static scene
<AccumulativeShadows temporal frames={100} scale={10} position={[0, -0.99, 0]}>
  <RandomizedLight amount={8} position={[5, 5, -10]} />
</AccumulativeShadows>;

<Sky sunPosition={[100, 20, 100]} />;
<Stars radius={100} depth={50} count={5000} factor={4} />;
```

---

## drei: abstractions

```tsx
import {
  Box, Sphere, Plane, Cylinder, Cone, Torus, TorusKnot, Circle, Ring, Tube,
  RoundedBox, Line, QuadraticBezierLine, Edges, Outlines, Wireframe,
  Billboard, Image, Svg, Gltf, Clone, Mask, useMask,
  MeshReflectorMaterial, MeshTransmissionMaterial, MeshWobbleMaterial,
  MeshDistortMaterial, GradientTexture, Decal,
} from "@react-three/drei";

// shape shorthands: a mesh with the geometry already inside
<Box args={[1, 1, 1]} position={[0, 0, 0]}>
  <meshStandardMaterial color="orange" />
</Box>;
<Sphere args={[1, 32, 32]}><meshNormalMaterial /></Sphere>;
<RoundedBox args={[1, 1, 1]} radius={0.1} smoothness={4}>
  <meshStandardMaterial />
</RoundedBox>;

// lines that actually have width (three's linewidth is ignored on most GPUs)
<Line points={[[0, 0, 0], [1, 1, 0], [2, 0, 0]]} color="red" lineWidth={3} />;

// outlines and edges
<mesh><boxGeometry /><meshStandardMaterial /><Edges color="white" /></mesh>;
<mesh><boxGeometry /><meshStandardMaterial /><Outlines thickness={0.02} /></mesh>;

// always face the camera
<Billboard><mesh><planeGeometry /><meshBasicMaterial /></mesh></Billboard>;

// deep-clone a loaded scene so several copies can differ
<Clone object={someScene} />;
declare const someScene: import("three").Object3D;

// materials worth knowing
<mesh>
  <planeGeometry args={[10, 10]} />
  <MeshReflectorMaterial mirror={0.5} blur={[300, 100]} resolution={1024} mixBlur={1} />
</mesh>;

<mesh>
  <sphereGeometry />
  <MeshTransmissionMaterial thickness={0.5} roughness={0} transmission={1} ior={1.5} />
</mesh>;

<mesh>
  <sphereGeometry />
  <MeshWobbleMaterial factor={0.6} speed={2} color="hotpink" />
</mesh>;
```

---

## drei: text and HTML

```tsx
import { Text, Text3D, Html, useCursor } from "@react-three/drei";

// SDF text, rendered in WebGL — crisp at any distance, no DOM
<Text
  fontSize={1}
  color="white"
  anchorX="center"
  anchorY="middle"
  maxWidth={10}
  textAlign="center"
  font="/Inter.woff"
>
  Hello
</Text>;

// extruded geometry text, needs a typeface JSON
<Text3D font="/helvetiker_regular.typeface.json" size={1} height={0.2} bevelEnabled>
  Hello
  <meshStandardMaterial color="gold" />
</Text3D>;

// real DOM inside the scene, positioned in 3D
<Html
  position={[0, 1, 0]}
  center
  distanceFactor={10}               // scale with distance
  occlude                           // hide when behind geometry
  transform                         // apply the 3D transform to the DOM node
>
  <div style={{ background: "#fff", padding: 8 }}>A real div</div>
</Html>;
```

```tsx
// <Html> is the bridge back to the DOM. It is also the expensive option —
// each instance is a positioned DOM node updated every frame. For labels on
// hundreds of objects, use <Text> instead.

// useCursor: the hover-cursor pattern, packaged
import { useState } from "react";
function Hoverable() {
  const [hovered, setHovered] = useState(false);
  useCursor(hovered);
  return (
    <mesh onPointerOver={() => setHovered(true)} onPointerOut={() => setHovered(false)}>
      <boxGeometry /><meshNormalMaterial />
    </mesh>
  );
}
```

---

## drei: helpers

```tsx
import {
  Grid, GizmoHelper, GizmoViewport, GizmoViewcube,
  Stats, StatsGl, Helper, useHelper,
  Bvh, Detailed, Preload, AdaptiveDpr, AdaptiveEvents, BakeShadows,
  KeyboardControls, useKeyboardControls, ScrollControls, useScroll,
} from "@react-three/drei";
import { useRef } from "react";
import * as THREE from "three";

<Grid infiniteGrid cellSize={0.5} sectionSize={2} fadeDistance={30} />;

<GizmoHelper alignment="bottom-right" margin={[80, 80]}>
  <GizmoViewport axisColors={["red", "green", "blue"]} />
</GizmoHelper>;

<Stats />;                          {/* fps panel */}

// visualise a light's frustum while tuning shadows
function LightWithHelper() {
  const light = useRef<THREE.DirectionalLight>(null!);
  useHelper(light, THREE.DirectionalLightHelper, 1, "red");
  return <directionalLight ref={light} position={[5, 5, 5]} />;
}

// level of detail
<Detailed distances={[0, 10, 20]}>
  <mesh><sphereGeometry args={[1, 64, 64]} /><meshNormalMaterial /></mesh>
  <mesh><sphereGeometry args={[1, 16, 16]} /><meshNormalMaterial /></mesh>
  <mesh><sphereGeometry args={[1, 8, 8]} /><meshNormalMaterial /></mesh>
</Detailed>;

// faster raycasting via a bounding volume hierarchy
<Bvh><mesh><torusKnotGeometry args={[1, 0.4, 256, 64]} /><meshNormalMaterial /></mesh></Bvh>;
```

```tsx
// keyboard input
function Game() {
  return (
    <KeyboardControls
      map={[
        { name: "forward", keys: ["ArrowUp", "w", "W"] },
        { name: "back", keys: ["ArrowDown", "s", "S"] },
        { name: "jump", keys: ["Space"] },
      ]}
    >
      <Player />
    </KeyboardControls>
  );
}

function Player() {
  const [, get] = useKeyboardControls();
  useFrame((_, delta) => {
    const { forward, back } = get();
    if (forward) { /* move */ }
  });
  return null;
}
declare function useFrame(cb: (s: unknown, d: number) => void): null;
```

```tsx
// scroll-driven scenes
import { ScrollControls, useScroll, Scroll } from "@react-three/drei";

function Scrolly() {
  return (
    <ScrollControls pages={3} damping={0.2}>
      <Rig />
      <Scroll html>
        <h1 style={{ position: "absolute", top: "100vh" }}>Section two</h1>
      </Scroll>
    </ScrollControls>
  );
}

function Rig() {
  const scroll = useScroll();
  useFrame(() => {
    scroll.offset;                  // 0..1 through the whole scroll
    scroll.range(0, 1 / 3);         // 0..1 within the first third
    scroll.curve(0, 1 / 3);         // a bell curve over that range
  });
  return null;
}
```

---

## Animation

### Manual, with useFrame

```tsx
import { useFrame } from "@react-three/fiber";
import { useRef } from "react";
import * as THREE from "three";
import { MathUtils } from "three";

function Lerped({ target }: { target: number }) {
  const ref = useRef<THREE.Mesh>(null!);

  useFrame((_, delta) => {
    // frame-rate independent easing — damp, not lerp with a fixed alpha
    ref.current.position.y = MathUtils.damp(ref.current.position.y, target, 4, delta);
  });

  return <mesh ref={ref}><boxGeometry /><meshNormalMaterial /></mesh>;
}

// MathUtils.damp(current, target, lambda, delta) is the correct easing
// primitive here. `lerp(current, target, 0.1)` is framerate-dependent: it
// moves twice as fast at 120fps as at 60fps.
```

### GLTF animation clips

```tsx
import { useAnimations, useGLTF } from "@react-three/drei";
import { useEffect, useRef } from "react";
import * as THREE from "three";

function Animated() {
  const group = useRef<THREE.Group>(null!);
  const { scene, animations } = useGLTF("/character.glb");
  const { actions, names, mixer } = useAnimations(animations, group);

  useEffect(() => {
    actions[names[0]!]?.reset().fadeIn(0.3).play();
    return () => { actions[names[0]!]?.fadeOut(0.3); };
  }, [actions, names]);

  return <group ref={group}><primitive object={scene} /></group>;
}
```

### react-spring

```tsx
// npm i @react-spring/three
import { useSpring, animated } from "@react-spring/three";
import { useState } from "react";

function Springy() {
  const [open, setOpen] = useState(false);
  const { scale, rotation } = useSpring({
    scale: open ? 1.5 : 1,
    rotation: open ? [0, Math.PI, 0] : [0, 0, 0],
    config: { mass: 1, tension: 200, friction: 20 },
  });

  return (
    <animated.mesh scale={scale} rotation={rotation as never} onClick={() => setOpen((o) => !o)}>
      <boxGeometry />
      <meshNormalMaterial />
    </animated.mesh>
  );
}
```

`animated.*` mirrors the intrinsic elements. The `rotation` cast is a known rough edge — spring's animated-value types do not line up with three's `Euler` tuple.

---

## Shaders

```tsx
import * as THREE from "three";
import { useRef } from "react";
import { useFrame, extend } from "@react-three/fiber";
import { shaderMaterial } from "@react-three/drei";

const WaveMaterial = shaderMaterial(
  // uniforms — the object's keys become typed props
  { uTime: 0, uColor: new THREE.Color("#00aaff"), uAmplitude: 0.2 },
  // vertex shader
  `
    uniform float uTime;
    uniform float uAmplitude;
    varying vec2 vUv;
    void main() {
      vUv = uv;
      vec3 p = position;
      p.z += sin(p.x * 4.0 + uTime) * uAmplitude;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(p, 1.0);
    }
  `,
  // fragment shader
  `
    uniform vec3 uColor;
    varying vec2 vUv;
    void main() {
      gl_FragColor = vec4(uColor * vUv.y, 1.0);
    }
  `,
);

const WaveMaterialEl = extend(WaveMaterial);

function Wave() {
  const ref = useRef<THREE.ShaderMaterial & { uTime: number }>(null!);
  useFrame((state) => { ref.current.uTime = state.clock.elapsedTime; });

  return (
    <mesh>
      <planeGeometry args={[4, 4, 64, 64]} />
      <WaveMaterialEl ref={ref} uColor={new THREE.Color("#00aaff")} uAmplitude={0.3} />
    </mesh>
  );
}
```

```tsx
// the raw form, without drei
<mesh>
  <planeGeometry />
  <shaderMaterial
    uniforms={{ uTime: { value: 0 } }}
    vertexShader={`void main(){ gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0); }`}
    fragmentShader={`void main(){ gl_FragColor = vec4(1.0,0.0,0.0,1.0); }`}
  />
</mesh>;
// note: uniforms is NOT reactive — mutate uniforms.uTime.value in useFrame,
// do not rebuild the object.

// patching a built-in material instead of writing one from scratch
<meshStandardMaterial
  onBeforeCompile={(shader) => {
    shader.uniforms.uTime = { value: 0 };
    shader.vertexShader = `uniform float uTime;\n${shader.vertexShader}`;
  }}
/>;
```

---

## State management

```tsx
// zustand is the community default, and it is already a dependency of R3F
// npm i zustand
import { create } from "zustand";
import { useFrame } from "@react-three/fiber";
import { useRef } from "react";
import * as THREE from "three";

type Store = {
  selected: string | null;
  select: (id: string | null) => void;
};

const useStore = create<Store>((set) => ({
  selected: null,
  select: (selected) => set({ selected }),
}));

function Selectable({ id }: { id: string }) {
  const selected = useStore((s) => s.selected === id);
  const select = useStore((s) => s.select);

  return (
    <mesh onClick={() => select(id)}>
      <boxGeometry />
      <meshStandardMaterial color={selected ? "hotpink" : "gray"} />
    </mesh>
  );
}
```

```tsx
// for values that change every frame, keep them OUT of React entirely —
// a plain mutable object read in useFrame costs nothing
const mutable = { velocity: new THREE.Vector3() };

function Physics() {
  const ref = useRef<THREE.Mesh>(null!);
  useFrame((_, d) => {
    ref.current.position.addScaledVector(mutable.velocity, d);
  });
  return <mesh ref={ref}><boxGeometry /><meshNormalMaterial /></mesh>;
}

// the rule: React state for things the USER changes, plain mutation for
// things the CLOCK changes.
```

```tsx
// context crosses the Canvas boundary only if you bridge it — <Canvas> is a
// separate reconciler root, so a provider outside does not reach inside.
// zustand stores work fine (they are not context). For real React context:
//   import { useContextBridge } from "@react-three/drei"  (older versions)
// or simply put the provider INSIDE <Canvas>.
```

---

## General idioms

### A component per object

```tsx
// each object owns its ref, its animation and its state
import { useRef, useState } from "react";
import { useFrame } from "@react-three/fiber";
import * as THREE from "three";

function Cube({ position }: { position: [number, number, number] }) {
  const ref = useRef<THREE.Mesh>(null!);
  const [hovered, setHovered] = useState(false);

  useFrame((_, d) => { ref.current.rotation.y += hovered ? d * 2 : d * 0.2; });

  return (
    <mesh
      ref={ref}
      position={position}
      onPointerOver={(e) => { e.stopPropagation(); setHovered(true); }}
      onPointerOut={() => setHovered(false)}
    >
      <boxGeometry />
      <meshStandardMaterial color={hovered ? "hotpink" : "orange"} />
    </mesh>
  );
}

export function Row() {
  return (
    <>
      {[-2, 0, 2].map((x) => <Cube key={x} position={[x, 0, 0]} />)}
    </>
  );
}
```

### Forwarding props to the underlying object

```tsx
import type { ThreeElements } from "@react-three/fiber";

// accept every mesh prop, add your own
type WidgetProps = ThreeElements["mesh"] & { color?: string };

function Widget({ color = "orange", ...rest }: WidgetProps) {
  return (
    <mesh {...rest}>
      <boxGeometry />
      <meshStandardMaterial color={color} />
    </mesh>
  );
}

<Widget position={[0, 1, 0]} scale={2} castShadow color="red" onClick={() => {}} />;
```

### A reusable scene shell

```tsx
"use client";
import { Canvas } from "@react-three/fiber";
import { OrbitControls, Environment, Center } from "@react-three/drei";
import { Suspense } from "react";

export function Viewer({ children }: { children: React.ReactNode }) {
  return (
    <div style={{ height: "100vh" }}>
      <Canvas shadows dpr={[1, 2]} camera={{ position: [3, 3, 3], fov: 50 }}>
        <Suspense fallback={null}>
          <Center>{children}</Center>
          <Environment preset="city" />
        </Suspense>
        <OrbitControls makeDefault />
      </Canvas>
    </div>
  );
}
```

### Debug-only helpers

```tsx
import { Stats, Grid, GizmoHelper, GizmoViewport } from "@react-three/drei";

const DEV = process.env.NODE_ENV === "development";

function Debug() {
  if (!DEV) return null;
  return (
    <>
      <Stats />
      <Grid infiniteGrid />
      <GizmoHelper alignment="bottom-right"><GizmoViewport /></GizmoHelper>
    </>
  );
}
```

---

## Gotchas

| Trap | Reality |
|---|---|
| zero-height canvas | `<Canvas>` fills its parent; give the parent a height |
| `setState` in `useFrame` | a React render every frame — mutate the instance instead |
| a `<div>` inside `<Canvas>` | throws; use drei's `<Html>` |
| `useThree` outside `<Canvas>` | throws — hooks only work inside the root |
| no `e.stopPropagation()` | the click also hits every object behind it |
| changing `args` | remounts the object; memoise arrays and objects |
| `<Canvas camera={...}>` after mount | read once; use a ref or `makeDefault` |
| `opacity` without `transparent` | no effect |
| `lerp(a, b, 0.1)` in `useFrame` | frame-rate dependent; use `MathUtils.damp` |
| manually created geometry/material | R3F does not dispose it — you must |
| `useLoader` without `<Suspense>` | the whole tree suspends with no fallback |
| a new `Float32Array` per render | rebuilds the buffer attribute |
| light intensity from old tutorials | r155+ uses physical units; values are much larger |
| `colorSpace` on a normal map | washes out lighting; only colour maps are sRGB |
| React context from outside the canvas | separate reconciler root — put the provider inside |
| `react@19.3` with fiber 9.7 | peer range is `>=19 <19.3` |

```tsx
// disposal: R3F disposes what IT created (elements you wrote as JSX).
// Anything you `new`ed yourself is yours.
import { useEffect, useMemo } from "react";
import * as THREE from "three";

function Owned() {
  const geometry = useMemo(() => new THREE.BoxGeometry(), []);
  useEffect(() => () => { geometry.dispose(); }, [geometry]);
  return <mesh geometry={geometry}><meshNormalMaterial /></mesh>;
}

// opting a JSX element out of automatic disposal
<bufferGeometry dispose={null} />;
```

```tsx
// the event-propagation surprise, in full
<>
  <mesh position={[0, 0, 0]} onClick={() => console.log("front")}>
    <boxGeometry /><meshBasicMaterial transparent opacity={0.5} />
  </mesh>
  <mesh position={[0, 0, -2]} onClick={() => console.log("back")}>
    <boxGeometry /><meshBasicMaterial />
  </mesh>
</>;
// clicking the front box logs BOTH "front" and "back" — the ray continues.
// e.stopPropagation() in the front handler stops it.
```

```tsx
// StrictMode double-mounts in development, so anything created in an effect
// without cleanup is created twice. This is the same rule as React generally,
// but it bites harder here because three.js objects hold GPU memory.
```

---

## v8 → v9

| Area | v8 | v9 |
|---|---|---|
| React | 18 | **19 required** (`>=19 <19.3` as of 9.7) |
| element prop types | `MeshProps`, `GroupProps`, … | `ThreeElements["mesh"]`, … |
| the generic node helper | `Object3DNode<T, C>`, `MaterialNode`, `BufferGeometryNode` | one `ThreeElement<T>` |
| augmentation target | `declare global { namespace JSX { … } }` | `declare module "@react-three/fiber" { interface ThreeElements { … } }` |
| `extend(SingleClass)` | not supported | returns a ready-made component |
| `extend({ Catalogue })` | the only form | still supported |
| `useRef<T>(null!)` | needed — `current` was readonly | optional; React 19 made it mutable |
| `state.mouse` | current | deprecated alias for `state.pointer` |
| `<Canvas>` children | same | same |

```tsx
// the v8 augmentation, for reference when reading older code:
//   import { Object3DNode, extend } from "@react-three/fiber";
//   extend({ CustomMesh });
//   declare global {
//     namespace JSX {
//       interface IntrinsicElements {
//         customMesh: Object3DNode<CustomMesh, typeof CustomMesh>;
//       }
//     }
//   }
//
// the v9 equivalent:
//   import { extend, type ThreeElement } from "@react-three/fiber";
//   extend({ CustomMesh });
//   declare module "@react-three/fiber" {
//     interface ThreeElements {
//       customMesh: ThreeElement<typeof CustomMesh>;
//     }
//   }
//
// or skip augmentation entirely:
//   const CustomMeshEl = extend(CustomMesh);
```

Most v8 code compiles unchanged on v9. The work is React 19, the prop-type names, and any module augmentation.

---

## Version notes

| Feature | Requires |
|---|---|
| hooks-based R3F | fiber 4 |
| `<Canvas>` event layer, `useThree` selectors | fiber 8 |
| React 19, `ThreeElements`, `ThreeElement<T>` | **fiber 9** |
| `extend(SingleClass)` → component | fiber 9 |
| drei for fiber 9 | drei 10 |
| physical light units | three r155 |
| `colorSpace` replacing `encoding` | three r152 |
| WebGPU renderer | three r167+ (separate entry point) |
| `three-stdlib` for loaders and controls | any |

```bash
npm ls three @types/three @react-three/fiber @react-three/drei react
```

Keep `three` and `@types/three` on the same minor. R3F's peer range on `three` is wide (`>=0.156`), so npm will happily install a mismatched pair.

### Where to look things up

Nothing here replaces the three.js documentation — R3F is a renderer, not an API. When a prop is not in this page, it is a property of the three.js class, and the three.js docs are the reference. The mapping is mechanical: `<meshStandardMaterial roughness={0.4} />` is `material.roughness = 0.4`.
