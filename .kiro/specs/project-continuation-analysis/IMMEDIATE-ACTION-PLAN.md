# Immediate Action Plan - Fix ACE-open MVP Blocker

**Created**: October 31, 2025  
**Priority**: P0 - CRITICAL BLOCKER  
**Owner**: Development Team  
**Target**: Ship-ready in 2-3 weeks

---

## Problem Statement

ACE-open MVP has a critical blocker: `ace/llm_openai.py` module is missing, causing 67% of examples to fail immediately. This must be fixed before any release.

---

## Action Items

### 🔴 CRITICAL - Must Complete Before Any Release

#### 1. Implement `ace/llm_openai.py` Module
**Priority**: P0  
**Effort**: 2-3 days  
**Owner**: Backend Developer  
**Blocker**: None

**Requirements**:
- Implement `OpenAILLMClient` class extending `LLMClient`
- Support both `complete()` and `complete_structured()` methods
- Handle API errors gracefully with retries
- Support all config parameters (temperature, max_tokens, timeout, etc.)
- Add proper logging for debugging
- Include docstrings and type hints

**Acceptance Criteria**:
- [ ] File `ace/llm_openai.py` exists and is importable
- [ ] `OpenAILLMClient` class implements all required methods
- [ ] All 3 MVP examples run without import errors
- [ ] Unit tests pass for OpenAI client
- [ ] Integration tests pass for all examples

**Reference Implementation**:
```python
"""OpenAI LLM client implementation for ACE framework."""

from __future__ import annotations

import json
import structlog
from typing import Any, Dict, Optional

from openai import OpenAI
from openai.types.chat import ChatCompletion

from ace.llm import LLMClient, LLMResponse

logger = structlog.get_logger(__name__)


class OpenAILLMClient(LLMClient):
    """LLM client powered by OpenAI API."""

    def __init__(
        self,
        api_key: str,
        model: str = "gpt-4o-mini",
        temperature: float = 0.0,
        max_tokens: int = 512,
        timeout: int = 60,
        max_retries: int = 3,
        **kwargs: Any,
    ) -> None:
        """Initialize OpenAI client.
        
        Args:
            api_key: OpenAI API key
            model: Model name (e.g., "gpt-4o-mini", "gpt-4")
            temperature: Sampling temperature (0.0 = deterministic)
            max_tokens: Maximum tokens to generate
            timeout: Request timeout in seconds
            max_retries: Number of retries on failure
            **kwargs: Additional arguments passed to OpenAI client
        """
        super().__init__(model=model)
        
        self.client = OpenAI(
            api_key=api_key,
            timeout=timeout,
            max_retries=max_retries,
        )
        self.temperature = temperature
        self.max_tokens = max_tokens
        self._kwargs = kwargs
        
        logger.info(
            "openai_client_initialized",
            model=model,
            temperature=temperature,
            max_tokens=max_tokens,
        )

    def complete(self, prompt: str, **kwargs: Any) -> LLMResponse:
        """Generate completion using OpenAI API.
        
        Args:
            prompt: The prompt to complete
            **kwargs: Additional arguments (overrides defaults)
            
        Returns:
            LLMResponse with generated text
        """
        # Merge default kwargs with call-specific kwargs
        call_kwargs = {
            "temperature": self.temperature,
            "max_tokens": self.max_tokens,
            **self._kwargs,
            **kwargs,
        }
        
        # Remove ACE-specific kwargs that OpenAI doesn't understand
        call_kwargs.pop("refinement_round", None)
        
        logger.debug(
            "openai_completion_request",
            model=self.model,
            prompt_length=len(prompt),
            **call_kwargs,
        )
        
        try:
            response: ChatCompletion = self.client.chat.completions.create(
                model=self.model,
                messages=[
                    {"role": "user", "content": prompt}
                ],
                **call_kwargs,
            )
            
            text = response.choices[0].message.content or ""
            
            logger.debug(
                "openai_completion_response",
                response_length=len(text),
                finish_reason=response.choices[0].finish_reason,
                usage=response.usage.model_dump() if response.usage else None,
            )
            
            return LLMResponse(
                text=text.strip(),
                raw={
                    "response": response.model_dump(),
                    "usage": response.usage.model_dump() if response.usage else None,
                }
            )
            
        except Exception as e:
            logger.error(
                "openai_completion_error",
                error=str(e),
                error_type=type(e).__name__,
            )
            raise

    def complete_structured(
        self, 
        prompt: str, 
        schema: Dict[str, Any], 
        **kwargs: Any
    ) -> LLMResponse:
        """Generate structured output using OpenAI JSON mode.
        
        Args:
            prompt: The prompt to complete
            schema: JSON schema for output validation
            **kwargs: Additional arguments
            
        Returns:
            LLMResponse with validated JSON text
        """
        # Add JSON mode if model supports it
        call_kwargs = kwargs.copy()
        
        # Models that support JSON mode
        json_mode_models = ["gpt-4o", "gpt-4o-mini", "gpt-4-turbo"]
        supports_json_mode = any(m in self.model for m in json_mode_models)
        
        if supports_json_mode:
            call_kwargs["response_format"] = {"type": "json_object"}
            # Add schema to prompt for JSON mode
            enhanced_prompt = (
                f"{prompt}\n\n"
                f"IMPORTANT: Respond with valid JSON matching this schema:\n"
                f"{json.dumps(schema, indent=2)}"
            )
        else:
            # Fall back to parent class implementation (extraction + validation)
            return super().complete_structured(prompt, schema, **kwargs)
        
        logger.debug(
            "openai_structured_request",
            model=self.model,
            json_mode=supports_json_mode,
        )
        
        response = self.complete(enhanced_prompt, **call_kwargs)
        
        # Validate against schema
        from ace.llm import extract_json_from_text, validate_json_schema
        
        try:
            data = extract_json_from_text(response.text)
            validate_json_schema(data, schema)
            
            # Return clean JSON
            return LLMResponse(
                text=json.dumps(data, ensure_ascii=False),
                raw=response.raw
            )
        except (ValueError, json.JSONDecodeError) as e:
            logger.error(
                "openai_structured_validation_error",
                error=str(e),
                response_text=response.text[:200],
            )
            raise ValueError(
                f"OpenAI response does not match schema: {e}"
            ) from e
```

