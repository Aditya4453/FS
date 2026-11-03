# Topic 07: DOM Manipulation

---

### 1. Plain-Language Explanation (Fast Pace)
The **DOM (Document Object Model)** is the tree of C++/JavaScript objects the browser creates from your HTML.
When you want to change what is on the screen in vanilla JS, you query the DOM, change a property, or add/remove nodes:
* Select elements: `document.getElementById('id')`, `document.querySelector('.class')`, `document.querySelectorAll('tag')`.
* Create elements: `const newEl = document.createElement('div')`.
* Attach elements: `parentElement.appendChild(newEl)` or `parent.prepend(newEl)`.
* Remove elements: `element.remove()` or `parent.removeChild(element)`.
* Change content:
  * `element.textContent`: Sets raw plain text (Fast, safe, escapes `<script>` tags, does not parse HTML).
  * `element.innerHTML`: Parses string into live HTML tags (Slow, vulnerable to Cross-Site Scripting XSS).
* Modify classes: `element.classList.add('active')`, `classList.remove('hidden')`, `classList.toggle('dark-mode')`.

---

### 2. The Root Idea Behind It
Web pages used to be static text documents. JavaScript was added to manipulate this DOM tree on the fly so web pages could respond dynamically to user clicks without contacting a web server to reload the page.

---

### 3. Exact Code From Your Files
**Source File 1:** [`D:/_3rd Year/Portfolio_Assignment/client/src/App.jsx`](file:///D:/_3rd%20Year/Portfolio_Assignment/client/src/App.jsx#L42-L58)  
**Lines 42–58 & 83–98:**
```javascript
42:     const loaderBar = document.getElementById('loaderBar');
43:     const loaderPercentEl = document.getElementById('loaderPercent');
...
50:       onUpdate: () => {
51:         const p = Math.round(progressObj.value);
52:         setLoadPercent(p);
53:         if (loaderPercentEl) loaderPercentEl.textContent = `${p}%`;
54:         if (loaderBar) gsap.set(loaderBar, { scaleX: p / 100 });
55:       },
...
84:     const shapes = Array.from(document.querySelectorAll('.shape'));
85:     const hero = document.getElementById('hero');
...
97:     hero.addEventListener('mousemove', handleMouseMove);
98:     hero.addEventListener('mouseleave', handleMouseLeave);
```

**Source File 2 (From your course exam solutions):** [`D:/_3rd Year/FS/DOM_Challenge_Practice.pptx`](file:///D:/_3rd%20Year/FS/DOM_Challenge_Practice.pptx) & [`Manual_Solutions.pdf`](file:///D:/_3rd%20Year/FS/Manual_Solutions.pdf):
```javascript
// Exam Challenge: Add and Remove DOM elements dynamically
const addBtn = document.getElementById('addBtn');
const list = document.getElementById('myList');

addBtn.addEventListener('click', () => {
  const li = document.createElement('li');
  li.textContent = `Item ${list.children.length + 1}`;
  
  const delBtn = document.createElement('button');
  delBtn.textContent = 'Delete';
  delBtn.addEventListener('click', () => li.remove()); // remove node
  
  li.appendChild(delBtn);
  list.appendChild(li); // append node
});
```

#### Line-by-Line Walkthrough:
* **Line 42-43:** `document.getElementById('loaderBar')`: Direct reference to the DOM element with `id="loaderBar"`.
* **Line 53:** `loaderPercentEl.textContent = `${p}%``: Directly modifies the plain text content of the element every animation tick.
* **Line 84:** `Array.from(document.querySelectorAll('.shape'))`: `querySelectorAll` returns a static `NodeList`. Wrapping it in `Array.from()` converts it into a true JavaScript Array so methods like `.map()`, `.filter()`, and `.forEach()` can be used!
* **Line 97:** `hero.addEventListener('mousemove', handleMouseMove)`: Registers an event listener on the container DOM element.

---

### 4. How It Connects to Other Topics
* In Vanilla JS: Developer directly modifies the real DOM (`document.createElement`, `appendChild`).
* In React: React maintains an in-memory representation called the **Virtual DOM**. When state changes via `useState`, React computes the diff (reconciliation) and updates the real DOM automatically behind the scenes!

---

### 5. Common Mistakes & Exact Error Messages
* **Mistake 1: Calling array methods directly on `HTMLCollection`:**
  ```javascript
  const items = document.getElementsByClassName('item');
  items.map(el => el.textContent); // ERROR!
  ```
  * **Error:** `TypeError: items.map is not a function`.
  * **Why:** `HTMLCollection` is an array-like object, NOT an array.
  * **Fix:** `Array.from(items).map(...)` or `[...items].map(...)`.
* **Mistake 2: Querying DOM element before HTML is parsed:**
  * **Error:** `TypeError: Cannot read properties of null (reading 'addEventListener')`.
  * **Fix:** Place `<script>` at the bottom of `<body>` or use `defer`.

---

### 6. Optimized & Best-Practice Version
Avoid querying the DOM in loops. Use DocumentFragment to batch multiple additions in a single repaint:

```javascript
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
  const p = document.createElement('p');
  p.textContent = `Paragraph ${i}`;
  fragment.appendChild(p);
}
document.getElementById('container').appendChild(fragment); // Single browser reflow!
```
