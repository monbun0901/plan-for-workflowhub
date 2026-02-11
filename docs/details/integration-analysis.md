# Backend-Frontend-Database Integration Analysis

**Version:** v1.0.0  
**Date:** 2026-02-11  
**Status:** ✅ RESOLVED

---

## 🎯 Analysis Summary

Full-stack integration audit đã phát hiện và fix **3 major conflicts**.

---

## ✅ RESOLVED ISSUES

### 🔴 P0: Task Status Table Conflict → **FIXED**

**Problem:**
- Database `tasks.md` referenced `workflow_statuses`
- Frontend created new `task_status` table spec
- Backend used hardcoded ENUM

**Solution Applied: Option A (Unified Workflow Statuses)**
- ✅ Deleted `database/tables/task_status.md`
- ✅ Updated `tasks.md` to confirm `workflow_statuses` usage
- ✅ Updated `backend/06-tasks-module.md` to use `status_id` FK
- ✅ Updated frontend Task List to reference `workflow_statuses` filtered by `target_type='task'`

**Benefits:**
- Single source of truth for all statuses (projects, issues, tasks)
- Per-organization customization
- Reduces table proliferation

---

### 🔴 P0: API Endpoint Inconsistency → **FIXED**

**Problem:**
Frontend `api-endpoints.md` missing `orgId` prefix:
```typescript
// ❌ Before
TASKS.LIST: (projectId) => `/projects/${projectId}/tasks`

// ✅ After  
TASKS.LIST: (orgId, projectId) => `/${orgId}/projects/${projectId}/tasks`
```

**Solution Applied:**
- ✅ Added `orgId` parameter to ALL endpoints
- ✅ Added missing endpoints: USERS, LOOKUPS, INVITE_MEMBER
- ✅ Added `WORKFLOW_STATUSES` lookup endpoint with `target_type` filter
- ✅ Now matches Backend `18-api-routes-map.md`

---

### 🟡 P1: Assignment Filter Ambiguity → **CLARIFIED**

**Problem:**
Frontend Task List had 2 filters: "Assignees" và "Assignments" - không rõ khác nhau thế nào.

**Solution Applied:**
- ✅ **"Assignees" filter** = Current assigned users (from `task_assignees` table)
- ✅ **Removed "Assignments" filter** from MVP
- ✅ Added future feature: "View assignment history" (🟡 Scale-up) using `task_assignments` audit log

**Clarification:**
```
task_assignees → Many-to-many current state (WHO is assigned NOW)
task_assignments → Audit log (WHO was assigned WHEN, by WHOM)
```

---

## 📋 FILES CHANGED

| File | Change Type | Description |
|------|-------------|-------------|
| `database/tables/task_status.md` | ❌ **DELETED** | Replaced by workflow_statuses |
| `frontend/constants/02-api-endpoints.md` | ✏️ **UPDATED** | Added orgId to all endpoints |
| `frontend/pages/tasks/01-task-list.md` | ✏️ **UPDATED** | Clarified filters, use workflow_statuses |
| `backend/06-tasks-module.md` | ✏️ **UPDATED** | Use status_id FK instead of enum |

---

## 🎯 CURRENT STATE: Fully Aligned

### Backend ↔ Database
- ✅ All modules reference correct tables
- ✅ Multi-tenant `orgId` enforced at repository layer
- ✅ Unified workflow_statuses for all entities

### Frontend ↔ Backend
- ✅ API endpoints match backend routes (with orgId)
- ✅ Frontend constants include all CRUD + lookup endpoints
- ✅ Filter semantics match database tables

### Database ↔ Frontend
- ✅ Frontend page specs reference correct table structures
- ✅ Assignees vs Assignment history clearly distinguished

---

## 🚀 Next Steps (Optional Enhancements)

1. **Add Backend API Route Tests** - Verify orgId validation works
2. **Create TenantStore** - Frontend needs to manage current orgId context
3. **Implement Backend Tasks Module** - Currently placeholder, expand to full CRUD
4. **Add Workflow Status Seeder** - Default statuses for new organizations

---

*Last Updated: 2026-02-11*
