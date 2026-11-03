# Topic 18: 40 Exam-Style Practice Questions & Complete Solutions

---

## Part 1: Conceptual MCQs (15 Questions)

#### Q1. When sending an object from a React client to an Express server via `fetch`, what must you do?
* A) Send the object directly as `body: formData`
* B) Convert it using `JSON.stringify(formData)` and set `'Content-Type': 'application/json'`
* C) Convert it using `JSON.parse(formData)`
* D) Encode it using `encodeURI(formData)`

#### Q2. What happens if you forget `e.preventDefault()` inside a form's `onSubmit` handler?
* A) React throws a syntax error.
* B) The form inputs become read-only.
* C) The browser performs a full page reload, wiping component state and aborting fetch requests.
* D) Express rejects the request with HTTP 400.

#### Q3. In a controlled React form, what connects the typed keystrokes to the state?
* A) `value={state}` and `onChange={(e) => setState(e.target.value)}`
* B) `defaultValue={state}` and `onClick`
* C) `id={state}` and `onSubmit`
* D) `ref={state}` and `onKeyPress`

#### Q4. In `FS_20/models/products.js`, if `price` is defined with `type: String`, what will be the result of sorting `.sort({ price: 1 })`?
* A) Numerical sorting: 5, 20, 100
* B) Lexicographical string sorting: "100", "20", "5"
* C) MongoDB throws a runtime CastError
* D) The collection returns in natural insertion order

#### Q5. In Express, what is the role of `server.use(express.json())`?
* A) Converts outgoing server responses into JSON.
* B) Intercepts incoming requests with JSON bodies and parses the raw text stream into `req.body`.
* C) Protects routes from Cross-Site Scripting.
* D) Connects Express to MongoDB.

#### Q6. What does `useEffect(() => { ... }, [])` with an empty dependency array do?
* A) Runs on every render cycle.
* B) Runs exactly once after the initial component mount.
* C) Runs only when component unmounts.
* D) Never runs.

#### Q7. Which MongoDB operator is used to group documents and compute aggregate totals like sum or max?
* A) `$match`
* B) `$project`
* C) `$group`
* D) `$lookup`

#### Q8. What HTTP status code should be returned after successfully creating a new document in MongoDB?
* A) 200 OK
* B) 201 Created
* C) 204 No Content
* D) 304 Not Modified

#### Q9. How do you extract the parameter `category` from the route `/products/:category` in Express?
* A) `req.query.category`
* B) `req.body.category`
* C) `req.params.category`
* D) `req.headers.category`

#### Q10. What is the difference between `textContent` and `innerHTML`?
* A) `textContent` parses HTML tags; `innerHTML` escapes them.
* B) `textContent` sets plain text safely; `innerHTML` parses strings as HTML elements.
* C) `textContent` only works on `<p>` tags.
* D) They are completely identical.

#### Q11. In React, what is the primary purpose of `useCallback`?
* A) Caches the returned calculation of a function.
* B) Caches the function definition reference between renders to prevent re-rendering memoized child components.
* C) Runs asynchronous operations like `fetch`.
* D) Creates a mutable reference that does not cause re-renders.

#### Q12. What does an index in MongoDB do?
* A) Compresses documents on disk.
* B) Replaces `COLLSCAN` with `IXSCAN` using a B-Tree structure in RAM.
* C) Enforces strict relational schemas.
* D) Automatically backs up the database.

#### Q13. In a MongoDB aggregation pipeline, where should `$match` ideally be placed?
* A) At the very end of the pipeline.
* B) At the very beginning to filter documents before grouping or projecting.
* C) Exactly in the middle.
* D) `$match` cannot be used in aggregation pipelines.

#### Q14. Why is `HTMLCollection` different from a standard JavaScript array?
* A) It stores only numbers.
* B) It is a live array-like DOM object that lacks methods like `.map()`, `.filter()`, and `.reduce()`.
* C) It cannot be iterated with a `for` loop.
* D) It is supported only in Internet Explorer.

#### Q15. Why does CORS block a frontend request from `localhost:5173` to `localhost:9000` by default?
* A) The ports are different, making them different origins under the Same-Origin Policy.
* B) The frontend does not have an SSL certificate.
* C) MongoDB Atlas requires an API key.
* D) JavaScript cannot run network requests across localhost.

---

