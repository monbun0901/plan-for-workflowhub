# Workflows Module

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`

---

## 🎯 Overview

Workflows module implements workflow automation engine với state machine pattern.

**Status:** 📝 To Be Implemented  
**Pattern:** State Machine + Event-Driven Architecture

---

## 📋 Implementation Guide

### Special Requirements

**Workflow module phức tạp hơn CRUD vì:**
- State machine management
- Event triggers
- Condition evaluation
- Action execution
- Idempotency

### Workflow Engine Service

```typescript
// apps/api/src/modules/workflows/services/workflow-engine.service.ts
export class WorkflowEngineService {
  /**
   * Execute workflow step
   */
  async executeStep(
    organizationId: string,
    workflowInstanceId: string,
    stepId: string
  ) {
    // 1. Get workflow instance
    const instance = await this.workflowRepo.findInstance(
      workflowInstanceId,
      organizationId
    );
    
    // 2. Get current step
    const step = instance.steps.find(s => s.id === stepId);
    
    // 3. Evaluate conditions
    if (!this.evaluateConditions(step.conditions, instance.context)) {
      throw new AppError(400, 'Step conditions not met');
    }
    
    // 4. Execute actions (idempotent)
    const result = await this.executeActions(step.actions, instance.context);
    
    // 5. Update instance state
    await this.workflowRepo.updateInstanceState(
      workflowInstanceId,
      organizationId,
      {
        current_step: this.getNextStep(step, result),
        context: { ...instance.context, ...result },
      }
    );
    
    return result;
  }
  
  private evaluateConditions(conditions: any[], context: any): boolean {
    return conditions.every(condition => {
      // Example: { field: 'status', operator: 'equals', value: 'approved' }
      const value = context[condition.field];
      
      switch (condition.operator) {
        case 'equals':
          return value === condition.value;
        case 'not_equals':
          return value !== condition.value;
        case 'contains':
          return value.includes(condition.value);
        default:
          return false;
      }
    });
  }
  
  private async executeActions(actions: any[], context: any) {
    const results = {};
    
    for (const action of actions) {
      switch (action.type) {
        case 'create_issue':
          results.issue = await this.issueService.createIssue(
            context.organization_id,
            action.params
          );
          break;
          
        case 'send_notification':
          await this.notificationService.send(action.params);
          break;
          
        case 'ai_analysis':
          results.analysis = await this.aiService.analyze(
            context.organization_id,
            action.params
          );
          break;
      }
    }
    
    return results;
  }
}
```

### Workflow Template Schema

```typescript
interface WorkflowTemplate {
  id: string;
  name: string;
  organization_id: string;
  trigger: {
    type: 'manual' | 'scheduled' | 'event';
    event?: 'issue_created' | 'task_completed';
    schedule?: string; // cron expression
  };
  steps: WorkflowStep[];
}

interface WorkflowStep {
  id: string;
  name: string;
  conditions: Condition[];
  actions: Action[];
  next_step: string | null;
}
```

---

## 🔒 Security Considerations

- ✅ **Tenant isolation** - Workflows scoped to organization
- ✅ **Permission checks** - Verify user can trigger workflow
- ✅ **Idempotency** - Safe to retry steps
- ✅ **Rate limiting** - Prevent workflow abuse

---

## 📚 Related Documents

- [04-projects-module.md](04-projects-module.md) - Base pattern
- [09-chat-module.md](09-chat-module.md) - AI integration
- [../../basics/step-3-architecture.md](../../basics/step-3-architecture.md) - Workflow flows

---

*Status: Placeholder - Complex module requiring state machine implementation*
