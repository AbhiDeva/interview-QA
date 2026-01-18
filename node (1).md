# Node.js Login SDK (Multi-Frontend Application Support)

This document describes how to build a **reusable Login SDK in Node.js** that can be consumed by **multiple frontend applications** (Web, Admin, Mobile, Partner Portal).

The SDK handles:
- Authentication
- Token management
- Role & permission validation
- Multi-app support
- Extensibility for OAuth providers

---

## 🎯 Enterprise Use Cases

- One auth system → many frontend apps
- Shared login & token logic
- Consistent RBAC across apps
- Centralized security enforcement

---

## 🧱 High-Level Architecture

```
Frontend Apps
(Web / Admin / Mobile)
        ↓
Login SDK (Node.js)
        ↓
Auth Service / DB / OAuth
```

---

## 📦 SDK Design Principles

- Framework-agnostic (Express, Fastify, Nest)
- Config-driven
- No UI coupling
- Pluggable auth providers
- Versioned & testable

---

## 📁 Folder Structure

```
login-sdk/
├── src/
│   ├── index.ts
│   ├── config/
│   │   └── sdk.config.ts
│   ├── auth/
│   │   ├── auth.service.ts
│   │   ├── token.service.ts
│   │   ├── password.service.ts
│   │   └── oauth.service.ts
│   ├── middleware/
│   │   ├── authenticate.ts
│   │   └── authorize.ts
│   ├── types/
│   │   └── user.types.ts
│   └── utils/
│       └── crypto.util.ts
├── package.json
└── README.md
```

---

## ⚙️ SDK Configuration

```ts
export interface LoginSdkConfig {
  jwtSecret: string;
  tokenExpiry: string;
  refreshTokenExpiry: string;
  issuer: string;
}
```

```ts
export const createLoginSDK = (config: LoginSdkConfig) => ({
  authService: new AuthService(config),
});
```

---

## 👤 User Model (Shared Contract)

```ts
export interface User {
  id: string;
  email: string;
  passwordHash: string;
  roles: string[];
  isActive: boolean;
}
```

---

## 🔐 Auth Service (Core SDK Logic)

```ts
export class AuthService {

  constructor(private config: LoginSdkConfig) {}

  async login(user: User, password: string) {
    const valid = await comparePassword(password, user.passwordHash);
    if (!valid) throw new Error('Invalid credentials');

    return this.generateTokens(user);
  }

  private generateTokens(user: User) {
    const payload = { sub: user.id, roles: user.roles };

    const accessToken = jwt.sign(payload, this.config.jwtSecret, {
      expiresIn: this.config.tokenExpiry,
      issuer: this.config.issuer
    });

    const refreshToken = jwt.sign(payload, this.config.jwtSecret, {
      expiresIn: this.config.refreshTokenExpiry
    });

    return { accessToken, refreshToken };
  }
}
```

---

## 🔑 Token Verification Service

```ts
export class TokenService {
  verify(token: string, secret: string) {
    return jwt.verify(token, secret);
  }
}
```

---

## 🧩 Authentication Middleware (SDK Export)

```ts
export const authenticate = (sdk: ReturnType<typeof createLoginSDK>) =>
  (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.sendStatus(401);

    try {
      req.user = sdk.authService.verify(token);
      next();
    } catch {
      res.sendStatus(403);
    }
  };
```

---

## 🛡 Authorization Middleware (RBAC)

```ts
export const authorize = (roles: string[]) =>
  (req, res, next) => {
    if (!roles.some(r => req.user.roles.includes(r))) {
      return res.sendStatus(403);
    }
    next();
  };
```

---

## 🌍 Multi-Frontend Support Strategy

Each frontend app gets:
- Separate client ID
- Scoped roles
- Same SDK

```ts
roles: ['ADMIN_APP', 'USER_APP', 'MOBILE_APP']
```

---

## 🔌 OAuth Extension (Optional)

```ts
loginWithGoogle(token: string) {}
loginWithFacebook(token: string) {}
```

---

## 🧪 Unit Testing Strategy

- Mock token service
- Validate RBAC rules
- Token expiry tests

---

## 📦 SDK Consumption (Express Example)

```ts
const sdk = createLoginSDK(config);

app.post('/login', async (req, res) => {
  const tokens = await sdk.authService.login(user, req.body.password);
  res.json(tokens);
});

app.get('/admin', authenticate(sdk), authorize(['ADMIN_APP']), handler);
```

