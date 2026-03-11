# Security Checklist for React Applications

## Table of Contents
1. [XSS Prevention](#xss-prevention)
2. [Authentication & Authorization](#authentication--authorization)
3. [Sensitive Data Handling](#sensitive-data-handling)
4. [API Security](#api-security)
5. [Dependencies](#dependencies)

---

## XSS Prevention

### dangerouslySetInnerHTML
```tsx
// ❌ CRITICAL: User input in dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: userComment }} />

// ✅ Sanitize if absolutely necessary
import DOMPurify from 'dompurify';

<div 
  dangerouslySetInnerHTML={{ 
    __html: DOMPurify.sanitize(userComment) 
  }} 
/>

// ✅ Better: Use text content
<div>{userComment}</div>
```

### URL Injection
```tsx
// ❌ User-controlled href
<a href={userProvidedUrl}>Link</a>

// ✅ Validate URL protocol
function SafeLink({ href, children }) {
  const isSafe = /^https?:\/\//.test(href);
  
  if (!isSafe) {
    console.warn('Blocked potentially unsafe URL:', href);
    return <span>{children}</span>;
  }
  
  return <a href={href} rel="noopener noreferrer">{children}</a>;
}
```

### JavaScript Protocol
```tsx
// ❌ Allows javascript: protocol
<a href={`javascript:${userInput}`}>Click</a>

// ❌ User-controlled onClick
<button onClick={new Function(userCode)}>Run</button>
```

### Iframe Security
```tsx
// ❌ Unrestricted iframe
<iframe src={externalUrl} />

// ✅ Sandboxed iframe
<iframe
  src={externalUrl}
  sandbox="allow-scripts allow-same-origin"
  referrerPolicy="no-referrer"
/>
```

---

## Authentication & Authorization

### Token Storage
```tsx
// ❌ Token in localStorage (XSS vulnerable)
localStorage.setItem('token', accessToken);

// ✅ HttpOnly cookies for refresh tokens (server-side)
// ✅ In-memory storage for access tokens
const [accessToken, setAccessToken] = useState(null);

// ✅ Or secure context provider
function AuthProvider({ children }) {
  const tokenRef = useRef(null);
  
  const setToken = useCallback((token) => {
    tokenRef.current = token;
  }, []);
  
  return (
    <AuthContext.Provider value={{ getToken: () => tokenRef.current, setToken }}>
      {children}
    </AuthContext.Provider>
  );
}
```

### Authorization Checks
```tsx
// ❌ Client-side only authorization
{user.role === 'admin' && <AdminPanel />}

// ✅ Server must also verify
// Client-side check is UX only, not security
function AdminRoute({ children }) {
  const { user } = useAuth();
  
  if (!user?.permissions?.includes('admin')) {
    return <Navigate to="/unauthorized" />;
  }
  
  // Server MUST verify on each API call
  return children;
}
```

### Session Management
```tsx
// ✅ Clear auth state on logout
function useLogout() {
  const { setToken } = useAuth();
  
  return useCallback(async () => {
    // Clear server session
    await api.post('/auth/logout');
    
    // Clear client state
    setToken(null);
    
    // Clear any cached data
    queryClient.clear();
    
    // Redirect
    window.location.href = '/login';
  }, [setToken]);
}
```

---

## Sensitive Data Handling

### Environment Variables
```tsx
// ❌ Secret in client bundle (VITE/CRA exposes VITE_/REACT_APP_ vars)
const API_KEY = import.meta.env.VITE_SECRET_API_KEY;

// ✅ Public keys only in client
const PUBLIC_STRIPE_KEY = import.meta.env.VITE_STRIPE_PUBLIC_KEY;

// ✅ Secrets stay on server
// Call your backend, which uses the secret
```

### Sensitive Form Data
```tsx
// ✅ Don't log sensitive data
function LoginForm() {
  const handleSubmit = (data) => {
    // ❌ Never log passwords
    console.log('Login attempt:', data);
    
    // ✅ Log safely
    console.log('Login attempt for:', data.email);
  };
}

// ✅ Clear password fields after use
function PasswordInput() {
  const [value, setValue] = useState('');
  
  const handleSubmit = () => {
    submitPassword(value);
    setValue(''); // Clear immediately
  };
}
```

### Error Handling
```tsx
// ❌ Exposing internal errors to users
catch (error) {
  setError(error.message); // May contain stack traces, DB info
}

// ✅ User-friendly messages
catch (error) {
  console.error('Login failed:', error); // Log full error server-side
  setError('Login failed. Please check your credentials.');
}
```

---

## API Security

### Request Forgery Prevention
```tsx
// ✅ Include CSRF token in requests
const api = axios.create({
  baseURL: '/api',
  withCredentials: true,
  headers: {
    'X-CSRF-Token': getCsrfToken(),
  },
});
```

### Input Validation
```tsx
// ✅ Validate and sanitize all inputs
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().positive().max(150),
});

function handleSubmit(formData) {
  const result = userSchema.safeParse(formData);
  
  if (!result.success) {
    setErrors(result.error.flatten());
    return;
  }
  
  // Safe to use result.data
  api.post('/users', result.data);
}
```

### Rate Limiting Awareness
```tsx
// ✅ Handle rate limit responses gracefully
async function fetchWithRetry(url, retries = 3) {
  try {
    return await api.get(url);
  } catch (error) {
    if (error.response?.status === 429 && retries > 0) {
      const delay = error.response.headers['retry-after'] * 1000 || 5000;
      await new Promise(r => setTimeout(r, delay));
      return fetchWithRetry(url, retries - 1);
    }
    throw error;
  }
}
```

---

## Dependencies

### Vulnerability Scanning
```bash
# Regular audits
npm audit
pnpm audit
yarn audit

# Fix automatically where possible
npm audit fix
```

### Dependency Review Checklist
- [ ] Package has active maintenance (recent commits)
- [ ] Package has reasonable download count
- [ ] No critical vulnerabilities in `npm audit`
- [ ] Minimal dependency tree (fewer transitive deps)
- [ ] Trusted publisher/organization

### Lock Files
```tsx
// ✅ Always commit lock files
// package-lock.json (npm)
// pnpm-lock.yaml (pnpm)
// yarn.lock (yarn)

// ✅ Use exact versions for critical packages in CI
npm ci  // Uses lock file exactly
```

### Subresource Integrity
```html
<!-- ✅ SRI for external scripts -->
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-xyz..."
  crossorigin="anonymous"
></script>
```

---

## Review Flags

When reviewing, flag these as **CRITICAL**:
- `dangerouslySetInnerHTML` with user input
- `eval()` or `new Function()` with user input
- Tokens/secrets in client code or localStorage
- Missing server-side authorization
- User-controlled URLs in `href` or `src`

Flag as **HIGH**:
- Missing input validation
- Verbose error messages to users
- Dependencies with known vulnerabilities
- Missing CSRF protection
- Hardcoded credentials (even if "test" credentials)