**Testing Checklist**:
```bash
# 1. Test import
python -c "from ace.llm_openai import OpenAILLMClient; print('✓ Import successful')"

# 2. Test basic completion
python -c "
from ace.llm_openai import OpenAILLMClient
import os
client = OpenAILLMClient(api_key=os.getenv('OPENAI_API_KEY'))
response = client.complete('Say hello')
print(f'✓ Completion: {response.text}')
"

# 3. Test structured output
python -c "
from ace.llm_openai import OpenAILLMClient
import os
client = OpenAILLMClient(api_key=os.getenv('OPENAI_API_KEY'))
schema = {'type': 'object', 'properties': {'greeting': {'type': 'string'}}}
response = client.complete_structured('Say hello in JSON', schema)
print(f'✓ Structured: {response.text}')
"

# 4. Run all examples
python examples/qa_task/run_qa.py
python examples/code_generation/run_codegen.py
python examples/classification/run_classification.py
```

---

#### 2. Add Integration Tests
**Priority**: P0  
**Effort**: 1 day  
**Owner**: QA/Test Engineer  
**Blocker**: Depends on #1

**Requirements**:
- Create `tests/test_examples.py`
- Add tests for all 3 MVP examples
- Add tests for OpenAI client
- Integrate into CI/CD pipeline

**Implementation**:
```python
"""Integration tests for MVP examples."""

import os
import subprocess
import pytest
from pathlib import Path

EXAMPLES_DIR = Path(__file__).parent.parent / "examples"


@pytest.mark.integration
def test_qa_example_imports():
    """Verify QA example can import without errors."""
    result = subprocess.run(
        ["python", "-c", "import sys; sys.path.insert(0, 'examples/qa_task'); import run_qa"],
        capture_output=True,
        text=True,
    )
    assert result.returncode == 0, f"Import failed: {result.stderr}"


@pytest.mark.integration
def test_codegen_example_imports():
    """Verify code generation example can import without errors."""
    result = subprocess.run(
        ["python", "-c", "import sys; sys.path.insert(0, 'examples/code_generation'); import run_codegen"],
        capture_output=True,
        text=True,
    )
    assert result.returncode == 0, f"Import failed: {result.stderr}"


@pytest.mark.integration
def test_classification_example_imports():
    """Verify classification example can import without errors."""
    result = subprocess.run(
        ["python", "-c", "import sys; sys.path.insert(0, 'examples/classification'); import run_classification"],
        capture_output=True,
        text=True,
    )
    assert result.returncode == 0, f"Import failed: {result.stderr}"


@pytest.mark.integration
@pytest.mark.skipif(not os.getenv("OPENAI_API_KEY"), reason="No OpenAI API key")
def test_openai_client_basic():
    """Test OpenAI client basic functionality."""
    from ace.llm_openai import OpenAILLMClient
    
    client = OpenAILLMClient(
        api_key=os.getenv("OPENAI_API_KEY"),
        model="gpt-4o-mini",
        temperature=0.0,
    )
    
    response = client.complete("Say 'test successful'")
    assert response.text
    assert len(response.text) > 0


@pytest.mark.integration
@pytest.mark.skipif(not os.getenv("OPENAI_API_KEY"), reason="No OpenAI API key")
def test_openai_client_structured():
    """Test OpenAI client structured output."""
    from ace.llm_openai import OpenAILLMClient
    
    client = OpenAILLMClient(
        api_key=os.getenv("OPENAI_API_KEY"),
        model="gpt-4o-mini",
    )
    
    schema = {
        "type": "object",
        "properties": {
            "message": {"type": "string"}
        },
        "required": ["message"]
    }
    
    response = client.complete_structured(
        "Respond with a JSON object containing a 'message' field",
        schema
    )
    
    import json
    data = json.loads(response.text)
    assert "message" in data
```

