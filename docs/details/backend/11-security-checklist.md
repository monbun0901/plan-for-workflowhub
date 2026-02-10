# Backend Security Checklist

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-security-coder`

---

## 🎯 Overview

Comprehensive security checklist cho WorkflowHub backend. **All items MUST be verified** before production deployment.

---

## 🔒 Authentication & Authorization

### Password Security
- [ ] **bcrypt hashing** - Minimum 10 rounds
- [ ] **Password policy enforced** - Min 8 chars, uppercase, lowercase, numbers
- [ ] **No password in logs** - Never log passwords (even hashed)
- [ ] **Password reset** - Secure token-based reset flow
- [ ] **Password history** - Prevent reuse of last N passwords (optional v1)

### JWT (JSON Web Tokens)
- [ ] **Short-lived access tokens** - 15 minutes maximum
- [ ] **Refresh token rotation** - New token on each refresh
- [ ] **Strong secrets** - 256-bit random keys minimum
- [ ] **HTTPS only** - Tokens transmitted over HTTPS only
- [ ] **HttpOnly cookies** - Refresh tokens in HttpOnly cookies
- [ ] **Token revocation** - Implement if needed (blacklist)

### Session Management
- [ ] **Session timeout** - Auto logout after inactivity
- [ ] **Concurrent session limits** - Optional: Limit sessions per user
- [ ] **Logout endpoint** - Clear tokens client-side, invalidate server-side

---

## 🛡️ Attack Prevention

### Rate Limiting
- [ ] **Auth endpoints** - 5 attempts per 15 min
- [ ] **API endpoints** - 100 requests per 15 min per user
- [ ] **Redis-backed** - Distributed rate limiting
- [ ] **Custom error messages** - Don't reveal system details

### Brute Force Protection
- [ ] **Login rate limiting** - Max 5 failed attempts
- [ ] **Account lockout** - Optional: Lock after N failed attempts
- [ ] **CAPTCHA** - After 3 failed attempts (optional v1)
- [ ] **Monitoring** - Alert on suspicious patterns

### SQL Injection
- [ ] **Sequelize parameterized queries** - NEVER use raw SQL with user input
- [ ] **Input validation** - Zod schemas on all inputs
- [ ] **ORM best practices** - Use Sequelize safely
- [ ] **Database least privilege** - App user has minimal permissions

### XSS (Cross-Site Scripting)
- [ ] **Output encoding** - Sanitize user-generated content
- [ ] **Content-Security-Policy** - CSP headers enabled
- [ ] **X-XSS-Protection header** - Browser XSS filter enabled
- [ ] **No eval()** - Never use eval with user input

### CSRF (Cross-Site Request Forgery)
- [ ] **CSRF tokens** - On state-changing operations (optional if JWT)
- [ ] **SameSite cookies** - Set SameSite=Strict or Lax
- [ ] **Origin validation** - Check request origin

---

## 🏢 Multi-Tenant Security

### Tenant Isolation
- [ ] **organization_id in all tables** - Every table has tenant ID
- [ ] **organization_id in all queries** - ALWAYS filter by tenant
- [ ] **Middleware validation** - Verify user access to organization
- [ ] **No direct ID access** - Always use /organizations/:orgId/ prefix
- [ ] **Integration tests** - Test cross-tenant isolation

### Data Leakage Prevention
- [ ] **Repository pattern** - All data access through repositories
- [ ] **Tenant filter required** - organizationId parameter mandatory
- [ ] **Join query filtering** - Filter both sides of joins
- [ ] **Search filtering** - Full-text search scoped to tenant

---

## 🌐 HTTP Security

### Headers (Helmet.js)
- [ ] **X-Content-Type-Options** - nosniff
- [ ] **X-Frame-Options** - DENY or SAMEORIGIN
- [ ] **X-XSS-Protection** - 1; mode=block
- [ ] **Strict-Transport-Security** - HSTS enabled (production)
- [ ] **Content-Security-Policy** - Restrict resource loading
- [ ] **Referrer-Policy** - no-referrer or strict-origin

### CORS (Cross-Origin Resource Sharing)
- [ ] **Whitelist origins** - Only allow frontend domain
- [ ] **Credentials allowed** - credentials: true if needed
- [ ] **Methods restricted** - Only necessary HTTP methods
- [ ] **No wildcard (*)** - NEVER use * in production

### HTTPS
- [ ] **HTTPS enforced** - Redirect HTTP to HTTPS
- [ ] **TLS 1.2+** - Minimum TLS version
- [ ] **Strong ciphers** - Modern cipher suites only
- [ ] **Certificate valid** - SSL cert not expired

---

## 📋 Input Validation

### Zod Schemas
- [ ] **All endpoints validated** - Zod schema on every POST/PUT
- [ ] **Type safety** - TypeScript types from Zod
- [ ] **Custom validators** - Business logic validation
- [ ] **Error messages sanitized** - Don't reveal system details

### File Uploads (if applicable)
- [ ] **File type validation** - Whitelist allowed extensions
- [ ] **File size limits** - Max upload size enforced
- [ ] **Virus scanning** - Scan uploaded files
- [ ] **Storage outside webroot** - Files not directly accessible

---

## 🗄️ Database Security

### SQL Injection Prevention
- [ ] **Sequelize parameterized** - NEVER raw queries with user input
- [ ] **Prepared statements** - Use placeholders
- [ ] **Input validation** - Validate before DB queries

### Database Configuration
- [ ] **Least privilege** - App user minimal permissions
- [ ] **Strong passwords** - Complex DB passwords
- [ ] **Network isolation** - DB not publicly accessible
- [ ] **Backup encryption** - Encrypted backups

### Data Encryption
- [ ] **Sensitive data encrypted** - PII, financial data
- [ ] **TLS for DB connection** - Encrypted connection to DB
- [ ] **Encryption at rest** - Optional: Disk encryption

---

## 📊 Logging & Monitoring

### Logging Best Practices
- [ ] **Never log passwords** - Mask sensitive data
- [ ] **Structured logging** - JSON logs
- [ ] **Log levels** - ERROR, WARN, INFO, DEBUG
- [ ] **Log rotation** - Prevent disk fill-up

### Security Monitoring
- [ ] **Failed login tracking** - Monitor brute force attempts
- [ ] **Suspicious activity alerts** - Alert on anomalies
- [ ] **Access logs** - Who accessed what, when
- [ ] **Audit trail** - Critical actions logged

---

## ⚙️ Dependency Management

### npm Packages
- [ ] **npm audit** - Run regularly
- [ ] **Update dependencies** - Keep packages current
- [ ] **Lock file committed** - package-lock.json in git
- [ ] **Minimal dependencies** - Only necessary packages

### Supply Chain Security
- [ ] **Verify package integrity** - Check checksums
- [ ] **Audit before install** - Review new packages
- [ ] **Renovate/Dependabot** - Auto dependency updates

---

## 🔧 Environment & Deployment

### Environment Variables
- [ ] **Secrets in env vars** - Never hardcode secrets
- [ ] **.env not in git** - .env in .gitignore
- [ ] **Strong secrets** - Random, 256-bit keys
- [ ] **Separate configs** - Dev/staging/prod configs

### Production Deployment
- [ ] **Error messages sanitized** - Generic errors to client
- [ ] **Debug mode OFF** - No debug logs in production
- [ ] **Source maps OFF** - Don't expose source code
- [ ] **Rate limiting enabled** - Production-strength limits

---

## 🧪 Security Testing

### Pre-Deployment
- [ ] **Unit tests** - Security-critical functions tested
- [ ] **Integration tests** - Tenant isolation tested
- [ ] **Penetration testing** - Optional: External audit
- [ ] **Code review** - Security review before merge

### Continuous Monitoring
- [ ] **npm audit in CI** - Fail build on vulnerabilities  
- [ ] **SAST tools** - Static analysis (e.g., SonarQube)
- [ ] **Dependency scanning** - Snyk, Dependabot, etc.

---

## 📝 Checklist Summary

### Critical (MUST FIX before production)
- ✅ bcrypt password hashing (10+ rounds)
- ✅ JWT short-lived access tokens (15 min)
- ✅ Rate limiting on auth endpoints
- ✅ organization_id filtering in ALL queries
- ✅ HTTPS enforced
- ✅ Helmet.js security headers
- ✅ CORS whitelist (no wildcards)
- ✅ Input validation (Zod schemas)

### High Priority
- ⚠️ Token revocation mechanism
- ⚠️ Account lockout after failed attempts
- ⚠️ CSP headers configured
- ⚠️ Audit logging for critical actions

### Medium Priority (Post-MVP)
- 📌 Password history tracking
- 📌 CAPTCHA on repeated failures
- 📌 File upload virus scanning
- 📌 Penetration testing

---

## 🎯 Skill Usage

```
@backend-security-coder
Security audit WorkflowHub backend:
- Review authentication implementation
- Verify SQL injection prevention
- Check tenant isolation
- Validate input validation
- Test rate limiting
- Review headers (Helmet.js)
```

---

## 📚 Related Documents

- [02-authentication.md](02-authentication.md) - Auth implementation
- [03-multi-tenant.md](03-multi-tenant.md) - Tenant isolation
- [01-architecture.md](01-architecture.md) - Error handling
- [../database/v1-database-security.md](../database/v1-database-security.md) - Database security

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
