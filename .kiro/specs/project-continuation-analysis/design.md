# ACE-Open Project Continuation Design

## Overview

This design document outlines the architecture and implementation strategy for continuing development of the ACE-Open framework. Based on comprehensive codebase analysis, the project is currently at **81.18% test coverage** with **368 passing tests** and **11/11 Tier 1 production readiness tasks complete**. The primary gap is the monitoring module (0% coverage) and achieving the 90% coverage target.

### Current State Assessment

**Strengths:**
- Excellent core architecture with clean separation of concerns
- Comprehensive test suite for critical components (adaptation, validation, deduplication)
- Production-ready features: secrets management, structured logging, input validation
- Well-documented codebase with API reference and architectural guides
- Performance baselines established with regression detection

**Gaps:**
- Monitoring module (164 lines) has 0% test coverage
- Overall coverage at 81.18% vs. 90% target
- No end-to-end integration examples
- Missing production deployment documentation
- Streaming adaptation mode not implemented (Tier 2 feature)

## Architecture

### System Context

```
┌─────────────────────────────────────────────────────────────┐
│                    ACE-Open Framework                        │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Generator   │  │  Reflector   │  │   Curator    │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
│         └──────────────────┴──────────────────┘              │
│                           │                                  │
│                    ┌──────▼───────┐                         │
│                    │   Playbook   │                         │
│                    └──────────────┘                         │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │           Monitoring & Observability                  │  │
│  │  - Prometheus Metrics                                 │  │
│  │  - Structured Logging                                 │  │
│  │  - Performance Tracking                               │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │           Configuration Management                    │  │
│  │  - YAML-based configs                                 │  │
│  │  - Pydantic validation                                │  │
│  │  - Environment overrides                              │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Component Architecture

#### 1. Monitoring Module Enhancement

**Current State:**
- 164 lines of code in `ace/monitoring.py`
- Comprehensive Prometheus metrics defined
- Decorator functions for instrumentation
- 0% test coverage

**Design:**
```python
# Test Architecture for Monitoring Module

class TestMonitoringDecorators:
    """Test decorator functions with mock metrics."""
    
    def test_monitor_adaptation_success(self):
        # Verify success metrics are recorded
        pass
    
    def test_monitor_adaptation_failure(self):
        # Verify failure metrics and error tracking
        pass
    
    def test_monitor_llm_call_latency(self):
        # Verify latency histogram updates
        pass
    
    def test_monitor_llm_call_token_tracking(self):
        # Verify token consumption counters
        pass

class TestMetricsUtilities:
    """Test utility functions for metric tracking."""
    
    def test_track_bullet_added(self):
        # Verify counter increments
        pass
    
    def test_update_playbook_size(self):
        # Verify gauge updates
        pass
    
    def test_track_json_parse_failure(self):
        # Verify error counter increments
        pass

class TestMetricsExport:
    """Test Prometheus metrics export."""
    
    def test_write_metrics_to_file(self):
        # Verify Prometheus format output
        pass
    
    def test_start_metrics_server(self):
        # Verify HTTP server starts correctly
        pass
```

**Key Design Decisions:**
- Use `unittest.mock` to mock Prometheus metrics objects
- Test decorators by wrapping dummy functions
- Verify metric values using `.labels().inc()` call counts
- Test file export by parsing generated Prometheus format

#### 2. Configuration-Based Workflow

**Current State:**
- Comprehensive `ace/settings.py` with Pydantic models (94.61% coverage)
- Factory functions for creating components from config
- YAML loading and validation implemented

**Design:**
```python
# High-Level Workflow API