**Acceptance Criteria**:
- [ ] All integration tests pass
- [ ] Tests run in CI/CD pipeline
- [ ] Tests catch the missing module issue
- [ ] Tests validate OpenAI client works

---

#### 3. Update Documentation
**Priority**: P0  
**Effort**: 1 day  
**Owner**: Technical Writer / Developer  
**Blocker**: None (can be done in parallel)

**Requirements**:
- Update README.md with setup instructions
- Add troubleshooting guide
- Document API key configuration
- Add known limitations section

**Changes to README.md**:

```markdown
## Prerequisites

- Python 3.9 or higher
- OpenAI API key (for examples using OpenAI models)
- 4GB+ RAM recommended

## Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/ACE-open.git
cd ACE-open
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Set up your OpenAI API key:
```bash
export OPENAI_API_KEY="your-api-key-here"
```

Or create a `.env` file:
```bash
echo "OPENAI_API_KEY=your-api-key-here" > .env
```

## Running Examples

### Quick Start

Run the QA example:
```bash
python examples/qa_task/run_qa.py
```

Run the code generation example:
```bash
python examples/code_generation/run_codegen.py
```

Run the classification example:
```bash
python examples/classification/run_classification.py
```

### Dry Run Mode

Test examples without making API calls:
```bash
python examples/qa_task/run_qa.py --dry-run
```

## Troubleshooting

### "ModuleNotFoundError: No module named 'ace.llm_openai'"

**Solution**: Make sure you have the latest version of the code. The `ace/llm_openai.py` file should exist in your installation.

### "OpenAI API key not found"

**Solution**: Set the `OPENAI_API_KEY` environment variable:
```bash
export OPENAI_API_KEY="your-api-key-here"
```

### "Rate limit exceeded"

**Solution**: OpenAI has rate limits. Wait a few minutes and try again, or use a different API key tier.

### Examples are slow

**Solution**: Examples make real API calls which can take time. Use `--dry-run` for faster testing, or reduce the number of samples in the config file.

## Known Limitations

- OpenAI API calls cost money (typically $0.01-0.10 per example run)
- Examples require internet connection for API access
- Rate limits apply based on your OpenAI account tier
- Some examples may take several minutes to complete

## Cost Estimation

Approximate costs for running examples (using gpt-4o-mini):
- QA Task: ~$0.02 per run (10 samples × 3 epochs)
- Code Generation: ~$0.05 per run (8 samples × 3 epochs)
- Classification: ~$0.03 per run (12 samples × 3 epochs)

Total: ~$0.10 for all three examples
```

**Acceptance Criteria**:
- [ ] README has clear setup instructions
- [ ] Troubleshooting section covers common issues
- [ ] API key configuration is documented
- [ ] Cost estimation is provided
- [ ] Known limitations are listed

---

### 🟡 HIGH PRIORITY - Should Complete Before Release

#### 4. Add Pre-Flight Validation
**Priority**: P1  
**Effort**: 1 day  
**Owner**: Developer  
**Blocker**: Depends on #1

**Requirements**:
- Add `--dry-run` flag to all examples
- Validate dependencies before running
- Check API key is present
- Estimate costs before running

