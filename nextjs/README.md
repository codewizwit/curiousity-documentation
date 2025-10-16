# Next.js: The Full-Stack React Framework

_Building React apps that are fast, SEO-friendly, and production-ready out of the box._

## Overview

Next.js is a framework built on top of React that solves a lot of the hard problems that come with building modern web apps. If React is the engine, Next.js is the complete car — it comes with routing, server-side rendering, static site generation, API routes, and a ton of optimizations baked in.

Think of React as giving you the tools to build a house, while Next.js gives you the blueprints, foundation, and structure to actually build that house the right way.

---

## The Restaurant Kitchen Metaphor

Understanding Next.js is easier if you think about a restaurant:

**Plain React (Client-Side Rendering):**
- Customer walks in (visits your site)
- You hand them raw ingredients and a recipe card (JavaScript bundle)
- Customer has to cook the entire meal themselves in their browser (rendering)
- Slow first experience, especially on slow devices
- Search engines see an empty plate (bad for SEO)

**Next.js (Server-Side Rendering + Static Generation):**
- Customer walks in
- You already have meals prepped in the kitchen (server)
- You can serve them a fully cooked meal immediately (pre-rendered HTML)
- Or you cook it fresh when they order (SSR)
- Or you prepared popular dishes ahead of time (SSG)
- Fast first experience, great for SEO
- Customer gets a hot meal right away, then the meal becomes interactive

That's the power of Next.js: **delivering content fast** while keeping the **interactive benefits of React**.

---

## What is Next.js?

Next.js is a **React framework** that provides:

1. **Routing** - File-based routing (no need for React Router)
2. **Rendering modes** - Server-side, client-side, static, or hybrid
3. **API routes** - Build backend endpoints right in your app
4. **Optimizations** - Image optimization, code splitting, prefetching automatically
5. **Developer experience** - Fast refresh, TypeScript support, great error messages
6. **Production-ready** - Built-in performance, security, and scaling best practices

It's made by Vercel (the company behind deployment platform Vercel), but you can deploy Next.js anywhere — not just Vercel.

---

## Why Next.js Exists: The Problems It Solves

### Problem 1: SEO with React

**The Issue:**
```jsx
// Plain React app
ReactDOM.render(<App />, document.getElementById('root'));

// What Google sees:
<html>
  <body>
    <div id="root"></div>  <!-- Empty! -->
    <script src="bundle.js"></script>
  </body>
</html>
```

Search engines struggle because the content is empty until JavaScript runs.

**Next.js Solution:**
Next.js pre-renders your pages on the server, so search engines (and users) get real HTML immediately.

```html
<!-- What Google sees with Next.js -->
<html>
  <body>
    <div id="root">
      <h1>Welcome to My App</h1>
      <p>This content is immediately visible!</p>
    </div>
    <script src="bundle.js"></script>
  </body>
</html>
```

---

### Problem 2: Slow Initial Load with React

**The Issue:**
- User visits your React app
- Browser downloads JavaScript (could be large)
- JavaScript executes and renders the page
- User finally sees content (slow on slow connections/devices)

**Next.js Solution:**
Send pre-rendered HTML first, then hydrate with React for interactivity.

```
Plain React:  [Download JS] → [Parse JS] → [Render] → Content visible
Next.js:      Content visible immediately → [Hydrate] → Interactive
```

---

### Problem 3: No Built-in Routing

**The Issue:**
React doesn't include routing — you need to install React Router, configure it, handle code splitting manually, etc.

**Next.js Solution:**
File-based routing. Just create files in the `pages/` or `app/` directory, and you get routes automatically.

```
pages/
├── index.js          → /
├── about.js          → /about
├── blog/
│   ├── index.js      → /blog
│   └── [slug].js     → /blog/:slug (dynamic)
└── api/
    └── hello.js      → /api/hello (API route)
```

---

### Problem 4: No Backend Integration

**The Issue:**
React is front-end only. You need a separate backend server for APIs, authentication, database queries.

**Next.js Solution:**
API routes let you build backend endpoints right inside your Next.js app.

```javascript
// pages/api/users.js
export default function handler(req, res) {
  // This runs on the server!
  const users = fetchUsersFromDatabase();
  res.status(200).json(users);
}

// Accessible at /api/users
```

---

## Key Concepts Explained

### 1. Rendering Modes

Next.js lets you choose **how** each page is rendered. Different pages can use different strategies.

#### Static Site Generation (SSG) - Pre-cook Everything

**The Restaurant Analogy:**
Like meal prepping popular dishes before customers arrive. You cook 100 burgers ahead of time, store them, and serve them instantly when ordered.

