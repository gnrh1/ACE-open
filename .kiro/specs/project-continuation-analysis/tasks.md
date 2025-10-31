# Implementation Plan

## Overview

This implementation plan converts the ACE-Open continuation design into actionable coding tasks. The plan prioritizes achieving 90% test coverage, implementing configuration-based workflows, and preparing for production deployment.

## Task List

- [ ] 1. Achieve 90% Test Coverage
  - Implement comprehensive test suite for monitoring module
  - Fill coverage gaps in existing modules
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 2.1, 2.2, 2.3, 2.4, 2.5_

- [x] 1.1 Create monitoring module test suite
  - Create `tests/test_monitoring.py` with test classes for decorators, utilities, and export functions
  - Mock Prometheus metrics objects using `unittest.mock.patch`
  - Test `monitor_adaptation` decorator with success and failure scenarios
  - Test `monitor_llm_call` decorator with latency and token tracking
  - Test `monitor_playbook_operation` decorator for load/save operations
  - Test `monitor_deduplication` decorator for semantic and exact deduplication
  - Test utility functions: `track_bullet_added`, `track_bullet_removed`, `update_playbook_size`, `track_json_parse_failure`
  - Test metrics export: `write_metrics_to_file` and `start_metrics_server`
  - Verify Prometheus format output by parsing generated files
  - Achieve 90%+ coverage for `ace/monitoring.py`
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

- [x] 1.2 Fill coverage gaps in ace/delta.py (currently 76.32%)
  - Add tests for edge cases in `DeltaOperation` validation
  - Test `DeltaBatch` serialization with complex operations
  - Test error handling for invalid operation types
  - Achieve 90%+ coverage for `ace/delta.py`
  - _Requirements: 2.1, 2.2, 2.3_

- [x] 1.3 Fill coverage gaps in ace/playbook.py (currently 85.61%)
  - Add tests for playbook section management edge cases
  - Test bullet removal with non-existent IDs
  - Test playbook merging and conflict resolution
  - Achieve 90%+ coverage for `ace/playbook.py`
  - _Requirements: 2.1, 2.2, 2.3_

- [x] 1.4 Fill coverage gaps in ace/llm.py (currently 89.16%)
  - Add tests for TransformersLLMClient edge cases
  - Test JSON extraction with deeply nested objects
  - Test schema validation with complex schemas
  - Achieve 90%+ coverage for `ace/llm.py`
  - _Requirements: 2.1, 2.2, 2.3_

- [x] 1.5 Fill coverage gaps in ace/roles.py (currently 83.92%)
  - Add tests for Generator retry logic edge cases
  - Test Reflector refinement rounds
  - Test Curator delta generation with empty reflections
  - Achieve 90%+ coverage for `ace/roles.py`
  - _Requirements: 2.1, 2.2, 2.3_

- [x] 1.6 Fill coverage gaps in ace/versioning.py (currently 90.21%)
  - Add tests for branch merging edge cases
  - Test tag management with duplicate tags
  - Test history export/import with large histories
  - Achieve 95%+ coverage for `ace/versioning.py`
  - _Requirements: 2.1, 2.2, 2.3_

- [x] 1.7 Fill coverage gaps in ace/config.py (currently 82.76%)
  - Add tests for API key loading with missing environment variables
  - Test secret redaction with various formats
  - Test .env file loading with malformed files
  - Achieve 90%+ coverage for `ace/config.py`
  - _Requirements: 2.1, 2.2, 2.3_

- [x] 1.8 Fill coverage gaps in ace/deduplication.py (currently 93.26%)
  - Add tests for embedding cache eviction
  - Test similarity computation with edge cases
  - Test bullet merging with conflicting metadata
  - Achieve 95%+ coverage for `ace/deduplication.py`
  - _Requirements: 2.1, 2.2, 2.3_

- [x] 1.9 Run full test suite and verify 90% overall coverage
  - Execute `pytest tests/ --cov=ace --cov-report=term-missing`
  - Verify no critical modules below 85% coverage
  - Document remaining uncovered lines with justification
  - Update CHANGELOG.md with test coverage milestone
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [ ] 2. Implement Configuration-Based Workflow
  - Create high-level experiment runner API
  - Implement end-to-end examples
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5_

