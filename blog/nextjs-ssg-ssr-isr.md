# Next.js Rendering Strategies: SSG, SSR, and ISR Explained

## Introduction

One of Next.js's most powerful features is its flexibility in how pages are rendered. Understanding the different rendering strategies - Static Site Generation (SSG), Server-Side Rendering (SSR), and Incremental Static Regeneration (ISR) - is crucial for building performant applications. This guide will help you understand when and how to use each approach.

## The Three Rendering Strategies

### 1. Static Site Generation (SSG)

Pages are pre-rendered at build time and served as static HTML files.

#### When to Use SSG

- Content doesn't change frequently
- Same content for all users
- SEO is important
- Fastest possible page loads

#### Basic Implementation

```jsx
// pages/blog/index.js
export default function Blog({ posts }) {
  return (
    <div>
      <h1>Blog Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// This runs at build time
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: { posts }
  };
}
```

#### With Dynamic Routes

```jsx
// pages/blog/[slug].js
export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Generate all paths at build time
export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map(post => ({
    params: { slug: post.slug }
  }));

  return {
    paths,
    fallback: false // Show 404 for non-existent paths
  };
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.slug}`);
  const post = await res.json();

  return {
    props: { post }
  };
}
```

#### Fallback Options

```jsx
export async function getStaticPaths() {
  return {
    paths: [
      { params: { slug: 'first-post' } },
      { params: { slug: 'second-post' } }
    ],
    fallback: false // or 'blocking' or true
  };
}

// fallback: false
// - Only pre-rendered paths exist
// - 404 for other paths

// fallback: 'blocking'
// - Pre-rendered paths load instantly
// - Other paths SSR on first request, then cached
// - User waits for page

// fallback: true
// - Pre-rendered paths load instantly
// - Other paths show loading state, then render
// - Need to handle loading state
```

Handling `fallback: true`:

```jsx
import { useRouter } from 'next/router';

