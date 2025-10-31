# Updated Implementation Plan: LLM Provider Modules

**Date**: October 31, 2025  
**Priority**: P0 - CRITICAL BLOCKER  
**Status**: NEW TASKS - Must be completed before MVP can ship

---

## Context

Dependency analysis revealed that **two critical modules are missing**:
1. `ace/llm_openai.py` - OpenAI integration
2. `ace/llm_anthropic.py` - Anthropic integration

These modules are imported by `ace/settings.py` but don't exist, causing 100% of production examples to fail.

**Good News**: Working implementation exists in `test_ace_openai.py` - just needs to be moved and adapted.

---

## New Tasks (Insert BEFORE Task 1 in tasks.md)

### 🔴 CRITICAL: Task 0 - Implement Missing LLM Provider Modules

**Priority**: P0 - BLOCKING  
**Estimated Effort**: 2-3 days  
**Must Complete Before**: Any MVP validation

- [ ] 0. Implement Missing LLM Provider Modules
  - Create OpenAI and Anthropic LLM client modules
  - Update tests and documentation
  - Add prevention mechanisms
  - _Requirements: ALL (blocks everything)_

- [ ] 0.1 Implement ace/llm_openai.py
  - Create `ace/llm_openai.py` module
  - Move `OpenAIClient` class from `test_ace_openai.py` (lines 40-165)
  - Rename `OpenAIClient` → `OpenAILLMClient` for consistency
  - Add `complete_structured()` method using OpenAI JSON mode
  - Update imports to use `ace.llm` base classes
  - Add comprehensive docstrings with examples
  - Add error handling for rate limits and API errors
  - Add logging using structlog
  - Add `__repr__()` method with redacted API key
  - _Requirements: ALL (blocks all examples)_


- [ ] 0.2 Implement ace/llm_anthropic.py
  - Create `ace/llm_anthropic.py` module
  - Implement `AnthropicLLMClient` class extending `LLMClient`
  - Implement `complete()` using Anthropic Messages API
  - Implement `complete_structured()` using extraction (Anthropic has no JSON mode)
  - Add error handling for rate limits and API errors
  - Add logging using structlog
  - Add `__repr__()` method with redacted API key
  - Follow same pattern as OpenAI client for consistency
  - _Requirements: ALL (needed for provider parity)_

- [ ] 0.3 Update ace/__init__.py exports
  - Add `from ace.llm_openai import OpenAILLMClient` to imports
  - Add `from ace.llm_anthropic import AnthropicLLMClient` to imports
  - Add both classes to `__all__` list
  - Test that imports work: `from ace import OpenAILLMClient`
  - _Requirements: ALL (user convenience)_

- [ ] 0.4 Create unit tests for OpenAI client
  - Create `tests/test_llm_openai.py`
  - Test client initialization with/without API key
  - Test `complete()` method with mocked OpenAI API
  - Test `complete_structured()` method with JSON mode
  - Test error handling (rate limits, API errors, network errors)
  - Test token usage tracking
  - Test `__repr__()` redacts API key
  - Achieve 90%+ coverage for `ace/llm_openai.py`
  - _Requirements: ALL (quality assurance)_

- [ ] 0.5 Create unit tests for Anthropic client
  - Create `tests/test_llm_anthropic.py`
  - Test client initialization with/without API key
  - Test `complete()` method with mocked Anthropic API
  - Test `complete_structured()` method with extraction
  - Test error handling (rate limits, API errors, network errors)
  - Test token usage tracking
  - Test `__repr__()` redacts API key
  - Achieve 90%+ coverage for `ace/llm_anthropic.py`
  - _Requirements: ALL (quality assurance)_

- [ ] 0.6 Update tests/test_config_factories.py
  - Remove mocking of `ace.llm_openai` module (lines 99, 120)
  - Test actual import: `from ace.llm_openai import OpenAILLMClient`
  - Test actual import: `from ace.llm_anthropic import AnthropicLLMClient`
  - Test `create_llm_client_from_config()` with real classes (mocked API calls)
  - Verify tests catch missing modules (would have caught this bug)
  - _Requirements: ALL (prevent regression)_