**Implementation**:
```python
def validate_environment():
    """Validate environment before running example."""
    issues = []
    
    # Check Python version
    import sys
    if sys.version_info < (3, 9):
        issues.append("Python 3.9+ required")
    
    # Check OpenAI package
    try:
        import openai
    except ImportError:
        issues.append("openai package not installed (pip install openai)")
    
    # Check API key
    import os
    if not os.getenv("OPENAI_API_KEY"):
        issues.append("OPENAI_API_KEY environment variable not set")
    
    # Check ACE modules
    try:
        from ace.llm_openai import OpenAILLMClient
    except ImportError:
        issues.append("ace.llm_openai module not found")
    
    if issues:
        print("❌ Environment validation failed:")
        for issue in issues:
            print(f"  - {issue}")
        return False
    
    print("✅ Environment validation passed")
    return True


def estimate_cost(num_samples: int, epochs: int, model: str = "gpt-4o-mini"):
    """Estimate cost of running example."""
    # Rough estimates (tokens per sample)
    tokens_per_sample = {
        "gpt-4o-mini": 500,  # input + output
        "gpt-4o": 500,
        "gpt-4": 500,
    }
    
    # Pricing per 1M tokens (as of Oct 2025)
    pricing = {
        "gpt-4o-mini": 0.15,  # $0.15 per 1M tokens
        "gpt-4o": 2.50,
        "gpt-4": 30.00,
    }
    
    tokens = tokens_per_sample.get(model, 500) * num_samples * epochs
    cost = (tokens / 1_000_000) * pricing.get(model, 2.50)
    
    print(f"💰 Estimated cost: ${cost:.4f}")
    print(f"   Model: {model}")
    print(f"   Samples: {num_samples} × {epochs} epochs")
    print(f"   Tokens: ~{tokens:,}")
    
    return cost
```

**Acceptance Criteria**:
- [ ] `--dry-run` flag works for all examples
- [ ] Environment validation catches missing dependencies
- [ ] Cost estimation is reasonably accurate
- [ ] Users can test without making API calls

---

#### 5. Fix Error Messages
**Priority**: P1  
**Effort**: 1 hour  
**Owner**: Developer  
**Blocker**: None

**Current Error** (Misleading):
```
ImportError: OpenAI provider requires 'openai' package. Install with: pip install openai
```

**Better Error**:
```
ImportError: OpenAI client module not found. 
The 'ace.llm_openai' module is required but not installed.

This may indicate:
1. Incomplete installation - try: pip install -e .
2. Missing file - ensure ace/llm_openai.py exists
3. Development version - check git status

For help, see: https://github.com/yourusername/ACE-open/issues
```

**Implementation**:
```python
# ace/settings.py
elif provider == LLMProvider.OPENAI:
    try:
        from ace.llm_openai import OpenAILLMClient
    except ImportError as e:
        error_msg = (
            "OpenAI client module not found.\n"
            "The 'ace.llm_openai' module is required but not available.\n\n"
            "This may indicate:\n"
            "1. Incomplete installation - try: pip install -e .\n"
            "2. Missing file - ensure ace/llm_openai.py exists\n"
            "3. Development version - check git status\n\n"
            "For help, see: https://github.com/yourusername/ACE-open/issues"
        )
        raise ImportError(error_msg) from e
```

**Acceptance Criteria**:
- [ ] Error message is clear and actionable
- [ ] Error message doesn't mislead users
- [ ] Error message provides helpful debugging steps

---

### 🟢 MEDIUM PRIORITY - Nice to Have

#### 6. Add CI/CD Pipeline
**Priority**: P2  
**Effort**: 2 days  
**Owner**: DevOps  
**Blocker**: Depends on #2

**Requirements**:
- Set up GitHub Actions workflow
- Run tests on every commit
- Run integration tests on PR
- Block merge if tests fail

**Implementation** (`.github/workflows/test.yml`):
```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install -e .
    
    - name: Run unit tests
      run: pytest tests/ -v --ignore=tests/test_examples.py
    
    - name: Run integration tests (imports only)
      run: pytest tests/test_examples.py -v -m "not skipif"
    
    - name: Verify all modules importable
      run: |
        python -c "from ace.llm_openai import OpenAILLMClient"
        python -c "import sys; sys.path.insert(0, 'examples/qa_task'); import run_qa"
        python -c "import sys; sys.path.insert(0, 'examples/code_generation'); import run_codegen"
        python -c "import sys; sys.path.insert(0, 'examples/classification'); import run_classification"
```

---

## Timeline

### Week 1: Implementation
| Day | Task | Owner | Status |
|-----|------|-------|--------|
| Mon | Implement llm_openai.py | Backend Dev | 🔴 Not Started |
| Tue | Complete llm_openai.py + unit tests | Backend Dev | 🔴 Not Started |
| Wed | Add integration tests | QA Engineer | 🔴 Not Started |
| Thu | Update documentation | Tech Writer | 🔴 Not Started |
| Fri | Add pre-flight validation | Developer | 🔴 Not Started |

