# Learning Next.js
To build server rendered JS web apps with React.

**Advantages:**
* Server-rendered by default
* Automatic code splitting for faster page loads
* Simple client-side routing (page based)
* Hot Module Replacement (Reload/Rendering)
* Able to implement with Express or any other Node.js HTTP server
* Customizable with your own Babel and Webpack configurations

## Setup
```bash
mkdir hello-next
cd hello-next
npm init -y
npm install --save react react-dom next
mkdir pages
```
Open the `package.json` in the `hello-next` directory and add to the `scripts` object:
```json
{
  "scripts": {
    "dev": "next"
  }
}
```
Create the index page at `pages/index.js`:
```js
const Index = () => (
  <div>
    <p>Hello Next.js</p>
  </div>
)

export default Index;
```

```bash
npm run dev
```

## Navigation
Create an about page at `pages/about.js`:
```js
export default () => (
  <div>
    <p>This is the about page</p>
  </div>
)
```

We will use Next's Link API for client side navigation. Update the `pages/index.js` file as below:
```js
import Link from 'next/link';

const Index = () => (
  <div>
    <Link href="/about"><a>About Page</a></Link>
    <p>Hello Next.js</p>
  </div>
)

export default Index;
```
You can add a style to the underlying component, `<a>` using jsx but not to `Link` directly as it is a higher order component (HOC) and only takes in certain props such as `href`.

You can place any of your custom React components or even a div within a Link. The only requirement for components placed inside a Link is they should accept an onClick prop.