- [ ] 0.7 Add integration tests for examples
  - Create `tests/test_examples_integration.py`
  - Test all examples can be imported without errors
  - Test `examples.qa_task.run_qa` imports successfully
  - Test `examples.code_generation.run_codegen` imports successfully
  - Test `examples.classification.run_classification` imports successfully
  - Add to CI/CD pipeline
  - _Requirements: ALL (prevent similar issues)_

- [ ] 0.8 Update documentation
  - Update README.md with LLM provider setup instructions
  - Create `docs/LLM_PROVIDERS.md` with provider guide
  - Document OpenAI setup (API key, installation, usage)
  - Document Anthropic setup (API key, installation, usage)
  - Add troubleshooting section for missing modules
  - Update example READMEs with provider requirements
  - _Requirements: ALL (user guidance)_

- [ ] 0.9 Add prevention mechanisms
  - Create `.git/hooks/pre-commit` to verify imports
  - Update `.github/workflows/test.yml` to verify imports
  - Add `tests/test_module_inventory.py` to verify all required modules exist
  - Add import linting with pylint (enable import-error)
  - Create `docs/MODULE_STATUS.md` listing all modules
  - _Requirements: ALL (prevent recurrence)_

- [ ] 0.10 Validate all examples work
  - Run `python examples/qa_task/run_qa.py` (should succeed)
  - Run `python examples/code_generation/run_codegen.py` (should succeed)
  - Run `python examples/classification/run_classification.py` (should succeed)
  - Verify all examples complete without errors
  - Verify playbook evolution works correctly
  - Document any issues found
  - _Requirements: ALL (final validation)_

---

## Implementation Details

### 0.1: OpenAI Client Implementation

**Source File**: `test_ace_openai.py` lines 40-165

**Key Changes Needed**:
1. Rename class: `OpenAIClient` → `OpenAILLMClient`
2. Add `complete_structured()` method:
   ```python
   def complete_structured(
       self, 
       prompt: str, 
       schema: Dict[str, Any], 
       **kwargs: Any
   ) -> LLMResponse:
       """Generate structured JSON output using OpenAI JSON mode."""
       # Use response_format={"type": "json_object"} for gpt-4o models
       # Fall back to extraction for older models
   ```
3. Update imports:
   ```python
   from ace.llm import LLMClient, LLMResponse
   from ace.config import get_api_key, redact_secret
   import structlog
   
   logger = structlog.get_logger(__name__)
   ```
4. Add logging to all methods
5. Improve error messages

**Interface Contract**:
```python
class OpenAILLMClient(LLMClient):
    """OpenAI API client for ACE framework."""
    
    def __init__(
        self,
        api_key: str,
        model: str = "gpt-4o-mini",
        temperature: float = 0.0,
        max_tokens: int = 512,
        timeout: int = 60,
        max_retries: int = 3,
    ) -> None:
        """Initialize OpenAI client."""
        
    def complete(self, prompt: str, **kwargs: Any) -> LLMResponse:
        """Generate completion using OpenAI Chat API."""
        
    def complete_structured(
        self, 
        prompt: str, 
        schema: Dict[str, Any], 
        **kwargs: Any
    ) -> LLMResponse:
        """Generate structured JSON output."""
```

### 0.2: Anthropic Client Implementation

**No Existing Code** - Must create from scratch

**Reference**: Follow OpenAI client pattern

**Key Implementation Points**:
1. Use Anthropic Messages API (not legacy Completions API)
2. Handle different message format (system vs. user messages)
3. No native JSON mode - use extraction + validation
4. Different token counting (prompt_tokens vs. input_tokens)
5. Different error types (AnthropicError vs. OpenAIError)

