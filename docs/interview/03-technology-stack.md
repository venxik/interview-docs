# Technology Stack

This guide covers the technology choices and foundational web concepts used in the project, including Express.js, Sequelize, web architecture, and security fundamentals.

---

## Topics Covered

- Express.js fundamentals
- Sequelize ORM and database concepts
- Middleware and request/response cycle
- HTTP vs HTTPS
- TLS/SSL and certificates
- Reverse proxies and load balancing
- CORS and security

---

### **Category 3: Technology Choices**

These questions are about the "what" and "why" of the tech stack.

**8. Why use an ORM like Sequelize instead of writing raw SQL?**

- **Answer:** "The documentation lists this as a key architectural decision. The trade-offs are:
  - **Benefits:** You get database abstraction (the code isn't tied to MySQL), protection against SQL injection, and often faster development for standard queries.
  - **Drawbacks:** There can be a performance overhead compared to highly optimized raw SQL, and sometimes it's harder to write very complex queries."

**9. What is the role of Express.js in this application?**

- **Answer:** "Express.js is the web server framework. Its main jobs here are:
  - **Routing:** It maps incoming HTTP requests (like `GET /api/class`) to the correct controller functions.
  - **Middleware Pipeline:** It manages the request-response cycle through middleware, as seen in `app.ts` with `cors()`, `bodyParser.json()`, and the `globalErrorHandler`."

**10. What is the difference between `app.ts` and `server.ts`? Why are they separate?**

- **Answer:** "This separation is a key architectural best practice that improves testability and maintains a clean separation of concerns.
  - **`app.ts` is the application configurator.** Its only job is to create and configure the Express application instance. This includes setting up all global middleware (like `cors`, `bodyParser`), attaching the main API router, and adding the final global error handler. It then **exports the configured `app` object without starting it.**
  - **`server.ts` is the application's entry point.** Its only job is to start the server. It imports the `app` object from `app.ts`, handles infrastructure concerns like connecting to the database (with a retry mechanism), and then starts the HTTP server by calling `app.listen()`.
  - **The primary reason for this separation is testability.** For integration tests, we can import the `app` object directly and send it requests without needing to run a live server on a network port. This makes tests faster, more reliable, and able to run in parallel without port conflicts."

**11. Why doesn't `server.ts` use `http.createServer()` directly?**

- **Answer:** "You don't need to use `http.createServer()` because the Express framework provides a convenience method, `app.listen()`, that does it for you. The `app.listen()` function is a wrapper that creates a Node.js `http.Server` instance and attaches the Express app to it. Using `app.listen()` is the standard, idiomatic way to start an Express server because it's more concise."

**12. The server runs on HTTP. Why not HTTPS?**

- **Answer:** "This is standard practice for production architecture. The Node.js server itself doesn't handle the complexity of HTTPS. Instead, **TLS/SSL Termination** is handled by a dedicated service in front of the application, like a reverse proxy (Nginx) or a load balancer (AWS ELB).
  - **How it works:** The load balancer receives the incoming `HTTPS` request, decrypts it, and forwards it to the Node server as a plain `HTTP` request over a secure internal network.
  - **The benefits are significant:** It separates concerns (the app server doesn't need to handle encryption), centralizes certificate management, and improves performance. For local development, using HTTP is also much simpler."

**13. What is "middleware"? Can you give an example from this project?**

- **Answer:** "Middleware functions are pieces of code that have access to the request and response objects. They run in a chain during the request cycle. A great example from this project is the `globalErrorHandler.ts`. It's configured as the very last middleware in `app.ts`, so it can catch any errors that occur in the preceding controllers and format a consistent JSON error response."

**13.1. What is `NextFunction` and why do controllers only call `next()` when there is an error?**

- **Answer:** "This question highlights a core pattern in Express.
  - **What is `NextFunction`?** It's the function, conventionally named `next`, that passes control from one middleware function to the next in the chain. It's the 'pass the baton' mechanism in the middleware relay race.
  - **Why not call `next()` on success?** A controller's main job is to **end the request-response cycle** by sending a response (e.g., `res.json()`, `res.sendStatus()`). Once the response is sent, the cycle is complete, and there's nothing 'next'.
  - **Why call `next(error)` on failure?** Calling `next()` with an argument is a special signal to Express to skip all remaining regular middleware and jump directly to the designated **error-handling middleware**. In this project, that's the `globalErrorHandler`. So, instead of handling errors inside the controller, we delegate them to a central handler for consistent error responses.
  - **In short:** On success, controllers send a response. On failure, they call `next(error)` to pass the problem to the global error handler."

**14. What is the role of `multer` in this project?**

- **Answer:** "`multer` is a specialized Node.js middleware that handles `multipart/form-data` requests, which is the format used for uploading files.
  - **Why it's needed:** Its role is critical for the 'CSV Data Import' feature. When an administrator uploads a CSV file of students or classes, the browser sends it as a `multipart/form-data` request.
  - **How it works:** Multer intercepts and processes this request. It extracts the file from the request body and saves it to a temporary location on the server. It then attaches information about the uploaded file (like its path, size, and original name) to the Express `req` object, typically as `req.file`.
  - **In this project:** The configuration in `src/api/middleware/multer.ts` sets up Multer, likely with file filters to ensure only `.csv` files are accepted. The `UploadDataController` then uses the Multer middleware on its route to receive the file and pass its path to the `DataUploadService` for processing."

**14.1. How would you implement rate limiting in this Express.js project? (Future Improvement)**

- **Answer:** "Rate limiting is a crucial security and performance feature that prevents abuse by limiting how many requests a client can make in a given time window. Here's how I would implement it in this project:

**What is Rate Limiting?**

Rate limiting controls the number of requests a client (identified by IP address or API key) can make to your API within a specific time period. For example:
- Maximum 100 requests per 15 minutes per IP address
- Maximum 10 file uploads per hour per IP address

**Why Do We Need Rate Limiting?**

1. **Prevent DDoS attacks:** Attackers can't overwhelm the server with millions of requests
2. **Prevent brute force attacks:** Limits password guessing or credential stuffing attempts
3. **Prevent resource exhaustion:** Protects database and server resources
4. **Fair usage:** Ensures all users get fair access to the API
5. **Cost control:** Prevents excessive API usage that could increase infrastructure costs

**Step-by-Step Implementation:**

**Step 1: Install the rate limiting package**

```bash
npm install express-rate-limit
npm install --save-dev @types/express-rate-limit
```

**Step 2: Create a rate limiting middleware file**

```typescript
// src/api/middleware/rateLimiter.ts
import rateLimit from 'express-rate-limit';
import { Request, Response } from 'express';

/**
 * General API rate limiter
 * Limits: 100 requests per 15 minutes per IP
 */
export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes in milliseconds
  max: 100, // Maximum 100 requests per windowMs
  message: {
    error: 'Too many requests from this IP, please try again after 15 minutes',
    code: 'RATE_LIMIT_EXCEEDED'
  },
  standardHeaders: true, // Return rate limit info in `RateLimit-*` headers
  legacyHeaders: false, // Disable `X-RateLimit-*` headers
  // Skip rate limiting for successful requests (only count errors)
  skip: (req: Request, res: Response) => res.statusCode < 400,
});

/**
 * Strict rate limiter for sensitive operations
 * Limits: 10 requests per hour per IP
 */
export const strictLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 10, // Maximum 10 requests per hour
  message: {
    error: 'Too many attempts, please try again after 1 hour',
    code: 'RATE_LIMIT_EXCEEDED'
  },
  standardHeaders: true,
  legacyHeaders: false,
});

/**
 * File upload rate limiter
 * Limits: 5 uploads per hour per IP
 */
export const uploadLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // Maximum 5 uploads per hour
  message: {
    error: 'Too many file uploads, please try again after 1 hour',
    code: 'UPLOAD_RATE_LIMIT_EXCEEDED'
  },
  standardHeaders: true,
  legacyHeaders: false,
  // Custom handler for clearer logging
  handler: (req: Request, res: Response) => {
    Logger.warn(`Rate limit exceeded for IP: ${req.ip} on ${req.path}`);
    res.status(429).json({
      error: 'Too many file uploads, please try again after 1 hour',
      code: 'UPLOAD_RATE_LIMIT_EXCEEDED',
      retryAfter: '1 hour'
    });
  }
});

/**
 * Lenient rate limiter for read-only operations
 * Limits: 300 requests per 15 minutes per IP
 */
export const readOnlyLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 300, // Higher limit for read operations
  message: {
    error: 'Too many requests, please try again later',
    code: 'RATE_LIMIT_EXCEEDED'
  },
  standardHeaders: true,
  legacyHeaders: false,
});
```

**Step 3: Apply rate limiters in app.ts (Global)**

```typescript
// src/app.ts
import Express from 'express';
import compression from 'compression';
import cors from 'cors';
import bodyParser from 'body-parser';
import router from './router';
import globalErrorHandler from '@middleware/globalErrorHandler';
import { apiLimiter } from '@middleware/rateLimiter'; // NEW

const App = Express();

App.use(compression());
App.use(cors());
App.use(bodyParser.json());
App.use(bodyParser.urlencoded({ extended: true }));

// Apply global rate limiting to all /api routes
App.use('/api', apiLimiter); // NEW - Apply before router

App.use('/api', router);
App.use(globalErrorHandler);

export default App;
```

**Step 4: Apply specific rate limiters to individual routes**

```typescript
// src/api/controllers/UploadDataController.ts
import { Router, Request, Response, NextFunction } from 'express';
import { uploadFile } from '@middleware/multer';
import { uploadLimiter } from '@middleware/rateLimiter'; // NEW
import { dataUploadService } from '@services';
import { Transaction } from '@repositories';

const router = Router();

// Apply uploadLimiter specifically to upload endpoint
router.post(
  '/upload-data',
  uploadLimiter, // NEW - Apply before uploadFile middleware
  uploadFile,
  async (req: Request, res: Response, next: NextFunction) => {
    // ... existing controller logic
  }
);

export default router;
```

**Step 5: Apply different limits to different route types**

```typescript
// src/router.ts
import Express from 'express';
import UploadDataController from '@controllers/UploadDataController';
import HealthcheckController from '@controllers/HealthcheckController';
import StudentController from '@controllers/StudentController';
import ClassController from '@controllers/ClassController';
import ReportController from '@controllers/ReportController';
import { readOnlyLimiter, strictLimiter } from '@middleware/rateLimiter'; // NEW

const router = Express.Router();

// No rate limit for healthcheck (needed for load balancers)
router.use('/', HealthcheckController);

// Upload has its own strict limiter in the controller
router.use('/', UploadDataController);

// Read-only endpoints get lenient limits
router.use('/', readOnlyLimiter, StudentController); // NEW
router.use('/', readOnlyLimiter, ClassController); // NEW
router.use('/', readOnlyLimiter, ReportController); // NEW

export default router;
```

**Step 6: Update barrel file to export limiters**

```typescript
// src/api/middleware/index.ts
export { default as globalErrorHandler } from './globalErrorHandler';
export { uploadFile } from './multer';
export { apiLimiter, strictLimiter, uploadLimiter, readOnlyLimiter } from './rateLimiter'; // NEW
```

**Advanced Configuration Options:**

**1. Custom Key Generator (Rate limit by user ID instead of IP)**

```typescript
export const userBasedLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  // Use user ID from JWT token instead of IP
  keyGenerator: (req: Request) => {
    return req.user?.id || req.ip; // Fallback to IP if no user
  }
});
```

**2. Skip Certain Routes or Users**

```typescript
export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  skip: (req: Request) => {
    // Skip rate limiting for admin users
    if (req.user?.role === 'admin') return true;

    // Skip rate limiting for whitelisted IPs
    const whitelistedIPs = ['127.0.0.1', '::1'];
    if (whitelistedIPs.includes(req.ip)) return true;

    return false;
  }
});
```

**3. Custom Store (Redis for Production)**

For production with multiple servers, use Redis to share rate limit state:

```typescript
import RedisStore from 'rate-limit-redis';
import Redis from 'ioredis';

const redisClient = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
});

export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:', // Rate limit key prefix
  }),
});
```

**4. Dynamic Rate Limits Based on Endpoint**

```typescript
export const dynamicLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: (req: Request) => {
    // Different limits for different endpoints
    if (req.path.includes('/upload')) return 5;
    if (req.path.includes('/report')) return 20;
    return 100; // Default
  }
});
```

**Response Headers:**

When rate limiting is active, these headers are sent to clients:

```
RateLimit-Limit: 100              # Maximum requests allowed
RateLimit-Remaining: 85           # Requests remaining in window
RateLimit-Reset: 1634567890       # Timestamp when limit resets
```

**HTTP 429 Response Example:**

When limit is exceeded:

```json
HTTP/1.1 429 Too Many Requests
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 1634567890
Content-Type: application/json

{
  "error": "Too many requests from this IP, please try again after 15 minutes",
  "code": "RATE_LIMIT_EXCEEDED"
}
```

**Testing Rate Limiting:**

```typescript
// src/api/middleware/__tests__/rateLimiter.test.ts
import request from 'supertest';
import App from '../../../app';

describe('Rate Limiter', () => {
  it('should allow requests within limit', async () => {
    // Make 5 requests (under limit)
    for (let i = 0; i < 5; i++) {
      const res = await request(App).get('/api/healthcheck');
      expect(res.status).toBe(200);
    }
  });

  it('should block requests exceeding limit', async () => {
    // Make 101 requests (over limit of 100)
    for (let i = 0; i < 101; i++) {
      const res = await request(App).get('/api/healthcheck');

      if (i < 100) {
        expect(res.status).toBe(200);
      } else {
        expect(res.status).toBe(429);
        expect(res.body.code).toBe('RATE_LIMIT_EXCEEDED');
      }
    }
  });

  it('should include rate limit headers', async () => {
    const res = await request(App).get('/api/healthcheck');

    expect(res.headers['ratelimit-limit']).toBeDefined();
    expect(res.headers['ratelimit-remaining']).toBeDefined();
    expect(res.headers['ratelimit-reset']).toBeDefined();
  });
});
```

**Best Practices:**

1. **Different Limits for Different Operations:**
   - Read operations: More lenient (300/15min)
   - Write operations: Moderate (100/15min)
   - Sensitive operations (uploads, password reset): Strict (5-10/hour)

2. **Use Redis in Production:**
   - In-memory store doesn't work with multiple server instances
   - Redis shares rate limit state across all servers

3. **Return Meaningful Headers:**
   - `standardHeaders: true` tells clients when they can retry
   - Helps client-side applications handle rate limits gracefully

4. **Log Rate Limit Violations:**
   - Monitor for potential attacks or abuse
   - Alert on suspicious patterns

5. **Whitelist Internal Services:**
   - Don't rate limit internal service-to-service calls
   - Use API keys or skip by IP address

**Real-World Example - DDoS Attack Scenario:**

**Without Rate Limiting:**
```
Attacker sends 100,000 requests/second
→ Database gets overwhelmed with queries
→ Server runs out of memory
→ Application crashes
→ ALL users affected (downtime)
```

**With Rate Limiting:**
```
Attacker sends 100,000 requests/second
→ Rate limiter blocks after 100 requests
→ Returns 429 status immediately (no database hit)
→ Attacker blocked for 15 minutes
→ Legitimate users unaffected
```

**When to Use Which Limiter:**

| Route/Operation              | Limiter          | Limit             | Reason                             |
|------------------------------|------------------|-------------------|------------------------------------|
| `GET /api/healthcheck`       | None             | Unlimited         | Needed for monitoring              |
| `GET /api/class/:id/students`| `readOnlyLimiter`| 300/15min         | Read-heavy, needs higher limit     |
| `PUT /api/class/:id`         | `apiLimiter`     | 100/15min         | Standard write operation           |
| `POST /api/upload-data`      | `uploadLimiter`  | 5/hour            | Expensive operation, strict limit  |
| `POST /api/auth/login`       | `strictLimiter`  | 10/hour           | Prevent brute force attacks        |
| `GET /api/reports/workload`  | `readOnlyLimiter`| 300/15min         | Complex query but read-only        |

**Summary:**

Rate limiting is essential for:
- ✅ Security (prevent attacks)
- ✅ Performance (prevent resource exhaustion)
- ✅ Fairness (ensure equal access)
- ✅ Cost control (limit infrastructure usage)

For this project, I would:
1. Install `express-rate-limit`
2. Create different limiters for different operation types
3. Apply global limiter to all `/api` routes
4. Apply stricter limiters to sensitive endpoints (uploads)
5. Use Redis store in production for multi-server setups
6. Monitor rate limit violations for security threats"

---
### **Category 8: Web Architecture & Security Fundamentals**

This section covers fundamental concepts about how the web works and how security is handled in a modern production environment.

**20. What is the difference between HTTP and HTTPS?**

- **Answer:**

  - **HTTP (Hypertext Transfer Protocol):** Is the foundational protocol for data communication on the web. It's a plain-text protocol, meaning any data sent—passwords, personal information, etc.—is unencrypted and can be intercepted and read by anyone monitoring the network. It's like sending a postcard.
  - **HTTPS (Hypertext Transfer Protocol Secure):** Is the secure version of HTTP. It uses an encryption layer, **TLS/SSL**, to create a secure, encrypted connection between the client (browser) and the server. This ensures that even if data is intercepted, it cannot be read. It's like sending a sealed, tamper-proof letter.

  | Feature          | HTTP                                 | HTTPS                                                |
  | :--------------- | :----------------------------------- | :--------------------------------------------------- |
  | **Security**     | Unencrypted (insecure)               | Encrypted (secure)                                   |
  | **Default Port** | 80                                   | 443                                                  |
  | **Trust**        | No verification of server identity   | Server identity is verified by a TLS/SSL Certificate |
  | **SEO**          | Can negatively impact search ranking | Preferred by search engines; improves ranking        |

**21. What is a TLS/SSL Certificate?**

- **Answer:** A TLS/SSL Certificate is a digital file that serves two main purposes:
  1.  **Authentication:** It proves that the server you are connecting to is legitimately who it claims to be (e.g., it really is `google.com`). This is verified by a trusted third party called a **Certificate Authority (CA)**.
  2.  **Encryption:** It contains the server's **public key**, which is used during the initial **TLS Handshake** to securely establish a shared secret key for encrypting all communication during the session.

**21.1. Can you explain the TLS Handshake in more detail?**

- **Answer:** "The TLS Handshake is the process where a client (like a browser) and a server establish a secure connection before transferring any data. The two main goals are to **authenticate the server** (prove it's who it says it is) and to **agree on a secret key** for encrypting the conversation. Here’s a simplified, step-by-step breakdown:
  1.  **Client Hello:** The browser sends a `Hello` message, which includes the TLS versions it supports and a random string of bytes.
  2.  **Server Hello & Certificate:** The server responds with its own `Hello`, confirming the TLS version to use. Crucially, it also sends its **SSL Certificate**.
  3.  **Certificate Verification:** The browser checks the server's certificate to ensure it's valid, belongs to the correct domain, and was signed by a trusted **Certificate Authority (CA)**. This is the authentication step.
  4.  **Client Key Exchange:** After verifying the certificate, the browser generates another secret (a "premaster secret"). It encrypts this secret using the **server's public key** (found in the SSL certificate) and sends it to the server.
  5.  **Server Decrypts:** The server uses its **private key** to decrypt the premaster secret. Now, both the client and server share the same secret.
  6.  **Session Keys Created:** Both client and server use the shared secret to independently generate an identical set of symmetric encryption keys, called **session keys**.
  7.  **Secure Communication Begins:** The handshake is complete. Both sides send a "Finished" message, which is the first message encrypted with the new session key. All subsequent data transfer is encrypted using these session keys."

**22. What is a Reverse Proxy?**

- **Answer:** A Reverse Proxy is a server that sits in front of one or more application servers, intercepting all incoming client requests. Instead of a client talking directly to our Node.js application, it talks to the reverse proxy, which then forwards the request to the Node.js server.
- **Key Functions:**
  - **TLS/SSL Termination:** As discussed previously, the reverse proxy handles all the HTTPS encryption/decryption, freeing up the Node.js application.
  - **Load Balancing:** If we have multiple instances of our Node.js application running, the reverse proxy can distribute requests among them to ensure no single instance gets overloaded.
  - **Caching:** It can store and serve static assets (like images, CSS, or some API responses) directly, reducing the load on the application server.
  - **Security:** It acts as a single, hardened entry point to our system, hiding the internal network and application servers from the public internet.

**23. Can you walk through a full request cycle, from client to response, involving all these components?**

- **Answer:** "Certainly. Here is a step-by-step breakdown of a typical secure web request:
  1.  **Client Request:** A user types `https://example.com` into their browser. The browser performs a DNS lookup to get the IP address of the server, which points to the **Reverse Proxy**.
  2.  **TLS Handshake:** The browser initiates a connection with the Reverse Proxy on port 443. The proxy presents its **SSL Certificate**. The browser verifies this certificate with a trusted Certificate Authority. They then perform the TLS handshake to agree on a secret session key. From this point on, communication between the browser and the proxy is encrypted.
  3.  **HTTPS Request:** The browser sends the encrypted HTTP request (e.g., `GET /api/students`) to the Reverse Proxy.
  4.  **TLS Termination:** The Reverse Proxy **decrypts** the request using the session key, revealing the plain HTTP request. This is called TLS Termination.
  5.  **Forwarding:** The Reverse Proxy forwards the plain `HTTP` request to one of the available Node.js application servers on the internal network (e.g., to port 3000).
  6.  **Application Processing:** Our Express application receives the plain `HTTP` request. It goes through the middleware, router, controller, service, and repository layers to fetch the required data from the database.
  7.  **Application Response:** The application sends a plain `HTTP` response (e.g., a JSON object) back to the Reverse Proxy.
  8.  **Encryption:** The Reverse Proxy receives the plain response and **encrypts** it using the session key from the TLS handshake.
  9.  **HTTPS Response:** The Reverse Proxy sends the encrypted `HTTPS` response back to the user's browser.
  10. **Client Renders:** The browser decrypts the response and renders the content to the user."

---
### **Category 9: Express.js Fundamentals for Beginners**

These are essential Express.js concepts you should understand as a beginner.

**24. What are the Request (`req`) and Response (`res`) objects in Express?**

- **Answer:**

  - **Request Object (`req`):** Contains all information about the incoming HTTP request:

    - `req.params`: URL parameters (e.g., `/api/teachers/:id` → `req.params.id`)
    - `req.query`: Query string parameters (e.g., `/api/teachers?email=test@test.com` → `req.query.email`)
    - `req.body`: Request body (parsed JSON, requires body-parser middleware)
    - `req.headers`: HTTP headers
    - `req.method`: HTTP method (GET, POST, etc.)

  - **Response Object (`res`):** Used to send responses back to the client:
    - `res.json(data)`: Send JSON response
    - `res.status(code)`: Set HTTP status code
    - `res.send(data)`: Send any type of response
    - `res.sendStatus(code)`: Send status code only

**Example from this project:**

```typescript
// From ClassController.ts
export const listAllClasses = async (
  req: Request,
  res: Response,
  next: NextFunction,
) => {
  try {
    const classes = await classService.findAll();
    res.json(classes); // Send JSON response
  } catch (error) {
    next(error); // Pass error to error handler
  }
};
```

**25. What's the difference between `req.params`, `req.query`, and `req.body`?**

- **Answer:**

  - **`req.params`**: URL path parameters defined in the route

    ```typescript
    // Route: GET /api/teachers/:id
    // URL: /api/teachers/5
    // Access: req.params.id = "5"
    ```

  - **`req.query`**: Query string parameters after `?` in URL

    ```typescript
    // Route: GET /api/teachers
    // URL: /api/teachers?email=test@test.com&status=active
    // Access: req.query.email = "test@test.com"
    //         req.query.status = "active"
    ```

  - **`req.body`**: Data sent in the request body (POST/PUT requests)
    ```typescript
    // Route: POST /api/teachers
    // Request body: { "name": "John", "email": "john@test.com" }
    // Access: req.body.name = "John"
    //         req.body.email = "john@test.com"
    // NOTE: Requires body-parser middleware to parse JSON
    ```

**26. What is the purpose of `body-parser` middleware?**

- **Answer:** "`body-parser` is a middleware that parses incoming request bodies before your handlers get to them. Without it, `req.body` would be `undefined`.
  - **In this project:** It's configured in `app.ts` as `bodyParser.json()`, which tells Express to parse JSON payloads.
  - **Example:** When a client sends `POST /api/upload-data` with JSON data, `body-parser` parses it so the controller can access it via `req.body`.

**27. What is CORS and why do we need it?**

- **Answer:** **CORS (Cross-Origin Resource Sharing)** is a browser security mechanism that controls which domains can access your API.

  **Understanding Origins:**

  - An **origin** is defined by: `protocol + domain + port`
  - Examples:
    - `http://localhost:3000` (different from `http://localhost:4200` - different port)
    - `https://example.com` (different from `http://example.com` - different protocol)
    - `https://api.example.com` (different from `https://example.com` - different subdomain)

  **The Security Problem (Same-Origin Policy):**

  - Browsers enforce the **Same-Origin Policy** for security
  - By default, JavaScript on `https://frontend.com` CANNOT make AJAX/fetch requests to `https://api.backend.com`
  - This prevents malicious websites from accessing your APIs

  **Example of Blocked Request:**

  ```
  Frontend: http://localhost:4200 (Angular/React app)
     ↓ Makes API call
  Backend: http://localhost:3000/api/students
     ↓ Browser blocks!
  Error: "Access to fetch at 'http://localhost:3000/api/students'
         from origin 'http://localhost:4200' has been blocked by CORS policy"
  ```

  **The Solution - CORS Headers:**
  The `cors()` middleware in `app.ts` adds HTTP response headers that tell the browser: "It's okay for other domains to access this API."

  **Key CORS Headers:**

  ```
  Access-Control-Allow-Origin: http://localhost:4200
  Access-Control-Allow-Methods: GET, POST, PUT, DELETE
  Access-Control-Allow-Headers: Content-Type, Authorization
  Access-Control-Allow-Credentials: true
  ```

  **How CORS Works (Preflight Request):**

  ```
  1. Browser sends OPTIONS request (preflight):
     OPTIONS /api/students
     Origin: http://localhost:4200
     Access-Control-Request-Method: POST
     Access-Control-Request-Headers: Content-Type

  2. Server responds with CORS headers:
     HTTP/1.1 200 OK
     Access-Control-Allow-Origin: http://localhost:4200
     Access-Control-Allow-Methods: GET, POST, PUT, DELETE
     Access-Control-Allow-Headers: Content-Type

  3. If approved, browser sends actual request:
     POST /api/students
     Content-Type: application/json
     { "name": "John", "email": "john@test.com" }

  4. Server responds with data + CORS headers:
     HTTP/1.1 200 OK
     Access-Control-Allow-Origin: http://localhost:4200
     { "id": 123, "name": "John", "email": "john@test.com" }
  ```

  **CORS Configuration Options:**

  ```typescript
  // In app.ts

  // Option 1: Allow all origins (development only)
  app.use(cors());

  // Option 2: Allow specific origin (production)
  app.use(
    cors({
      origin: "https://frontend.example.com",
    }),
  );

  // Option 3: Allow multiple origins
  app.use(
    cors({
      origin: ["https://frontend.example.com", "https://admin.example.com"],
      methods: ["GET", "POST", "PUT", "DELETE"],
      allowedHeaders: ["Content-Type", "Authorization"],
      credentials: true, // Allow cookies
    }),
  );

  // Option 4: Dynamic origin (most flexible)
  app.use(
    cors({
      origin: (origin, callback) => {
        const allowedOrigins = [
          "http://localhost:4200",
          "https://frontend.example.com",
        ];

        if (!origin || allowedOrigins.includes(origin)) {
          callback(null, true);
        } else {
          callback(new Error("Not allowed by CORS"));
        }
      },
    }),
  );
  ```

  **Why We Need CORS in This Project:**

  - Our frontend (Angular/React running on `localhost:4200`) needs to call our backend API (Express running on `localhost:3000`)
  - Different ports = different origins
  - Without CORS, the browser would block all API requests from the frontend
  - The `cors()` middleware allows cross-origin requests during development

  **Security Best Practices:**

  1. **Development:** Use `cors()` to allow all origins for convenience
  2. **Production:** Specify exact allowed origins (whitelist)
  3. **Never use** `Access-Control-Allow-Origin: *` with credentials in production
  4. **Validate** origin on the server-side for sensitive operations

  **Common CORS Issues:**

  - **Issue:** CORS error even with middleware

    - **Cause:** CORS middleware must be registered before routes
    - **Fix:** Place `app.use(cors())` at the top of `app.ts`

  - **Issue:** Preflight request fails

    - **Cause:** Server doesn't handle OPTIONS method
    - **Fix:** CORS middleware automatically handles OPTIONS

  - **Issue:** Credentials not sent
    - **Cause:** Need both server and client config
    - **Fix:** Server: `credentials: true`, Client: `withCredentials: true`

**28. What's the difference between synchronous and asynchronous code? Why use `async/await`?**

- **Answer:**

  - **Synchronous:** Code runs line by line, blocking execution until each operation completes. Bad for I/O operations (database, file system).
  - **Asynchronous:** Code doesn't wait for operations to complete, allowing other code to run. Essential for server applications.

  **Why use `async/await` (instead of callbacks or `.then()`):**

  - **Readability:** Looks like synchronous code, easier to understand
  - **Error Handling:** Can use try/catch instead of `.catch()`
  - **Cleaner:** No callback hell or promise chains

  **Example from project:**

  ```typescript
  // Clean async/await (what we use)
  async findAll() {
    const classes = await classRepository.findAll();  // Wait for DB
    return classes;
  }

  // vs Promise chains (harder to read)
  findAll() {
    return classRepository.findAll()
      .then(classes => classes)
      .catch(error => { throw error; });
  }
  ```

**29. What does `try/catch` do and why is it in every controller?**

- **Answer:** `try/catch` handles errors in async code:

  - **`try` block:** Code that might fail (database queries, file operations)
  - **`catch` block:** Handles errors if they occur

  **In controllers:** Every controller has try/catch because:

  1. Database operations can fail (connection lost, constraint violation)
  2. We want to catch these errors and pass them to the global error handler via `next(error)`
  3. Without try/catch, the app would crash on unhandled errors

**30. What is the difference between `app.use()` and `app.get()` / `app.post()`?**

- **Answer:**

  - **`app.use(middleware)`**: Applies middleware to **all routes** or routes matching a path prefix

    ```typescript
    app.use(cors()); // Applies to all routes
    app.use("/api", router); // Applies router to all /api/* routes
    ```

  - **`app.get()`, `app.post()`, etc.**: Define specific route handlers for specific HTTP methods
    ```typescript
    app.get("/api/classes", getAllClasses); // Only GET requests
    app.post("/api/classes", createClass); // Only POST requests
    ```

---
### **Category 10: Database & Sequelize Concepts**

**31. What is the difference between `create`, `upsert`, and `findOrCreate` in Sequelize?**

- **Answer:** (Based on our recent refactoring)

  | Method             | When to Use                                | Returns               | Behavior if Exists  | Behavior if Not Exists |
  | ------------------ | ------------------------------------------ | --------------------- | ------------------- | ---------------------- |
  | **`create`**       | Creating new records                       | Instance              | ❌ Throws error     | ✅ Creates record      |
  | **`upsert`**       | Create or update                           | `[instance, created]` | ✅ Updates existing | ✅ Creates new         |
  | **`findOrCreate`** | Get or create (deprecated in this project) | `[instance, created]` | ✅ Returns existing | ✅ Creates new         |

  **Our Pattern:** We use `upsert` in `findOrCreateRelationship` and `findOrCreateData` methods for atomic create-or-get operations.

**32. What is a database transaction and why use them?**

- **Answer:** A transaction is a group of database operations that must **all succeed or all fail together** (atomicity).

  **Example scenario from DataUploadService:**

  ```typescript
  // Upload 100 students
  // Operation 1: Create 50 students ✅
  // Operation 2: Create 50 more students ❌ ERROR!

  // WITHOUT transaction: 50 students are in DB, 50 are missing (inconsistent!)
  // WITH transaction: ALL 100 are rolled back (database stays consistent!)
  ```

  **ACID Properties:**

  - **Atomicity:** All or nothing
  - **Consistency:** Database stays valid
  - **Isolation:** Transactions don't interfere with each other
  - **Durability:** Committed changes persist even after crashes

**33. What is an ORM? What are the trade-offs?**

- **Answer:** **ORM (Object-Relational Mapping)** is a technique that lets you interact with a database using objects instead of SQL.

  **Benefits:**

  - ✅ Database agnostic (switch from MySQL to PostgreSQL easily)
  - ✅ Prevents SQL injection automatically
  - ✅ Faster development (no SQL writing for simple queries)
  - ✅ Type-safe with TypeScript

  **Drawbacks:**

  - ❌ Performance overhead vs optimized raw SQL
  - ❌ Complex queries can be difficult
  - ❌ Learning curve for the ORM itself

**34. What are Sequelize Models and why define associations?**

- **Answer:**

  - **Models** are classes that represent database tables (e.g., `Teacher`, `Student`, `Class`)
  - **Associations** define relationships between models (e.g., "A Class has many Students")

  **Why define associations?**

  - **Enables eager loading:** Fetch related data in one query

    ```typescript
    // Instead of 2 queries
    const teacher = await Teacher.findOne({ where: { id: 1 } });
    const subjects = await TeacherClassSubject.findAll({
      where: { teacherId: teacher.id },
    });

    // Single query with include
    const teacher = await Teacher.findOne({
      where: { id: 1 },
      include: [Subject], // Automatically joined!
    });
    ```

  - **Cleaner code:** Sequelize handles the JOIN logic
  - **Type safety:** TypeScript knows the shape of associated data

**35. What's the difference between `belongsTo`, `hasMany`, and `belongsToMany`?**

- **Answer:** These define different relationship types in Sequelize:

  | Association         | Relationship | Foreign Key Location | Example                                                       |
  | ------------------- | ------------ | -------------------- | ------------------------------------------------------------- |
  | **`belongsTo`**     | Many-to-One  | Current model        | `ClassStudent.belongsTo(Class)` - ClassStudent has classId    |
  | **`hasMany`**       | One-to-Many  | Related model        | `Class.hasMany(ClassStudent)` - Class owns many ClassStudents |
  | **`belongsToMany`** | Many-to-Many | Junction table       | `Class.belongsToMany(Student, { through: ClassStudent })`     |

  **From our project (models/index.ts):**

  ```typescript
  // ClassStudent belongs to ONE Class
  ClassStudent.belongsTo(Class, { foreignKey: "classId" });

  // Class has MANY ClassStudents
  Class.hasMany(ClassStudent, { foreignKey: "classId" });

  // Class has MANY Students (through ClassStudent junction table)
  Class.belongsToMany(Student, { through: ClassStudent });
  ```

**35.1. Are `belongsToMany` associations optional? Why did I get "Class is not associated to Student" error?**

- **Answer:** This is a common misconception. The `belongsToMany` association is **optional for your data model** (the database doesn't require it), but it's **required if your queries actively use it**.

  **The Problem:**

  - In `StudentListingService.ts:44-50`, the code uses `include` with `model: Class` and `through: { attributes: [] }`
  - The `through` option is specifically for `belongsToMany` associations - it tells Sequelize to use a many-to-many relationship through a junction table
  - When you removed the `belongsToMany` from `models/index.ts`, Sequelize no longer knew how `Student` and `Class` are related through the `ClassStudent` table
  - The error "Class is not associated to Student" occurs at **query time**, not at model definition time

  **The Original Query Pattern:**

  ```typescript
  // This requires belongsToMany to be defined:
  const localStudents = await studentRepository.findAll({
    include: [
      {
        model: Class, // ← Uses Class model directly
        where: { id: classEntity.id },
        through: { attributes: [] }, // ← through implies belongsToMany!
        attributes: [],
      },
    ],
  });
  ```

  **Two Solutions:**

  1. **Keep `belongsToMany`** (if you prefer the simpler query syntax):

     ```typescript
     // In models/index.ts - add these back:
     Class.belongsToMany(Student, {
       through: ClassStudent,
       foreignKey: "classId",
     });
     Student.belongsToMany(Class, {
       through: ClassStudent,
       foreignKey: "studentId",
     });
     ```

  2. **Use existing `hasMany`/`belongsTo` through ClassStudent** (what we did):
     ```typescript
     // Query through the junction table directly:
     const localStudents = await studentRepository.findAll({
       include: [
         {
           model: ClassStudent, // ← Use junction table model
           where: { classId: classEntity.id },
           attributes: [],
         },
       ],
     });
     ```

  **Key Takeaway:** Associations are optional for your data model but **mandatory if your queries depend on them**. Always check your query patterns before removing associations.

**36. What is `.get({ plain: true })` in Sequelize and when to use it?**

- **Answer:** (From our WorkloadReportService analysis)

  - **What it does:** Converts a Sequelize Model instance into a plain JavaScript object
  - **Why use it:**
    - Removes Sequelize metadata (methods, getters, internal state)
    - Better for business logic that doesn't need database operations
    - Safer for JSON serialization
    - Lighter weight in memory

  **Example:**

  ```typescript
  const teacher = await Teacher.findOne({ include: [Subject] });
  // teacher is a Sequelize Model instance with 100s of properties

  const plainTeacher = teacher.get({ plain: true });
  // plainTeacher is { id: 1, name: "John", email: "...", Subject: {...} }
  ```

---