- [x] 2.1 Create experiment orchestration module
  - Create `ace/experiments.py` with `ExperimentRunner` class
  - Implement `run_experiment_from_config(config_path)` function
  - Integrate logging configuration from config
  - Integrate monitoring setup from config
  - Create components (LLM client, adapter, deduplicator) from config
  - Implement result saving with config metadata
  - Add `ExperimentResults` dataclass for structured results
  - _Requirements: 3.1, 3.2, 3.3_

- [x] 2.2 Create end-to-end QA example
  - Create `examples/qa_task/` directory
  - Create `examples/qa_task/config.yaml` with QA-specific settings
  - Create `examples/qa_task/run_qa.py` script using `run_experiment_from_config`
  - Create `examples/qa_task/data/` with sample QA dataset
  - Implement `QAEnvironment` for evaluation
  - Add README.md with setup and usage instructions
  - _Requirements: 5.1, 5.4, 5.5_

- [x] 2.3 Create end-to-end code generation example
  - Create `examples/code_generation/` directory
  - Create `examples/code_generation/config.yaml` with code-specific settings
  - Create `examples/code_generation/run_codegen.py` script
  - Create `examples/code_generation/data/` with sample coding tasks
  - Implement `CodeExecutionEnvironment` for evaluation
  - Add README.md with setup and usage instructions
  - _Requirements: 5.2, 5.4, 5.5_

- [x] 2.4 Create end-to-end classification example
  - Create `examples/classification/` directory
  - Create `examples/classification/config.yaml` with classification settings
  - Create `examples/classification/run_classification.py` script
  - Create `examples/classification/data/` with sample classification dataset
  - Implement `ClassificationEnvironment` for evaluation
  - Add README.md with setup and usage instructions
  - _Requirements: 5.3, 5.4, 5.5_

- [x] 2.5 Create config template generator
  - Implement `save_config_template(filepath)` function in `ace/settings.py` (already exists)
  - Add CLI command `python -m ace.config template` to generate template
  - Add comprehensive comments to generated template
  - Document all configuration options with examples
  - _Requirements: 3.4_

- [x] 2.6 Add integration tests for config-based workflow
  - Create `tests/test_experiments.py` with integration tests
  - Test `run_experiment_from_config` with DummyLLMClient
  - Test config validation error handling
  - Test result saving and loading
  - Verify all examples run successfully in CI
  - _Requirements: 3.1, 3.2, 3.5_

---

## 🔀 MVP BRANCH-OFF POINT

**NOTICE**: Tasks 1.1 through 2.6 above constitute the **MVP (Minimum Viable Product) scope**.

- **MVP Status**: ✅ COMPLETE - Ready for validation test run
- **MVP Document**: See [MVP-SCOPE.md](./MVP-SCOPE.md) for focused MVP checklist
- **What's Next**: 
  - For MVP validation: Use [MVP-SCOPE.md](./MVP-SCOPE.md)
  - For production implementation: Continue with Tasks 3+ below

**Tasks below this line are DEFERRED until after MVP validation.**

Rationale: MVP validates the core hypothesis ("ACE improves LLM performance through iterative context refinement"). Production infrastructure (deployment, optimization, streaming) should be built after validation confirms the approach works.

---

- [ ] 3. Create Production Deployment Documentation
  - Write comprehensive deployment guide
  - Create Docker and Kubernetes manifests
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

- [ ] 3.1 Create Dockerfile for ACE
  - Create `Dockerfile` with multi-stage build
  - Install all dependencies including optional packages
  - Copy source code and configs
  - Add health check endpoint
  - Optimize image size (target: <500MB)
  - Test Docker build locally
  - _Requirements: 4.1_

- [ ] 3.2 Create Kubernetes manifests
  - Create `k8s/deployment.yaml` with resource limits and health checks
  - Create `k8s/service.yaml` for metrics endpoint
  - Create `k8s/configmap.yaml` for configuration
  - Create `k8s/secret.yaml` template for API keys
  - Create `k8s/servicemonitor.yaml` for Prometheus integration
  - Add namespace and labels for organization
  - _Requirements: 4.2, 4.3_