**Interface Contract**:
```python
class AnthropicLLMClient(LLMClient):
    """Anthropic API client for ACE framework."""
    
    def __init__(
        self,
        api_key: str,
        model: str = "claude-3-5-sonnet-20241022",
        temperature: float = 0.0,
        max_tokens: int = 512,
    ) -> None:
        """Initialize Anthropic client."""
        
    def complete(self, prompt: str, **kwargs: Any) -> LLMResponse:
        """Generate completion using Anthropic Messages API."""
        
    def complete_structured(
        self, 
        prompt: str, 
        schema: Dict[str, Any], 
        **kwargs: Any
    ) -> LLMResponse:
        """Generate structured JSON output (extraction-based)."""
```

---

## Testing Strategy

### Unit Tests (Mocked APIs)

**Purpose**: Test client logic without making real API calls

**Approach**:
- Mock `openai.OpenAI` and `anthropic.Anthropic` clients
- Mock API responses with realistic data
- Test error handling with mocked exceptions
- Test edge cases (empty responses, malformed JSON, etc.)

**Example**:
```python
@patch('openai.OpenAI')
def test_openai_complete(mock_openai):
    mock_response = Mock()
    mock_response.choices = [Mock(message=Mock(content="Hello"))]
    mock_client = Mock()
    mock_client.chat.completions.create.return_value = mock_response
    mock_openai.return_value = mock_client
    
    client = OpenAILLMClient(api_key="sk-test")
    response = client.complete("Test")
    
    assert response.text == "Hello"
```

### Integration Tests (Import Validation)

**Purpose**: Verify modules can be imported and examples work