def run_experiment_from_config(config_path: str) -> ExperimentResults:
    """
    Run complete ACE experiment from YAML configuration.
    
    Workflow:
    1. Load and validate config
    2. Create LLM client from config
    3. Create adapter with all components
    4. Configure logging and monitoring
    5. Run adaptation loop
    6. Save results with config metadata
    """
    config = load_config(config_path)
    
    # Configure observability
    configure_logging(
        level=config.logging.level.value,
        format=config.logging.format
    )
    
    # Create components
    llm_client = create_llm_client_from_config(config)
    adapter = create_adapter_from_config(config, llm_client=llm_client)
    deduplicator = create_deduplicator_from_config(config)
    
    # Run adaptation
    results = adapter.run(
        samples=load_samples(config.experiment.metadata.get('dataset')),
        environment=create_environment(config.experiment.metadata.get('task')),
        epochs=config.adaptation.epochs
    )
    
    # Save results with config
    save_results(results, config, output_dir=config.experiment.metadata.get('output_dir'))
    
    return results
```

**Integration Points:**
- `ace/settings.py`: Configuration models (already implemented)
- `ace/autoadapter.py`: High-level API (already implemented)
- New: `ace/experiments.py`: Experiment orchestration
- New: `examples/`: End-to-end examples

#### 3. Production Deployment Architecture

**Design:**
```yaml
# Kubernetes Deployment Pattern

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ace-adapter
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: ace
        image: ace-open:latest
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: ace-secrets
              key: openai-api-key
        ports:
        - containerPort: 8000  # Prometheus metrics
          name: metrics
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
```

**Components:**
- Dockerfile with multi-stage build
- Kubernetes manifests (Deployment, Service, ConfigMap, Secret)
- Prometheus ServiceMonitor for metrics scraping
- Grafana dashboard JSON for visualization
- Health check endpoints

#### 4. Streaming Adaptation Mode (Tier 2)

**Design:**
```python
# Async Streaming Architecture

class StreamingAdapter(AdapterBase):
    """
    Async streaming adapter for real-time learning.
    
    Key Features:
    - Async LLM calls with concurrent processing
    - Immediate playbook updates
    - Backpressure handling
    - Graceful cancellation
    """
    
    async def run_streaming(
        self,
        samples: AsyncIterator[Sample],
        environment: TaskEnvironment,
    ) -> AsyncIterator[AdapterStepResult]:
        """Process samples as they arrive."""
        async for sample in samples:
            # Process sample asynchronously
            result = await self._process_sample_async(sample, environment)
            
            # Update playbook immediately
            self.playbook.apply_delta(result.delta_batch)
            
            # Yield result for streaming response
            yield result
    
    async def _process_sample_async(
        self,
        sample: Sample,
        environment: TaskEnvironment
    ) -> AdapterStepResult:
        """Async version of sample processing."""
        # Concurrent LLM calls where possible
        output = await self.generator.generate_async(sample, self.playbook)
        result = environment.evaluate(sample, output)
        reflection = await self.reflector.reflect_async(
            sample, output, result, self.playbook
        )
        delta = await self.curator.curate_async([reflection], self.playbook)
        
        return AdapterStepResult(
            generator_output=output,
            environment_result=result,
            reflection=reflection,
            delta_batch=delta
        )
```

**Refactoring Required:**
- Make `LLMClient.complete()` async
- Add `async` versions of Generator, Reflector, Curator methods
- Implement async tests with `pytest-asyncio`
- Add backpressure and cancellation handling

## Components and Interfaces

### 1. Monitoring Test Suite

**Interface:**
```python
# tests/test_monitoring.py

class TestAdaptationMonitoring:
    """Test adaptation loop monitoring."""
    
    @pytest.fixture
    def mock_metrics(self):
        """Mock Prometheus metrics."""
        with patch('ace.monitoring.adaptation_runs_total') as mock:
            yield mock
    
    def test_monitor_adaptation_tracks_success(self, mock_metrics):
        """Verify success metrics are recorded."""
        @monitor_adaptation
        def dummy_adaptation(samples, env, epochs=1):
            return "success"
        
        result = dummy_adaptation([], None, epochs=5)
        
        mock_metrics.labels.assert_called_with(status='success', epochs='5')
        mock_metrics.labels().inc.assert_called_once()

