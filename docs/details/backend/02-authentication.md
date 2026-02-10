# Authentication & Authorization

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-security-coder`, `nodejs-best-practices`

---

## 🎯 Overview

Authentication implementation sử dụng JWT (JSON Web Tokens) và bcrypt cho password hashing.

**Strategy:**
- Access Token: 15 minutes (short-lived)
- Refresh Token: 7 days (long-lived)
- Password: bcrypt with 10 rounds
- Rate Limiting: Prevent brute force attacks

---

## 🔐 Password Service

### Implementation

```typescript
// apps/api/src/modules/auth/services/password.service.ts
import bcrypt from 'bcrypt';

/**
 * Password service - Handle secure password operations
 * @follows backend-security-coder best practices
 */
export class PasswordService {
  private readonly SALT_ROUNDS = 10;

  /**
   * Hash password using bcrypt
   * @param plainPassword - Plain text password
   * @returns Hashed password
   */
  async hash(plainPassword: string): Promise<string> {
    return bcrypt.hash(plainPassword, this.SALT_ROUNDS);
  }

  /**
   * Verify password against hash
   * @param plainPassword - Plain text password
   * @param hashedPassword - Stored hash
   * @returns True if password matches
   */
  async verify(plainPassword: string, hashedPassword: string): Promise<boolean> {
    return bcrypt.compare(plainPassword, hashedPassword);
  }

  /**
   * Validate password strength
   * @param password - Password to validate
   * @throws Error if password is weak
   */
  validateStrength(password: string): void {
    if (password.length < 8) {
      throw new Error('Password must be at least 8 characters');
    }
    
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    
    if (!hasUpperCase || !hasLowerCase || !hasNumbers) {
      throw new Error('Password must contain uppercase, lowercase, and numbers');
    }
  }
}
```

### Password Policy

| Rule | Requirement | Reason |
|------|-------------|--------|
| **Length** | >= 8 characters | OWASP minimum |
| **Uppercase** | At least 1 | Complexity |
| **Lowercase** | At least 1 | Complexity |
| **Numbers** | At least 1 | Complexity |
| **Special Chars** | Optional for MVP | Can add later |
| **Bcrypt Rounds** | 10 | Balance security & performance |

---

## 🎫 JWT Service

### Implementation

```typescript
// apps/api/src/modules/auth/services/jwt.service.ts
import jwt from 'jsonwebtoken';
import { v4 as uuidv4 } from 'uuid';

interface TokenPayload {
  userId: string;
  email: string;
  tokenId: string;
}

/**
 * JWT service - Handle token generation and verification
 * @follows backend-security-coder best practices
 */
export class JWTService {
  private readonly accessTokenSecret = process.env.JWT_ACCESS_SECRET!;
  private readonly refreshTokenSecret = process.env.JWT_REFRESH_SECRET!;
  private readonly accessTokenExpiry = '15m';
  private readonly refreshTokenExpiry = '7d';

  /**
   * Generate access and refresh tokens
   * @param userId - User ID
   * @param email - User email
   * @returns Access and refresh tokens with expiry
   */
  generateTokens(userId: string, email: string) {
    const tokenId = uuidv4(); // For token revocation
    
    const accessToken = jwt.sign(
      { userId, email, tokenId } as TokenPayload,
      this.accessTokenSecret,
      { expiresIn: this.accessTokenExpiry }
    );
    
    const refreshToken = jwt.sign(
      { userId, email, tokenId } as TokenPayload,
      this.refreshTokenSecret,
      { expiresIn: this.refreshTokenExpiry }
    );
    
    return {
      accessToken,
      refreshToken,
      expiresIn: 15 * 60, // 15 minutes in seconds
    };
  }

  /**
   * Verify access token
   * @param token - JWT token
   * @returns Decoded payload
   * @throws Error if token is invalid
   */
  verifyAccessToken(token: string): TokenPayload {
    return jwt.verify(token, this.accessTokenSecret) as TokenPayload;
  }

  /**
   * Verify refresh token
   * @param token - JWT refresh token
   * @returns Decoded payload
   * @throws Error if token is invalid
   */
  verifyRefreshToken(token: string): TokenPayload {
    return jwt.verify(token, this.refreshTokenSecret) as TokenPayload;
  }
}
```

### Token Strategy

| Token Type | Lifetime | Storage | Purpose |
|------------|----------|---------|---------|
| **Access Token** | 15 minutes | Memory (frontend) | API authentication |
| **Refresh Token** | 7 days | HttpOnly cookie | Token renewal |

**Why short-lived access token?**
- If stolen, expires quickly
- Forces regular token refresh
- Better security posture

---

## 🛡️ Rate Limiting

### Implementation

```typescript
// apps/api/src/shared/middleware/rate-limiter.middleware.ts
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

