# task_assignments Table

**Type:** Assignment History (Audit Log)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE task_assignments (
  id              VARCHAR(36) PRIMARY KEY,
  task_id         VARCHAR(36) NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  assignee_id     VARCHAR(36) NOT NULL REFERENCES users(id),
  assigned_by     VARCHAR(36) REFERENCES users(id),  -- NULL if AI assigned
  assigned_by_agent VARCHAR(36) REFERENCES agents(id),
  
  reason          TEXT,                          -- Why reassigned?
  
  assigned_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  unassigned_at   TIMESTAMP,                     -- When reassigned to someone else
  
  INDEX idx_task (task_id),
  INDEX idx_assignee (assignee_id),
  INDEX idx_org (organization_id)
);
```

---

## 🎯 Purpose

**Track:**
- **Audit Trail:** Toàn bộ lịch sử các lần giao việc (kể cả song song).
- **Assigner:** Ai là người đã giao task cho user nào.
- **Duration:** Thời gian mỗi người tham gia vào task (từ `assigned_at` đến `unassigned_at`).
- **Parallel Work:** Hỗ trợ mô hình nhiều người làm cùng lúc (nhiều bản ghi có `unassigned_at IS NULL`).

---

## 🔗 Associations (Sequelize)

```typescript
// models/task-assignment.model.ts
TaskAssignment.belongsTo(Task, {
  foreignKey: 'task_id',
  as: 'task'
});

TaskAssignment.belongsTo(User, {
  foreignKey: 'assignee_id',
  as: 'assignee'
});

TaskAssignment.belongsTo(User, {
  foreignKey: 'assigned_by',
  as: 'assigner'
});

TaskAssignment.belongsTo(Agent, {
  foreignKey: 'assigned_by_agent',
  as: 'agent'
});
```

**Explanation:**
- `belongsTo(Task)` - Assignment thuộc task nào
- `belongsTo(User, assignee)` - Người được assign
- `belongsTo(User, assigned_by)` - Người assign (có thể NULL nếu AI)
- `belongsTo(Agent)` - AI agent assign (nếu auto)

---

## 📝 Fields Explanation

| Field | Description | Why? |
|-------|-------------|------|
| `assignee_id` | Current assignee | Who is doing it |
| `assigned_by` | Who assigned it | Manager/PM who delegated |
| `assigned_by_agent` | AI that assigned it | Workflow auto-assignment |
| `reason` | Why reassigned | "John on leave, reassign to Jane" |
| `unassigned_at` | When stopped | Track assignment duration |

---

## 🎯 Common Queries

### Get current assignee

```typescript
const currentAssignment = await TaskAssignment.findOne({
  where: {
    task_id: taskId,
    organization_id: orgId,
    unassigned_at: null  // Still active
  },
  include: [{ model: User, as: 'assignee' }]
});
```

### Get assignment history

```typescript
const history = await TaskAssignment.findAll({
  where: {
    task_id: taskId,
    organization_id: orgId
  },
  include: [
    { model: User, as: 'assignee' },
    { model: User, as: 'assigner' }
  ],
  order: [['assigned_at', 'DESC']]
});
```

### Reassign task

```typescript
// 1. Close previous assignment
await TaskAssignment.update(
  { unassigned_at: new Date() },
  {
    where: {
      task_id: taskId,
      organization_id: orgId,
      unassigned_at: null
    }
  }
);

// 2. Create new assignment
await TaskAssignment.create({
  id: uuidv4(),
  task_id: taskId,
  organization_id: orgId,
  assignee_id: newUserId,
  assigned_by: currentUserId,
  reason: "Original assignee on leave",
  assigned_at: new Date()
});

// 3. Update tasks.assignee_id (denormalized for quick access)
await Task.update(
  { assignee_id: newUserId },
  { where: { id: taskId, organization_id: orgId } }
);
```

---

## 📊 Denormalization Strategy

### Why keep `tasks.assignee_id`?

**Performance:**
```typescript
// ✅ FAST: Direct query (no join)
const tasks = await Task.findAll({
  where: {
    assignee_id: userId,
    status: 'todo'
  }
});

// ❌ SLOW: Join required
const tasks = await Task.findAll({
  include: [{
    model: TaskAssignment,
    where: { assignee_id: userId, unassigned_at: null }
  }]
});
```

**Best Practice:**
- `tasks.assignee_id` - Current assignee (denormalized, fast query)
- `task_assignments` - Full history (audit trail)

---

## 🔄 Assignment Flow

```
1. Task created
   ├─ tasks.assignee_id = NULL
   └─ No assignment record yet

2. First assignment
   ├─ INSERT task_assignments (assignee: John)
   └─ UPDATE tasks SET assignee_id = John

3. Reassignment
   ├─ UPDATE task_assignments SET unassigned_at = NOW() WHERE assignee = John
   ├─ INSERT task_assignments (assignee: Jane, reason: "John on leave")
   └─ UPDATE tasks SET assignee_id = Jane

4. Unassign
   ├─ UPDATE task_assignments SET unassigned_at = NOW()
   └─ UPDATE tasks SET assignee_id = NULL
```

---

## 📈 Analytics Queries

### Who is overloaded?

```typescript
const workload = await TaskAssignment.findAll({
  where: {
    organization_id: orgId,
    unassigned_at: null  // Active assignments
  },
  attributes: [
    'assignee_id',
    [sequelize.fn('COUNT', sequelize.col('id')), 'task_count']
  ],
  include: [{ model: User, as: 'assignee' }],
  group: ['assignee_id'],
  order: [[sequelize.literal('task_count'), 'DESC']]
});
```

### Average time per assignment

```typescript
const avgTime = await sequelize.query(`
  SELECT 
    assignee_id,
    AVG(TIMESTAMPDIFF(HOUR, assigned_at, unassigned_at)) as avg_hours
  FROM task_assignments
  WHERE organization_id = :orgId 
    AND unassigned_at IS NOT NULL
  GROUP BY assignee_id
`, {
  replacements: { orgId },
  type: QueryTypes.SELECT
});
```

---

## ✅ Benefits

| Feature | Without History | With History |
|---------|----------------|--------------|
| **Current assignee** | ✅ Fast (tasks.assignee_id) | ✅ Fast (tasks.assignee_id) |
| **Past assignees** | ❌ Lost | ✅ Full history |
| **Who assigned** | ❌ Unknown | ✅ Tracked |
| **Why reassigned** | ❌ No context | ✅ Reason field |
| **Time tracking** | ❌ No data | ✅ Duration per assignee |
| **Audit trail** | ❌ No | ✅ Complete |

---

## 📚 Related Tables

- **tasks** - Parent task
- **users** - Assignee and assigner
- **agents** - AI that auto-assigned

---

*Table Version: 1.0*  
*Last Updated: 2026-02-11*