class TestLLMMonitoring:
    """Test LLM call monitoring."""
    
    def test_monitor_llm_call_tracks_latency(self):
        """Verify latency histogram updates."""
        pass
    
    def test_monitor_llm_call_tracks_tokens(self):
        """Verify token consumption tracking."""
        pass

class TestMetricsExport:
    """Test Prometheus metrics export."""
    
    def test_write_metrics_to_file_creates_valid_format(self, tmp_path):
        """Verify Prometheus format output."""
        filepath = tmp_path / "metrics.prom"
        write_metrics_to_file(str(filepath))
        
        content = filepath.read_text()
        assert "# HELP" in content
        assert "# TYPE" in content
```

### 2. Configuration Workflow Interface

**Interface:**
```python
# ace/experiments.py

class ExperimentRunner:
    """High-level experiment orchestration."""
    
    def __init__(self, config: ACEConfig):
        self.config = config
        self.logger = get_logger(__name__)
    
    def run(self, samples: List[Sample], environment: TaskEnvironment) -> ExperimentResults:
        """Run complete experiment with monitoring."""
        # Setup
        self._configure_logging()
        self._configure_monitoring()
        
        # Create components
        llm_client = create_llm_client_from_config(self.config)
        adapter = create_adapter_from_config(self.config, llm_client=llm_client)
        
        # Run adaptation
        with self._monitor_experiment():
            results = adapter.run(
                samples=samples,
                environment=environment,
                epochs=self.config.adaptation.epochs
            )
        
        # Save results
        self._save_results(results)
        
        return ExperimentResults(
            config=self.config,
            results=results,
            metrics=self._collect_metrics()
        )
    
    def _configure_logging(self):
        """Configure structured logging from config."""
        configure_logging(
            level=self.config.logging.level.value,
            format=self.config.logging.format
        )
    
    def _configure_monitoring(self):
        """Start Prometheus metrics server if enabled."""
        if self.config.experiment.metadata.get('enable_metrics_server'):
            port = self.config.experiment.metadata.get('metrics_port', 8000)
            start_metrics_server(port)
```

### 3. Deployment Components

**Interface:**
```dockerfile
# Dockerfile

FROM python:3.11-slim as builder

WORKDIR /app

# Install dependencies
COPY pyproject.toml .
RUN pip install --no-cache-dir -e .[all]

# Copy source
COPY ace/ ace/
COPY configs/ configs/

FROM python:3.11-slim

WORKDIR /app

# Copy from builder
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /app /app

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD python -c "import requests; requests.get('http://localhost:8000/health')"

# Run
CMD ["python", "-m", "ace.server"]
```

## Data Models

### 1. Experiment Results Model

```python
@dataclass
class ExperimentResults:
    """Complete experiment results with metadata."""
    
    config: ACEConfig
    results: List[AdapterStepResult]
    metrics: Dict[str, Any]
    playbook_final: Playbook
    duration_seconds: float
    timestamp: str
    
    def to_dict(self) -> Dict[str, Any]:
        """Serialize to dictionary."""
        return {
            'config': self.config.to_dict(),
            'results': [r.to_dict() for r in self.results],
            'metrics': self.metrics,
            'playbook': self.playbook_final.to_dict(),
            'duration_seconds': self.duration_seconds,
            'timestamp': self.timestamp
        }
    
    def save(self, filepath: Path):
        """Save results to JSON file."""
        filepath.write_text(json.dumps(self.to_dict(), indent=2))
```

### 2. Monitoring Metrics Model

```python
@dataclass
class MonitoringSnapshot:
    """Snapshot of monitoring metrics."""
    
    adaptation_runs_total: int
    adaptation_success_rate: float
    llm_calls_total: int
    llm_avg_latency_seconds: float
    llm_total_tokens: int
    llm_estimated_cost_usd: float
    playbook_bullets_total: int
    playbook_bullets_added: int
    playbook_bullets_removed: int
    timestamp: str
    
    @classmethod
    def capture(cls) -> 'MonitoringSnapshot':
        """Capture current metrics state."""
        # Query Prometheus metrics
        return cls(
            adaptation_runs_total=get_metric_value('ace_adaptation_runs_total'),
            # ... other metrics
            timestamp=datetime.utcnow().isoformat()
        )
