# Building APIs with Next.js API Routes

## Introduction

Next.js API Routes provide a simple solution for building your API within your Next.js application. They allow you to create serverless API endpoints without needing a separate backend server. In this guide, we'll explore how to build robust APIs using Next.js API Routes.

## What Are API Routes?

API Routes are server-side functions that run in a Node.js environment. Any file inside the `pages/api` folder is mapped to `/api/*` and treated as an API endpoint instead of a page.

### Key Benefits

- **Full-Stack in One Place**: Backend and frontend in the same codebase
- **Serverless by Default**: Deploy as serverless functions
- **Simple Routing**: File-based routing like pages
- **TypeScript Support**: Full type safety for your APIs
- **Built-in Request Helpers**: Easy access to request data

## Creating Your First API Route

Create a file in `pages/api/hello.js`:

```javascript
export default function handler(req, res) {
  res.status(200).json({ message: 'Hello from Next.js API!' });
}
```

Access it at: `http://localhost:3000/api/hello`

## Request and Response Objects

API routes receive two parameters:

- **req**: An instance of http.IncomingMessage with additional helpers
- **res**: An instance of http.ServerResponse with additional helpers

### Common Request Properties

```javascript
export default function handler(req, res) {
  const {
    method,    // HTTP method (GET, POST, etc.)
    body,      // Request body (parsed)
    query,     // Query string parameters
    cookies,   // Cookies object
    headers    // Request headers
  } = req;

  console.log('Method:', method);
  console.log('Query params:', query);
  console.log('Body:', body);
  
  res.status(200).json({ received: true });
}
```

### Response Methods

```javascript
export default function handler(req, res) {
  // Set status code
  res.status(200);
  
  // Send JSON response
  res.json({ data: 'value' });
  
  // Send plain text
  res.send('Hello World');
  
  // Redirect
  res.redirect(307, '/other-page');
  
  // Set headers
  res.setHeader('Content-Type', 'application/json');
  
  // End response
  res.end();
}
```

## Handling Different HTTP Methods

```javascript
// pages/api/posts.js
export default async function handler(req, res) {
  const { method } = req;

  switch (method) {
    case 'GET':
      // Fetch all posts
      const posts = await getPosts();
      res.status(200).json(posts);
      break;
    
    case 'POST':
      // Create a new post
      const newPost = await createPost(req.body);
      res.status(201).json(newPost);
      break;
    
    case 'PUT':
      // Update a post
      const updatedPost = await updatePost(req.body);
      res.status(200).json(updatedPost);
      break;
    
    case 'DELETE':
      // Delete a post
      await deletePost(req.query.id);
      res.status(204).end();
      break;
    
    default:
      res.setHeader('Allow', ['GET', 'POST', 'PUT', 'DELETE']);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}
```

## Dynamic API Routes

Create dynamic routes using square brackets:

```javascript
// pages/api/posts/[id].js
export default async function handler(req, res) {
  const { id } = req.query;
  const { method } = req;

  switch (method) {
    case 'GET':
      const post = await getPostById(id);
      if (!post) {
        return res.status(404).json({ message: 'Post not found' });
      }
      res.status(200).json(post);
      break;
    
    case 'PUT':
      const updated = await updatePost(id, req.body);
      res.status(200).json(updated);
      break;
    
    case 'DELETE':
      await deletePost(id);
      res.status(204).end();
      break;
    
    default:
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}
```

## Middleware and Request Validation

Create reusable middleware functions:

```javascript
// lib/middleware.js
export function withAuth(handler) {
  return async (req, res) => {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
      return res.status(401).json({ message: 'Unauthorized' });
    }
    
    try {
      const user = await verifyToken(token);
      req.user = user;
      return handler(req, res);
    } catch (error) {
      return res.status(401).json({ message: 'Invalid token' });
    }
  };
}

export function validateBody(schema) {
  return (handler) => {
    return async (req, res) => {
      try {
        const validated = await schema.validate(req.body);
        req.body = validated;
        return handler(req, res);
      } catch (error) {
        return res.status(400).json({ message: error.message });
      }
    };
  };
}

// Usage in API route
import { withAuth, validateBody } from '@/lib/middleware';

async function handler(req, res) {
  // This handler only runs if auth succeeds
  const { user } = req;
  res.status(200).json({ message: `Hello ${user.name}` });
}

export default withAuth(handler);
```