**Technical:**
HTML is generated at **build time**. Same HTML served to everyone. Super fast, great for SEO.

**When to use:**
- Marketing pages
- Blog posts
- Documentation
- Product pages
- Anything that doesn't change often

```jsx
// pages/about.js
export default function About({ data }) {
  return <h1>About Us: {data.title}</h1>;
}

// This runs at BUILD TIME (not on every request)
export async function getStaticProps() {
  const data = await fetch('https://api.example.com/about').then(r => r.json());

  return {
    props: { data },  // Passed to component
    revalidate: 60    // Re-generate page every 60 seconds (ISR)
  };
}
```

**Result:** HTML is created once during build, served instantly to all users.

---

#### Server-Side Rendering (SSR) - Cook to Order

**The Restaurant Analogy:**
Like cooking a fresh meal when the customer orders. Takes a bit longer, but it's personalized and fresh every time.

**Technical:**
HTML is generated on **every request**. Server runs your React code, creates HTML, sends it to the browser.

**When to use:**
- User dashboards
- Personalized content
- Real-time data
- Pages that need up-to-the-second freshness

```jsx
// pages/dashboard.js
export default function Dashboard({ user, data }) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>Your balance: ${data.balance}</p>
    </div>
  );
}

// This runs on EVERY REQUEST
export async function getServerSideProps(context) {
  const { req } = context;
  const user = getUserFromRequest(req);
  const data = await fetchUserData(user.id);

  return {
    props: { user, data }
  };
}
```

**Result:** Fresh HTML generated for each user on each request.

---

#### Client-Side Rendering (CSR) - Customer Cooks

**The Restaurant Analogy:**
Like giving customers ingredients to cook themselves (regular React behavior).

**Technical:**
HTML is minimal, JavaScript renders everything in the browser. Good for highly interactive apps where SEO doesn't matter.

**When to use:**
- Admin dashboards
- Interactive tools
- Apps behind authentication
- Real-time collaborative features

```jsx
// pages/admin.js
import { useEffect, useState } from 'react';

export default function Admin() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Fetch on client side
    fetch('/api/admin-data')
      .then(r => r.json())
      .then(setData);
  }, []);

  if (!data) return <p>Loading...</p>;

  return <div>Admin Data: {data.stats}</div>;
}
```

**Result:** Empty shell sent to browser, React fetches and renders content client-side.

---

#### Incremental Static Regeneration (ISR) - Smart Meal Prep

**The Restaurant Analogy:**
Like meal prepping burgers, but every hour you make a fresh batch to keep them from getting stale. Best of both worlds.

**Technical:**
Generate static HTML at build time, but **regenerate it in the background** after a certain time period. Users always get fast static pages, but content stays fresh.

**When to use:**
- Blog posts (update occasionally)
- Product catalogs (prices change)
- News sites
- E-commerce product pages

```jsx
// pages/blog/[slug].js
export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);

  return {
    props: { post },
    revalidate: 60  // Regenerate page every 60 seconds if someone visits
  };
}

export async function getStaticPaths() {
  const posts = await fetchAllPostSlugs();

  return {
    paths: posts.map(slug => ({ params: { slug } })),
    fallback: 'blocking'  // Generate new pages on-demand
  };
}
```

**Result:** Static page served instantly, but Next.js regenerates it in the background every 60 seconds.

---

### 2. File-Based Routing

No need to configure routes manually. The file structure **is** the routing.

```
pages/
├── index.js                    → /
├── about.js                    → /about
├── blog/
│   ├── index.js                → /blog
│   └── [slug].js               → /blog/react-hooks
├── posts/
│   └── [id].js                 → /posts/123
└── api/
    ├── hello.js                → /api/hello
    └── users/
        └── [id].js             → /api/users/456
```

**Dynamic Routes:**

```jsx
// pages/blog/[slug].js
import { useRouter } from 'next/router';

export default function BlogPost() {
  const router = useRouter();
  const { slug } = router.query;  // Get dynamic part from URL

  return <h1>Post: {slug}</h1>;
}

// /blog/my-first-post → slug = "my-first-post"
// /blog/react-tips → slug = "react-tips"
```

**Catch-All Routes:**

```jsx
// pages/docs/[...slug].js
// Matches: /docs/a, /docs/a/b, /docs/a/b/c

export default function Docs() {
  const router = useRouter();
  const { slug } = router.query;  // slug = ['a', 'b', 'c']

  return <h1>Docs: {slug.join(' / ')}</h1>;
}
```