```

## Error Handling

### 1. Configuration Validation Errors

```python
try:
    config = load_config('configs/experiment.yaml')
except ValidationError as e:
    logger.error(
        "config_validation_failed",
        errors=e.errors(),
        config_path='configs/experiment.yaml'
    )
    # Provide user-friendly error message
    for error in e.errors():
        print(f"Error in {error['loc']}: {error['msg']}")
    sys.exit(1)
```

### 2. Monitoring Failures

```python
try:
    start_metrics_server(port=8000)
except Exception as e:
    logger.warning(
        "metrics_server_failed",
        port=8000,
        error=str(e),
        fallback="Continuing without metrics server"
    )
    # Continue execution without metrics
```

### 3. Streaming Adapter Errors

```python
async def run_streaming(self, samples, environment):
    try:
        async for sample in samples:
            try:
                result = await self._process_sample_async(sample, environment)
                yield result
            except asyncio.CancelledError:
                logger.info("streaming_cancelled", sample_id=sample.metadata.get('id'))
                raise
            except Exception as e:
                logger.error(
                    "sample_processing_failed",
                    sample_id=sample.metadata.get('id'),
                    error=str(e)
                )
                if self.fail_fast:
                    raise
                # Yield fallback result
                yield self._create_fallback_result(sample, e)
    except asyncio.CancelledError:
        logger.info("streaming_adapter_cancelled")
        # Cleanup resources
        await self._cleanup()
        raise
```

## Testing Strategy

### 1. Monitoring Module Tests

**Approach:**
- Mock Prometheus metrics objects using `unittest.mock`
- Test decorators by wrapping dummy functions
- Verify metric updates using call assertions
- Test file export by parsing output format

**Coverage Target:** 90%+ for `ace/monitoring.py`

### 2. Integration Tests

**Approach:**
- Create end-to-end tests using DummyLLMClient
- Test complete workflow from config to results
- Verify all components integrate correctly
- Test error handling and recovery

**Coverage Target:** 100% of critical paths

### 3. Performance Tests

**Approach:**
- Benchmark new features against baselines
- Ensure no regression in existing performance
- Test streaming adapter latency
- Profile memory usage

**Regression Threshold:** 130% of baseline

## Performance Considerations

### 1. Monitoring Overhead

**Concern:** Prometheus metrics collection adds latency

**Mitigation:**
- Use lightweight counter/gauge operations
- Avoid expensive computations in hot paths
- Make metrics collection optional via config
- Benchmark overhead (target: <1% impact)

### 2. Streaming Adapter Latency

**Concern:** Async overhead may increase latency

**Mitigation:**
- Use `asyncio.gather()` for concurrent LLM calls
- Implement connection pooling for API clients
- Cache embeddings for deduplication
- Target: <100ms playbook update latency

### 3. Configuration Loading

**Concern:** YAML parsing and validation adds startup time

**Mitigation:**
- Cache validated configs
- Lazy-load optional components
- Provide fast-path for programmatic configs
- Target: <100ms config loading time

## Security Considerations

### 1. Secrets in Configuration

**Risk:** API keys in YAML files

**Mitigation:**
- Use environment variable references in configs
- Validate that secrets are not hardcoded
- Redact secrets in logs and error messages
- Document secure configuration patterns

### 2. Monitoring Endpoint Security

**Risk:** Unauthenticated metrics endpoint

**Mitigation:**
- Bind metrics server to localhost by default
- Support authentication via reverse proxy
- Document security best practices
- Provide Kubernetes NetworkPolicy examples

### 3. Checkpoint File Security

**Risk:** Sensitive data in checkpoint files

**Mitigation:**
- Encrypt checkpoints at rest (optional)
- Validate checkpoint paths to prevent traversal
- Set restrictive file permissions
- Document data retention policies