### Week 2: Testing & Polish
| Day | Task | Owner | Status |
|-----|------|-------|--------|
| Mon | Run full test suite | QA Team | 🔴 Not Started |
| Tue | Fix any issues found | Dev Team | 🔴 Not Started |
| Wed | Code review | Tech Lead | 🔴 Not Started |
| Thu | Final testing | QA Team | 🔴 Not Started |
| Fri | Prepare release notes | Tech Writer | 🔴 Not Started |

### Week 3: Release
| Day | Task | Owner | Status |
|-----|------|-------|--------|
| Mon | Beta release to limited users | Product | 🔴 Not Started |
| Tue | Monitor for issues | Support | 🔴 Not Started |
| Wed | Fix critical issues | Dev Team | 🔴 Not Started |
| Thu | Final validation | QA Team | 🔴 Not Started |
| Fri | Public release | Product | 🔴 Not Started |

---

## Success Criteria

### Definition of Done

- [ ] `ace/llm_openai.py` file exists and is fully implemented
- [ ] All 3 MVP examples run without errors
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Documentation is complete and accurate
- [ ] Error messages are clear and helpful
- [ ] Pre-flight validation works
- [ ] CI/CD pipeline is set up
- [ ] Code review is complete
- [ ] Beta testing is successful
- [ ] No P0 or P1 bugs remain

### Release Checklist

- [ ] All examples tested on clean environment
- [ ] Documentation reviewed and approved
- [ ] Release notes prepared
- [ ] GitHub release created
- [ ] PyPI package published (if applicable)
- [ ] Announcement prepared
- [ ] Support channels ready

---

## Risk Management

### Risks & Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| OpenAI API changes | Low | High | Pin API version, add tests |
| Implementation takes longer | Medium | Medium | Start immediately, daily standups |
| Tests reveal more issues | Medium | High | Allocate buffer time in week 2 |
| Documentation incomplete | Low | Medium | Start docs in parallel with code |
| Beta users find critical bugs | Medium | High | Limited beta, quick response team |

### Contingency Plans

**If implementation takes >3 days**:
- Simplify initial implementation (basic features only)
- Defer structured output to v1.1
- Focus on getting examples working

**If tests reveal major issues**:
- Extend timeline by 1 week
- Prioritize P0 bugs only
- Defer nice-to-haves to v1.1

**If beta testing finds critical bugs**:
- Delay public release by 1 week
- Fix critical bugs immediately
- Re-run beta testing

---

## Communication Plan

### Daily Standups
- Time: 9:00 AM
- Duration: 15 minutes
- Attendees: Dev team, QA, Product
- Format: What did you do? What will you do? Any blockers?

### Weekly Status Updates
- Time: Friday 4:00 PM
- Duration: 30 minutes
- Attendees: All stakeholders
- Format: Progress review, risks, next week plan

### Release Announcement
- Channels: GitHub, Twitter, Blog, Email list
- Content: Features, improvements, getting started guide
- Timing: Friday Week 3 (after public release)

---

## Metrics to Track

### Development Metrics
- Lines of code added
- Test coverage percentage
- Number of bugs found/fixed
- Time to complete each task

### Quality Metrics
- Test pass rate
- Number of P0/P1/P2 bugs
- Code review feedback items
- Documentation completeness

### Release Metrics
- Number of beta users
- Beta user feedback score
- Time to first successful run
- Number of support requests

---

## Next Steps

1. **Assign owners** to each task
2. **Set up project board** (GitHub Projects or Jira)
3. **Schedule kickoff meeting** for Monday
4. **Create feature branch** for implementation
5. **Start daily standups** from Day 1

---

## Questions & Answers

**Q: Can we ship without the OpenAI client?**  
A: No. 67% of examples depend on it. Shipping without it would damage reputation.

**Q: Can we use a different LLM provider?**  
A: Yes, but examples are configured for OpenAI. Would need to update configs and test thoroughly.

**Q: How much will this cost to test?**  
A: ~$1-2 for development testing, ~$10-20 for full test suite runs.

**Q: What if we find more missing modules?**  
A: Integration tests will catch them. That's why we're adding them now.

**Q: Can we parallelize any tasks?**  
A: Yes! Documentation (#3) and error messages (#5) can be done in parallel with implementation (#1).

---

## Conclusion

This is a **fixable problem** with a **clear solution** and a **realistic timeline**.

The path forward:
1. Implement the missing module (2-3 days)
2. Add proper testing (1 day)
3. Update documentation (1 day)
4. Test thoroughly (1 week)
5. Release with confidence (1 week)

**Total time**: 2-3 weeks from start to production-ready release.

**Let's get started!** 🚀
