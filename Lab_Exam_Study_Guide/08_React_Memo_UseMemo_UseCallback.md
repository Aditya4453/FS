# Topic 08: `React.memo`, `useMemo`, and `useCallback`

---

### 1. Plain-Language Explanation (Fast Pace)
By default in React, when a parent component re-renders, **every single child component re-renders too**, even if their props did not change!
* `React.memo(Component)`: Wraps a child component. Tells React: *"Only re-render this child if its props have actually changed."*
* `useMemo(() => computeValue(a, b), [a, b])`: **Caches a calculated value**. If you are filtering or sorting 10,000 items, `useMemo` prevents recalculating the array on every render unless the source data or filter query changed.
* `useCallback(() => { doSomething(); }, [dependencies])`: **Caches a function instance**. In JavaScript, functions are objects: `() => {} !== () => {}`. Every time a parent re-renders, it creates brand new function references, which breaks `React.memo` on children! `useCallback` keeps the function reference identical between renders.

---

### 2. The Root Idea Behind It
JavaScript compares objects and functions by reference identity, not structural value (`{} === {}` is `false`). Re-creating functions and arrays during render causes unnecessary child renders and expensive re-calculations. These three tools memoize memory references to preserve performance.

---

### 3. Exact Code From Your Files
**Source File:** [`D:/_3rd Year/Portfolio_Assignment/client/src/components/Projects.jsx`](file:///D:/_3rd%20Year/Portfolio_Assignment/client/src/components/Projects.jsx#L14-L24)  
**Lines 14–24:**
```javascript
14:   // Filter projects based on selected tag
15:   const filteredProjects = projects.filter((item) => {
16:     if (filter === 'All') return true;
17:     if (filter === 'AI & ML') return item.category.includes('AI') || item.category.includes('VISION') || item.category.includes('ML');
18:     if (filter === 'Full-Stack') return item.category.includes('FULL-STACK') || item.category.includes('TRANSIT');
19:     if (filter === 'Web3') return item.category.includes('WEB3');
20:     if (filter === 'Creative & Frontend') return item.category.includes('FRONT-END') || item.category.includes('CREATIVE');
21:     return true;
22:   });
```

#### Line-by-Line Walkthrough:
* **Current status in your code:** Notice that `filteredProjects` is calculated as an inline variable right inside the render body!
* Every time `expandedIndex` changes (when a user hovers or clicks an accordion row), the entire `Projects` component re-renders, and `.filter()` runs again over the entire project dataset.
* This is a prime exam optimization opportunity.

---

### 4. How It Connects to Other Topics
1. **Parent Re-render:** User clicks accordion or types in parent -> Parent re-renders.
2. **Without Memoization:** New function references and recalculated filters run -> Child components re-render.
3. **With `useMemo` / `useCallback`:** React checks dependency array -> Reuses cached reference -> `React.memo` on child sees identical props -> Child re-render is SKIPPED!

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Using `useMemo` for cheap operations:**
  ```javascript
  const sum = useMemo(() => a + b, [a, b]); // Anti-pattern!
  ```
  * Overhead of `useMemo` memory allocation is greater than the 0.0001ms cost of addition!
* **Mistake 2: Missing dependencies in `useCallback`:**
  ```javascript
  const handleDelete = useCallback(() => {
    deleteItem(selectedId);
  }, []); // missing selectedId!
  ```
  * **Result:** Stale closure! `handleDelete` always sees the initial `selectedId`.
* **Mistake 3: Passing inline arrow function to `React.memo` child:**
  ```javascript
  <MemoizedChild onClick={() => doSomething()} /> // New reference every render!
  ```
  * Breaks `React.memo` completely. Must wrap in `useCallback`.

---

### 6. Optimized & Best-Practice Version (From Your Code)
Here is the production-optimized version for your `Projects.jsx`:

```javascript
import React, { useState, useMemo, useCallback } from 'react';

// 1. Memoize the expensive filter calculation with useMemo
const filteredProjects = useMemo(() => {
  return projects.filter((item) => {
    if (filter === 'All') return true;
    if (filter === 'AI & ML') return item.category.includes('AI') || item.category.includes('VISION');
    if (filter === 'Full-Stack') return item.category.includes('FULL-STACK');
    return true;
  });
}, [projects, filter]); // Only recalculates when projects or filter changes!

// 2. Memoize toggle callback with useCallback
const toggleExpand = useCallback((idx) => {
  setExpandedIndex((prev) => (prev === idx ? null : idx));
}, []); // Stable reference across all renders
```
