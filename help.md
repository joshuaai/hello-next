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

You can place any of your custom React components or even a `div` within a `Link`. The only requirement for components placed inside a `Link` is they should accept an `onClick` prop.

## Using Shared Components
We can create a `page` by exporting a React component, and putting that component inside the `pages` directory. Then it will have a fixed URL based on the file name.

Let us create a `components/Header.js` component to be used across multiple screens:
```js
import Link from 'next/link'

const linkStyle = {
  marginRight: 15
}

const Header = () => (
    <div>
        <Link href="/">
          <a style={linkStyle}>Home</a>
        </Link>
        <Link href="/about">
          <a style={linkStyle}>About</a>
        </Link>
    </div>
)

export default Header
```
Import this into the `index` and `about` pages and simplify the code:
```js
import Header from '../components/Header'

const Index = () => (
  <div>
    <Header />
    <p>Hello Next.js</p>
  </div>
)

export default Index
```

## Create Dynamic Pages
In a real app, we need to create pages dynamically in order to display dynamic content.

We are starting with creating dynamic pages by using *query strings*.

Update the `pages/index.js` file thus:
```js
import Link from 'next/link'

const PostLink = (props) => (
  <li>
    <Link href={`/post?title=${props.title}`}>
      <a>{props.title}</a>
    </Link>
  </li>
)

const Index = () => (
  <Layout>
    <h1>My Blog</h1>
    <ul>
      <PostLink title="Hello Next.js"/>
      <PostLink title="Learn Next.js is awesome"/>
      <PostLink title="Deploy apps with Zeit"/>
    </ul>
  </Layout>
)

export default Index
```

Then add a `pages/post.js` component that will put the link to work:
```js
import Layout from '../components/MyLayout.js'

export default (props) => (
    <Layout>
       <h1>{props.url.query.title}</h1>
       <p>This is the blog post content.</p>
    </Layout>
)
```

Here's what's happening in the above code:

* Every page will get a prop called “URL” which has some details related to the current URL.
* In this case, we are using the “query” object, which has the query string params.
* Therefore, we get the title with `props.url.query.title`.

## Clean URLs with Route Masking
Route masking basically shows a different URL on the browser than the actual URL that your app sees.

In `pages/index.js`, replace the `PostLink` component with:
```js
const PostLink = (props) => (
  <li>
    <Link as={`/p/${props.id}`} href={`/post?title=${props.title}`}>
      <a>{props.title}</a>
    </Link>
  </li>
)

const Index = () => (
  <Layout>
    <h1>My Blog</h1>
    <ul>
      <PostLink id="hello-nextjs" title="Hello Next.js"/>
      <PostLink id="learn-nextjs" title="Learn Next.js is awesome"/>
      <PostLink id="deploy-nextjs" title="Deploy apps with Zeit"/>
    </ul>
  </Layout>
)
```

## Next.js Custom Server API - Server Side Support for Clean URLs
Now we are going to create a custom server for our app using Express. It's pretty simple.
```bash
npm install --save express
```

Add the express `server.js` file to the project:
```js
const express = require('express')
const next = require('next')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare()
.then(() => {
  const server = express()

  server.get('*', (req, res) => {
    return handle(req, res)
  })

  server.listen(3000, (err) => {
    if (err) throw err
    console.log('> Ready on http://localhost:3000')
  })
})
.catch((ex) => {
  console.error(ex.stack)
  process.exit(1)
})
```

Now update your npm dev script to:
```json
{
  "scripts": {
    "dev": "node server.js"
  }
}
```

Now we are going to add a custom route to match our blog post URLs. Add the new route to our `server.js` will look like this:
```js
server.get('/p/:id', (req, res) => {
  const actualPage = '/post'
  const queryParams = { title: req.params.id } 
  app.render(req, res, actualPage, queryParams)
})
```

## Fetching Data for Pages
In practice, we usually need to fetch data from a remote data source. Next.js comes with a standard API to fetch data for pages. We do it using an async function called `getInitialProps`.

First of all we need to install *isomorphic-unfetch*. That's the library we are going to use to fetch data. It's a simple implementation of the browser *fetch* API, but works both in client and server environments.
```bash
npm install --save isomorphic-unfetch
```

Update the `pages/index.js` to have:
```js
import Layout from '../components/Layout.js'
import Link from 'next/link'
import fetch from 'isomorphic-unfetch'

const Index = (props) => (
  <Layout>
    <h1>Batman TV Shows</h1>
    <ul>
      {props.shows.map(({show}) => (
        <li key={show.id}>
          <Link as={`/p/${show.id}`} href={`/post?id=${show.id}`}>
            <a>{show.name}</a>
          </Link>
        </li>
      ))}
    </ul>
  </Layout>
)

Index.getInitialProps = async function() {
  const res = await fetch('http://api.tvmaze.com/search/shows?q=batman')
  const data = await res.json()

  console.log(`Show data fetched. Count: ${data.length}`)

  return {
    shows: data
  }
}

export default Index
```