## Database Integration

Example with Prisma ORM:

```javascript
// pages/api/users/index.js
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export default async function handler(req, res) {
  const { method } = req;

  try {
    switch (method) {
      case 'GET':
        const users = await prisma.user.findMany({
          select: {
            id: true,
            email: true,
            name: true,
            createdAt: true
          }
        });
        res.status(200).json(users);
        break;
      
      case 'POST':
        const { email, name, password } = req.body;
        const hashedPassword = await hashPassword(password);
        
        const user = await prisma.user.create({
          data: {
            email,
            name,
            password: hashedPassword
          }
        });
        
        res.status(201).json({ id: user.id, email: user.email });
        break;
      
      default:
        res.status(405).end(`Method ${method} Not Allowed`);
    }
  } catch (error) {
    console.error(error);
    res.status(500).json({ message: 'Internal server error' });
  } finally {
    await prisma.$disconnect();
  }
}
```

## File Upload Handling

Handle file uploads with multipart/form-data:

```javascript
// pages/api/upload.js
import formidable from 'formidable';
import fs from 'fs/promises';

export const config = {
  api: {
    bodyParser: false, // Disable default body parser
  },
};

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  const form = formidable({ multiples: true });

  form.parse(req, async (err, fields, files) => {
    if (err) {
      return res.status(500).json({ message: 'Error parsing files' });
    }

    const file = files.file;
    const data = await fs.readFile(file.filepath);
    const filename = `uploads/${Date.now()}-${file.originalFilename}`;
    
    await fs.writeFile(`public/${filename}`, data);
    await fs.unlink(file.filepath);

    res.status(200).json({ url: `/${filename}` });
  });
}
```

## Error Handling

Implement consistent error handling:

```javascript
// lib/errors.js
export class ApiError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}

// lib/errorHandler.js
export function errorHandler(handler) {
  return async (req, res) => {
    try {
      await handler(req, res);
    } catch (error) {
      console.error(error);
      
      if (error instanceof ApiError) {
        return res.status(error.statusCode).json({
          message: error.message
        });
      }
      
      res.status(500).json({
        message: 'Internal server error'
      });
    }
  };
}

// Usage
import { errorHandler } from '@/lib/errorHandler';
import { ApiError } from '@/lib/errors';

async function handler(req, res) {
  const user = await getUser(req.query.id);
  
  if (!user) {
    throw new ApiError('User not found', 404);
  }
  
  res.status(200).json(user);
}

export default errorHandler(handler);
```

## CORS Configuration

Enable CORS for external API access:

```javascript
// pages/api/public-data.js
export default async function handler(req, res) {
  // Set CORS headers
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');

  // Handle preflight request
  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }

  // Your API logic here
  res.status(200).json({ data: 'public data' });
}
```

## Best Practices

1. **Use Environment Variables**: Keep sensitive data in `.env.local`
2. **Implement Rate Limiting**: Protect against abuse
3. **Validate Input**: Always validate and sanitize user input
4. **Use TypeScript**: Add type safety to your APIs
5. **Error Handling**: Implement consistent error responses
6. **Authentication**: Secure endpoints that need protection
7. **Logging**: Log important events and errors
8. **API Documentation**: Document your endpoints
9. **Versioning**: Plan for API version changes
10. **Testing**: Write tests for your API routes

## Conclusion

Next.js API Routes provide a powerful, yet simple way to build full-stack applications. They eliminate the need for a separate backend while providing all the features you need to build production-ready APIs. Start building your API today and enjoy the benefits of a unified full-stack framework!
