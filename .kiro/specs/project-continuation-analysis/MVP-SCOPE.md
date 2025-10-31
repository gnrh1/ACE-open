# ACE-Open MVP Scope Document

## 🔀 BRANCH-OFF NOTICE

**This document is a SUBSET of the full production task list.**

- **Source of Truth**: [tasks.md](./tasks.md) - Complete production-grade implementation plan
- **This Document**: MVP-focused subset for initial hypothesis validation
- **Branch-Off Point**: After Task 2.6 (Configuration-Based Workflow Complete)
- **Relationship**: This MVP scope represents the minimum viable implementation needed to validate the core ACE hypothesis before proceeding with production infrastructure

---

## Document Purpose

This MVP scope document:
1. ✅ Contains ONLY tasks required for MVP validation
2. ✅ References original task IDs from `tasks.md` for traceability
3. ✅ Excludes production infrastructure tasks (deployment, optimization, streaming)
4. ✅ Focuses on validating: "Can ACE improve LLM performance through iterative context refinement?"

**When to use this document**: For MVP test run execution and validation
**When to use tasks.md**: For full production implementation planning

---

## MVP Completion Status

### ✅ COMPLETED (Ready for MVP Test Run)

All tasks below are **COMPLETE** and ready for validation:

#### Task 1: Achieve 90% Test Coverage ✅
- **Status**: COMPLETE (95.61% coverage achieved)
- **Reference**: tasks.md Tasks 1.1-1.9
- **Evidence**: 535 tests passing, all critical modules >90% coverage
- **MVP Value**: Ensures code quality and reliability for validation

#### Task 2: Implement Configuration-Based Workflow ✅
- **Status**: COMPLETE
- **Reference**: tasks.md Tasks 2.1-2.6
- **Components**:
  - ✅ 2.1: Experiment orchestration module (`ace/experiments.py`)
  - ✅ 2.2: QA example (`examples/qa_task/`)
  - ✅ 2.3: Code generation example (`examples/code_generation/`)
  - ✅ 2.4: Classification example (`examples/classification/`)
  - ✅ 2.5: Config template generator (`python -m ace template`)
  - ✅ 2.6: Integration tests (19 tests in `tests/test_experiments.py`)
- **MVP Value**: Provides 3 diverse examples to validate ACE across different task types

---

## MVP Pre-Flight Checklist

Before running MVP validation, complete these quick verifications:

### 1. Security Verification (5 minutes)
```bash
# Verify existing security tests pass
python -m pytest tests/test_security.py -v

# Expected: All tests pass
# If any fail, investigate before proceeding
```

**Traceability**: Lean version of tasks.md Task 4.1
**Why**: Ensures basic security for trustworthy results

### 2. Example Smoke Test (5 minutes)
```bash
# Verify each example environment can be imported
python -c "from examples.qa_task.qa_environment import QAEnvironment; print('✓ QA OK')"
python -c "from examples.code_generation.code_environment import CodeExecutionEnvironment; print('✓ Code OK')"
python -c "from examples.classification.classification_environment import ClassificationEnvironment; print('✓ Classification OK')"

# Expected: All print "✓ [Type] OK"
```

**Traceability**: Verification of tasks.md Tasks 2.2-2.4
**Why**: Confirms examples are ready to run

---

## MVP Test Run Execution

### Objective
Validate the core hypothesis: **"ACE improves LLM performance through iterative context refinement"**

### Test Scenarios

#### Scenario 1: Question Answering
```bash
cd /path/to/ACE-open
python examples/qa_task/run_qa.py

# What to observe:
# - Accuracy improvement from epoch 1 to epoch 3
# - Playbook growth with QA strategies
# - No crashes or errors
```

**Success Criteria**: Accuracy improves by ≥5% from first to last epoch

#### Scenario 2: Code Generation
```bash
python examples/code_generation/run_codegen.py

# What to observe:
# - Test pass rate improvement across epochs
# - Playbook growth with coding strategies
# - Successful code execution
```

**Success Criteria**: Test pass rate improves by ≥10% from first to last epoch

#### Scenario 3: Text Classification
```bash
python examples/classification/run_classification.py

# What to observe:
# - F1 score improvement across epochs
# - Playbook growth with classification strategies
# - Balanced per-class performance
```

**Success Criteria**: Macro F1 score improves by ≥5% from first to last epoch

### Overall MVP Success Threshold

**MVP is successful if**:
- ✅ At least 2 out of 3 examples show measurable improvement
- ✅ Playbook grows with useful strategies (manual inspection)
- ✅ No crashes or errors during execution
- ✅ Results are reproducible across runs

---

## Tasks EXCLUDED from MVP Scope

The following tasks from `tasks.md` are **DEFERRED** until after MVP validation:

### Task 3: Production Deployment Documentation
- **Reference**: tasks.md Tasks 3.1-3.5
- **Reason**: MVP runs locally; Docker/K8s not needed for validation
- **When to implement**: After MVP validation confirms approach works

