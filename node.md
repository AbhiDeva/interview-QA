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

## ➕ Want Next?

- Refresh token rotation implementation
- OAuth (Google, Facebook) full flow
- SDK published as private NPM package
- Swagger-based auth API
- Diagram & threat model

Tell me 👍

