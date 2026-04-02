---
created: 2026-04-02
tags: [ui-architecture, design-systems]
related: [ [[UI architecture MOC]], [[Design systems MOC]] ]
status: active
---

# Atomic design pattern

Atomic design is a methodology for creating design systems by treating UI components as basic building blocks. Proposed by Brad Frost, it helps in scaling component architecture from the smallest parts to full pages.

## The Hierarchy

### 1. Atoms
The smallest functional units. They cannot be broken down further without losing their functionality.
- **Example**: Buttons, Inputs, Labels, Icons.
- **Role**: Define basic styling (typography, colors, spacing).

### 2. Molecules
Groups of atoms functioning together as a unit.
- **Example**: A Search bar (Input atom + Button atom + Label atom).
- **Role**: Simple, reusable UI components with logical local state.

### 3. Organisms
Relatively complex UI components comprised of groups of molecules and/or atoms.
- **Example**: Global Navbar (Logo atom + Menu molecules + Search molecule).
- **Role**: Defining a distinct section of an interface.

### 4. Templates
Page-level objects that place components into a layout.
- **Example**: A "Dashboard Template" with a sidebar, header, and content grid area.
- **Role**: Defining the underlying content structure without actual data.

### 5. Pages
Specific instances of templates that show what a UI looks like with real content/data.
- **Example**: "User Profile Page" (Template + Alice's data).
- **Role**: Testing the effectiveness of the design system with real-world scenarios.

---

## Why Use Atomic Design?

1.  **Consistency**: Smallest pieces (Atoms) ensure the application looks uniform.
2.  **Scalability**: Reusing Molecules and Organisms reduces duplication.
3.  **Code Splitting**: Easier to maintain and split bundles by component type.
4.  **Documentation**: Provides a clear roadmap for what components exist.

## React Implementation Example

```tsx
// Atom
const Button = ({ label, onClick }) => <button className="btn" onClick={onClick}>{label}</button>;

// Molecule
const SearchBar = ({ onSearch }) => (
  <div className="search-bar">
    <Input placeholder="Search..." />
    <Button label="Go" onClick={onSearch} />
  </div>
);

// Organism
const Header = () => (
  <header>
    <Logo />
    <SearchBar />
    <UserMenu />
  </header>
);
```

---
## Related Patterns
- [[Compound components]]
- [[Design systems and shared libraries]]
- [[UI architecture MOC]]
