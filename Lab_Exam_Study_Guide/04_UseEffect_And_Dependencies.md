# Topic 04: `useEffect` and Dependencies

---

### 1. Plain-Language Explanation (Fast Pace)
React components should primarily be pure functions: take props/state, return JSX. But real apps need to do things outside React: fetch data from an Express server, set document titles, run timers, attach scroll animations. These are called **Side Effects**.
* `useEffect(() => { /* effect code */ return () => { /* cleanup */ }; }, [dependencies]);`
* **Dependency Array Rules:**
  * No array: Runs on EVERY single render.
  * Empty array `[]`: Runs ONCE when component mounts (appears on screen).
  * Array with variables `[category, page]`: Runs on mount AND whenever `category` or `page` changes value.
  * Cleanup function (`return () => {...}`): Runs before re-running the effect and when the component unmounts (destroys timers, aborts fetches).

---

### 2. The Root Idea Behind It
React's rendering cycle must be synchronous to calculate UI differences. If you ran asynchronous network requests directly in the component body, the render would freeze or loop infinitely (calling `setPortfolio` inside the render body triggers another render, which triggers another fetch, forever). `useEffect` defers side effects until AFTER the browser has painted the screen.

---

### 3. Exact Code From Your Files
**Source File:** [`D:/_3rd Year/Portfolio_Assignment/client/src/App.jsx`](file:///D:/_3rd%20Year/Portfolio_Assignment/client/src/App.jsx#L22-L38)  
**Lines 22–38:**
```javascript
22:   // Fetch portfolio data from MERN Express API
23:   useEffect(() => {
24:     const fetchPortfolioData = async () => {
25:       try {
26:         const response = await fetch('/api/portfolio');
27:         if (response.ok) {
28:           const resJson = await response.json();
29:           if (resJson.success && resJson.data) {
30:             setPortfolio(resJson.data);
31:           }
32:         }
33:       } catch (err) {
34:         console.warn('API fetch defaulted to fallback data:', err.message);
35:       }
36:     };
37: 
38:     fetchPortfolioData();
39:   }, []);
```

#### Line-by-Line Walkthrough:
* **Line 23:** `useEffect(() => {`: Declares a side-effect hook.
* **Line 24:** `const fetchPortfolioData = async () => {`: Declares an inner async helper function. (React does not allow passing an `async` function directly as the top argument to `useEffect` because async functions return a Promise, whereas `useEffect` requires returning a cleanup function or nothing).
* **Line 26:** Calls the backend Express endpoint `/api/portfolio`.
* **Line 30:** `setPortfolio(resJson.data);`: Stores fetched projects and skills into React state.
* **Line 38:** `fetchPortfolioData();`: Executes the inner function.
* **Line 39:** `}, []);`: The empty array `[]` ensures this API call runs exactly once when the App loads, not on subsequent animation updates or re-renders!

---

### 4. How It Connects to Other Topics
1. **Mount:** Browser loads React app -> Component mounts.
2. **Initial Paint:** UI renders initial state (`fallbackPortfolioData` from Line 18).
3. **Effect Execution (Here):** React executes `useEffect` -> calls `fetch('/api/portfolio')`.
4. **Backend Processing:** Express route `/api/portfolio` responds with MongoDB data.
5. **State Update:** `setPortfolio(resJson.data)` triggers a re-render.
6. **Repaint:** React re-renders components (`Projects.jsx`, `Skills.jsx`) displaying live MongoDB data.

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Passing `async` directly to `useEffect`:**
  ```javascript
  useEffect(async () => { await fetch(...); }, []); // ILLEGAL
  ```
  * **Error Warning:** `Warning: useEffect must not return anything besides a function, which is used for clean-up. You returned: [object Promise].`
* **Mistake 2: Missing dependency causing stale values:**
  ```javascript
  useEffect(() => {
    fetch(`/api/products?cat=${category}`);
  }, []); // missing category in deps!
  ```
  * **Bug:** When user changes category filter, the effect never re-runs; old products remain on screen.
* **Mistake 3: Infinite loop by mutating dependency inside effect:**
  ```javascript
  useEffect(() => {
    setData(data + 1); // mutates data
  }, [data]);          // triggers re-run because data changed -> INFINITE LOOP!
  ```

---

### 6. Optimized & Best-Practice Version (With AbortController Cleanup)
Always implement an `AbortController` cleanup so rapid unmounting or prop changes do not cause race conditions or memory leaks:

```javascript
useEffect(() => {
  const controller = new AbortController();

  const loadData = async () => {
    try {
      const res = await fetch(`/api/portfolio`, { signal: controller.signal });
      if (!res.ok) throw new Error(`HTTP error: ${res.status}`);
      const json = await res.json();
      setPortfolio(json.data);
    } catch (err) {
      if (err.name !== 'AbortError') {
        console.error('Fetch error:', err.message);
      }
    }
  };

  loadData();

  // Cleanup function: runs on unmount or before next effect run
  return () => controller.abort();
}, []);
```