export default function Post({ post }) {
  const router = useRouter();

  // Show loading state while page is being generated
  if (router.isFallback) {
    return <div>Loading...</div>;
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

### 2. Server-Side Rendering (SSR)

Pages are rendered on each request on the server.

#### When to Use SSR

- Content changes frequently
- Content is user-specific
- Need to access request data (cookies, headers)
- Real-time data is critical

#### Basic Implementation

```jsx
// pages/dashboard.js
export default function Dashboard({ user, data }) {
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <div>
        {data.map(item => (
          <div key={item.id}>{item.content}</div>
        ))}
      </div>
    </div>
  );
}

// Runs on every request
export async function getServerSideProps(context) {
  const { req, res, query, params } = context;
  
  // Access cookies
  const token = req.cookies.token;
  
  // Fetch user-specific data
  const userRes = await fetch('https://api.example.com/user', {
    headers: { Authorization: `Bearer ${token}` }
  });
  const user = await userRes.json();

  const dataRes = await fetch(`https://api.example.com/data?userId=${user.id}`);
  const data = await dataRes.json();

  return {
    props: { user, data }
  };
}
```

#### Redirect and Not Found

```jsx
export async function getServerSideProps(context) {
  const { slug } = context.params;
  
  const res = await fetch(`https://api.example.com/posts/${slug}`);
  
  // Handle 404
  if (res.status === 404) {
    return {
      notFound: true
    };
  }
  
  const post = await res.json();
  
  // Conditional redirect
  if (post.draft && !context.req.cookies.isAdmin) {
    return {
      redirect: {
        destination: '/',
        permanent: false
      }
    };
  }

  return {
    props: { post }
  };
}
```

#### Caching SSR with Cache-Control

```jsx
export async function getServerSideProps({ res }) {
  // Cache for 60 seconds, revalidate in background
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=60, stale-while-revalidate=120'
  );

  const data = await fetchData();

  return {
    props: { data }
  };
}
```

### 3. Incremental Static Regeneration (ISR)

Update static content after deployment without rebuilding the entire site.

#### When to Use ISR

- Content updates periodically
- Want benefits of static sites with fresh content
- High traffic sites that need to scale
- E-commerce product pages

#### Basic Implementation

```jsx
// pages/products/[id].js
export default function Product({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>Price: ${product.price}</p>
      <p>Stock: {product.stock}</p>
    </div>
  );
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/products/${params.id}`);
  const product = await res.json();

  return {
    props: { product },
    revalidate: 60 // Regenerate page every 60 seconds
  };
}

export async function getStaticPaths() {
  return {
    paths: [], // Generate on-demand
    fallback: 'blocking'
  };
}
```

#### On-Demand Revalidation

Trigger revalidation programmatically:

```jsx
// pages/api/revalidate.js
export default async function handler(req, res) {
  // Check for secret to confirm this is a valid request
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  try {
    // Revalidate specific path
    await res.revalidate('/products/1');
    await res.revalidate('/blog/my-post');
    
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}
```

Trigger from CMS webhook:

```jsx
// pages/api/webhook.js
export default async function handler(req, res) {
  const { slug, type } = req.body;
  
  // Verify webhook signature
  if (!verifySignature(req)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Revalidate based on content type
  if (type === 'post.published') {
    await res.revalidate(`/blog/${slug}`);
    await res.revalidate('/blog');
  }

  return res.json({ revalidated: true });
}
```

## Comparing the Strategies

### Performance

```
SSG (Fastest)
↓
ISR (Fast, occasionally rebuilds)
↓
SSR (Slower, but always fresh)
↓
CSR (Client-Side Rendering - slowest initial load)
```

### Use Case Matrix

| Strategy | Build Time | Request Time | Freshness | Best For |
|----------|-----------|--------------|-----------|----------|
| SSG | Long | Instant | Stale | Marketing pages, documentation |
| ISR | Short | Instant | Fresh enough | E-commerce, blogs, news |
| SSR | N/A | Slow | Always fresh | Dashboards, personalized content |

## Hybrid Approach

Mix strategies in the same application:

```jsx
// pages/index.js - SSG for homepage
export async function getStaticProps() {
  return { props: { /* ... */ } };
}

// pages/blog/[slug].js - ISR for blog posts
export async function getStaticProps() {
  return {
    props: { /* ... */ },
    revalidate: 60
  };
}

// pages/dashboard.js - SSR for user dashboard
export async function getServerSideProps() {
  return { props: { /* ... */ } };
}

// pages/settings.js - CSR for settings page
// No data fetching function = client-side rendering
export default function Settings() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/settings').then(/* ... */);
  }, []);
  
  return <div>{/* ... */}</div>;
}
```

## Best Practices

### 1. Choose the Right Strategy

```jsx
// Ask yourself:
// - Does content change? → ISR or SSR
// - Is it user-specific? → SSR
// - Is it the same for everyone? → SSG
// - How often does it change? → ISR (periodic) or SSR (always)
```

### 2. Optimize Data Fetching

```jsx
// Parallel requests
export async function getStaticProps() {
  const [posts, categories, settings] = await Promise.all([
    fetch('https://api.example.com/posts').then(r => r.json()),
    fetch('https://api.example.com/categories').then(r => r.json()),
    fetch('https://api.example.com/settings').then(r => r.json())
  ]);

  return {
    props: { posts, categories, settings }
  };
}
```

### 3. Handle Errors Gracefully

```jsx
export async function getStaticProps() {
  try {
    const res = await fetch('https://api.example.com/data');
    
    if (!res.ok) {
      throw new Error('Failed to fetch data');
    }
    
    const data = await res.json();
    
    return {
      props: { data }
    };
  } catch (error) {
    console.error('Error fetching data:', error);
    
    return {
      props: { data: [] }, // Fallback data
      revalidate: 10 // Try again sooner
    };
  }
}
```

### 4. Use Preview Mode

Enable preview mode for draft content:

```jsx
// pages/api/preview.js
export default async function handler(req, res) {
  const { slug, secret } = req.query;

  if (secret !== process.env.PREVIEW_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  res.setPreviewData({});
  res.redirect(`/blog/${slug}`);
}

// pages/api/exit-preview.js
export default async function handler(req, res) {
  res.clearPreviewData();
  res.redirect('/');
}

// pages/blog/[slug].js
export async function getStaticProps({ params, preview = false }) {
  const data = await getPostData(params.slug, preview);
  
  return {
    props: { data, preview }
  };
}
```

## Conclusion

Next.js's flexible rendering strategies allow you to optimize each page individually. Use SSG for content that rarely changes, ISR for content that updates periodically, and SSR for user-specific or real-time data. The key is understanding your content's requirements and choosing the appropriate strategy.

Master these patterns to build lightning-fast, scalable Next.js applications!
