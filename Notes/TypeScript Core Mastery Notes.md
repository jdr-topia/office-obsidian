---
tags:
  - front-end
  - back-end
---

- [[#🏷️ 1. The Type Hierarchy|🏷️ 1. The Type Hierarchy]]
- [[#📐 2. Structural Typing|📐 2. Structural Typing]]
- [[#🔗 3. Unions, Intersections, & Literals|🔗 3. Unions, Intersections, & Literals]]
- [[#🧬 4. Generics|🧬 4. Generics]]
- [[#🔍 5. Narrowing & Type Guards|🔍 5. Narrowing & Type Guards]]
- [[#🛠️ 6. Utility Types|🛠️ 6. Utility Types]]
- [[#⚙️ 7. The Compiler (`tsconfig.json`)|⚙️ 7. The Compiler (`tsconfig.json`)]]

## 🏷️ 1. The Type Hierarchy

Understanding the "levels" of types to avoid logic errors.

- **`any`**: 🛑 **The Danger Zone.** Turns off all type checking. Use only as a last resort.
- **`unknown`**: 🛡️ **The Safe Alternative.** Similar to `any`, but requires a type check (narrowing) before use.
- **`void`**: 📭 Used for functions that return no value.
- **`never`**: 🚫 Represents values that _cannot_ occur (e.g., a function that always throws an error).

---

## 📐 2. Structural Typing

TypeScript is "Shape-based," not "Name-based."

- **Definition**: If it walks like a duck and quacks like a duck, it’s a duck.
- **Logic**: If an object has the required properties of an interface, it is accepted, even if it has extra properties not defined in that interface. test.

---

## 🔗 3. Unions, Intersections, & Literals

The "Lego bricks" of type construction.

- **Unions (`|`)**: 🏗️ **"Either/Or."** Useful for state management.
    - `type Result = "success" | "error";`
- **Intersections (`&`)**: 🧬 **"Both/And."** Combines multiple types.
    - `type DraggableCircle = Circle & Draggable;`
- **Literal Types**: 🎯 Restricting a variable to an exact specific value (e.g., `let speed: 60;`).

---

## 🧬 4. Generics

Creating flexible, reusable components that "remember" their type.

- **The Concept**: Using a placeholder (usually `<T>`) that gets filled with a specific type when the code is actually called.
- **Analogy**: A shipping container that can hold anything, but once it's loaded with "Apples," it is treated strictly as an "Apple Container."

---

## 🔍 5. Narrowing & Type Guards

The art of making types more specific as the code executes.

- **`typeof`**: 🧪 Identifies primitives (string, number).
- **`instanceof`**: 🏛️ Identifies specific classes (Error, Date).
- **Discriminated Unions**: 🔑 Using a "tag" or "kind" property to let TS know exactly which object in a union you are working with.

---

## 🛠️ 6. Utility Types

Built-in "Type Transformers" to save time.

|**Icon**|**Utility**|**Effect**|
|---|---|---|
|🏗️|**`Partial<T>`**|Makes all properties **optional**.|
|🔒|**`Readonly<T>`**|Prevents properties from being **modified**.|
|✂️|**`Pick<T, K>`**|Selects only specific properties from a type.|
|🗑️|**`Omit<T, K>`**|Removes specific properties from a type.|

---

## ⚙️ 7. The Compiler (`tsconfig.json`)

Controlling how TypeScript behaves.

- **`strict: true`** 🦾: The most important setting. Prevents `null` bugs and implicit `any`.
- **`target`** 🎯: Set to `ESNext` for modern Node.js environments.
- **`experimentalStripTypes`** 🚀: New in Node.js 23+; allows running `.ts` files without a separate build step.