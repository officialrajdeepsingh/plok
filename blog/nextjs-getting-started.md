# Getting Started with Next.js: The React Framework for Production

## Introduction

Next.js is a powerful React framework that makes building production-ready web applications easier than ever. It provides features like server-side rendering, static site generation, API routes, and automatic code splitting out of the box. In this guide, we'll explore what makes Next.js special and how to get started.

## What is Next.js?

Next.js is a React framework developed by Vercel that enables functionality such as server-side rendering and generating static websites. It's designed to make the development process faster and the resulting applications more performant.

### Key Features

- **Hybrid Static & Server Rendering**: Choose between static generation (SSG) or server-side rendering (SSR) per page
- **File-System Based Routing**: No need for complex routing configuration
- **API Routes**: Build your API with serverless functions
- **Automatic Code Splitting**: Faster page loads with optimized bundles
- **Built-in CSS and Sass Support**: Style your application with ease
- **Fast Refresh**: See your changes instantly during development
- **Image Optimization**: Automatic image optimization with the Image component
- **TypeScript Support**: First-class TypeScript support out of the box

## Creating Your First Next.js Application

Install Next.js using create-next-app:

```bash
npx create-next-app@latest my-nextjs-app
cd my-nextjs-app
npm run dev
```

For TypeScript support:

```bash
npx create-next-app@latest my-nextjs-app --typescript
```

Your application will be running at `http://localhost:3000`.

## Project Structure

A typical Next.js project structure looks like this:

```
my-nextjs-app/
├── pages/
│   ├── api/
│   │   └── hello.js
│   ├── _app.js
│   ├── index.js
│   └── about.js
├── public/
│   └── favicon.ico
├── styles/
│   └── globals.css
├── next.config.js
└── package.json
```

## File-Based Routing

One of Next.js's most powerful features is its file-based routing system:

```jsx
// pages/index.js → /
export default function Home() {
  return <h1>Home Page</h1>;
}

// pages/about.js → /about
export default function About() {
  return <h1>About Page</h1>;
}

// pages/blog/[slug].js → /blog/:slug
export default function BlogPost({ slug }) {
  return <h1>Blog Post: {slug}</h1>;
}
```

### Dynamic Routes

Create dynamic routes using square brackets:

```jsx
// pages/posts/[id].js
import { useRouter } from 'next/router';

export default function Post() {
  const router = useRouter();
  const { id } = router.query;

  return <div>Post ID: {id}</div>;
}
```

## Data Fetching Methods

Next.js provides multiple ways to fetch data:

### Static Generation (SSG) with getStaticProps

```jsx
// pages/blog.js
export default function Blog({ posts }) {
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return {
    props: { posts },
    revalidate: 60, // Regenerate page every 60 seconds
  };
}
```

### Server-Side Rendering (SSR) with getServerSideProps

```jsx
// pages/profile.js
export default function Profile({ user }) {
  return <div>Welcome, {user.name}!</div>;
}

export async function getServerSideProps(context) {
  const { req } = context;
  const res = await fetch(`https://api.example.com/user`, {
    headers: { cookie: req.headers.cookie },
  });
  const user = await res.json();

  return {
    props: { user },
  };
}
```

### Client-Side Data Fetching

```jsx
import { useState, useEffect } from 'react';

export default function Dashboard() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/dashboard')
      .then(res => res.json())
      .then(setData);
  }, []);

  if (!data) return <div>Loading...</div>;
  return <div>{data.content}</div>;
}
```

## API Routes

Create backend API endpoints within your Next.js app:

```jsx
// pages/api/users.js
export default async function handler(req, res) {
  if (req.method === 'GET') {
    const users = await fetchUsers();
    res.status(200).json(users);
  } else if (req.method === 'POST') {
    const user = await createUser(req.body);
    res.status(201).json(user);
  } else {
    res.status(405).json({ message: 'Method not allowed' });
  }
}
```

## Image Optimization

Use the Next.js Image component for automatic optimization:

```jsx
import Image from 'next/image';

export default function Avatar() {
  return (
    <Image
      src="/profile.jpg"
      alt="Profile"
      width={200}
      height={200}
      priority
    />
  );
}
```

## Deployment

Deploying to Vercel is incredibly simple:

1. Push your code to GitHub
2. Import your repository on [vercel.com](https://vercel.com)
3. Vercel automatically detects Next.js and configures the build

You can also deploy to other platforms like AWS, Google Cloud, or DigitalOcean.

## Best Practices

1. **Use Static Generation when possible**: It's faster than SSR
2. **Optimize images**: Always use the Next.js Image component
3. **Implement incremental static regeneration**: Keep static pages fresh
4. **Use environment variables**: Keep sensitive data secure
5. **Enable TypeScript**: Catch errors early in development

## Conclusion

Next.js combines the best of both worlds: the simplicity of static sites and the power of server-side rendering. Its intuitive API and powerful features make it an excellent choice for building modern web applications.

Start your Next.js journey today and experience the difference!