## Part 2: Predict the Output (10 Questions)

#### Q16. Event Loop & Microtask Priority:
What is the exact console output order?
```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
```

#### Q17. State Update Batching:
What does the alert display when the button is clicked once?
```javascript
function Counter() {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  };
  return <button onClick={handleClick}>{count}</button>;
}
```

#### Q18. Functional State Updater:
What is the value of `count` after one click?
```javascript
const handleClick = () => {
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
};
```

#### Q19. `textContent` vs `innerHTML`:
What appears visually on screen?
```javascript
const div = document.createElement('div');
div.textContent = '<h1>Hello World</h1>';
document.body.appendChild(div);
```

#### Q20. Closure Stale State in Timers:
What is logged after 3 seconds if the user clicks the button once, and then clicks it again at 1 second?
```javascript
function TimerBug() {
  const [count, setCount] = useState(0);
  const handleAlert = () => {
    setTimeout(() => {
      console.log('Count is:', count);
    }, 3000);
  };
  return (
    <>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={handleAlert}>Alert</button>
    </>
  );
}
```

#### Q21. Array Transformation with Higher-Order Functions:
What is the value of `result`?
```javascript
const items = [{ p: 10 }, { p: 20 }, { p: 30 }];
const result = items
  .filter(x => x.p > 15)
  .map(x => x.p * 2)
  .reduce((acc, curr) => acc + curr, 0);
console.log(result);
```

#### Q22. Express Middleware Execution Order:
What gets logged when `GET /test` is called?
```javascript
app.use((req, res, next) => { console.log('A'); next(); console.log('B'); });
app.use((req, res, next) => { console.log('C'); next(); });
app.get('/test', (req, res) => { console.log('D'); res.send('OK'); });
```

#### Q23. MongoDB Comparison Query:
Given collection: `[{ x: 5 }, { x: 10 }, { x: 15 }, { x: 20 }]`.
What documents match?
```javascript
db.items.find({ x: { $gt: 5, $lte: 15 } })
```

#### Q24. MongoDB Aggregation Output:
Given collection `[{ cat: 'A', v: 10 }, { cat: 'A', v: 20 }, { cat: 'B', v: 30 }]`.
What is the output of:
```javascript
db.items.aggregate([
  { $group: { _id: "$cat", total: { $sum: "$v" } } },
  { $sort: { total: -1 } }
])
```

#### Q25. `React.memo` Re-render Count:
How many times does `Child` render when the parent re-renders due to `text` changing?
```javascript
const Child = React.memo(({ onClick }) => {
  console.log("Child render");
  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const [text, setText] = useState('');
  const handleClick = () => console.log('clicked');
  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <Child onClick={handleClick} />
    </div>
  );
}
```

---

## Part 3: Coding Tasks (10 Tasks)

#### Task 26 (Classic Exam Problem): Fill in the 6 Blanks in `handleSignup`
Fill in blanks (1) through (6):
```javascript
import React, { useState } from 'react';

export default function SignupForm() {
  const [username, setUsername] = __(1)__('');
  const [password, setPassword] = useState('');

  const handleSignup = __(2)__(e) => {
    e.__(3)__();
    const userData = { username, password };
    console.log("Submitting:", userData);

    const res = await fetch('/api/signup', {
      method: 'POST',
      headers: { 'Content-Type': '__(4)__' },
      body: JSON.__(5)__(userData)
    });
  };

  return (
    <form onSubmit={__(6)__}>
      <input value={username} onChange={(e) => setUsername(e.target.value)} />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
      <button type="submit">Signup</button>
    </form>
  );
}
```

#### Task 27: Dynamic DOM Element Addition & Deletion
Write a vanilla JS script that:
1. Selects a container `#container`.
2. Creates a card `div` with class `card`.
3. Adds a title `<h3 class="card-title">` with text `User Card`.
4. Adds a delete button `<button>Delete</button>` that removes the card when clicked.
5. Appends the card to `#container`.

#### Task 28: Express REST API CRUD for Products
Write an Express router in `routes/productRoutes.js` supporting:
* `GET /`: returns all products.
* `POST /`: validates `name` and `price` from `req.body`, creates product with `Product.create()`, returns status 201.

#### Task 29: MongoDB Pagination Query
Write a query function taking `page` and `limit`, sorting products by `price` ascending, skipping the appropriate documents, and limiting results.