---

### 3. API Routes

Build backend endpoints right in your Next.js app. No separate Express server needed (though you can still use one if you want).

```javascript
// pages/api/hello.js
export default function handler(req, res) {
  res.status(200).json({ message: 'Hello from Next.js API!' });
}

// Fetch from your frontend:
// fetch('/api/hello')
```

**Real-World Example: Form Submission**

```javascript
// pages/api/contact.js
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { name, email, message } = req.body;

  // Validate
  if (!name || !email || !message) {
    return res.status(400).json({ error: 'Missing fields' });
  }

  // Save to database
  await saveContactForm({ name, email, message });

  // Send email
  await sendEmail({ to: 'support@example.com', subject: 'New contact form', body: message });

  return res.status(200).json({ success: true });
}
```

**Front-end usage:**

```jsx
// pages/contact.js
export default function Contact() {
  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);

    const response = await fetch('/api/contact', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: formData.get('name'),
        email: formData.get('email'),
        message: formData.get('message')
      })
    });

    const data = await response.json();
    if (data.success) {
      alert('Message sent!');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" placeholder="Name" />
      <input name="email" type="email" placeholder="Email" />
      <textarea name="message" placeholder="Message"></textarea>
      <button type="submit">Send</button>
    </form>
  );
}
```

---

### 4. Built-in Optimizations

Next.js automatically optimizes your app without you having to think about it.

#### Image Optimization

```jsx
import Image from 'next/image';

export default function Profile() {
  return (
    <div>
      {/* ❌ Old way - Manual optimization needed */}
      <img src="/profile.jpg" alt="Profile" width={500} height={500} />

      {/* ✅ Next.js way - Automatic optimization */}
      <Image
        src="/profile.jpg"
        alt="Profile"
        width={500}
        height={500}
        priority  // Load this image first
      />
    </div>
  );
}
```

**What Next.js does automatically:**
- Resizes images to the right size for the device
- Converts to modern formats (WebP, AVIF) when supported
- Lazy loads images below the fold
- Serves from CDN
- Prevents layout shift

---

#### Code Splitting

Next.js automatically splits your JavaScript into smaller chunks. Each page only loads the code it needs.

```jsx
// pages/about.js uses large charting library
import Chart from 'huge-chart-library';  // Only loaded on /about

export default function About() {
  return <Chart data={data} />;
}

// Visiting / doesn't download the chart library
// Visiting /about downloads it only when needed
```

---

#### Prefetching

Next.js automatically prefetches pages you're likely to visit.

```jsx
import Link from 'next/link';

export default function Home() {
  return (
    <div>
      {/* Next.js prefetches /about when this link is visible */}
      <Link href="/about">About Us</Link>
    </div>
  );
}
```

When the link appears in the viewport, Next.js downloads the JavaScript for `/about` in the background. When you click, it loads **instantly**.

---

## App Router vs Pages Router

Next.js has **two routing systems**. Understanding the difference matters.

### Pages Router (Legacy, but still widely used)

```
pages/
├── index.js          → /
├── about.js          → /about
└── api/
    └── hello.js      → /api/hello
```

- Uses `getStaticProps`, `getServerSideProps`
- File = Route
- Simple and straightforward

### App Router (New, recommended for new projects)

```
app/
├── page.js           → /
├── layout.js         → Root layout
├── about/
│   └── page.js       → /about
└── blog/
    ├── page.js       → /blog
    └── [slug]/
        └── page.js   → /blog/:slug
```

- Uses React Server Components
- More powerful, more flexible
- Better for complex layouts and data fetching
- File = Route segment, `page.js` = actual page

**Key Differences:**

| Feature | Pages Router | App Router |
|---------|-------------|------------|
| **Routing** | `pages/about.js` | `app/about/page.js` |
| **Layouts** | Wrap manually | `layout.js` nesting |
| **Data Fetching** | `getStaticProps`, `getServerSideProps` | `async` components, `fetch()` |
| **Server Components** | No | Yes |
| **Streaming** | No | Yes |
| **Maturity** | Stable, battle-tested | Newer, evolving |

**Recommendation:** Use **App Router** for new projects, but **Pages Router** is still perfectly fine and widely used.

---

## Real-World Examples

### Example 1: Blog with SSG + ISR

```jsx
// app/blog/[slug]/page.js (App Router)
export default async function BlogPost({ params }) {
  const post = await fetchPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <p className="date">{post.publishedAt}</p>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Generate static pages for all blog posts at build time
export async function generateStaticParams() {
  const posts = await fetchAllPosts();
  return posts.map(post => ({ slug: post.slug }));
}

// Revalidate every 60 seconds
export const revalidate = 60;
```