- [ ] 3.3 Create Grafana dashboard
  - Create `monitoring/grafana-dashboard.json` with ACE metrics
  - Add panels for adaptation loop performance
  - Add panels for playbook growth and quality
  - Add panels for LLM API latency and cost
  - Add panels for error rates and types
  - Document dashboard import instructions
  - _Requirements: 6.2, 6.3, 6.4, 6.5_

- [ ] 3.4 Write deployment documentation
  - Create `docs/DEPLOYMENT.md` with step-by-step instructions
  - Document Docker build and run commands
  - Document Kubernetes deployment process
  - Document secrets management patterns
  - Document scaling guidelines for high-throughput
  - Document monitoring and alerting setup
  - Add troubleshooting section
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

- [ ] 3.5 Add health check endpoints
  - Create `ace/server.py` with Flask/FastAPI server
  - Implement `/health` endpoint for liveness probe
  - Implement `/ready` endpoint for readiness probe
  - Implement `/metrics` endpoint for Prometheus scraping
  - Add graceful shutdown handling
  - Test endpoints with curl/httpie
  - _Requirements: 4.2, 6.1_

- [ ] 4. Validate Security Audit Completeness
  - Review and enhance security tests
  - Document security limitations
  - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_

- [ ] 4.1 Review existing security tests
  - Run `pytest tests/test_security.py -v` and verify all 20 tests pass
  - Review test coverage for path traversal prevention
  - Review test coverage for JSON DoS prevention
  - Review test coverage for API key redaction
  - Identify any missing security test cases
  - _Requirements: 7.1, 7.2, 7.3, 7.5_

- [ ] 4.2 Enhance security documentation
  - Update `SECURITY.md` with known limitations
  - Document prompt injection risks and mitigations
  - Document log sanitization best practices
  - Add security checklist for production deployment
  - Document incident response procedures
  - _Requirements: 7.4_

- [ ] 4.3 Add security validation to CI/CD
  - Add Bandit security linter to GitHub Actions workflow
  - Add dependency vulnerability scanning with Safety
  - Add secret scanning with detect-secrets
  - Configure security test failure thresholds
  - Document security CI/CD pipeline
  - _Requirements: 7.5_

- [ ] 5. Optimize Performance Bottlenecks
  - Profile and optimize critical paths
  - Validate performance baselines
  - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_

- [ ] 5.1 Profile deduplication performance
  - Run performance benchmarks for deduplication with 200 bullets
  - Identify bottlenecks using cProfile or py-spy
  - Optimize embedding computation (batch processing, caching)
  - Optimize similarity computation (vectorized operations)
  - Verify performance meets <500ms target
  - _Requirements: 8.2_

- [ ] 5.2 Profile playbook serialization
  - Run performance benchmarks for serialization with 100 bullets
  - Identify bottlenecks in JSON encoding
  - Optimize serialization (use orjson if needed)
  - Verify performance meets <200ms target
  - _Requirements: 8.3_

- [ ] 5.3 Profile memory usage
  - Run memory profiling for 10,000 bullets
  - Identify memory leaks or excessive allocations
  - Optimize data structures (use __slots__ if needed)
  - Verify memory usage stays under 100MB
  - _Requirements: 8.4_

- [ ] 5.4 Run full performance regression suite
  - Execute `pytest tests/test_performance_baseline.py --benchmark-only`
  - Compare results against documented baselines
  - Identify any regressions exceeding 130% threshold
  - Document performance improvements in CHANGELOG.md
  - _Requirements: 8.1, 8.5_

- [ ] 6. Implement Streaming Adaptation Mode (Tier 2)
  - Refactor LLM clients for async
  - Implement StreamingAdapter
  - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_

- [ ] 6.1 Refactor LLMClient for async support
  - Add `async def complete_async(prompt, **kwargs)` to `LLMClient` base class
  - Implement async version in `DummyLLMClient` for testing
  - Implement async version in `TransformersLLMClient` using asyncio
  - Add async tests with pytest-asyncio
  - Maintain backward compatibility with sync `complete()` method
  - _Requirements: 9.1, 9.4_

- [ ] 6.2 Refactor roles for async support
  - Add `async def generate_async()` to `Generator` class
  - Add `async def reflect_async()` to `Reflector` class
  - Add `async def curate_async()` to `Curator` class
  - Implement concurrent LLM calls using `asyncio.gather()` where possible
  - Add async tests for each role
  - _Requirements: 9.1, 9.4_