#### Task 30: MongoDB Aggregation Pipeline
Write an aggregation pipeline on a `sales` collection that:
1. Matches sales with `status: "completed"`.
2. Groups by `category`.
3. Calculates `totalRevenue` (`$sum` of `$amount`), `maxAmount` (`$max` of `$amount`), and `orderCount` (`$sum: 1`).
4. Sorts by `totalRevenue` descending.

#### Task 31: Generic React Controlled Change Handler
Write a generic `handleChange` function in React that updates a single `formData` state object for inputs with varied `name` attributes.

#### Task 32: Express Request Logger Middleware
Write an Express middleware that logs the HTTP method, request URL, and response time in milliseconds.

#### Task 33: React Optimization with `useMemo`
Given an array of 5,000 product objects, write a `useMemo` hook that filters items where `price < maxPrice` and sorts by `name`.

#### Task 34: Safe MongoDB Update with `$inc` and `$set`
Write a MongoDB command that updates user `user123`: increments `loginCount` by 1 and sets `lastLogin` to `new Date()`.

#### Task 35: React Custom Hook for Fetch
Write a custom hook `useFetch(url)` that returns `{ data, loading, error }`.

---

## Part 4: Debugging Tasks (5 Tasks)

#### Task 36: The `req.body` Await Bug (From `FS_20/server.js`)
Find the error in this code and rewrite it:
```javascript
server.post('/products', async (req, res) => {
  const data = await req.body;
  const postResult = await products.insertMany(data);
  res.send("data posted");
});
```

#### Task 37: The Missing `return` Header Error
Why does this Express controller crash?
```javascript
app.post('/api/contact', async (req, res) => {
  if (!req.body.email) {
    res.status(400).json({ error: 'Email required' });
  }
  const contact = await Contact.create(req.body);
  res.status(201).json(contact);
});
```

#### Task 38: The Broken `HTMLCollection` Map
Why does this line fail and how do you fix it?
```javascript
const buttons = document.getElementsByClassName('action-btn');
buttons.map(btn => btn.disabled = true);
```

#### Task 39: The Missing `_id` in `$group`
Fix the syntax error in this aggregation stage:
```javascript
db.orders.aggregate([
  { $group: { totalSales: { $sum: "$price" } } }
])
```

#### Task 40: The Infinite Render Loop
Why does this component freeze the browser?
```javascript
function ProductList() {
  const [products, setProducts] = useState([]);
  fetch('/api/products')
    .then(res => res.json())
    .then(data => setProducts(data));
  return <div>{products.length} products</div>;
}
```

---

## Complete Answer Key & Explanations

### MCQs
* **Q1: B** – `JSON.stringify` converts the live object to a string, and `Content-Type: application/json` instructs Express to run `express.json()`.
* **Q2: C** – Browser standard form submission triggers a full page reload.
* **Q3: A** – React controlled inputs pair `value={state}` with `onChange={(e) => setState(e.target.value)}`.
* **Q4: B** – Strings sort alphabetically ("100" < "20" < "5"), corrupting numerical logic.
* **Q5: B** – Reads incoming HTTP stream bytes and populates `req.body`.
* **Q6: B** – Empty dependency array runs only once after component mount.
* **Q7: C** – `$group` groups documents by `_id`.
* **Q8: B** – 201 Created signifies resource creation.
* **Q9: C** – Route parameters are populated in `req.params`.
* **Q10: B** – `textContent` safely sets raw text; `innerHTML` parses tags.
* **Q11: B** – Preserves function references across re-renders for memoized children.
* **Q12: B** – B-Tree in RAM allows binary search instead of full collection scans.
* **Q13: B** – Putting `$match` early filters rows before expensive grouping.
* **Q14: B** – `HTMLCollection` is an array-like DOM list, not a JS Array prototype.
* **Q15: A** – Port difference creates different origins under the Same-Origin Policy.

### Predict the Output
* **Q16 Output:**
  ```
  1
  4
  3
  2
  ```
  *Explanation:* `1` and `4` are synchronous. Promise microtask `3` runs immediately after call stack empties. Timer macrotask `2` runs on next turn.
* **Q17 Output:** `1` (Not 3!).
  *Explanation:* All three calls close over `count = 0`.
