---
created: 2026-04-02
tags: [ui-architecture, react-patterns]
related: [ [[UI architecture MOC]], [[Headless components]] ]
status: active
---

# Compound components

A **compound component** is a pattern for building complex, state-sharing components where multiple sub-components work together to manage shared internal state.

## Why use it?

Instead of passing dozens of props (Prop Drilling) to a single monolithic component, we break the component down into smaller pieces that communicate implicitly.

### Comparison:
**Standard way (Prop Heavy):**
`<Select options={options} value={value} onChange={onChange} />`

**Compound way (Declarative):**
```tsx
<Select value={value} onChange={onChange}>
  <Select.Label>Select an option</Select.Label>
  <Select.Button />
  <Select.List>
    <Select.Option value="1">Option 1</Select.Option>
    <Select.Option value="2">Option 2</Select.Option>
  </Select.List>
</Select>
```

---

## How it works (The React Pattern)

It typically uses the **Context API** to share state between a parent component and its nested children.

1.  **Parent**: Creates a context provider and manages common state (e.g., `isOpen`).
2.  **Children**: Consume the context using `useContext` to read and update state.

```tsx
const SelectContext = React.createContext();

function Select({ children, ...props }) {
  const [isOpen, setOpen] = useState(false);
  return (
    <SelectContext.Provider value={{ isOpen, setOpen }}>
      {children}
    </SelectContext.Provider>
  );
}

Select.Button = function Button() {
  const { setOpen } = useContext(SelectContext);
  return <button onClick={() => setOpen(s => !s)}>Open</button>;
};
```

---

## Why Use Compound Components?

1.  **Cleaner Props**: No need to drill props through multiple layers.
2.  **Flexibility**: Consumers can reorder children or add their own custom markup between them.
3.  **Encapsulation**: State logic is hidden from the consumer but shared reliably.
4.  **Implicit State**: State is shared behind the scenes, making the API feel natural.

---
## Related Patterns
- [[Headless components]]
- [[UI architecture MOC]]
- [[Atomic design pattern]]