/**
 * Rate limiter for auth endpoints
 * @follows backend-security-coder: Prevent brute force
 */
export const authLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:auth:',
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 requests per window
  message: 'Too many login attempts, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

/**
 * General API rate limiter
 */
export const apiLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:api:',
  }),
  windowMs: 15 * 60 * 1000,
  max: 100, // 100 requests per 15 minutes
  message: 'Too many requests, please try again later',
});
```

### Rate Limit Rules

| Endpoint | Limit | Window | Reason |
|----------|-------|--------|--------|
| `/auth/login` | 5 requests | 15 min | Prevent brute force |
| `/auth/register` | 3 requests | 1 hour | Prevent spam |
| `/auth/refresh` | 10 requests | 15 min | Normal usage |
| All API endpoints | 100 requests | 15 min | Abuse prevention |

---

## 🔒 Auth Middleware

```typescript
// apps/api/src/shared/middleware/auth.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { JWTService } from '../../modules/auth/services/jwt.service';

const jwtService = new JWTService();

/**
 * Authentication middleware
 * Verifies JWT token and attaches user to request
 */
export async function authenticate(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({
        success: false,
        error: { code: 'UNAUTHORIZED', message: 'No token provided' },
      });
    }
    
    const token = authHeader.substring(7);
    const payload = jwtService.verifyAccessToken(token);
    
    // Attach user to request
    req.user = {
      id: payload.userId,
      email: payload.email,
    };
    
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      error: { code: 'INVALID_TOKEN', message: 'Invalid or expired token' },
    });
  }
}
```

**Usage:**
```typescript
// Protected route
router.get('/api/projects', authenticate, projectController.list);
```

---

## 📋 Security Checklist

### Password Security
- [x] **bcrypt** - 10 rounds minimum
- [x] **Password Policy** - 8+ chars, uppercase, lowercase, numbers
- [ ] **Password History** - Prevent reuse (optional for v1)
- [ ] **Password Expiry** - Force change after N days (optional for v1)

### JWT Security
- [x] **Short-lived Access Token** - 15 minutes
- [x] **Refresh Token Rotation** - New token on each refresh
- [x] **Strong Secrets** - Random, 256-bit minimum
- [ ] **Token Revocation** - Blacklist (implement when needed)
- [x] **Payload Minimal** - Only userId, email, tokenId

### Attack Prevention
- [x] **Rate Limiting** - Redis-backed rate limiter
- [x] **Brute Force Protection** - 5 attempts / 15 min
- [ ] **Account Lockout** - After N failed attempts (optional for v1)
- [x] **HTTPS Only** - Cookies set to secure in production
- [x] **CORS** - Whitelist frontend origin

### HTTP Headers (Helmet.js)
- [ ] **X-Content-Type-Options** - nosniff
- [ ] **X-Frame-Options** - DENY
- [ ] **X-XSS-Protection** - 1; mode=block
- [ ] **Strict-Transport-Security** - HSTS enabled
- [ ] **Content-Security-Policy** - CSP headers

---

## 🧪 Testing

### Unit Tests

```typescript
// apps/api/src/modules/auth/services/__tests__/password.service.test.ts
import { PasswordService } from '../password.service';

describe('PasswordService', () => {
  const passwordService = new PasswordService();
  
  it('should hash password', async () => {
    const password = 'Test1234';
    const hash = await passwordService.hash(password);
    
    expect(hash).not.toBe(password);
    expect(hash).toMatch(/^\$2[aby]\$/); // bcrypt format
  });
  
  it('should verify correct password', async () => {
    const password = 'Test1234';
    const hash = await passwordService.hash(password);
    
    const isValid = await passwordService.verify(password, hash);
    expect(isValid).toBe(true);
  });
  
  it('should reject weak passwords', () => {
    expect(() => {
      passwordService.validateStrength('weak');
    }).toThrow('Password must be at least 8 characters');
  });
});
```

---

## 🔧 Environment Variables

```bash
# .env
JWT_ACCESS_SECRET=your-super-secret-access-key-256-bits-minimum
JWT_REFRESH_SECRET=your-super-secret-refresh-key-256-bits-minimum
REDIS_URL=redis://localhost:6379
```

**Generate secrets:**
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

---

## 📚 Related Documents

- [01-architecture.md](01-architecture.md) - Backend architecture
- [03-multi-tenant.md](03-multi-tenant.md) - Tenant isolation
- [11-security-checklist.md](11-security-checklist.md) - Security best practices
- [../../basics/step-5-data-design.md](../../basics/step-5-data-design.md) - User schema

---

## 🎯 Skill Usage

```
@backend-security-coder
Implement authentication module với:
- bcrypt password hashing (10 rounds)
- JWT (access 15min, refresh 7 days)
- Rate limiting (Redis-backed)
- Security headers (Helmet.js)
```

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