### Task 4: Security Audit Enhancement
- **Reference**: tasks.md Tasks 4.2-4.3 (4.1 done in lean form)
- **Reason**: Existing security tests provide baseline; enhanced docs/CI not needed for MVP
- **When to implement**: Before production deployment

### Task 5: Performance Optimization
- **Reference**: tasks.md Tasks 5.1-5.4
- **Reason**: Correctness before speed; MVP validates approach, not performance
- **When to implement**: After MVP validation, before production scale

### Task 6: Streaming Adaptation Mode
- **Reference**: tasks.md Tasks 6.1-6.3
- **Reason**: Tier 2 feature; offline mode sufficient for validation
- **When to implement**: After production deployment, as enhancement

### Task 7-8: Final Integration and Release
- **Reference**: tasks.md Tasks 7.1-8.4
- **Reason**: Release activities come after validation
- **When to implement**: After MVP validation and production readiness

---

## Traceability Matrix

| MVP Component | tasks.md Reference | Status | MVP Critical? |
|---------------|-------------------|--------|---------------|
| Test Coverage | Tasks 1.1-1.9 | ✅ Complete | Yes - Quality |
| Experiment Orchestration | Task 2.1 | ✅ Complete | Yes - Core API |
| QA Example | Task 2.2 | ✅ Complete | Yes - Validation |
| Code Gen Example | Task 2.3 | ✅ Complete | Yes - Validation |
| Classification Example | Task 2.4 | ✅ Complete | Yes - Validation |
| Config Template | Task 2.5 | ✅ Complete | Yes - Usability |
| Integration Tests | Task 2.6 | ✅ Complete | Yes - Quality |
| Security Verification | Task 4.1 (lean) | ⏳ Pre-flight | Yes - Trust |
| Deployment Docs | Tasks 3.1-3.5 | ⏸️ Deferred | No - Post-MVP |
| Performance Optimization | Tasks 5.1-5.4 | ⏸️ Deferred | No - Post-MVP |
| Streaming Mode | Tasks 6.1-6.3 | ⏸️ Deferred | No - Post-MVP |

---

## Post-MVP Transition Plan

### If MVP Validation Succeeds

1. **Document Results** (30 minutes)
   - Record improvement metrics for each example
   - Capture sample playbook strategies learned
   - Note any issues or limitations observed

2. **Update tasks.md** (15 minutes)
   - Mark MVP-related tasks as validated
   - Prioritize remaining tasks based on findings
   - Update task estimates based on MVP learnings

3. **Plan Production Path** (1 hour)
   - Review deferred tasks (3, 4, 5, 6)
   - Prioritize based on deployment needs
   - Create production implementation timeline

### If MVP Validation Fails

1. **Root Cause Analysis** (1 hour)
   - Identify why improvement didn't occur
   - Check for implementation bugs
   - Validate test methodology

2. **Iterate on Core** (variable)
   - Fix identified issues
   - Re-run MVP validation
   - Do NOT proceed to production tasks until core works

---

## Key Principles

### Single Source of Truth
- **tasks.md** is the authoritative task list
- This document is a **view** of tasks.md, not a replacement
- Changes to task status should be reflected in both documents

### Clear Separation
- MVP scope = validation-critical tasks only
- Production scope = everything in tasks.md
- No ambiguity about what's in/out of MVP

### Traceability
- Every MVP task references its tasks.md origin
- Deferred tasks are explicitly listed with rationale
- Easy to navigate between MVP and production views

### Change Management
- MVP task completion → Update both documents
- New MVP insights → Feed back to tasks.md
- Production task changes → Evaluate MVP impact

---

## FAQ

### Q: Why not just use tasks.md with filters?
**A**: This document provides a focused, unambiguous MVP checklist without the cognitive load of seeing 50+ production tasks. It's optimized for execution, not planning.

### Q: What if I want to implement a deferred task?
**A**: Check tasks.md for the full task details, dependencies, and requirements. This document only contains MVP-scoped information.

### Q: How do I know if something is MVP-critical?
**A**: Ask: "Does this block validation of the core hypothesis?" If no, it's not MVP-critical.

### Q: When should I stop using this document?
**A**: After MVP validation is complete and you're transitioning to production implementation. At that point, use tasks.md exclusively.

---

## Document Metadata

- **Created**: 2025-10-31
- **Source**: tasks.md (ACE-Open Project Continuation)
- **Scope**: MVP validation only
- **Maintenance**: Update when MVP scope changes or validation completes
- **Owner**: Development team
- **Review Frequency**: After each MVP test run

---

## Navigation

- 📋 [Full Task List (tasks.md)](./tasks.md) - Complete production implementation plan
- 📐 [Design Document (design.md)](./design.md) - Architecture and component design
- 📝 [Requirements (requirements.md)](./requirements.md) - EARS-compliant requirements
- 🚀 [Examples Directory](../../examples/) - MVP validation examples

---

**Remember**: This is a branch-off document. The source of truth is `tasks.md`. Use this for MVP execution, use `tasks.md` for production planning.