Now let's try to implement the “/post” page where it shows the detailed information about the TV show.

First, open the server.js and change the `/p/:id` route with the following content:
```js
server.get('/p/:id', (req, res) => {
    const actualPage = '/post'
    const queryParams = { id: req.params.id }
    app.render(req, res, actualPage, queryParams)
})
```

Now replace the `pages/post.js` with the following content:
```js
import Layout from '../components/Layout.js'
import fetch from 'isomorphic-unfetch'

const Post =  (props) => (
  <Layout>
    <h1>{props.show.name}</h1>
    <p>{props.show.summary.replace(/<[/]?p>/g, '')}</p>
    <img src={props.show.image.medium}/>
  </Layout>
)

Post.getInitialProps = async function (context) {
  const { id } = context.query
  const res = await fetch(`http://api.tvmaze.com/shows/${id}`)
  const show = await res.json()

  console.log(`Fetched show: ${show.name}`)

  return { show }
}

export default Post
```

## Styling Components
Next.js comes preloaded with a CSS in JS framework called *styled-jsx*.

It allows you to write familiar CSS rules for your components; rules will have no impact on anything other than the components (not even child components).

Now let's add some styles into our home page. Create a Blog `pages/blog.js` component with the code below:
```js
import Layout from '../components/Layout.js'
import Link from 'next/link'

function getPosts () {
  return [
    { id: 'hello-nextjs', title: 'Hello Next.js'},
    { id: 'learn-nextjs', title: 'Learn Next.js is awesome'},
    { id: 'deploy-nextjs', title: 'Deploy apps with ZEIT'},
  ]
}

const PostLink = ({ post }) => (
  <li>
    <Link as={`/p/${post.id}`} href={`/post?title=${post.title}`}>
      <a>{post.title}</a>
    </Link>
    <style jsx>{`
      li {
        list-style: none;
        margin: 5px 0;
      }

      a {
        text-decoration: none;
        color: blue;
        font-family: "Arial";
      }

      a:hover {
        opacity: 0.6;
      }
    `}</style>
  </li>
)

export default () => (
  <Layout>
    <h1>My Blog</h1>
    <ul>
      {getPosts().map((post) => (
        <PostLink key={post.id} post={post}/>
      ))}
    </ul>
    <style jsx>{`
      h1, a {
        font-family: "Arial";
      }

      ul {
        padding: 0;
      }

      li {
        list-style: none;
        margin: 5px 0;
      }

      a {
        text-decoration: none;
        color: blue;
      }

      a:hover {
        opacity: 0.6;
      }
    `}</style>
  </Layout>
)
```
CSS rules have no effect on elements inside of a child component. You need to style the child component directly.

### Global Styles
Sometimes, we do need to change styles inside of a child component. This is especially true when using markdown with React. You can see that on our post page. Apply the following content to `pages/blogPost.js`:
```js
import Layout from '../components/Layout.js'
import Markdown from '../components/Markdown'

export default (props) => (
  <Layout>
   <h1>{props.url.query.title}</h1>
   <div className="markdown">
     <Markdown content={`
This is our blog post.
Yes. We can have a [link](/link).
And we can have a title as well.

### This is a title

And here's the content.
     `}/>
   </div>
   <style jsx global>{`
     .markdown {
       font-family: 'Arial';
     }

     .markdown a {
       text-decoration: none;
       color: blue;
     }

     .markdown a:hover {
       opacity: 0.6;
     }

     .markdown h3 {
       margin: 0;
       padding: 0;
       text-transform: uppercase;
     }
  `}</style>
  </Layout>
)
```

Install the markdown package:
```bash
npm install marked --save
```

Add the `components/Markdown.js` component:
```js
import React from 'react'
import marked from 'marked'

marked.setOptions({
  gfm: true,
  tables: true,
  breaks: true,
  sanitize: false
})

export default class Markdown extends React.Component {
  render () {
    const { content } = this.props

    return (
      <div dangerouslySetInnerHTML={{ __html: marked(content) }}/>
    )
  }
}
```

## Deploying a Next.js App
First of all, we need to build our Next.js app for production. It will produce an optimized set of code for production.

For that, simply add the following npm script:
```json
"scripts": {
  "build": "next build"
}
```

Now we are going to run two instances of our app. Usually, we do this to horizontally scale our app. First of all, do the following changes to our start npm script:
```json
"scripts": {
  "start": "next start -p $PORT"
}
```

Then run: 
```bash
npm run build

PORT=8000 npm start
PORT=9000 npm start
```