---

## 🏢 Enterprise Enhancements

- Refresh token rotation
- Device fingerprinting
- Rate limiting
- Audit logging
- Session revocation

---

## 🔚 Summary

✔ Single SDK for many frontends  
✔ Centralized auth logic  
✔ RBAC enforced consistently  
✔ Easy to extend (OAuth, MFA)  
✔ Framework agnostic

---

## 🌐 Angular Frontend Integration

This section shows how to integrate the **Node.js Login SDK** with an **Angular frontend** (Enterprise-ready, scalable).

---

## 🧱 Angular Auth Architecture

```
Angular App
│
├── AuthService (API calls)
├── TokenService (Storage)
├── AuthInterceptor (JWT attach)
├── AuthGuard (Route protection)
└── Login / Dashboard / Admin Modules
```

---

## 📦 Step 1: Auth Models

```ts
export interface LoginRequest {
  email: string;
  password: string;
}

export interface LoginResponse {
  accessToken: string;
  refreshToken: string;
}

export interface UserClaims {
  sub: string;
  roles: string[];
  exp: number;
}
```

---

## 🔐 Step 2: Angular AuthService

```ts
@Injectable({ providedIn: 'root' })
export class AuthService {

  private API = '/api/auth';

  constructor(private http: HttpClient) {}

  login(payload: LoginRequest) {
    return this.http.post<LoginResponse>(`${this.API}/login`, payload);
  }

  refresh(token: string) {
    return this.http.post<LoginResponse>(`${this.API}/refresh`, { token });
  }
}
```

---

## 💾 Step 3: Token Storage Service

```ts
@Injectable({ providedIn: 'root' })
export class TokenService {

  private ACCESS = 'access_token';
  private REFRESH = 'refresh_token';

  save(tokens: LoginResponse) {
    localStorage.setItem(this.ACCESS, tokens.accessToken);
    localStorage.setItem(this.REFRESH, tokens.refreshToken);
  }

  getAccess() {
    return localStorage.getItem(this.ACCESS);
  }

  clear() {
    localStorage.clear();
  }
}
```

---

## 🔄 Step 4: HTTP Interceptor (JWT Attach)

```ts
@Injectable()
export class AuthInterceptor implements HttpInterceptor {

  constructor(private tokenService: TokenService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler) {
    const token = this.tokenService.getAccess();

    if (!token) return next.handle(req);

    const authReq = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });

    return next.handle(authReq);
  }
}
```

---

## 🛡 Step 5: Route Guard (RBAC)

```ts
@Injectable({ providedIn: 'root' })
export class RoleGuard implements CanActivate {

  canActivate(route: ActivatedRouteSnapshot): boolean {
    const roles = route.data['roles'];
    const token = localStorage.getItem('access_token');

    if (!token) return false;

    const payload = JSON.parse(atob(token.split('.')[1]));
    return roles.some((r: string) => payload.roles.includes(r));
  }
}
```

---

## 🧭 Step 6: Secure Routing

```ts
{
  path: 'admin',
  canActivate: [RoleGuard],
  data: { roles: ['ADMIN_APP'] },
  loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
}
```

---

## 🔑 Step 7: Login Component

```ts
onLogin() {
  this.auth.login(this.form.value).subscribe(res => {
    this.token.save(res);
    this.router.navigate(['/dashboard']);
  });
}
```

---

## 🌍 Multi-Frontend Strategy (Angular)

| App | Role |
|----|-----|
| Web App | USER_APP |
| Admin Portal | ADMIN_APP |
| Mobile Web | MOBILE_APP |

Same SDK → Different Angular builds

---

## ⚠️ Enterprise Security Notes

- Prefer HttpOnly cookies for refresh tokens
- Add token expiry auto-refresh
- Use backend logout (revoke refresh token)
- Enable CSP & XSS protection

---

## 🧪 Testing Strategy

- Mock AuthService
- Guard unit tests
- Interceptor integration tests

---

## 🔚 End-to-End Flow

```
Angular Login → SDK Login API → JWT Issued → Interceptor Attaches → Guard Validates
```

---

## ➕ Want Next?


- Refresh token rotation implementation
- OAuth (Google, Facebook) full flow
- SDK published as private NPM package
- Swagger-based auth API
- Diagram & threat model

Tell me 👍

