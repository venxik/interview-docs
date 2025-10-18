# Common Mistakes & Debugging

This guide covers common beginner mistakes and practical debugging tips.

---

## Topics Covered

- Common JavaScript/TypeScript pitfalls
- Express.js best practices
- Sequelize common mistakes
- Debugging strategies and tools
- Error handling patterns

---

### **Category 11: Common Beginner Mistakes & Best Practices**

**37. Why shouldn't you modify `req` or `res` objects directly?**

- **Answer:** While technically possible, it breaks Express conventions:
  - ❌ **Bad:** `req.userId = 5` (mutating request)
  - ✅ **Better:** Use middleware to add typed properties or pass data through function parameters
  - **Reason:** Makes code harder to understand, test, and maintain

**38. What happens if you call `res.json()` twice?**

- **Answer:** **Error!** You can only send one response per request.

  ```typescript
  // This will crash
  res.json({ status: "ok" });
  res.json({ data: [] }); // Error: Cannot set headers after they are sent
  ```

  **Common mistake in beginners:**

  ```typescript
  if (error) {
    res.status(500).json({ error });
    // FORGOT TO RETURN! Code continues...
  }
  res.json({ success: true }); // CRASHES!
  ```

  **Fix:** Always `return` after sending a response:

  ```typescript
  if (error) {
    return res.status(500).json({ error }); // ✅ Return stops execution
  }
  res.json({ success: true });
  ```

**39. Why use `async/await` in controllers but the service doesn't always need it?**

- **Answer:**
  - **Controllers** use `async` because they call async services and need `try/catch` for error handling
  - **Services** use `async` when they perform async operations (database calls)
  - **Rule of thumb:** If your function calls `await`, it must be `async`

**40. What's the difference between throwing an error and calling `next(error)`?**

- **Answer:**

  - **`throw error`** inside try/catch: Caught by catch block, then passed to `next(error)`
  - **`next(error)`**: Directly passes error to Express error-handling middleware

  **In our controllers:**

  ```typescript
  try {
    const result = await service.doSomething();
    // If doSomething() throws, catch block handles it
    res.json(result);
  } catch (error) {
    next(error); // Pass to globalErrorHandler
  }
  ```

---
### **Category 12: Practical Debugging Tips**

**41. How would you debug if an API endpoint returns 404?**

- **Answer:** Check in this order:
  1. **Route defined?** Check `router.ts` - is the route registered?
  2. **HTTP method correct?** Using POST but route expects GET?
  3. **URL typo?** `/api/class` vs `/api/classes`
  4. **Router mounted?** Check `app.ts` - is router attached with `app.use('/api', router)`?
  5. **Middleware blocking?** Any middleware rejecting the request early?

**42. How would you debug if the API returns 500 Internal Server Error?**

- **Answer:**
  1. **Check server console** for error logs
  2. **Look at the stack trace** - which file/line caused the error?
  3. **Common causes:**
     - Database connection failed
     - Typo in variable name (`res.jason()` instead of `res.json()`)
     - Accessing undefined property (`user.email` when `user` is null)
     - Missing await on async function
  4. **Use try/catch** and log the error: `console.error(error)`

**43. How do you test an API endpoint manually?**

- **Answer:** Multiple tools:

  1. **Postman** or **Insomnia**: GUI tools for API testing
  2. **curl** command line:

     ```bash
     # GET request
     curl http://localhost:3000/api/classes

     # POST request with JSON
     curl -X POST http://localhost:3000/api/classes \
       -H "Content-Type: application/json" \
       -d '{"code":"P1-1", "name":"Primary 1 Class 1"}'
     ```

  3. **Browser** (only for GET requests):
     - Open `http://localhost:3000/api/classes`

---