**Approach**:
- Test imports resolve without errors
- Test examples can be imported (don't run them)
- Test factory functions create correct clients
- Run in CI/CD on every commit

**Example**:
```python
def test_openai_client_importable():
    from ace.llm_openai import OpenAILLMClient
    assert OpenAILLMClient is not None

def test_qa_example_importable():
    import examples.qa_task.run_qa
    # Just verify it imports, don't run it
```

### End-to-End Tests (Real APIs - Optional)

**Purpose**: Validate clients work with real APIs

**Approach**:
- Only run if API keys are present
- Use `@pytest.mark.skipif` to skip if no keys
- Use minimal prompts to reduce cost
- Run nightly, not on every commit

**Example**:
```python
@pytest.mark.skipif(not os.getenv("OPENAI_API_KEY"), reason="No API key")
def test_openai_real_api():
    client = OpenAILLMClient(api_key=os.getenv("OPENAI_API_KEY"))
    response = client.complete("Say 'test'")
    assert len(response.text) > 0
```

---

## Prevention Mechanisms

### 1. Pre-commit Hook

**File**: `.git/hooks/pre-commit`

```bash
#!/bin/bash
echo "Verifying all imports resolve..."

python -c "from ace.llm_openai import OpenAILLMClient" || {
    echo "ERROR: ace.llm_openai module not found"
    exit 1
}

python -c "from ace.llm_anthropic import AnthropicLLMClient" || {
    echo "ERROR: ace.llm_anthropic module not found"
    exit 1
}

echo "✓ All imports verified"
```

### 2. CI/CD Check

**File**: `.github/workflows/test.yml`

```yaml
- name: Verify module imports
  run: |
    python -c "from ace.llm_openai import OpenAILLMClient"
    python -c "from ace.llm_anthropic import AnthropicLLMClient"
    python -c "import examples.qa_task.run_qa"
    python -c "import examples.code_generation.run_codegen"
    python -c "import examples.classification.run_classification"
```

### 3. Module Inventory Test

**File**: `tests/test_module_inventory.py`

```python
"""Verify all expected modules exist."""

REQUIRED_MODULES = [
    "ace.llm",
    "ace.llm_openai",
    "ace.llm_anthropic",
    "ace.settings",
    "ace.adaptation",
    "ace.roles",
    "ace.playbook",
    "ace.delta",
    "ace.deduplication",
    "ace.monitoring",
    "ace.versioning",
    "ace.config",
]

def test_all_required_modules_exist():
    """Test that all required modules can be imported."""
    for module_name in REQUIRED_MODULES:
        __import__(module_name)
```

### 4. Import Linting

**File**: `pyproject.toml` or `.pylintrc`

```toml
[tool.pylint]
enable = ["import-error"]

[tool.pylint.messages_control]
disable = ["all"]
enable = ["import-error", "undefined-variable"]
```

**Run in CI**:
```bash
pylint ace/ --disable=all --enable=import-error
```

---

## Timeline

### Day 1: OpenAI Implementation (6 hours)
- [ ] 0.1: Implement `ace/llm_openai.py` (4 hours)
- [ ] 0.4: Create unit tests (2 hours)

### Day 2: Anthropic Implementation (8 hours)
- [ ] 0.2: Implement `ace/llm_anthropic.py` (5 hours)
- [ ] 0.5: Create unit tests (3 hours)

### Day 3: Integration & Testing (6 hours)
- [ ] 0.3: Update `ace/__init__.py` (30 min)
- [ ] 0.6: Update `test_config_factories.py` (1 hour)
- [ ] 0.7: Add integration tests (2 hours)
- [ ] 0.10: Validate all examples (2.5 hours)

### Day 4: Documentation & Prevention (4 hours)
- [ ] 0.8: Update documentation (2 hours)
- [ ] 0.9: Add prevention mechanisms (2 hours)

**Total Effort**: 24 hours (3 days of focused work)

---

## Success Criteria

### Implementation Success
- [ ] `ace/llm_openai.py` exists and is importable
- [ ] `ace/llm_anthropic.py` exists and is importable
- [ ] Both clients implement full `LLMClient` interface
- [ ] Unit tests pass with 90%+ coverage
- [ ] Integration tests pass

### Example Success
- [ ] All 3 MVP examples run without errors
- [ ] QA example completes successfully
- [ ] Code generation example completes successfully
- [ ] Classification example completes successfully
- [ ] Playbook evolution works correctly

### Prevention Success
- [ ] Pre-commit hook catches missing modules
- [ ] CI/CD catches missing modules
- [ ] Module inventory test catches missing modules
- [ ] Import linting catches unresolved imports
- [ ] Documentation is complete and accurate

---

## Risk Mitigation

### Risk: Implementation takes longer than expected
**Mitigation**: Start with OpenAI only (Day 1-2), defer Anthropic to later

### Risk: Tests reveal more issues
**Mitigation**: Allocate buffer time on Day 3-4 for fixes

### Risk: API changes break implementation
**Mitigation**: Pin API versions, add version checks

### Risk: Examples still don't work
**Mitigation**: Test each example individually before declaring success

---

## Next Steps

1. **Review this plan** with team
2. **Assign owner** to Task 0 (backend developer)
3. **Create feature branch** (`feature/llm-providers`)
4. **Start Day 1** (OpenAI implementation)
5. **Daily check-ins** to track progress
6. **Merge to main** after all success criteria met

---

## Integration with Existing Plan

**Insert Point**: These tasks should be inserted as **Task 0** in the existing `tasks.md` file, BEFORE Task 1 (Test Coverage).

**Rationale**: These are blocking tasks that must be completed before any MVP validation can occur. Without these modules, 100% of production examples fail.

**Updated Task Order**:
1. **Task 0**: Implement Missing LLM Provider Modules (NEW - THIS DOCUMENT)
2. Task 1: Achieve 90% Test Coverage (EXISTING)
3. Task 2: Implement Configuration-Based Workflow (EXISTING)
4. ... (rest of existing tasks)

---

## Conclusion

This is a **critical but straightforward fix**:
- ✅ Working code exists (just needs to be moved)
- ✅ Clear pattern to follow (existing clients)
- ✅ No architectural changes needed
- ✅ High confidence in timeline (3 days)

**Let's get started!** 🚀