- [ ] 6.3 Implement StreamingAdapter class
  - Create `ace/streaming.py` with `StreamingAdapter` class
  - Implement `async def run_streaming(samples, environment)` method
  - Implement `async def _process_sample_async(sample, environment)` method
  - Add immediate playbook updates after each sample
  - Implement backpressure handling with asyncio queues
  - Implement graceful cancellation with cleanup
  - _Requirements: 9.1, 9.2, 9.3_

- [ ] 6.4 Add streaming adapter tests
  - Create `tests/test_streaming.py` with async tests
  - Test streaming with AsyncIterator of samples
  - Test immediate playbook updates
  - Test backpressure handling
  - Test cancellation and cleanup
  - Verify no memory leaks with long-running streams
  - _Requirements: 9.4_

- [ ] 6.5 Benchmark streaming adapter latency
  - Add performance tests for streaming adapter
  - Measure playbook update latency
  - Measure end-to-end sample processing time
  - Verify sub-100ms latency target
  - Document streaming performance characteristics
  - _Requirements: 9.5_

- [ ] 7. Create Comprehensive API Documentation
  - Generate Sphinx documentation
  - Write API reference
  - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_

- [ ] 7.1 Setup Sphinx documentation
  - Install Sphinx and sphinx-rtd-theme
  - Create `docs/` directory with Sphinx configuration
  - Configure autodoc extension for API reference
  - Configure napoleon extension for Google/NumPy docstrings
  - Add Makefile for building docs
  - _Requirements: 10.5_

- [ ] 7.2 Enhance docstrings for all public APIs
  - Add comprehensive docstrings to all classes in `ace/playbook.py`
  - Add comprehensive docstrings to all classes in `ace/roles.py`
  - Add comprehensive docstrings to all classes in `ace/adaptation.py`
  - Add comprehensive docstrings to all classes in `ace/settings.py`
  - Include type hints, parameter descriptions, return values, and examples
  - _Requirements: 10.1, 10.2_

- [ ] 7.3 Create API reference pages
  - Create `docs/api/playbook.rst` with Playbook API reference
  - Create `docs/api/roles.rst` with Generator/Reflector/Curator API reference
  - Create `docs/api/adaptation.rst` with Adapter API reference
  - Create `docs/api/settings.rst` with Configuration API reference
  - Add usage examples for each major class
  - _Requirements: 10.1, 10.2, 10.3_

- [ ] 7.4 Create migration guide
  - Create `docs/MIGRATION.md` with version migration instructions
  - Document breaking changes between versions
  - Provide code examples for migrating from old to new APIs
  - Add deprecation warnings to old APIs
  - _Requirements: 10.4_

- [ ] 7.5 Build and publish documentation
  - Build Sphinx documentation with `make html`
  - Verify all pages render correctly
  - Add documentation build to CI/CD pipeline
  - Publish documentation to GitHub Pages or Read the Docs
  - Update README.md with documentation links
  - _Requirements: 10.5_

- [ ] 8. Final Integration and Release
  - Verify all tasks complete
  - Update documentation
  - Create release

- [ ] 8.1 Run complete test suite
  - Execute `pytest tests/ -v --cov=ace --cov-report=html`
  - Verify 90%+ overall coverage
  - Verify all 400+ tests pass
  - Review coverage report for any critical gaps
  - _Requirements: 2.5_

- [ ] 8.2 Update project documentation
  - Update README.md with new features
  - Update CHANGELOG.md with all changes
  - Update ROADMAP.md with completed tasks
  - Update ARCHITECTURE.md with new components
  - Update API.md with new APIs
  - _Requirements: 10.3_

- [ ] 8.3 Create release notes
  - Create `RELEASE_NOTES.md` for version 1.0
  - Summarize all new features
  - Document breaking changes
  - Provide upgrade instructions
  - Include performance improvements
  - _Requirements: 10.4_

- [ ] 8.4 Tag release and publish
  - Create git tag `v1.0.0`
  - Push tag to GitHub
  - Create GitHub release with notes
  - Publish package to PyPI (if applicable)
  - Announce release to community
  - _Requirements: All_
