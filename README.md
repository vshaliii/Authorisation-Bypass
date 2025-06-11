# Authorization Bypass

**Authorization bypass** is a type of security vulnerability where an attacker gains access to **resources, functionality, or data** they should **not be authorized** to access. It happens when an application **fails to properly enforce access controls** on user actions or objects.

### **Difference Between Authentication vs Authorization**

| Term | Meaning |
| --- | --- |
| **Authentication** | Verifies *who* the user is (e.g., login) |
| **Authorization** | Verifies *what* the user is allowed to do (e.g., view or modify specific data) |

### Real-World Example

Imagine an online banking site where user A can access their account at:

```
https://bank.com/account/12345
```

But if they manually change the URL to:

```
https://bank.com/account/12346
```

And can now view user B's account — the app **is not checking if the user is authorized** to access that resource. This is a **direct object reference** that leads to **authorization bypass**.

## Common Authorization Bypass Techniques

## 1. IDOR (Insecure Direct Object Reference)

IDOR occurs when an application exposes a reference to an internal implementation object (like a file, user ID, or database key) and does not properly validate if the user has access to it.

### Example

A web application has a profile page that loads data using a user ID in the URL:

```
GET https://example.com/api/user/profile?id=102
```

You are logged in as user `john` (ID: 102), and you receive:

```json
{
  "id": 102,
  "name": "John Doe",
  "email": "john@example.com"
}
```

Now you **change the `id` to 103**:

```
GET https://example.com/api/user/profile?id=103
```

If you receive this:

```json
{
  "id": 103,
  "name": "Jane Smith",
  "email": "jane@example.com"
}
```

**Impact**: This is a clear case of IDOR. You accessed another user’s private data without authorization.

**Checklist**:

- Access another user's resource
- Change numeric or UUID values
- Check if access is blocked or allowed

### **Prevention**

- Enforce access control on the backend (not just the UI).
- Use indirect references (e.g., hashes or tokens).
- Implement object-level authorization.

---

## 2. Forced Browsing

**Test Case**: Manually access unauthorized pages

**Example**:

```
GET /admin
GET /internal/config
```

**Checklist**:

- Try accessing pages without logging in
- Check for misconfigured `robots.txt` or `.git` leaks

---

## 3. Privilege Escalation

Occurs when a low-privilege user gains access to higher-privilege actions or resources, typically due to missing or broken access controls.

**Example**

You log in as a normal user with role `user`. You intercept a request when viewing your dashboard:

```
GET /api/dashboard
Authorization: Bearer <your_JWT_token>
```

The JWT token contains:

```json
{
  "user": "normaluser",
  "role": "user"
}
```

You change it to:

```json

  "user": "normaluser",
  "role": "admin"
}
```

Then re-encode and send it again. Now you try:

```
GET /api/admin/users
```

If you now see all user data or access admin functions, the system **fails to verify** your actual permissions.

**Impact**: This allows unauthorized access to sensitive admin-level features. Critical vulnerability.
**How to Test:**

- Look for role-based parameters in requests or JWTs.
- Try altering them from `user → admin` or `false → true`.
- Check endpoints that may exist for admins (e.g., `/admin`, `/users/all`).
- Look for hidden UI elements you can call directly via API.

P**revention**

- Role/privilege validation must be enforced server-side.
- Avoid relying only on frontend/UI controls.
- Use Role-Based Access Control (RBAC) with strict checks.

---

## 4. Parameter Tampering

**Test Case**: Modify request parameters to escalate

**Example**:

```
POST /transfer
account_id=123 → account_id=124
```

**Checklist**:

- Change IDs and observe response
- Check for hidden fields in POST bodies

---

## 5. JWT Token Manipulation

**Test Case**: Modify JWT payload

**Example**:

```json
{
  "alg": "none",
  "role": "admin"
}
```

**Checklist**:

- Try unsigned JWT (`"alg": "none"`)
- Modify and re-sign JWTs using weak secrets or `HS256`

---

## 6. Missing Access Control

**Test Case**: Replay authorized requests without tokens

**Example**:

```
GET /admin/dashboard without session token
```

**Checklist**:

- Bypass UI and directly access APIs
- Test expired/invalid session tokens

---

## 7. Header Injection

**Test Case**: Add custom headers

**Example**:

```
X-User-Role: admin
X-Forwarded-For: 127.0.0.1
```

**Checklist**:

- Spoof internal IPs
- Fake authentication headers

---

## 8. HTTP Method Bypass

**Test Case**: Try different HTTP methods

**Example**:

```
PUT /admin/delete
HEAD /admin
```

**Checklist**:

- Use GET, POST, PUT, DELETE, PATCH, HEAD
- Identify if any method exposes unintended behaviour

## How to Prevent Authorization Bypass

1. Enforce authorization on **every API and endpoint**
2. Use **role-based access control (RBAC)** on server side
3. Never trust client-side data (hidden fields, headers)
4. Use secure and validated **session management**
5. Log and monitor all access to sensitive endpoints
