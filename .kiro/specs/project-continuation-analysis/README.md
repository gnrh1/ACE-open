# ACE-Open Project Continuation - Documentation Guide

## Quick Navigation

### 📋 For MVP Validation
**Start here**: [MVP-SCOPE.md](./MVP-SCOPE.md)
- Focused checklist for MVP test run
- Pre-flight verification steps
- Test execution instructions
- Success criteria

### 📐 For Production Planning
**Start here**: [tasks.md](./tasks.md)
- Complete task list (source of truth)
- All production infrastructure tasks
- Full dependency tracking
- Comprehensive requirements mapping

### 📝 For Understanding Requirements
**Start here**: [requirements.md](./requirements.md)
- EARS-compliant acceptance criteria
- User stories for all features
- Glossary of terms

### 🏗️ For Architecture Details
**Start here**: [design.md](./design.md)
- System architecture
- Component designs
- Integration patterns
- Technical decisions

---

## Document Relationships

```
requirements.md (What we need to build)
    ↓
design.md (How we'll build it)
    ↓
tasks.md (Step-by-step implementation)
    ↓
    ├─→ MVP-SCOPE.md (Subset for validation)
    └─→ [Full production tasks]
```

---

## Current Status

### ✅ MVP Ready
- **Test Coverage**: 95.61% (535 tests passing)
- **Examples**: 3 complete (QA, Code Gen, Classification)
- **Infrastructure**: Experiment orchestration, config system, CLI
- **Documentation**: Comprehensive READMEs for all examples

### ⏸️ Deferred (Post-MVP)
- Production deployment (Docker, K8s, monitoring)
- Performance optimization
- Streaming adaptation mode
- Enhanced security documentation

---

## How to Use This Documentation

### If you want to...

**Run MVP validation**
1. Read [MVP-SCOPE.md](./MVP-SCOPE.md)
2. Complete pre-flight checklist (10 min)
3. Run 3 examples (30-60 min)
4. Document results

**Understand what was built**
1. Read [design.md](./design.md) - Architecture overview
2. Check [tasks.md](./tasks.md) - See completed tasks (marked with ✅)
3. Review [CHANGELOG.md](../../../CHANGELOG.md) - Detailed progress log

**Plan production deployment**
1. Read [tasks.md](./tasks.md) - See Task 3 onwards
2. Review [requirements.md](./requirements.md) - Understand requirements
3. Check [design.md](./design.md) - See deployment architecture

**Add new features**
1. Check [requirements.md](./requirements.md) - Ensure requirement exists
2. Update [design.md](./design.md) - Add design if needed
3. Add to [tasks.md](./tasks.md) - Create implementation tasks
4. Evaluate MVP impact - Update [MVP-SCOPE.md](./MVP-SCOPE.md) if critical

---

## Key Principles

### Single Source of Truth
- **tasks.md** is the authoritative task list
- Other documents are views/subsets, not replacements

### Clear Separation
- MVP scope is explicitly marked in both tasks.md and MVP-SCOPE.md
- No ambiguity about what's in/out of MVP

### Traceability
- Every task references requirements
- MVP tasks reference their tasks.md origin
- Changes propagate clearly

---

## Maintenance

### When MVP validation completes
1. Update [MVP-SCOPE.md](./MVP-SCOPE.md) with results
2. Mark validation status in [tasks.md](./tasks.md)
3. Prioritize remaining tasks based on findings

### When adding new tasks
1. Add to [tasks.md](./tasks.md) first (source of truth)
2. Evaluate if MVP-critical
3. Update [MVP-SCOPE.md](./MVP-SCOPE.md) only if MVP-critical

### When completing tasks
1. Mark complete in [tasks.md](./tasks.md)
2. Update [MVP-SCOPE.md](./MVP-SCOPE.md) if MVP task
3. Log in [CHANGELOG.md](../../../CHANGELOG.md)

---

## Questions?

- **"Which document should I use?"** → See "How to Use This Documentation" above
- **"Is this task MVP-critical?"** → Check [MVP-SCOPE.md](./MVP-SCOPE.md) traceability matrix
- **"What's the full scope?"** → See [tasks.md](./tasks.md)
- **"What's been completed?"** → Check ✅ marks in [tasks.md](./tasks.md) or [CHANGELOG.md](../../../CHANGELOG.md)

---

## Document Metadata

- **Created**: 2025-10-31
- **Purpose**: Navigation guide for project continuation documentation
- **Audience**: Developers, project managers, stakeholders
- **Maintenance**: Update when documentation structure changes