* **Q18 Output:** `3`.
  *Explanation:* Functional updater `prev => prev + 1` chains sequentially.
* **Q19 Output:** Literally the text string `<h1>Hello World</h1>` visible on screen. `textContent` escapes `<` and `>` into entities.
* **Q20 Output:** `Count is: 0`.
  *Explanation:* Stale closure! `handleAlert` captured `count = 0` when clicked.
* **Q21 Output:** `100`.
  *Explanation:* Filter leaves `[20, 30]`. Map gives `[40, 60]`. Reduce sums to `100`.
* **Q22 Output:**
  ```
  A
  C
  D
  B
  ```
  *Explanation:* Express middleware unwinds like a call stack. `console.log('B')` runs after `next()` finishes.
* **Q23 Output:** `[{ x: 10 }, { x: 15 }]`. (5 is excluded by `$gt`; 15 is included by `$lte`).
* **Q24 Output:**
  ```json
  [
    { "_id": "A", "total": 30 },
    { "_id": "B", "total": 30 }
  ]
  ```
* **Q25 Output:** `Child` re-renders on EVERY parent keystroke!
  *Explanation:* `handleClick` is re-created every render, passing a new reference to `Child`, breaking `React.memo`. Wrap with `useCallback` to fix.

### Coding Solutions
* **Task 26 Blanks:**
  (1) `useState`  
  (2) `async`  
  (3) `preventDefault`  
  (4) `application/json`  
  (5) `stringify`  
  (6) `handleSignup`
* **Task 27:**
  ```javascript
  const container = document.getElementById('container');
  const card = document.createElement('div');
  card.className = 'card';
  const title = document.createElement('h3');
  title.className = 'card-title';
  title.textContent = 'User Card';
  const btn = document.createElement('button');
  btn.textContent = 'Delete';
  btn.addEventListener('click', () => card.remove());
  card.appendChild(title);
  card.appendChild(btn);
  container.appendChild(card);
  ```
* **Task 28:**
  ```javascript
  const express = require('express');
  const router = express.Router();
  const Product = require('../models/Product');

  router.get('/', async (req, res) => {
    const products = await Product.find();
    res.status(200).json(products);
  });

  router.post('/', async (req, res) => {
    const { name, price } = req.body;
    if (!name || price === undefined) {
      return res.status(400).json({ error: 'Name and price required' });
    }
    const newProduct = await Product.create({ name, price });
    res.status(201).json(newProduct);
  });
  module.exports = router;
  ```
* **Task 29:**
  ```javascript
  const getPaged = async (page = 1, limit = 10) => {
    return await Product.find()
      .sort({ price: 1 })
      .skip((page - 1) * limit)
      .limit(limit);
  };
  ```
* **Task 30:**
  ```javascript
  db.sales.aggregate([
    { $match: { status: "completed" } },
    { $group: {
        _id: "$category",
        totalRevenue: { $sum: "$amount" },
        maxAmount: { $max: "$amount" },
        orderCount: { $sum: 1 }
    }},
    { $sort: { totalRevenue: -1 } }
  ]);
  ```

### Debugging Solutions
* **Task 36 Fix:** Remove `await` from `req.body` and return proper HTTP 201 status:
  ```javascript
  server.post('/products', async (req, res) => {
    try {
      const data = req.body;
      const postResult = await products.insertMany(data);
      res.status(201).json({ success: true, count: postResult.length });
    } catch (err) {
      res.status(400).json({ success: false, error: err.message });
    }
  });
  ```
* **Task 37 Fix:** Add `return` before `res.status(400)` so execution terminates:
  ```javascript
  if (!req.body.email) {
    return res.status(400).json({ error: 'Email required' });
  }
  ```
* **Task 38 Fix:** Convert `HTMLCollection` to an array:
  ```javascript
  Array.from(buttons).forEach(btn => btn.disabled = true);
  ```
* **Task 39 Fix:** Add `_id: null` to group across the whole collection:
  ```javascript
  db.orders.aggregate([
    { $group: { _id: null, totalSales: { $sum: "$price" } } }
  ]);
  ```
* **Task 40 Fix:** Wrap network fetch inside `useEffect` with dependency array `[]`:
  ```javascript
  useEffect(() => {
    fetch('/api/products')
      .then(res => res.json())
      .then(data => setProducts(data));
  }, []);
  ```
