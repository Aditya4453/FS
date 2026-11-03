# React Routing Demo – Old Way vs New Way

## Run it
```bash
npm install
npm run dev
```
Open the URL Vite prints (usually http://localhost:5173) and open the browser console (F12).

## Switch between the two routing styles
Open `src/main.jsx` and change which line is commented:
```js
import App from "./old-way/AppOld";   // OLD WAY
// import App from "./new-way/AppNew"; // NEW WAY
```
The yellow strip under the header tells you which one is running.

## Folder structure
```
src/
├── main.jsx               ← choose old or new here
├── index.css
├── lifecycleLog.js        ← shows lifecycle messages on screen + console
├── old-way/AppOld.jsx     ← <BrowserRouter> + <Routes> + <Route>
├── new-way/AppNew.jsx     ← createBrowserRouter([...]) + <RouterProvider>
├── components/
│   ├── Layout.jsx         ← Header + LeftNavbar + <Outlet/> + RightNavbar + Footer
│   ├── Header.jsx         ← HOME / ABOUT / CONTACT links
│   ├── Footer.jsx
│   ├── LeftNavbar.jsx
│   └── RightNavbar.jsx    ← live lifecycle log
└── pages/
    ├── Home.jsx           ← useEffect: mount / update / unmount
    ├── About.jsx          ← + timer cleanup example
    ├── Contact.jsx        ← + update on every key press
    └── NotFound.jsx
```

## Old way vs New way

| | Old way (declarative) | New way (data router) |
|---|---|---|
| Setup | `<BrowserRouter>` | `createBrowserRouter([...])` |
| Routes written as | JSX `<Route>` tags | JavaScript objects |
| Where | Inside a component | Outside, at file level |
| Render | `<Routes>` | `<RouterProvider router={router} />` |
| Nested routes | `<Route>` inside `<Route>` | `children: [...]` |
| Home route | `<Route index …/>` | `{ index: true, … }` |
| Extras | — | `loader`, `action`, `errorElement` |

Both use the same `<Layout>` with `<Outlet />` — that's the spot where Home / About / Contact appear.

## useEffect = lifecycle
| Class lifecycle | useEffect version |
|---|---|
| componentDidMount | `useEffect(() => { … }, [])` |
| componentDidUpdate | `useEffect(() => { … }, [value])` |
| componentWillUnmount | `return () => { … }` inside the effect |
| every render | `useEffect(() => { … })` (no array) |

## Things to try in class
1. Click HOME → ABOUT: log shows 🔴 HOME unmounted, then 🟢 ABOUT mounted.
2. Click the Home button: 🟡 update. Go away and come back: the count resets to 0 (the component was destroyed).
3. On ABOUT, watch the console timer. Leave the page: it stops because of `clearInterval` in the cleanup. Remove that line and see what happens!
4. Header, footer and navbars never log anything — they stay mounted; only the middle changes.
5. Click "Broken link" → 404 page.

> `React.StrictMode` is left out of `main.jsx` on purpose: in development it mounts every component twice, which confuses the lifecycle demo. You can add it back to show students that effect.
