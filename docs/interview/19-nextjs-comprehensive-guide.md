# Next.js Comprehensive Guide & Interview Questions

## Table of Contents
1. [Next.js Fundamentals](#nextjs-fundamentals)
2. [Routing & Navigation](#routing--navigation)
3. [Rendering Strategies](#rendering-strategies)
4. [Data Fetching](#data-fetching)
5. [API Routes](#api-routes)
6. [Image & Font Optimization](#image--font-optimization)
7. [Performance Optimization](#performance-optimization)
8. [App Router vs Pages Router](#app-router-vs-pages-router)
9. [Server Components vs Client Components](#server-components-vs-client-components)
10. [Authentication & Middleware](#authentication--middleware)
11. [Deployment & Production](#deployment--production)
12. [Common Interview Questions](#common-interview-questions)

---

## Next.js Fundamentals

### What is Next.js?

**Next.js** is a React framework for building full-stack web applications with:
- ✅ Server-Side Rendering (SSR)
- ✅ Static Site Generation (SSG)
- ✅ API Routes (Backend)
- ✅ File-based routing
- ✅ Built-in optimization (images, fonts, code splitting)

### Why Next.js over Create React App (CRA)?

```
Create React App (CRA):
❌ Client-side rendering only (slow initial load)
❌ Poor SEO (content not in HTML)
❌ No backend API routes
❌ Manual optimization needed
❌ No built-in routing

Next.js:
✅ SSR + SSG + CSR (best performance)
✅ Great SEO (content in HTML)
✅ API routes (full-stack in one app)
✅ Automatic optimization
✅ File-based routing
✅ Incremental Static Regeneration (ISR)
```

### Next.js Project Structure

```
my-nextjs-app/
├── app/                    # App Router (Next.js 13+)
│   ├── layout.tsx          # Root layout (shared across pages)
│   ├── page.tsx            # Home page (/)
│   ├── about/
│   │   └── page.tsx        # About page (/about)
│   ├── api/                # API routes
│   │   └── students/
│   │       └── route.ts    # /api/students endpoint
│   └── students/
│       ├── page.tsx        # /students (list)
│       └── [id]/
│           └── page.tsx    # /students/:id (detail)
│
├── components/             # Reusable components
│   ├── Navbar.tsx
│   └── StudentCard.tsx
│
├── lib/                    # Utility functions
│   ├── db.ts               # Database connection
│   └── utils.ts
│
├── public/                 # Static assets
│   ├── images/
│   └── favicon.ico
│
├── styles/                 # Global styles
│   └── globals.css
│
├── next.config.js          # Next.js configuration
├── package.json
└── tsconfig.json
```

---

## Routing & Navigation

### File-based Routing (App Router)

```
app/
├── page.tsx                        → /
├── about/page.tsx                  → /about
├── students/
│   ├── page.tsx                    → /students
│   ├── [id]/page.tsx               → /students/:id
│   ├── [id]/edit/page.tsx          → /students/:id/edit
│   └── new/page.tsx                → /students/new
├── blog/
│   ├── [...slug]/page.tsx          → /blog/* (catch-all)
│   └── [[...slug]]/page.tsx        → /blog/* (optional catch-all)
└── (dashboard)/                    → / (route group, no URL segment)
    ├── layout.tsx
    ├── analytics/page.tsx          → /analytics
    └── settings/page.tsx           → /settings
```

### Dynamic Routes

```typescript
// app/students/[id]/page.tsx
interface PageProps {
  params: { id: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export default async function StudentPage({ params }: PageProps) {
  const student = await fetchStudent(params.id);

  return (
    <div>
      <h1>{student.name}</h1>
      <p>{student.email}</p>
    </div>
  );
}

// Generate static params for SSG
export async function generateStaticParams() {
  const students = await fetchAllStudents();

  return students.map((student) => ({
    id: student.id.toString(),
  }));
}
```

### Navigation

```typescript
// Using Link component (client-side navigation)
import Link from 'next/link';

export default function StudentList({ students }) {
  return (
    <ul>
      {students.map((student) => (
        <li key={student.id}>
          <Link href={`/students/${student.id}`}>
            {student.name}
          </Link>
        </li>
      ))}
    </ul>
  );
}

// Programmatic navigation
'use client';  // Client component

import { useRouter } from 'next/navigation';

export default function CreateStudent() {
  const router = useRouter();

  const handleSubmit = async (data) => {
    await createStudent(data);
    router.push('/students');  // Navigate after creation
  };

  return <form onSubmit={handleSubmit}>...</form>;
}

// Prefetching (automatic with Link)
<Link href="/students" prefetch={true}>  {/* Default */}
  Students
</Link>
```

### Route Groups

```typescript
// app/(dashboard)/layout.tsx
// Shared layout for dashboard pages, but "(dashboard)" not in URL

export default function DashboardLayout({ children }) {
  return (
    <div className="dashboard">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}

// app/(dashboard)/analytics/page.tsx  → URL: /analytics
// app/(dashboard)/settings/page.tsx   → URL: /settings
```

---

## Rendering Strategies

Next.js supports 4 rendering strategies:

### 1. Static Site Generation (SSG) - Default

**When:** Content doesn't change often (blog posts, documentation)

```typescript
// app/blog/[slug]/page.tsx
export default async function BlogPost({ params }: { params: { slug: string } }) {
  // Fetched at BUILD TIME
  const post = await fetchPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Generate static pages at build time
export async function generateStaticParams() {
  const posts = await fetchAllPosts();

  return posts.map((post) => ({
    slug: post.slug,
  }));
}

// Result:
// - Pages pre-rendered at build time
// - Served as static HTML (fastest)
// - Great for SEO
```

### 2. Server-Side Rendering (SSR)

**When:** Need fresh data on every request (user-specific, real-time)

```typescript
// app/dashboard/page.tsx
export const dynamic = 'force-dynamic';  // Disable caching

export default async function Dashboard() {
  // Fetched on EVERY REQUEST
  const user = await getCurrentUser();
  const stats = await getUserStats(user.id);

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>Total students: {stats.studentCount}</p>
    </div>
  );
}

// Result:
// - Page rendered on server for each request
// - Fresh data every time
// - Slower than SSG, but always up-to-date
```

### 3. Incremental Static Regeneration (ISR)

**When:** Want static benefits but need periodic updates

```typescript
// app/products/page.tsx
export const revalidate = 3600;  // Revalidate every hour

export default async function Products() {
  const products = await fetchProducts();

  return (
    <div>
      {products.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Result:
// - Static page generated at build time
// - Regenerated in background every 3600 seconds (1 hour)
// - Users get static page (fast), but data refreshes periodically
```

### 4. Client-Side Rendering (CSR)

**When:** Highly interactive, doesn't need SEO

```typescript
'use client';  // Client component

import { useState, useEffect } from 'react';

export default function StudentSearch() {
  const [students, setStudents] = useState([]);
  const [query, setQuery] = useState('');

  useEffect(() => {
    // Fetch on client side
    fetch(`/api/students?q=${query}`)
      .then(res => res.json())
      .then(setStudents);
  }, [query]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search students..."
      />
      <ul>
        {students.map(s => <li key={s.id}>{s.name}</li>)}
      </ul>
    </div>
  );
}

// Result:
// - Rendered in browser
// - Interactive (responds to user input)
// - No SEO (not in initial HTML)
```

### Comparison Table

| Strategy | When Rendered | Data Freshness | Performance | SEO | Use Case |
|----------|---------------|----------------|-------------|-----|----------|
| **SSG** | Build time | Stale until rebuild | Fastest | Best | Blog, docs |
| **ISR** | Build + periodic | Fresh every X seconds | Fast | Best | Product catalog |
| **SSR** | Every request | Always fresh | Slower | Good | Dashboard, user data |
| **CSR** | Client side | Fresh | Fast (after load) | Poor | Search, chat |

---

## Data Fetching

### Server Components (Default in App Router)

```typescript
// app/students/page.tsx (Server Component)
async function getStudents() {
  const res = await fetch('https://api.example.com/students', {
    cache: 'no-store',  // SSR (fetch on every request)
    // cache: 'force-cache',  // SSG (cache forever)
    // next: { revalidate: 3600 }  // ISR (revalidate every hour)
  });

  if (!res.ok) throw new Error('Failed to fetch');
  return res.json();
}

export default async function StudentsPage() {
  const students = await getStudents();  // Async component!

  return (
    <div>
      <h1>Students</h1>
      <ul>
        {students.map(s => <li key={s.id}>{s.name}</li>)}
      </ul>
    </div>
  );
}
```

### Parallel Data Fetching

```typescript
// app/dashboard/page.tsx
async function getUser() {
  const res = await fetch('/api/user');
  return res.json();
}

async function getStats() {
  const res = await fetch('/api/stats');
  return res.json();
}

export default async function Dashboard() {
  // Fetch in parallel (not sequential)
  const [user, stats] = await Promise.all([
    getUser(),
    getStats(),
  ]);

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>Students: {stats.studentCount}</p>
    </div>
  );
}
```

### Sequential Data Fetching (When Dependent)

```typescript
export default async function StudentPage({ params }: { params: { id: string } }) {
  // Fetch student first
  const student = await fetchStudent(params.id);

  // Then fetch classes (depends on student)
  const classes = await fetchStudentClasses(student.id);

  return (
    <div>
      <h1>{student.name}</h1>
      <h2>Classes:</h2>
      <ul>
        {classes.map(c => <li key={c.id}>{c.name}</li>)}
      </ul>
    </div>
  );
}
```

### Client-Side Data Fetching (SWR)

```typescript
'use client';

import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

export default function Students() {
  const { data, error, isLoading } = useSWR('/api/students', fetcher, {
    revalidateOnFocus: true,
    refreshInterval: 5000,  // Refresh every 5 seconds
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map(s => <li key={s.id}>{s.name}</li>)}
    </ul>
  );
}
```

---

## API Routes

### Creating API Routes

```typescript
// app/api/students/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';

// GET /api/students
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const query = searchParams.get('q');

  const students = await db.student.findMany({
    where: query ? {
      name: { contains: query }
    } : undefined
  });

  return NextResponse.json(students);
}

// POST /api/students
export async function POST(request: NextRequest) {
  const body = await request.json();

  // Validate
  if (!body.name || !body.email) {
    return NextResponse.json(
      { error: 'Name and email required' },
      { status: 400 }
    );
  }

  // Create student
  const student = await db.student.create({
    data: {
      name: body.name,
      email: body.email,
    }
  });

  return NextResponse.json(student, { status: 201 });
}
```

### Dynamic API Routes

```typescript
// app/api/students/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';

interface RouteParams {
  params: { id: string }
}

// GET /api/students/:id
export async function GET(request: NextRequest, { params }: RouteParams) {
  const student = await db.student.findUnique({
    where: { id: parseInt(params.id) }
  });

  if (!student) {
    return NextResponse.json(
      { error: 'Student not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(student);
}

// PUT /api/students/:id
export async function PUT(request: NextRequest, { params }: RouteParams) {
  const body = await request.json();

  const student = await db.student.update({
    where: { id: parseInt(params.id) },
    data: body
  });

  return NextResponse.json(student);
}

// DELETE /api/students/:id
export async function DELETE(request: NextRequest, { params }: RouteParams) {
  await db.student.delete({
    where: { id: parseInt(params.id) }
  });

  return NextResponse.json({ success: true }, { status: 204 });
}
```

### API Route with Authentication

```typescript
// app/api/students/route.ts
import { auth } from '@/lib/auth';
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  // Check authentication
  const session = await auth(request);

  if (!session) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    );
  }

  // Check authorization
  if (session.user.role !== 'ADMIN') {
    return NextResponse.json(
      { error: 'Forbidden' },
      { status: 403 }
    );
  }

  const students = await db.student.findMany();
  return NextResponse.json(students);
}
```

---

## Image & Font Optimization

### Image Optimization

```typescript
import Image from 'next/image';

export default function StudentCard({ student }) {
  return (
    <div>
      {/* Automatic optimization */}
      <Image
        src={student.profilePicture}
        alt={student.name}
        width={200}
        height={200}
        priority  // Load immediately (above fold)
      />

      {/* Responsive images */}
      <Image
        src="/hero.jpg"
        alt="Hero"
        fill  // Fill parent container
        sizes="(max-width: 768px) 100vw, 50vw"
        style={{ objectFit: 'cover' }}
      />

      {/* External images (configure in next.config.js) */}
      <Image
        src="https://example.com/image.jpg"
        alt="External"
        width={500}
        height={300}
        loader={({ src, width, quality }) => {
          return `${src}?w=${width}&q=${quality || 75}`;
        }}
      />
    </div>
  );
}
```

**next.config.js:**

```javascript
module.exports = {
  images: {
    domains: ['example.com', 'cdn.example.com'],  // Allow external domains
    formats: ['image/avif', 'image/webp'],  // Modern formats
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
};
```

**Benefits:**
- ✅ Automatic lazy loading
- ✅ Automatic format conversion (WebP, AVIF)
- ✅ Automatic responsive images
- ✅ Prevents layout shift (width/height required)

### Font Optimization

```typescript
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google';

// Google Fonts
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
});

const robotoMono = Roboto_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-roboto-mono',
});

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={`${inter.variable} ${robotoMono.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

**styles/globals.css:**

```css
body {
  font-family: var(--font-inter), sans-serif;
}

code {
  font-family: var(--font-roboto-mono), monospace;
}
```

**Benefits:**
- ✅ Self-hosted (no external request to Google Fonts)
- ✅ Zero layout shift
- ✅ Automatic font subsetting

---

## Performance Optimization

### Code Splitting (Automatic)

```typescript
// Automatic code splitting per route
// Each page only loads its own code

// app/students/page.tsx → students.js (only loaded when visiting /students)
// app/classes/page.tsx → classes.js (only loaded when visiting /classes)
```

### Dynamic Imports (Manual)

```typescript
import dynamic from 'next/dynamic';

// Load component only when needed
const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false,  // Don't render on server (client-only)
});

export default function Dashboard() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && <HeavyChart />}  {/* Loaded only when button clicked */}
    </div>
  );
}
```

### Streaming & Suspense

```typescript
import { Suspense } from 'react';

async function SlowComponent() {
  const data = await slowDataFetch();  // Takes 3 seconds
  return <div>{data}</div>;
}

export default function Page() {
  return (
    <div>
      <h1>Fast content (shows immediately)</h1>

      <Suspense fallback={<p>Loading...</p>}>
        <SlowComponent />  {/* Streams in when ready */}
      </Suspense>

      <p>More fast content</p>
    </div>
  );
}

// User sees:
// 1. "Fast content" immediately
// 2. "Loading..." placeholder
// 3. More fast content
// 4. SlowComponent streams in when ready (3 seconds later)
```

### Metadata for SEO

```typescript
// app/students/[id]/page.tsx
import { Metadata } from 'next';

export async function generateMetadata({ params }): Promise<Metadata> {
  const student = await fetchStudent(params.id);

  return {
    title: `${student.name} - Student Profile`,
    description: `View ${student.name}'s profile, classes, and grades`,
    openGraph: {
      title: student.name,
      description: `Student at Example School`,
      images: [student.profilePicture],
    },
  };
}

export default async function StudentPage({ params }) {
  const student = await fetchStudent(params.id);
  return <div>...</div>;
}
```

---

## App Router vs Pages Router

### Pages Router (Legacy, still supported)

```typescript
// pages/students/[id].tsx
import { GetServerSideProps } from 'next';

export default function StudentPage({ student }) {
  return <div>{student.name}</div>;
}

export const getServerSideProps: GetServerSideProps = async ({ params }) => {
  const student = await fetchStudent(params.id);

  return {
    props: { student }
  };
};
```

### App Router (New, recommended)

```typescript
// app/students/[id]/page.tsx
export default async function StudentPage({ params }) {
  const student = await fetchStudent(params.id);  // Async component!
  return <div>{student.name}</div>;
}
```

### Comparison

```
Pages Router:
- getServerSideProps, getStaticProps, getInitialProps
- pages/ directory
- _app.tsx, _document.tsx
- Older, stable

App Router:
- Async Server Components
- app/ directory
- layout.tsx, loading.tsx, error.tsx
- React Server Components
- Streaming, Suspense
- Better performance
- More flexible
```

---

## Server Components vs Client Components

### Server Components (Default)

```typescript
// app/students/page.tsx
// No 'use client' directive = Server Component

export default async function Students() {
  const students = await db.student.findMany();  // Direct DB access!

  return (
    <ul>
      {students.map(s => <li key={s.id}>{s.name}</li>)}
    </ul>
  );
}
```

**Benefits:**
- ✅ Can access backend directly (DB, file system)
- ✅ No JavaScript sent to client
- ✅ Better performance
- ✅ Better security (API keys, secrets stay on server)

**Limitations:**
- ❌ No useState, useEffect, event handlers
- ❌ No browser APIs

### Client Components

```typescript
'use client';  // Must be at top of file

import { useState } from 'react';

export default function StudentSearch() {
  const [query, setQuery] = useState('');

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}  // Event handler
    />
  );
}
```

**Benefits:**
- ✅ Can use hooks (useState, useEffect, etc.)
- ✅ Event handlers
- ✅ Browser APIs

**Limitations:**
- ❌ Cannot access backend directly
- ❌ JavaScript sent to client

### Composition Pattern

```typescript
// app/students/page.tsx (Server Component)
import StudentSearch from '@/components/StudentSearch';  // Client component

export default async function Students() {
  const students = await db.student.findMany();  // Server-side

  return (
    <div>
      <StudentSearch />  {/* Client component */}
      <StudentList students={students} />  {/* Server component */}
    </div>
  );
}

// components/StudentSearch.tsx (Client Component)
'use client';

import { useState } from 'react';

export default function StudentSearch() {
  const [query, setQuery] = useState('');
  // Interactive search UI
}

// components/StudentList.tsx (Server Component)
export default function StudentList({ students }) {
  return (
    <ul>
      {students.map(s => <li key={s.id}>{s.name}</li>)}
    </ul>
  );
}
```

**Rule of thumb:** Use Server Components by default, only use Client Components when you need interactivity.

---

## Authentication & Middleware

### Middleware

```typescript
// middleware.ts (root of project)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Check authentication
  const token = request.cookies.get('auth-token');

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Add custom header
  const response = NextResponse.next();
  response.headers.set('X-Custom-Header', 'value');

  return response;
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/students/:path*'],
};
```

### Authentication Example (NextAuth.js)

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { compare } from 'bcrypt';
import { db } from '@/lib/db';

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" }
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null;
        }

        const user = await db.user.findUnique({
          where: { email: credentials.email }
        });

        if (!user) return null;

        const isValid = await compare(credentials.password, user.passwordHash);
        if (!isValid) return null;

        return {
          id: user.id,
          email: user.email,
          name: user.name,
        };
      }
    })
  ],
  pages: {
    signIn: '/login',
  },
  session: {
    strategy: 'jwt',
  },
});

export { handler as GET, handler as POST };

// app/dashboard/page.tsx
import { getServerSession } from 'next-auth';

export default async function Dashboard() {
  const session = await getServerSession();

  if (!session) {
    redirect('/login');
  }

  return <div>Welcome, {session.user.name}</div>;
}

// components/LoginButton.tsx
'use client';

import { signIn, signOut, useSession } from 'next-auth/react';

export default function LoginButton() {
  const { data: session } = useSession();

  if (session) {
    return <button onClick={() => signOut()}>Sign out</button>;
  }

  return <button onClick={() => signIn()}>Sign in</button>;
}
```

---

## Deployment & Production

### Build & Deploy

```bash
# Build for production
npm run build

# Output:
# - .next/static/ → Static assets
# - .next/server/ → Server-side code
# - Pre-rendered pages

# Start production server
npm run start

# Or deploy to Vercel (recommended)
vercel deploy
```

### Environment Variables

```bash
# .env.local (development)
DATABASE_URL=postgresql://localhost:5432/school_admin
NEXT_PUBLIC_API_URL=http://localhost:3000/api

# .env.production (production)
DATABASE_URL=postgresql://prod-db.com:5432/school_admin
NEXT_PUBLIC_API_URL=https://api.school-admin.com
```

```typescript
// Server-side (secure)
const dbUrl = process.env.DATABASE_URL;  // Not exposed to client

// Client-side (public)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;  // Available in browser
```

### Performance Monitoring

```typescript
// next.config.js
module.exports = {
  experimental: {
    instrumentationHook: true,
  },
};

// instrumentation.ts
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    // Server-side monitoring
    const { NodeSDK } = await import('@opentelemetry/sdk-node');
    const sdk = new NodeSDK();
    sdk.start();
  }
}
```

---

## Common Interview Questions

### Q1: What is the difference between SSR, SSG, and CSR in Next.js?

**Answer:**

"Next.js supports three main rendering strategies:

**1. SSR (Server-Side Rendering):**
- Page rendered on server for EVERY request
- Fresh data every time
- Good SEO (content in HTML)

```typescript
export const dynamic = 'force-dynamic';

export default async function Page() {
  const data = await fetch('/api/data', { cache: 'no-store' });
  return <div>{data}</div>;
}
```

**When to use:** User-specific data, dashboards, real-time content

**2. SSG (Static Site Generation):**
- Page pre-rendered at BUILD time
- Served as static HTML (fastest)
- Great SEO

```typescript
export default async function Page() {
  const data = await fetch('/api/data', { cache: 'force-cache' });
  return <div>{data}</div>;
}
```

**When to use:** Blog posts, documentation, marketing pages

**3. CSR (Client-Side Rendering):**
- Rendered in browser
- No initial HTML content
- Interactive

```typescript
'use client';

export default function Page() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/data').then(r => r.json()).then(setData);
  }, []);

  return <div>{data}</div>;
}
```

**When to use:** Highly interactive UIs, doesn't need SEO

**Comparison:**

| Strategy | Speed | SEO | Data Freshness |
|----------|-------|-----|----------------|
| SSG | Fastest | Best | Stale until rebuild |
| SSR | Medium | Good | Always fresh |
| CSR | Slow (initial) | Poor | Fresh |

**In my school admin system, I would use:**
- SSG for marketing pages (homepage, about)
- SSR for dashboard (user-specific data)
- CSR for search (interactive, doesn't need SEO)"

### Q2: How does Next.js handle code splitting and why is it important?

**Answer:**

"Next.js automatically code splits at the route level, which means each page only loads the JavaScript it needs.

**Automatic Code Splitting:**

```
Without code splitting (CRA):
bundle.js (500 KB) → Loads everything on first page

With Next.js:
/_app.js (50 KB) → Shared code
/students.js (30 KB) → Only when visiting /students
/classes.js (25 KB) → Only when visiting /classes
```

**Benefits:**
1. **Faster initial load** (smaller bundle)
2. **Better performance** (less JavaScript to parse)
3. **Better caching** (chunks cached separately)

**Manual Code Splitting (dynamic imports):**

```typescript
import dynamic from 'next/dynamic';

// Load heavy component only when needed
const HeavyChart = dynamic(() => import('@/components/Chart'), {
  loading: () => <p>Loading...</p>,
  ssr: false,  // Client-side only
});

export default function Dashboard() {
  return (
    <div>
      <HeavyChart />  {/* Loaded in separate chunk */}
    </div>
  );
}
```

**Real Example:**
In my school admin system, I would use dynamic imports for:
- CSV upload component (only loaded when uploading)
- PDF report generator (only loaded when generating reports)
- Chart library (heavy, only loaded on analytics page)

This reduced initial bundle from 300 KB → 150 KB (50% reduction)."

### Q3: Explain the difference between Server Components and Client Components

**Answer:**

"Next.js 13+ introduced React Server Components, which run only on the server.

**Server Components (default):**

```typescript
// No 'use client' directive
export default async function Students() {
  // Direct database access!
  const students = await db.student.findMany();

  return (
    <ul>
      {students.map(s => <li key={s.id}>{s.name}</li>)}
    </ul>
  );
}
```

**Pros:**
✅ Access backend directly (no API needed)
✅ Zero JavaScript to client
✅ Can use secret keys (never exposed)
✅ Better performance

**Cons:**
❌ No useState, useEffect, event handlers
❌ No browser APIs

**Client Components:**

```typescript
'use client';

import { useState } from 'react';

export default function Search() {
  const [query, setQuery] = useState('');

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}  // Event handler
    />
  );
}
```

**Pros:**
✅ Full React features (hooks, event handlers)
✅ Browser APIs

**Cons:**
❌ JavaScript sent to client
❌ Cannot access backend directly

**Composition Pattern (Best Practice):**

```typescript
// app/students/page.tsx (Server)
import Search from '@/components/Search';  // Client

export default async function Students() {
  const students = await db.student.findMany();  // Server

  return (
    <div>
      <Search />  {/* Client component for interactivity */}
      <StudentList students={students} />  {/* Server component */}
    </div>
  );
}
```

**Rule:** Use Server Components by default, Client Components only when you need interactivity.

In my school admin system:
- Student list → Server Component (fetch from DB)
- Search box → Client Component (interactive)
- Student details → Server Component (fetch from DB)"

### Q4: How would you implement authentication in Next.js?

**Answer:**

"I would use **NextAuth.js** with middleware for protected routes.

**1. Setup NextAuth:**

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      async authorize(credentials) {
        const user = await verifyUser(credentials);
        return user || null;
      }
    })
  ],
  session: { strategy: 'jwt' },
  pages: { signIn: '/login' },
});

export { handler as GET, handler as POST };
```

**2. Protect routes with middleware:**

```typescript
// middleware.ts
import { getToken } from 'next-auth/jwt';
import { NextResponse } from 'next/server';

export async function middleware(request) {
  const token = await getToken({ req: request });

  // Protect /dashboard routes
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    if (!token) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*']
};
```

**3. Use session in Server Components:**

```typescript
// app/dashboard/page.tsx
import { getServerSession } from 'next-auth';

export default async function Dashboard() {
  const session = await getServerSession();

  if (!session) redirect('/login');

  return <div>Welcome, {session.user.name}</div>;
}
```

**4. Use session in Client Components:**

```typescript
'use client';

import { useSession, signIn, signOut } from 'next-auth/react';

export default function Profile() {
  const { data: session, status } = useSession();

  if (status === 'loading') return <p>Loading...</p>;
  if (!session) return <button onClick={() => signIn()}>Sign in</button>;

  return (
    <div>
      <p>{session.user.email}</p>
      <button onClick={() => signOut()}>Sign out</button>
    </div>
  );
}
```

**5. Protect API routes:**

```typescript
// app/api/students/route.ts
import { getServerSession } from 'next-auth';

export async function GET(request) {
  const session = await getServerSession();

  if (!session) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const students = await db.student.findMany();
  return NextResponse.json(students);
}
```

**Benefits:**
✅ Secure (JWT tokens)
✅ Easy integration
✅ Multiple providers (Google, GitHub, etc.)
✅ Built-in CSRF protection"

### Q5: How do you optimize images in Next.js?

**Answer:**

"Next.js provides the `Image` component with automatic optimization:

**1. Basic usage:**

```typescript
import Image from 'next/image';

<Image
  src="/profile.jpg"
  alt="Profile"
  width={200}
  height={200}
  priority  // Load immediately (above fold)
/>
```

**2. Responsive images:**

```typescript
<Image
  src="/hero.jpg"
  alt="Hero"
  fill  // Fill parent container
  sizes="(max-width: 768px) 100vw, 50vw"
  style={{ objectFit: 'cover' }}
/>
```

**3. External images:**

```javascript
// next.config.js
module.exports = {
  images: {
    domains: ['cdn.example.com'],
    formats: ['image/avif', 'image/webp'],
  },
};
```

**Optimizations applied automatically:**
✅ **Lazy loading** (images load when entering viewport)
✅ **Format conversion** (WebP/AVIF for modern browsers)
✅ **Responsive images** (srcset with multiple sizes)
✅ **Blur placeholder** (prevents layout shift)
✅ **Size optimization** (compressed based on device)

**Example with blur placeholder:**

```typescript
import blurDataUrl from '@/lib/blurDataUrl';

<Image
  src={student.photo}
  alt={student.name}
  width={300}
  height={300}
  placeholder="blur"
  blurDataURL={blurDataUrl}  // Low-quality placeholder
/>
```

**Performance impact:**
In my school admin system, optimizing images:
- Reduced page load by 60% (800ms → 320ms)
- Reduced bandwidth by 70% (500 KB → 150 KB per image)
- Improved Lighthouse score from 70 → 95"

### Q6: What is ISR (Incremental Static Regeneration) and when would you use it?

**Answer:**

"**ISR** allows you to update static pages after build without rebuilding the entire site.

**How it works:**

```typescript
// app/products/page.tsx
export const revalidate = 3600;  // Revalidate every hour

export default async function Products() {
  const products = await fetchProducts();

  return (
    <div>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}
```

**Timeline:**

```
Build time:
- Generate static page with products

User A visits (1 hour later):
- Serves static page (fast!)
- Triggers background regeneration
- User A sees old data

User B visits (30 seconds later):
- Serves NEW static page (regenerated)
- User B sees fresh data

Future users:
- Serve new static page until next revalidation
```

**Benefits:**
✅ Static performance (fast)
✅ Fresh data (periodic updates)
✅ No rebuild needed

**On-demand revalidation:**

```typescript
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache';

export async function POST(request) {
  const { path } = await request.json();

  revalidatePath(path);

  return Response.json({ revalidated: true });
}

// Trigger revalidation when product updated
await fetch('/api/revalidate', {
  method: 'POST',
  body: JSON.stringify({ path: '/products' })
});
```

**Use cases:**
- E-commerce product catalog (prices change occasionally)
- Blog (new posts added periodically)
- News site (articles updated every hour)

**In my school admin system:**
- Student directory → ISR (revalidate every 30 min)
- Class schedule → ISR (revalidate when schedule changes)
- Announcements → ISR (revalidate every 15 min)

This gives static performance while keeping data reasonably fresh."

### Q7: How would you handle error boundaries in Next.js?

**Answer:**

"Next.js provides built-in error handling with `error.tsx` files:

**1. Route-level error boundary:**

```typescript
// app/students/error.tsx
'use client';  // Error components must be Client Components

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Log error to error reporting service
    console.error(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

**2. Global error boundary:**

```typescript
// app/global-error.tsx
'use client';

export default function GlobalError({ error, reset }) {
  return (
    <html>
      <body>
        <h2>Application Error</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  );
}
```

**3. Not found pages:**

```typescript
// app/students/[id]/not-found.tsx
import Link from 'next/link';

export default function NotFound() {
  return (
    <div>
      <h2>Student Not Found</h2>
      <p>Could not find the requested student.</p>
      <Link href="/students">Back to students</Link>
    </div>
  );
}

// app/students/[id]/page.tsx
import { notFound } from 'next/navigation';

export default async function Student({ params }) {
  const student = await fetchStudent(params.id);

  if (!student) {
    notFound();  // Shows not-found.tsx
  }

  return <div>{student.name}</div>;
}
```

**4. Loading states:**

```typescript
// app/students/loading.tsx
export default function Loading() {
  return <div>Loading students...</div>;
}

// Automatically shown while page loads
```

**5. Error logging:**

```typescript
// lib/errorReporting.ts
export function logError(error: Error, context?: any) {
  // Send to error tracking service (Sentry, etc.)
  if (process.env.NODE_ENV === 'production') {
    // Sentry.captureException(error, { extra: context });
  } else {
    console.error(error, context);
  }
}

// app/students/error.tsx
useEffect(() => {
  logError(error, { route: '/students' });
}, [error]);
```

This provides graceful error handling at every level."

---

## Best Practices Summary

### Performance Checklist

```
☐ Use Server Components by default
☐ Use Image component for all images
☐ Implement ISR for semi-static content
☐ Use dynamic imports for heavy components
☐ Enable streaming with Suspense
☐ Optimize fonts (use next/font)
☐ Minimize Client Components
☐ Use proper caching strategies
```

### SEO Checklist

```
☐ Add metadata to all pages
☐ Use semantic HTML
☐ Add alt text to images
☐ Use SSR/SSG for public pages
☐ Implement sitemap.xml
☐ Add robots.txt
☐ Use structured data (JSON-LD)
```

### Security Checklist

```
☐ Never expose secrets in client code
☐ Use environment variables
☐ Implement authentication (NextAuth)
☐ Protect API routes
☐ Use HTTPS in production
☐ Sanitize user input
☐ Implement CSRF protection
```

---

**Next.js is powerful because it gives you the best of both worlds: static performance AND dynamic capabilities. Choose the right rendering strategy for each page!**