---

### Example 2: Dashboard with SSR

```jsx
// app/dashboard/page.js
import { getUserFromSession } from '@/lib/auth';
import { fetchUserData } from '@/lib/db';

export default async function Dashboard() {
  // This runs on the server for every request
  const user = await getUserFromSession();

  if (!user) {
    redirect('/login');
  }

  const data = await fetchUserData(user.id);

  return (
    <div>
      <h1>Welcome back, {user.name}</h1>
      <div>
        <h2>Your Stats</h2>
        <p>Balance: ${data.balance}</p>
        <p>Orders: {data.orders.length}</p>
      </div>
    </div>
  );
}

// Force dynamic rendering (SSR)
export const dynamic = 'force-dynamic';
```

---

### Example 3: E-commerce Product Page

```jsx
// app/products/[id]/page.js
import Image from 'next/image';
import { fetchProduct } from '@/lib/products';

export default async function ProductPage({ params }) {
  const product = await fetchProduct(params.id);

  return (
    <div className="product">
      <Image
        src={product.image}
        alt={product.name}
        width={600}
        height={600}
        priority
      />
      <h1>{product.name}</h1>
      <p className="price">${product.price}</p>
      <p>{product.description}</p>
      <button>Add to Cart</button>
    </div>
  );
}

// Pre-generate popular products at build time
export async function generateStaticParams() {
  const popularProducts = await fetchPopularProducts();
  return popularProducts.map(p => ({ id: p.id }));
}

// Regenerate every 5 minutes
export const revalidate = 300;

// Generate other products on-demand
export const dynamicParams = true;
```

---

## When to Use Next.js

### ✅ Great For:

- **Marketing sites** - SEO matters, content is mostly static
- **Blogs** - Great content management with SSG + ISR
- **E-commerce** - Product pages need SEO and speed
- **SaaS applications** - Mix of public (SSG) and private (SSR/CSR) pages
- **Documentation sites** - Fast, searchable, great DX
- **Full-stack apps** - API routes let you build backend and frontend together

### ⚠️  Maybe Not Ideal For:

- **Highly interactive apps** - If you don't need SEO (admin tools, games, creative tools), plain React might be simpler
- **Real-time collaboration** - WebSocket-heavy apps might need more custom server setup
- **Existing large React app** - Migration can be complex; might not be worth it

---

## Common Patterns

### Pattern 1: Hybrid Rendering

```jsx
// Public marketing page - SSG
// app/page.js
export default async function Home() {
  const data = await fetchMarketingContent();
  return <LandingPage data={data} />;
}
export const revalidate = 3600;  // Regenerate every hour

// User dashboard - SSR
// app/dashboard/page.js
export default async function Dashboard() {
  const user = await getCurrentUser();
  return <DashboardView user={user} />;
}
export const dynamic = 'force-dynamic';

// Interactive tool - CSR
// app/editor/page.js
'use client';  // Client component
export default function Editor() {
  const [content, setContent] = useState('');
  return <RichTextEditor value={content} onChange={setContent} />;
}
```

---

### Pattern 2: API Routes for Backend Logic

```javascript
// app/api/checkout/route.js
import { NextResponse } from 'next/server';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function POST(request) {
  const { items, userId } = await request.json();

  // Create Stripe checkout session
  const session = await stripe.checkout.sessions.create({
    payment_method_types: ['card'],
    line_items: items,
    customer: userId,
    success_url: `${process.env.DOMAIN}/success`,
    cancel_url: `${process.env.DOMAIN}/cart`,
  });

  return NextResponse.json({ url: session.url });
}
```

---

## Quick Reference

```bash
# Create new Next.js app
npx create-next-app@latest my-app

# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

### Key Files

```
my-next-app/
├── app/                  # App Router (new)
│   ├── page.js           # Homepage
│   ├── layout.js         # Root layout
│   └── about/
│       └── page.js       # /about
├── pages/                # Pages Router (legacy)
│   ├── index.js          # Homepage
│   └── api/
│       └── hello.js      # API endpoint
├── public/               # Static files (images, fonts)
├── next.config.js        # Configuration
└── package.json
```

---

## Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Learn Next.js](https://nextjs.org/learn) - Official interactive tutorial
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [App Router Guide](https://nextjs.org/docs/app)
- [Pages Router Guide](https://nextjs.org/docs/pages)

---

_From client-only React apps to production-ready, SEO-friendly, full-stack applications — Next.js makes it feel obvious._
