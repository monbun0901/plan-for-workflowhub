# WorkflowHub - Database Security & Optimization

**Date:** 2026-02-11  
**Version:** v1  
**Skills:** `database-architect`, `backend-security-coder`

---

## 🎯 Overview

Database security và optimization guide cho WorkflowHub MySQL database.

---

## Phase 6: Database Security & Optimization

**Duration:** Throughout project  
**Skills:** `database-architect`, `backend-security-coder`

### 6.1 Database Security

#### With `database-architect` + `backend-security-coder`:
```
@database-architect @backend-security-coder
Implement database security:
- SQL injection prevention
- Least privilege principle
- Encryption at rest
- Backup strategy
```

**Security Measures:**

#### 1. Sequelize Auto-Escaping
```typescript
// ✅ SAFE: Sequelize escapes automatically
const projects = await Project.findAll({
  where: { 
    organization_id: organizationId, // Escaped
    status: req.query.status         // Escaped
  }
});

// ❌ UNSAFE: Raw queries without binding
const projects = await sequelize.query(
  `SELECT * FROM projects WHERE status = '${req.query.status}'` // SQL INJECTION!
);

// ✅ SAFE: Raw query with bindings
const projects = await sequelize.query(
  'SELECT * FROM projects WHERE status = :status',
  {
    replacements: { status: req.query.status },
    type: QueryTypes.SELECT
  }
);
```

#### 2. Database User Permissions
```sql
-- Create app user with limited privileges
CREATE USER 'workflowhub_app'@'%' IDENTIFIED BY 'strong_password';

-- Grant only necessary privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON workflowhub_prod.* TO 'workflowhub_app'@'%';

-- NO DROP, CREATE, ALTER permissions
FLUSH PRIVILEGES;
```

#### 3. Encryption at Rest
```sql
CREATE TABLE users (
  id VARCHAR(36) PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  api_key_encrypted VARBINARY(256), -- Encrypted sensitive data
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB ENCRYPTION='Y'; -- MySQL 8.0+ encryption
```

### 6.2 Query Optimization

#### Index Strategy
```sql
-- Primary indexes (organization_id on ALL tables)
CREATE INDEX idx_projects_org_id ON projects(organization_id);
CREATE INDEX idx_issues_org_id ON issues(organization_id);
CREATE INDEX idx_tasks_org_id ON tasks(organization_id);

-- Composite indexes for common queries
CREATE INDEX idx_projects_org_status ON projects(organization_id, status);
CREATE INDEX idx_issues_org_project ON issues(organization_id, project_id);

-- Full-text search
CREATE FULLTEXT INDEX idx_documents_content ON documents(content);
```

#### Prevent N+1 Queries
```typescript
// ❌ BAD: N+1 query problem
const projects = await Project.findAll();
for (const project of projects) {
  project.issues = await Issue.findAll({ where: { project_id: project.id } });
}

// ✅ GOOD: Use include (JOIN)
const projects = await Project.findAll({
  include: [
    {
      model: Issue,
      as: 'issues',
    },
  ],
});
```

#### Connection Pooling
```javascript
// apps/api/config/database.js
module.exports = {
  production: {
    use_env_variable: 'DATABASE_URL',
    dialect: 'mysql',
    pool: {
      max: 20,        // Maximum connections
      min: 5,         // Minimum connections
      acquire: 60000, // Max time (ms) to acquire connection
      idle: 10000,    // Max time (ms) connection can be idle
    },
    logging: false,
  },
};
```

### 6.3 Backup Strategy

#### Automated Backups
```bash
#!/bin/bash
# backup-database.sh

# Configuration
DB_NAME="workflowhub_prod"
DB_USER="workflowhub_backup"
DB_PASSWORD="backup_password"
BACKUP_DIR="/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
FILENAME="workflowhub_${DATE}.sql.gz"

# Create backup directory if not exists
mkdir -p $BACKUP_DIR

# Dump database and compress
mysqldump \
  --user=$DB_USER \
  --password=$DB_PASSWORD \
  --single-transaction \
  --routines \
  --triggers \
  $DB_NAME | gzip > $BACKUP_DIR/$FILENAME

# Keep only last 30 days of backups
find $BACKUP_DIR -name "workflowhub_*.sql.gz" -mtime +30 -delete

echo "Backup completed: $FILENAME"
```

#### Cron Job
```cron
# Run daily at 2 AM
0 2 * * * /scripts/backup-database.sh >> /var/log/backup.log 2>&1
```

#### Restore from Backup
```bash
# Decompress and restore
gunzip < /backups/mysql/workflowhub_20260211_020000.sql.gz | \
  mysql -u root -p workflowhub_prod
```

### 6.4 Monitoring & Performance

#### Slow Query Log
```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2; -- Log queries > 2 seconds
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow-query.log';
```

#### Query Execution Plan
```sql
-- Analyze query performance
EXPLAIN SELECT * FROM projects 
WHERE organization_id = 'org-123' 
AND status = 'active';

-- Check index usage
SHOW INDEX FROM projects;
```

#### Database Size Monitoring
```sql
-- Check table sizes
SELECT 
  table_name AS 'Table',
  ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)'
FROM information_schema.TABLES
WHERE table_schema = 'workflowhub_prod'
ORDER BY (data_length + index_length) DESC;
```

---

## Security Best Practices Checklist

- [ ] **SQL Injection:** All queries parameterized or using ORM
- [ ] **Least Privilege:** App user has minimal permissions
- [ ] **Encryption:** Sensitive data encrypted
- [ ] **Backups:** Daily automated backups
- [ ] **Connection Pooling:** Configured appropriately
- [ ] **Indexes:** organization_id indexed on all tables
- [ ] **Monitoring:** Slow query log enabled
- [ ] **Password Rotation:** Database passwords rotated quarterly

---

## Performance Best Practices Checklist

- [ ] **Indexes:** Composite indexes for common queries
- [ ] **N+1 Prevention:** Use eager loading (include)
- [ ] **Pagination:** All list endpoints paginated
- [ ] **Connection Limits:** Pool size appropriate for load
- [ ] **Query Optimization:** EXPLAIN used for slow queries
- [ ] **Data Archival:** Old data archived regularly
- [ ] **Cache Strategy:** Redis for hot data

---

## Skill Usage Examples

### Example 1: Optimize Slow Query
```
@database-architect
Optimize this slow query:
SELECT * FROM documents WHERE content LIKE '%keyword%'

Suggestions:
- Full-text search index
- Elasticsearch integration
- Query rewrite
```

### Example 2: Security Audit
```
@backend-security-coder @database-architect
Audit database security:
- User permissions review
- Encryption verification
- SQL injection vulnerabilities
- Backup integrity check
```

### Example 3: Scale Database
```
@database-architect
Plan database scaling:
- Read replicas for read-heavy workload
- Sharding strategy
- Multi-region deployment
```

---

## Related Documents

- [../common/v1-setup-and-workflows.md](../common/v1-setup-and-workflows.md) - Initial database setup
- [../backend/03-multi-tenant.md](../backend/03-multi-tenant.md) - Multi-tenant isolation
- [../backend/04-projects-module.md](../backend/04-projects-module.md) - Repository pattern examples
- [../../basics/step-5-data-design.md](../../basics/step-5-data-design.md) - Full database schema


---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
