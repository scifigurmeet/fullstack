# Next.js Introduction & File-Based Routing
**Duration: 1 Hour | Faculty Development Workshop**

## What is Next.js? (10 minutes)

Next.js is a React framework that provides:
- **Zero-config setup** - No complex webpack configurations
- **File-based routing** - Pages are created by adding files to the `pages` directory
- **Built-in optimizations** - Automatic code splitting, image optimization
- **Multiple rendering methods** - Static, server-side, and client-side rendering
- **API routes** - Build full-stack applications
- **Production ready** - Built by Vercel with performance in mind

### Why Next.js over plain React?
- **Routing**: No need for React Router
- **Performance**: Automatic optimizations
- **SEO**: Server-side rendering support
- **Developer Experience**: Hot reloading, error overlay
- **Production**: Easy deployment

## Setting Up Next.js (10 minutes)

### Prerequisites
- Node.js 16.8 or later
- npm or yarn package manager

### Create a New Next.js Project

```bash
# Using create-next-app (recommended)
npx create-next-app@latest my-nextjs-app

# Navigate to project
cd my-nextjs-app

# Start development server
npm run dev
```

### Manual Setup (Alternative)
```bash
# Create project folder
mkdir my-nextjs-app
cd my-nextjs-app

# Initialize npm
npm init -y

# Install Next.js, React, and React DOM
npm install next react react-dom

# Add scripts to package.json
```

Add these scripts to `package.json`:
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

## Project Structure Overview (5 minutes)

```
my-nextjs-app/
├── pages/          # File-based routing
├── public/         # Static assets
├── styles/         # CSS files
├── package.json
└── next.config.js  # Next.js configuration
```

## File-Based Routing Fundamentals (25 minutes)

### Basic Pages

Create files in the `pages` directory to define routes:

**pages/index.js** (Home page - `/`)
```jsx
export default function Home() {
  return (
    <div>
      <h1>Welcome to Next.js!</h1>
      <p>This is the home page</p>
    </div>
  )
}
```

**pages/about.js** (About page - `/about`)
```jsx
export default function About() {
  return (
    <div>
      <h1>About Us</h1>
      <p>This is the about page</p>
    </div>
  )
}
```

**pages/contact.js** (Contact page - `/contact`)
```jsx
export default function Contact() {
  return (
    <div>
      <h1>Contact</h1>
      <p>Get in touch with us</p>
    </div>
  )
}
```

### Nested Routes

Create folders to organize nested routes:

**pages/blog/index.js** (Blog home - `/blog`)
```jsx
export default function Blog() {
  return (
    <div>
      <h1>Blog</h1>
      <p>Welcome to our blog</p>
    </div>
  )
}
```

**pages/blog/first-post.js** (Blog post - `/blog/first-post`)
```jsx
export default function FirstPost() {
  return (
    <div>
      <h1>My First Post</h1>
      <p>This is my first blog post using Next.js!</p>
    </div>
  )
}
```

### Dynamic Routes

Use square brackets for dynamic segments:

**pages/blog/[slug].js** (Dynamic blog post - `/blog/any-slug`)
```jsx
import { useRouter } from 'next/router'

export default function BlogPost() {
  const router = useRouter()
  const { slug } = router.query

  return (
    <div>
      <h1>Blog Post: {slug}</h1>
      <p>This is a dynamic route for slug: {slug}</p>
    </div>
  )
}
```

**pages/user/[id].js** (User profile - `/user/123`)
```jsx
import { useRouter } from 'next/router'

export default function UserProfile() {
  const router = useRouter()
  const { id } = router.query

  return (
    <div>
      <h1>User Profile</h1>
      <p>User ID: {id}</p>
    </div>
  )
}
```

### Catch-All Routes

**pages/docs/[...slug].js** (Catches `/docs/a`, `/docs/a/b`, etc.)
```jsx
import { useRouter } from 'next/router'

export default function Docs() {
  const router = useRouter()
  const { slug } = router.query

  return (
    <div>
      <h1>Documentation</h1>
      <p>Path segments: {JSON.stringify(slug)}</p>
    </div>
  )
}
```

## Navigation Between Pages (8 minutes)

### Using the Link Component

**components/Navigation.js**
```jsx
import Link from 'next/link'

export default function Navigation() {
  return (
    <nav style={{ padding: '20px', borderBottom: '1px solid #eee' }}>
      <Link href="/">
        <a style={{ marginRight: '15px' }}>Home</a>
      </Link>
      <Link href="/about">
        <a style={{ marginRight: '15px' }}>About</a>
      </Link>
      <Link href="/blog">
        <a style={{ marginRight: '15px' }}>Blog</a>
      </Link>
      <Link href="/contact">
        <a>Contact</a>
      </Link>
    </nav>
  )
}
```

### Programmatic Navigation

```jsx
import { useRouter } from 'next/router'

export default function HomePage() {
  const router = useRouter()

  const handleClick = () => {
    router.push('/about')
  }

  return (
    <div>
      <h1>Home Page</h1>
      <button onClick={handleClick}>
        Go to About Page
      </button>
    </div>
  )
}
```

## Practical Exercise (10 minutes)

Create a simple blog structure with the following pages:

1. **Homepage** (`pages/index.js`) - Welcome message with navigation
2. **Blog listing** (`pages/blog/index.js`) - List of blog posts
3. **Individual blog post** (`pages/blog/[slug].js`) - Dynamic blog post page
4. **Author profile** (`pages/author/[name].js`) - Dynamic author page

### Complete Example

**pages/_app.js** (Add global navigation)
```jsx
import Navigation from '../components/Navigation'

export default function MyApp({ Component, pageProps }) {
  return (
    <>
      <Navigation />
      <main style={{ padding: '20px' }}>
        <Component {...pageProps} />
      </main>
    </>
  )
}
```

**pages/index.js**
```jsx
import Link from 'next/link'

export default function Home() {
  const posts = [
    { slug: 'getting-started-nextjs', title: 'Getting Started with Next.js' },
    { slug: 'react-hooks-guide', title: 'React Hooks Guide' },
  ]

  return (
    <div>
      <h1>Welcome to My Blog</h1>
      <h2>Recent Posts:</h2>
      <ul>
        {posts.map(post => (
          <li key={post.slug}>
            <Link href={`/blog/${post.slug}`}>
              <a>{post.title}</a>
            </Link>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

## Key Commands Summary

```bash
# Create new Next.js app
npx create-next-app@latest my-app

# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

## Quick Reference

| Route | File Path |
|-------|-----------|
| `/` | `pages/index.js` |
| `/about` | `pages/about.js` |
| `/blog` | `pages/blog/index.js` |
| `/blog/post-1` | `pages/blog/post-1.js` |
| `/blog/[slug]` | `pages/blog/[slug].js` |
| `/docs/[...slug]` | `pages/docs/[...slug].js` |

## Next Steps

In the next session, we'll explore:
- Static Site Generation (SSG)
- Server-Side Rendering (SSR)  
- Data fetching methods (`getStaticProps`, `getServerSideProps`)
- Performance optimizations

---

**Workshop Progress: 1/4 Complete** ✅