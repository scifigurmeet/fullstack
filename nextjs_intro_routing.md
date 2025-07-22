# Next.js Introduction & File-Based Routing (App Router)
**Duration: 1 Hour | Faculty Development Workshop** 🚀

## 🤔 What is Next.js? (10 minutes)

Next.js = **React with superpowers** 🔋

**Key Benefits:**
- 🎯 Zero-config setup  
- 📁 File-based routing (folder = page)
- ⚡ Built-in optimizations
- 🔌 Full-stack capabilities

**App Router vs Pages Router:**
- **Old:** `pages/about.js` 
- **New:** `app/about/page.tsx` ✅

---

## 🛠️ Setup (10 minutes)

```bash
# Create project
npx create-next-app@latest my-app

# Choose: TypeScript✅, App Router✅, Tailwind✅

cd my-app
npm run dev
```

**Visit:** `http://localhost:3000` 🎉

---

## 📁 Project Structure (5 minutes)

```
my-app/
├── 📁 app/                    👈 Routes go here
│   ├── 📄 layout.tsx          👈 Wraps all pages
│   ├── 📄 page.tsx            👈 Homepage (/)
│   └── 📄 globals.css         
├── 📁 public/                 👈 Static files
└── 📄 package.json            
```

---

## 🗺️ File-Based Routing (25 minutes)

### 🏠 Basic Routes

```
📁 app/
├── 📄 page.tsx              👈 /
├── 📁 about/
│   └── 📄 page.tsx          👈 /about
└── 📁 contact/
    └── 📄 page.tsx          👈 /contact
```

**Create `app/about/page.tsx`:**
```tsx
export default function About() {
  return (
    <div className="p-8">
      <h1>👋 About Page</h1>
      <p>URL: /about</p>
    </div>
  )
}
```

### 📂 Nested Routes

```
📁 app/
└── 📁 blog/
    ├── 📄 page.tsx              👈 /blog
    └── 📁 first-post/
        └── 📄 page.tsx          👈 /blog/first-post
```

**Create `app/blog/page.tsx`:**
```tsx
import Link from 'next/link'

export default function Blog() {
  return (
    <div className="p-8">
      <h1>📝 Blog</h1>
      <Link href="/blog/first-post" className="text-blue-500">
        Read First Post →
      </Link>
    </div>
  )
}
```

### 🎯 Dynamic Routes

```
📁 app/
└── 📁 blog/
    └── 📁 [slug]/
        └── 📄 page.tsx          👈 /blog/anything
```

**Create `app/blog/[slug]/page.tsx`:**
```tsx
interface Props {
  params: { slug: string }
}

export default function BlogPost({ params }: Props) {
  return (
    <div className="p-8">
      <h1>📖 Post: {params.slug}</h1>
      <p>Dynamic route for: <code>{params.slug}</code></p>
    </div>
  )
}
```

**Test URLs:**
- `/blog/my-story` ✅
- `/blog/react-tips` ✅
- `/blog/anything-works` ✅

### 🎪 Catch-All Routes

```
📁 app/
└── 📁 docs/
    └── 📁 [...slug]/
        └── 📄 page.tsx          👈 /docs/a/b/c
```

**Create `app/docs/[...slug]/page.tsx`:**
```tsx
interface Props {
  params: { slug: string[] }
}

export default function Docs({ params }: Props) {
  return (
    <div className="p-8">
      <h1>📚 Docs</h1>
      <p>Path: /docs/{params.slug?.join('/')}</p>
      <p>Segments: {JSON.stringify(params.slug)}</p>
    </div>
  )
}
```

**Test URLs:**
- `/docs/getting-started` ✅
- `/docs/api/routes` ✅
- `/docs/a/b/c/d` ✅

---

## 🔗 Navigation (8 minutes)

### 🎯 Link Component

**Create `components/Navigation.tsx`:**
```tsx
import Link from 'next/link'

export default function Navigation() {
  return (
    <nav className="bg-gray-800 p-4">
      <div className="flex space-x-4">
        <Link href="/" className="text-white hover:text-blue-300">
          🏠 Home
        </Link>
        <Link href="/about" className="text-white hover:text-blue-300">
          👋 About
        </Link>
        <Link href="/blog" className="text-white hover:text-blue-300">
          📝 Blog
        </Link>
      </div>
    </nav>
  )
}
```

### 🎮 Programmatic Navigation

**Create `app/demo/page.tsx`:**
```tsx
'use client'
import { useRouter } from 'next/navigation'

export default function Demo() {
  const router = useRouter()

  return (
    <div className="p-8">
      <h1>Navigation Demo</h1>
      <button 
        onClick={() => router.push('/about')}
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        Go to About
      </button>
    </div>
  )
}
```

### 📋 Add Navigation to Layout

**Update `app/layout.tsx`:**
```tsx
import Navigation from '../components/Navigation'
import './globals.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <Navigation />
        <main>{children}</main>
      </body>
    </html>
  )
}
```

---

## 🧪 Test Your Routes (2 minutes)

```bash
npm run dev
```

**Visit these URLs:**
- `localhost:3000/` (Homepage)
- `localhost:3000/about` (About)
- `localhost:3000/blog` (Blog list)
- `localhost:3000/blog/my-story` (Dynamic)
- `localhost:3000/docs/getting-started` (Catch-all)

---

## 📚 Quick Reference

| Route Type | Structure | Example |
|------------|-----------|---------|
| **Static** | `app/about/page.tsx` | `/about` |
| **Nested** | `app/blog/post/page.tsx` | `/blog/post` |
| **Dynamic** | `app/blog/[slug]/page.tsx` | `/blog/anything` |
| **Catch-all** | `app/docs/[...slug]/page.tsx` | `/docs/a/b/c` |

### 🎯 Key Rules
- ✅ Folder + `page.tsx` = Route
- ✅ `[name]` = Dynamic segment  
- ✅ `[...name]` = Catch multiple segments
- ✅ Use `<Link>` for navigation
- ✅ Add `'use client'` for interactivity

### 🚨 Common Mistakes
- ❌ Missing `page.tsx` → No route
- ❌ Using `<a>` → Page refresh
- ❌ Forgetting `'use client'` → Hook errors

---

**🎉 Workshop Progress: 1/4 Complete** ✅

**Next:** SSG & SSR with App Router! 🚀
