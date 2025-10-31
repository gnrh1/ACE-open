# ACE-open Framework: Comprehensive Architecture

## System Overview

The ACE (Adaptive Context Engine) framework implements a Generator-Evaluator-Reflector pattern with continuous learning through a structured playbook. This document provides complete architectural diagrams with detailed annotations.

---

## 1. High-Level Architecture: Three-Layer ACE Pattern

```mermaid
graph TB
    subgraph "ACE Framework Core"
        subgraph "Layer 1: Generation"
            G[Generator<br/>LLM-powered]
            PB1[(Playbook<br/>Read)]
        end
        
        subgraph "Layer 2: Evaluation"
            E[Task Environment<br/>Evaluator]
            GT[(Ground Truth<br/>Source)]
        end
        
        subgraph "Layer 3: Reflection & Learning"
            R[Reflector<br/>LLM-powered]
            C[Curator<br/>LLM-powered]
            PB2[(Playbook<br/>Write)]
        end
    end
    
    subgraph "External Dependencies"
        LLM[OpenAI/Anthropic API]
        EMB[Embedding Model<br/>sentence-transformers]
    end
    
    %% Data Flow
    Sample[Input Sample] --> G
    PB1 -.Guidance.-> G
    G -->|Generated Answer| E
    GT -.Ground Truth.-> E
    E -->|Feedback + Metrics| R
    R -->|Reflection| C
    C -->|Delta Operations| PB2
    PB2 -.Updated Knowledge.-> PB1
    
    %% External Calls
    G -.API Call.-> LLM
    R -.API Call.-> LLM
    C -.API Call.-> LLM
    C -.Deduplication.-> EMB
    
    %% Styling
    classDef llmComponent fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef storage fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef external fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    
    class G,R,C llmComponent
    class PB1,PB2,GT storage
    class LLM,EMB external
```

**Key Points:**
- **Generator**: Reads playbook, generates answers using LLM
- **Evaluator**: Compares output to ground truth, provides feedback
- **Reflector**: Analyzes performance, extracts lessons
- **Curator**: Transforms lessons into playbook updates
- **Playbook**: Persistent knowledge store (read by Generator, written by Curator)



---

## 2. Detailed Component Architecture with Data Flows

```mermaid
graph TB
    subgraph "Input Layer"
        S[Sample<br/>question + context<br/>+ ground_truth]
        ENV[TaskEnvironment<br/>Custom Evaluator]
    end
    
    subgraph "Adapter Orchestration"
        ADP[OfflineAdapter/OnlineAdapter<br/>Orchestrates ACE Loop]
        RC[Reflection Context<br/>Last 3 reflections]
    end
    
    subgraph "Generation Phase"
        GEN[Generator]
        GLLM[LLM Client<br/>OpenAI/Anthropic]
        GPROMPT[Generator Prompt<br/>+ Playbook<br/>+ Reflection Context]
        GOUT[GeneratorOutput<br/>reasoning<br/>final_answer<br/>bullet_ids]
    end
    
    subgraph "Evaluation Phase"
        EVAL[Environment.evaluate]
        EOUT[EnvironmentResult<br/>feedback<br/>ground_truth<br/>metrics]
    end
    
    subgraph "Reflection Phase"
        REF[Reflector]
        RLLM[LLM Client<br/>OpenAI/Anthropic]
        RPROMPT[Reflector Prompt<br/>+ Question<br/>+ Output<br/>+ Feedback]
        ROUT[ReflectorOutput<br/>reasoning<br/>error_identification<br/>root_cause<br/>correct_approach<br/>key_insight<br/>bullet_tags]
    end
    
    subgraph "Curation Phase"
        CUR[Curator]
        CLLM[LLM Client<br/>OpenAI/Anthropic]
        CPROMPT[Curator Prompt<br/>+ Reflection<br/>+ Playbook Stats]
        DEDUP[Deduplication<br/>Semantic Similarity]
        COUT[CuratorOutput<br/>DeltaBatch<br/>operations]
    end
    
    subgraph "Storage Layer"
        PB[(Playbook<br/>Bullets + Sections)]
        CKPT[(Checkpoints<br/>JSON Files)]
        CACHE[(Embedding Cache<br/>In-Memory Dict)]
    end
    
    %% Flow
    S --> ADP
    ENV --> ADP
    ADP --> GEN
    PB -.Read.-> GEN
    RC -.Context.-> GEN
    GEN --> GPROMPT
    GPROMPT --> GLLM
    GLLM --> GOUT
    
    GOUT --> EVAL
    S -.Ground Truth.-> EVAL
    EVAL --> EOUT
    
    EOUT --> REF
    GOUT -.Output.-> REF
    PB -.Bullets.-> REF
    REF --> RPROMPT
    RPROMPT --> RLLM
    RLLM --> ROUT
    
    ROUT --> RC
    ROUT --> CUR
    PB -.Stats.-> CUR
    CUR --> CPROMPT
    CPROMPT --> CLLM
    CLLM --> COUT
    
    COUT --> DEDUP
    DEDUP -.Check Similarity.-> CACHE
    DEDUP --> PB
    
    ADP -.Save State.-> CKPT
    
    %% Styling
    classDef input fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    classDef process fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef llm fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef storage fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef output fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    
    class S,ENV input
    class ADP,GEN,EVAL,REF,CUR,DEDUP process
    class GLLM,RLLM,CLLM llm
    class PB,CKPT,CACHE storage
    class GOUT,EOUT,ROUT,COUT output
```

**Component Responsibilities:**

1. **Adapter**: Orchestrates the entire loop, manages state, handles checkpointing
2. **Generator**: Creates answers using playbook guidance and reflection context
3. **Evaluator**: Task-specific evaluation logic (user-provided)
4. **Reflector**: Analyzes performance and extracts structured lessons
5. **Curator**: Transforms reflections into playbook delta operations
6. **Deduplication**: Prevents redundant bullets using semantic similarity



---

## 3. LLM API Integration Points

```mermaid
sequenceDiagram
    participant A as Adapter
    participant G as Generator
    participant R as Reflector
    participant C as Curator
    participant API as OpenAI/Anthropic API
    participant PB as Playbook
    
    Note over A: Process Single Sample
    
    A->>G: generate(question, context, playbook)
    G->>PB: as_prompt() - Read bullets
    PB-->>G: Formatted playbook text
    G->>API: complete(prompt) - API Call #1
    Note over API: GPT-4o-mini generates answer
    API-->>G: JSON response
    G-->>A: GeneratorOutput
    
    A->>A: environment.evaluate()
    Note over A: Compare to ground truth
    
    A->>R: reflect(output, feedback, ground_truth)
    R->>PB: get_bullet(ids) - Read referenced bullets
    PB-->>R: Bullet details
    R->>API: complete(prompt) - API Call #2
    Note over API: GPT-4o-mini analyzes performance
    API-->>R: JSON response
    R-->>A: ReflectorOutput
    
    A->>C: curate(reflection, playbook)
    C->>PB: stats() - Read playbook statistics
    PB-->>C: Bullet counts, tags
    C->>API: complete(prompt) - API Call #3
    Note over API: GPT-4o-mini creates delta ops
    API-->>C: JSON response
    C-->>A: CuratorOutput (DeltaBatch)
    
    A->>PB: apply_delta(operations)
    Note over PB: Add/Update/Remove bullets
    
    Note over A: 3 API calls per sample<br/>Cost: ~$0.001 per sample
```

**API Call Breakdown:**
- **Call #1 (Generator)**: Generate answer based on question + playbook
- **Call #2 (Reflector)**: Analyze performance and extract lessons
- **Call #3 (Curator)**: Transform reflection into playbook operations

**Cost Analysis** (GPT-4o-mini):
- Input tokens: ~1,500 per call (playbook + prompts)
- Output tokens: ~500 per call (JSON responses)
- Total per sample: ~6,000 tokens = $0.001
- 1,000 samples: ~$1.00



---

## 4. Data Architecture & Storage

```mermaid
graph TB
    subgraph "In-Memory Storage"
        PB[Playbook Object<br/>_bullets: Dict[str, Bullet]<br/>_sections: Dict[str, List[str]]<br/>_next_id: int]
        RC[Reflection Context<br/>List[str] - Last 3 reflections]
        ECACHE[Embedding Cache<br/>Dict[str, np.ndarray]<br/>Global cache for deduplication]
    end
    
    subgraph "Persistent Storage - File System"
        CKPT[Checkpoint Files<br/>checkpoint_epoch1_step1.json<br/>checkpoint_epoch1_step2.json<br/>...]
        LOGS[Log Files<br/>qa_experiment_*.log<br/>json_failures.log]
        RESULTS[Results Files<br/>experiment_results.json]
    end
    
    subgraph "External Storage - Not Implemented"
        VDB[(Vector Database<br/>OPTIONAL<br/>Pinecone/Weaviate/Chroma)]
        CACHE_DB[(Redis/Memcached<br/>OPTIONAL<br/>Distributed cache)]
    end
    
    subgraph "Data Structures"
        BULLET[Bullet<br/>id: str<br/>section: str<br/>content: str<br/>helpful: int<br/>harmful: int<br/>neutral: int<br/>created_at: str<br/>updated_at: str]
        
        SAMPLE[Sample<br/>question: str<br/>context: str<br/>ground_truth: Optional[str]<br/>metadata: Dict]
        
        ENVRES[EnvironmentResult<br/>feedback: str<br/>ground_truth: Optional[str]<br/>metrics: Dict[str, float]]
    end
    
    %% Relationships
    PB -->|Contains| BULLET
    PB -.Serialized to.-> CKPT
    CKPT -.Restored to.-> PB
    
    SAMPLE -.Provides.-> ENVRES
    
    ECACHE -.Used by.-> PB
    
    PB -.Could use.-> VDB
    ECACHE -.Could use.-> CACHE_DB
    
    %% Styling
    classDef memory fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef disk fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef optional fill:#f5f5f5,stroke:#9e9e9e,stroke-width:2px,stroke-dasharray: 5 5
    classDef struct fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    
    class PB,RC,ECACHE memory
    class CKPT,LOGS,RESULTS disk
    class VDB,CACHE_DB optional
    class BULLET,SAMPLE,ENVRES struct
```

**Storage Details:**

### In-Memory Storage
- **Playbook**: Primary knowledge store, held in RAM during execution
- **Reflection Context**: Sliding window of last 3 reflections for context
- **Embedding Cache**: Global dictionary caching sentence embeddings for deduplication

### Persistent Storage (File System)
- **Checkpoints**: JSON snapshots of playbook state after each sample
  - Format: `checkpoint_epoch{N}_step{M}.json`
  - Contains: playbook dict, epoch, step, timestamp, recent reflections
  - Used for: Resume from failure, analysis, debugging

- **Logs**: Structured logging output
  - `qa_experiment_*.log`: Experiment execution logs
  - `json_failures.log`: Failed JSON parsing attempts for debugging

- **Results**: Final experiment outputs
  - `experiment_results.json`: Complete results with config, metrics, playbook

### External Storage (Optional - Not Currently Implemented)
- **Vector Database**: Could store bullet embeddings for semantic search
- **Distributed Cache**: Could share embeddings across multiple processes



---

## 5. Ground Truth Handling - CRITICAL CLARIFICATION

```mermaid
graph TB
    subgraph "Training/Evaluation Mode - WITH Ground Truth"
        S1[Sample<br/>question: 'What is 2+2?'<br/>context: '...'<br/>ground_truth: '4']
        
        G1[Generator<br/>Generates: '4']
        
        E1[TaskEnvironment.evaluate<br/>Compares output to ground_truth]
        
        ER1[EnvironmentResult<br/>feedback: '✓ Correct!'<br/>ground_truth: '4'<br/>metrics: correct=True, score=1.0]
        
        R1[Reflector<br/>Analyzes: Why correct?<br/>What patterns worked?]
        
        S1 --> G1
        G1 --> E1
        S1 -.Provides ground_truth.-> E1
        E1 --> ER1
        ER1 --> R1
        
        style S1 fill:#c8e6c9,stroke:#2e7d32,stroke-width:3px
        style E1 fill:#fff9c4,stroke:#f57f17,stroke-width:3px
        style ER1 fill:#b2dfdb,stroke:#00695c,stroke-width:3px
    end
    
    subgraph "Production/Inference Mode - WITHOUT Ground Truth"
        S2[Sample<br/>question: 'What is 2+2?'<br/>context: '...'<br/>ground_truth: None]
        
        G2[Generator<br/>Generates: '4']
        
        E2[TaskEnvironment.evaluate<br/>NO ground truth available<br/>Can only provide heuristic feedback]
        
        ER2[EnvironmentResult<br/>feedback: 'Answer generated'<br/>ground_truth: None<br/>metrics: confidence=0.8]
        
        R2[Reflector<br/>Limited analysis<br/>No correctness feedback]
        
        S2 --> G2
        G2 --> E2
        S2 -.ground_truth=None.-> E2
        E2 --> ER2
        ER2 --> R2
        
        style S2 fill:#ffccbc,stroke:#d84315,stroke-width:3px
        style E2 fill:#ffecb3,stroke:#ff6f00,stroke-width:3px
        style ER2 fill:#d7ccc8,stroke:#5d4037,stroke-width:3px
    end
    
    subgraph "Ground Truth Sources"
        GT1[Pre-labeled Dataset<br/>qa_samples.json<br/>Each sample has ground_truth]
        GT2[Human Annotation<br/>Expert provides correct answer<br/>After generation]
        GT3[Automated Oracle<br/>Code execution, API call<br/>Deterministic verification]
        GT4[User Feedback<br/>Thumbs up/down<br/>Corrections]
        GT5[No Ground Truth<br/>Production inference<br/>No learning possible]
        
        GT1 -.Used in.-> S1
        GT2 -.Could provide.-> S1
        GT3 -.Could provide.-> S1
        GT4 -.Could provide.-> S1
        GT5 -.Results in.-> S2
    end
```

### Ground Truth Modes Explained

#### Mode 1: Training/Evaluation (WITH Ground Truth)
**When**: Offline training, benchmarking, testing
**Ground Truth Source**: 
- Pre-labeled datasets (e.g., `qa_samples.json`)
- Human annotations
- Automated oracles (code execution, API verification)
- Historical data with known correct answers

**Behavior**:
- Evaluator compares generated output to ground truth
- Provides accurate feedback ("✓ Correct" or "✗ Incorrect")
- Reflector can analyze WHY answer was right/wrong
- Playbook learns from mistakes
- Metrics are meaningful (accuracy, exact match, etc.)

**Example**:
```python
sample = Sample(
    question="What is the capital of France?",
    context="France is a country in Western Europe...",
    ground_truth="Paris"  # ← Ground truth provided
)
# Evaluator can verify: output == "Paris" → Correct!
```



#### Mode 2: Production/Inference (WITHOUT Ground Truth)
**When**: Real-world deployment, user-facing applications
**Ground Truth Source**: None (or delayed/partial feedback)

**Behavior**:
- Evaluator CANNOT verify correctness
- Can only provide heuristic feedback:
  - "Answer generated successfully"
  - Confidence scores (if available)
  - Format validation
  - Consistency checks
- Reflector has limited learning capability
- Playbook growth is minimal or disabled
- Metrics are heuristic (confidence, not accuracy)

**Example**:
```python
sample = Sample(
    question="What will the stock market do tomorrow?",
    context="Current market conditions...",
    ground_truth=None  # ← No ground truth available
)
# Evaluator cannot verify correctness
# Can only check: "Answer is well-formed"
```

**Production Strategies**:
1. **No Learning**: Disable adaptation, use pre-trained playbook
2. **Delayed Learning**: Collect user feedback, retrain offline later
3. **Heuristic Learning**: Learn from proxy signals (user engagement, consistency)
4. **Human-in-Loop**: Route uncertain cases to human review

#### Mode 3: Hybrid (Partial Ground Truth)
**When**: Production with feedback mechanisms
**Ground Truth Source**: User feedback, delayed verification

**Behavior**:
- Initial inference without ground truth
- User provides feedback (thumbs up/down, corrections)
- Feedback becomes "ground truth" for learning
- Playbook updates based on user corrections

**Example**:
```python
# Step 1: Generate without ground truth
sample = Sample(question="...", ground_truth=None)
output = generator.generate(sample)

# Step 2: User provides feedback
user_feedback = "Incorrect, should be X"

# Step 3: Create learning sample
learning_sample = Sample(
    question=sample.question,
    ground_truth="X"  # ← User-provided ground truth
)
# Now can learn from this feedback
```

### Key Insight: Ground Truth is ONLY for Training

**IMPORTANT**: Ground truth is NOT required for the Generator to work. It's ONLY needed for:
1. **Evaluation**: Measuring performance
2. **Reflection**: Learning from mistakes
3. **Playbook Growth**: Improving over time

**In production**:
- Generator works fine without ground truth
- Evaluator provides generic feedback
- Reflection/learning is disabled or limited
- Use pre-trained playbook from offline training



---

## 6. Embedding Generation & Deduplication Flow

```mermaid
graph TB
    subgraph "Curator Output Processing"
        COUT[CuratorOutput<br/>DeltaBatch with operations]
        OPS[Delta Operations<br/>ADD, UPDATE, REMOVE, TAG]
    end
    
    subgraph "Deduplication Pipeline"
        CHECK{Deduplication<br/>Enabled?}
        NEW[New Bullet Content<br/>'Maintain factual knowledge...']
        
        EMB1[Generate Embedding<br/>sentence-transformers]
        CACHE{In Cache?}
        MODEL[SentenceTransformer<br/>'all-MiniLM-L6-v2']
        
        EXISTING[Existing Bullets<br/>in Same Section]
        EMB2[Get Embeddings<br/>from Cache/Generate]
        
        SIM[Compute Cosine<br/>Similarity]
        THRESH{Similarity ≥<br/>Threshold?<br/>Default: 0.85}
        
        MERGE[Merge Bullets<br/>Combine metadata<br/>Keep longest content]
        ADD[Add New Bullet<br/>to Playbook]
    end
    
    subgraph "Storage"
        ECACHE[(Embedding Cache<br/>MD5 hash → embedding<br/>In-Memory Dict)]
        PB[(Playbook<br/>Bullets + Sections)]
    end
    
    %% Flow
    COUT --> OPS
    OPS --> CHECK
    CHECK -->|Yes| NEW
    CHECK -->|No| ADD
    
    NEW --> EMB1
    EMB1 --> CACHE
    CACHE -->|Hit| EMB1
    CACHE -->|Miss| MODEL
    MODEL --> ECACHE
    ECACHE --> EMB1
    
    EMB1 --> SIM
    EXISTING --> EMB2
    EMB2 --> ECACHE
    EMB2 --> SIM
    
    SIM --> THRESH
    THRESH -->|Similar| MERGE
    THRESH -->|Unique| ADD
    
    MERGE --> PB
    ADD --> PB
    
    %% Styling
    classDef process fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef decision fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef storage fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef ml fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    
    class NEW,EMB1,EMB2,SIM,MERGE,ADD process
    class CHECK,CACHE,THRESH decision
    class ECACHE,PB storage
    class MODEL ml
```

**Deduplication Process:**

1. **Trigger**: When Curator creates ADD operation for new bullet
2. **Embedding Generation**: 
   - Compute sentence embedding for new bullet content
   - Use lightweight model: `all-MiniLM-L6-v2` (384 dimensions)
   - Cache embeddings using MD5 hash as key
3. **Similarity Check**:
   - Compare new bullet to all existing bullets in same section
   - Use cosine similarity on embeddings
   - Threshold: 0.85 (85% similar = duplicate)
4. **Merge or Add**:
   - If similar bullet found: Merge (keep longest content, sum metadata)
   - If unique: Add as new bullet

**Performance Optimization:**
- **Caching**: Embeddings cached in memory (global dict)
- **Batch Processing**: Can process multiple bullets at once
- **Section Filtering**: Only compare within same section
- **Lazy Loading**: Model loaded on first use

**Example**:
```python
# Bullet 1: "Maintain factual knowledge for accuracy"
# Bullet 2: "Maintaining factual knowledge is crucial"
# Similarity: 0.92 → MERGE

# Bullet 3: "Precision in terminology is important"
# Similarity to 1: 0.45 → ADD as new bullet
```



---

## 7. Complete Adaptation Loop - Single Sample

```mermaid
sequenceDiagram
    autonumber
    participant S as Sample
    participant A as Adapter
    participant G as Generator
    participant E as Environment
    participant R as Reflector
    participant C as Curator
    participant D as Deduplicator
    participant PB as Playbook
    participant API as LLM API
    participant FS as File System
    
    Note over S,FS: Process Single Sample (e.g., "What is 2+2?")
    
    S->>A: Input sample with ground_truth
    
    rect rgb(230, 245, 255)
        Note over A,PB: GENERATION PHASE
        A->>PB: Read playbook bullets
        PB-->>A: Formatted playbook text
        A->>G: generate(question, context, playbook)
        G->>API: complete(prompt) [API Call #1]
        API-->>G: JSON: {reasoning, final_answer, bullet_ids}
        G-->>A: GeneratorOutput
    end
    
    rect rgb(255, 243, 224)
        Note over A,E: EVALUATION PHASE
        A->>E: evaluate(sample, output)
        E->>E: Compare output to ground_truth
        E-->>A: EnvironmentResult(feedback, metrics)
    end
    
    rect rgb(232, 245, 233)
        Note over A,R: REFLECTION PHASE
        A->>PB: Get referenced bullets
        PB-->>A: Bullet details
        A->>R: reflect(output, feedback, ground_truth)
        R->>API: complete(prompt) [API Call #2]
        API-->>R: JSON: {reasoning, errors, insights, tags}
        R-->>A: ReflectorOutput
        A->>A: Update reflection context (sliding window)
        A->>PB: Apply bullet tags (helpful/harmful)
    end
    
    rect rgb(243, 229, 245)
        Note over A,C: CURATION PHASE
        A->>PB: Get playbook stats
        PB-->>A: Bullet counts, sections
        A->>C: curate(reflection, playbook, context)
        C->>API: complete(prompt) [API Call #3]
        API-->>C: JSON: {operations: [ADD, UPDATE, ...]}
        C-->>A: CuratorOutput(DeltaBatch)
    end
    
    rect rgb(255, 249, 196)
        Note over A,D: DEDUPLICATION PHASE
        A->>D: Check for duplicate bullets
        D->>D: Generate embeddings
        D->>D: Compute similarities
        D->>PB: Merge or add bullets
    end
    
    rect rgb(255, 235, 238)
        Note over A,FS: CHECKPOINT PHASE
        A->>FS: Save checkpoint JSON
        Note over FS: checkpoint_epoch1_step1.json
    end
    
    A-->>S: AdapterStepResult (complete)
    
    Note over S,FS: Total: 3 API calls, ~6000 tokens, ~$0.001
```

**Step-by-Step Breakdown:**

1. **Input**: Sample with question, context, and ground_truth
2. **Read Playbook**: Get current knowledge base
3. **Generate**: LLM creates answer using playbook guidance (API Call #1)
4. **Evaluate**: Compare output to ground truth, calculate metrics
5. **Reflect**: LLM analyzes performance and extracts lessons (API Call #2)
6. **Update Context**: Add reflection to sliding window (last 3)
7. **Tag Bullets**: Mark referenced bullets as helpful/harmful
8. **Get Stats**: Retrieve playbook statistics
9. **Curate**: LLM transforms reflection into operations (API Call #3)
10. **Deduplicate**: Check for similar bullets, merge if needed
11. **Update Playbook**: Apply delta operations (add/update/remove)
12. **Checkpoint**: Save state to disk
13. **Return**: Complete result with all outputs

**Timing** (approximate):
- Generation: 2-3 seconds
- Evaluation: <0.1 seconds
- Reflection: 2-3 seconds
- Curation: 2-3 seconds
- Deduplication: 0.5-1 seconds
- **Total**: ~8-10 seconds per sample



---

## 8. Multi-Epoch Training Architecture

```mermaid
graph TB
    subgraph "Epoch 1: Initial Learning"
        E1S[10 Samples]
        E1PB0[Playbook: 0 bullets]
        E1LOOP[Adaptation Loop<br/>10 iterations]
        E1PB1[Playbook: 23 bullets<br/>Accuracy: 90%]
        
        E1S --> E1LOOP
        E1PB0 -.Initial State.-> E1LOOP
        E1LOOP --> E1PB1
    end
    
    subgraph "Epoch 2: Refinement"
        E2S[Same 10 Samples<br/>Shuffled order]
        E2PB1[Playbook: 23 bullets<br/>From Epoch 1]
        E2LOOP[Adaptation Loop<br/>10 iterations<br/>WITH playbook guidance]
        E2PB2[Playbook: 35 bullets<br/>Accuracy: 95%]
        
        E2S --> E2LOOP
        E2PB1 -.Guides Generation.-> E2LOOP
        E2LOOP --> E2PB2
    end
    
    subgraph "Epoch 3: Fine-tuning"
        E3S[Same 10 Samples<br/>Shuffled again]
        E3PB2[Playbook: 35 bullets<br/>From Epoch 2]
        E3LOOP[Adaptation Loop<br/>10 iterations<br/>WITH refined playbook]
        E3PB3[Playbook: 40 bullets<br/>Accuracy: 98%]
        
        E3S --> E3LOOP
        E3PB2 -.Guides Generation.-> E3LOOP
        E3LOOP --> E3PB3
    end
    
    E1PB1 -.Carries Forward.-> E2PB1
    E2PB2 -.Carries Forward.-> E3PB2
    
    subgraph "Configuration"
        CFG[config.yaml<br/>epochs: 3<br/>shuffle_samples: true<br/>checkpoint_enabled: true]
    end
    
    CFG -.Controls.-> E1LOOP
    CFG -.Controls.-> E2LOOP
    CFG -.Controls.-> E3LOOP
    
    %% Styling
    classDef epoch1 fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    classDef epoch2 fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px
    classDef epoch3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    classDef config fill:#fff3e0,stroke:#e65100,stroke-width:2px
    
    class E1S,E1PB0,E1LOOP,E1PB1 epoch1
    class E2S,E2PB1,E2LOOP,E2PB2 epoch2
    class E3S,E3PB2,E3LOOP,E3PB3 epoch3
    class CFG config
```

**Multi-Epoch Training Explained:**

### Epoch 1: Initial Learning (Baseline)
- **Playbook State**: Empty (0 bullets)
- **Generator Behavior**: Uses only base LLM knowledge
- **Learning**: Discovers fundamental patterns
- **Result**: 23 bullets, 90% accuracy
- **Bullets Created**: Basic patterns like "maintain factual knowledge"

### Epoch 2: Guided Improvement
- **Playbook State**: 23 bullets from Epoch 1
- **Generator Behavior**: Reads playbook before answering
- **Learning**: Refines existing patterns, adds edge cases
- **Result**: 35 bullets (+12), 95% accuracy (+5%)
- **Improvement**: Fixes photosynthesis error using "precision in terminology" bullet

### Epoch 3: Fine-tuning
- **Playbook State**: 35 bullets from Epoch 2
- **Generator Behavior**: Uses refined playbook
- **Learning**: Minimal new patterns, mostly reinforcement
- **Result**: 40 bullets (+5), 98% accuracy (+3%)
- **Improvement**: Handles rare cases, optimizes phrasing

**Key Mechanisms:**

1. **Sample Shuffling**: Prevents order-dependent learning
2. **Playbook Persistence**: Knowledge carries forward between epochs
3. **Checkpointing**: State saved after each sample for recovery
4. **Deduplication**: Prevents redundant bullets across epochs

**Configuration**:
```yaml
adaptation:
  mode: "offline"
  epochs: 3
  shuffle_samples: true
  checkpoint_enabled: true
  checkpoint_frequency: 1  # Save after each sample
```



---

## 9. Architectural Patterns & Design Decisions

### Pattern 1: Feedback Loop Architecture

```mermaid
graph LR
    A[Action<br/>Generator] --> B[Observation<br/>Environment]
    B --> C[Reflection<br/>Reflector]
    C --> D[Learning<br/>Curator]
    D --> E[Memory<br/>Playbook]
    E -.Guides.-> A
    
    style A fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style B fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style C fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style D fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style E fill:#fff9c4,stroke:#f57f17,stroke-width:2px
```

**Pattern**: Observe-Orient-Decide-Act (OODA) Loop
- **Observe**: Environment evaluates output
- **Orient**: Reflector analyzes performance
- **Decide**: Curator determines updates
- **Act**: Generator uses updated playbook

### Pattern 2: Pipeline Architecture

```
Input → Generator → Evaluator → Reflector → Curator → Output
         ↑                                              ↓
         └──────────────── Playbook ←───────────────────┘
```

**Characteristics**:
- Sequential processing
- Each stage has single responsibility
- Playbook provides feedback loop
- Stateful (playbook persists across samples)

### Pattern 3: Agent Architecture

```
Agent = Generator + Playbook (Memory)
Environment = Evaluator
Trainer = Reflector + Curator
```

**Characteristics**:
- Agent learns from environment feedback
- Memory (playbook) improves agent over time
- Trainer extracts and stores lessons

---

## Key Design Decisions

### Decision 1: Synchronous vs Asynchronous

**Current**: **Synchronous** (blocking)
- Each sample processed sequentially
- Wait for LLM responses before continuing
- Simple, predictable, easy to debug

**Alternative**: Asynchronous (non-blocking)
- Process multiple samples in parallel
- Batch LLM API calls
- Complex state management

**Rationale**: Synchronous chosen for MVP simplicity. Async can be added later for scale.



### Decision 2: Stateful vs Stateless Components

**Stateful Components**:
- **Playbook**: Accumulates knowledge across samples
- **Adapter**: Maintains reflection context (last 3)
- **Embedding Cache**: Stores computed embeddings

**Stateless Components**:
- **Generator**: No state between calls
- **Reflector**: No state between calls
- **Curator**: No state between calls
- **Evaluator**: No state between calls

**Rationale**: 
- Roles (Generator/Reflector/Curator) are stateless for testability
- Adapter manages all state for clear separation of concerns
- Playbook is the only persistent knowledge store

### Decision 3: In-Memory vs Persistent Storage

**In-Memory**:
- Playbook (during execution)
- Reflection context
- Embedding cache

**Persistent**:
- Checkpoints (JSON files)
- Logs (text files)
- Results (JSON files)

**Rationale**:
- In-memory for performance during execution
- Persistent for recovery, analysis, and reproducibility
- No database required for MVP (file system sufficient)

### Decision 4: Centralized vs Distributed

**Current**: **Centralized** (single process)
- All components in one Python process
- Shared memory for playbook and cache
- File system for persistence

**Alternative**: Distributed (multiple processes/machines)
- Vector database for playbook (Pinecone, Weaviate)
- Redis for embedding cache
- Message queue for coordination

**Rationale**: Centralized sufficient for MVP scale (1000s of samples). Distributed can be added for production scale (millions of samples).

### Decision 5: LLM Provider Abstraction

**Design**: Abstract `LLMClient` interface
- Supports OpenAI, Anthropic, Transformers
- Easy to add new providers
- Consistent API across providers

**Benefits**:
- Provider-agnostic framework
- Easy testing with `DummyLLMClient`
- Cost optimization (switch providers)

**Implementation**:
```python
class LLMClient(ABC):
    @abstractmethod
    def complete(self, prompt: str) -> LLMResponse:
        pass
```



### Decision 6: Error Handling Strategy

**Approach**: Fail-fast with fallback option

**Configuration**:
```yaml
adaptation:
  fail_fast: true  # Raise exception on error
  # OR
  fail_fast: false  # Continue with fallback result
```

**Fail-Fast Mode** (default):
- Exception raised on any error
- Execution stops immediately
- Best for development and debugging

**Fallback Mode**:
- Errors logged but execution continues
- Fallback result created with error details
- Best for production resilience

**Error Types Handled**:
- LLM API failures (timeout, rate limit)
- JSON parsing errors (malformed output)
- Evaluation errors (missing ground truth)
- Deduplication errors (embedding failures)

### Decision 7: Scalability Considerations

**Current Scale** (MVP):
- 10-1,000 samples per experiment
- Single machine, single process
- File system storage
- In-memory playbook

**Future Scale** (Production):
- 100,000+ samples per experiment
- Distributed processing
- Vector database for playbook
- Distributed cache for embeddings

**Scaling Path**:
1. **Vertical**: Increase machine resources (RAM, CPU)
2. **Horizontal**: Distribute across multiple machines
3. **Optimization**: Batch API calls, cache aggressively
4. **Infrastructure**: Add vector DB, message queue, distributed cache

**Bottlenecks**:
- LLM API rate limits (primary bottleneck)
- Embedding computation (secondary)
- Playbook size (grows linearly with samples)

**Mitigation**:
- Batch API calls (reduce latency)
- Aggressive deduplication (limit playbook growth)
- Caching (reduce redundant computation)
- Async processing (improve throughput)



---

## 10. Deployment Architecture Options

### Option A: Research/Development (Current MVP)

```mermaid
graph TB
    subgraph "Single Machine"
        SCRIPT[Python Script<br/>run_qa.py]
        ACE[ACE Framework<br/>In-Memory]
        FS[(File System<br/>Checkpoints + Logs)]
    end
    
    subgraph "External Services"
        OPENAI[OpenAI API<br/>GPT-4o-mini]
        HF[HuggingFace<br/>sentence-transformers]
    end
    
    SCRIPT --> ACE
    ACE --> FS
    ACE -.API Calls.-> OPENAI
    ACE -.Model Download.-> HF
    
    style SCRIPT fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style ACE fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style FS fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style OPENAI fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style HF fill:#fff9c4,stroke:#f57f17,stroke-width:2px
```

**Characteristics**:
- Single Python process
- Local file system storage
- Direct API calls to OpenAI
- No infrastructure required

**Use Cases**:
- Research experiments
- Prototyping
- Small datasets (<10,000 samples)
- Development and testing

---

### Option B: Production Inference (No Learning)

```mermaid
graph TB
    subgraph "Application Server"
        API[REST API<br/>FastAPI/Flask]
        GEN[Generator Only<br/>No Reflector/Curator]
        PB[(Pre-trained Playbook<br/>Read-Only)]
    end
    
    subgraph "External Services"
        LLM[LLM API<br/>OpenAI/Anthropic]
    end
    
    subgraph "Clients"
        WEB[Web App]
        MOBILE[Mobile App]
        CLI[CLI Tool]
    end
    
    WEB --> API
    MOBILE --> API
    CLI --> API
    
    API --> GEN
    GEN --> PB
    GEN -.API Call.-> LLM
    
    style API fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style GEN fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style PB fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style LLM fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
```

**Characteristics**:
- Generator only (no learning loop)
- Pre-trained playbook loaded at startup
- Fast inference (<3 seconds per request)
- Stateless API (horizontally scalable)

**Use Cases**:
- User-facing applications
- Real-time inference
- No ground truth available
- High throughput required



---

### Option C: Production with Continuous Learning

```mermaid
graph TB
    subgraph "Inference Tier"
        API[REST API<br/>FastAPI]
        GEN[Generator<br/>Read-Only Playbook]
        CACHE[(Redis Cache<br/>Embeddings)]
    end
    
    subgraph "Feedback Collection"
        QUEUE[Message Queue<br/>RabbitMQ/Kafka]
        FB[(Feedback DB<br/>PostgreSQL)]
    end
    
    subgraph "Training Tier"
        WORKER[Background Worker<br/>Celery]
        FULL[Full ACE Loop<br/>Generator + Reflector + Curator]
        VDB[(Vector DB<br/>Pinecone/Weaviate<br/>Playbook Storage)]
    end
    
    subgraph "External Services"
        LLM[LLM API]
        EMB[Embedding API]
    end
    
    subgraph "Clients"
        USER[Users]
    end
    
    USER -->|Request| API
    API --> GEN
    GEN --> CACHE
    GEN -.API Call.-> LLM
    API -->|Response| USER
    
    USER -->|Feedback| QUEUE
    QUEUE --> FB
    
    FB --> WORKER
    WORKER --> FULL
    FULL --> VDB
    FULL -.API Calls.-> LLM
    FULL -.Embeddings.-> EMB
    
    VDB -.Sync Playbook.-> GEN
    
    style API fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style GEN fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style FULL fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style VDB fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style QUEUE fill:#fff9c4,stroke:#f57f17,stroke-width:2px
```

**Characteristics**:
- Separate inference and training tiers
- Inference uses read-only playbook (fast)
- User feedback collected asynchronously
- Background workers retrain periodically
- Playbook synced from vector DB

**Use Cases**:
- Production with user feedback
- Continuous improvement
- A/B testing of playbooks
- Large-scale deployment

**Workflow**:
1. User sends request → Fast inference with current playbook
2. User provides feedback → Queued for training
3. Background worker processes feedback batch
4. Full ACE loop runs with feedback as ground truth
5. Updated playbook saved to vector DB
6. Inference tier syncs new playbook (hourly/daily)

---

## 11. Security & Privacy Considerations

### Data Flow Security

```mermaid
graph LR
    subgraph "User Data"
        Q[Question<br/>May contain PII]
        C[Context<br/>May contain sensitive data]
    end
    
    subgraph "ACE Framework"
        G[Generator]
        PB[(Playbook<br/>Stores patterns, not data)]
    end
    
    subgraph "External Services"
        API[LLM API<br/>OpenAI/Anthropic]
    end
    
    subgraph "Storage"
        CKPT[(Checkpoints<br/>Contains full samples)]
        LOGS[(Logs<br/>May contain PII)]
    end
    
    Q --> G
    C --> G
    G -.Sends to.-> API
    G --> PB
    G --> CKPT
    G --> LOGS
    
    style Q fill:#ffcdd2,stroke:#c62828,stroke-width:3px
    style C fill:#ffcdd2,stroke:#c62828,stroke-width:3px
    style API fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style CKPT fill:#ffccbc,stroke:#d84315,stroke-width:2px
    style LOGS fill:#ffccbc,stroke:#d84315,stroke-width:2px
    style PB fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
```

**Security Concerns**:

1. **PII in Samples**: Questions/context may contain personal information
   - **Risk**: Sent to LLM API, stored in checkpoints/logs
   - **Mitigation**: Anonymize data before processing, use private LLM deployment

2. **Checkpoint Storage**: Contains full sample data including ground truth
   - **Risk**: Sensitive data persisted to disk
   - **Mitigation**: Encrypt checkpoints, secure file permissions, delete after training

3. **Log Files**: May contain PII from samples and outputs
   - **Risk**: Unencrypted logs accessible to admins
   - **Mitigation**: Redact PII in logs, encrypt log files, rotate frequently

4. **Playbook Content**: Stores patterns, not raw data
   - **Risk**: Low (abstracts away specific data)
   - **Benefit**: Safe to share and version control

5. **API Keys**: Required for LLM access
   - **Risk**: Exposed keys allow unauthorized API usage
   - **Mitigation**: Environment variables, secret management, key rotation

**Best Practices**:
- Use environment variables for API keys (never commit to git)
- Encrypt checkpoints and logs at rest
- Anonymize PII before processing
- Use private LLM deployment for sensitive data
- Implement data retention policies (delete old checkpoints)
- Audit log access and API usage



---

## 12. Summary: Key Architectural Insights

### Core Architecture
- **Pattern**: Generator-Evaluator-Reflector with feedback loop
- **Components**: 5 main (Generator, Evaluator, Reflector, Curator, Playbook)
- **LLM Calls**: 3 per sample (Generator, Reflector, Curator)
- **Storage**: In-memory during execution, file system for persistence

### Ground Truth Handling
- **Training Mode**: Ground truth required for evaluation and learning
- **Production Mode**: Ground truth optional, learning disabled
- **Sources**: Pre-labeled datasets, human feedback, automated oracles
- **Key Insight**: Generator works without ground truth, but can't learn

### Data Architecture
- **Playbook**: In-memory dict, serialized to JSON checkpoints
- **Embeddings**: Cached in-memory dict (global), used for deduplication
- **Checkpoints**: JSON files saved after each sample
- **No Database**: File system sufficient for MVP scale

### External Dependencies
- **LLM API**: OpenAI or Anthropic (3 calls per sample)
- **Embedding Model**: sentence-transformers (local, cached)
- **No Vector DB**: Not required for MVP (optional for scale)

### Design Decisions
- **Synchronous**: Sequential processing (simple, debuggable)
- **Stateful Adapter**: Manages playbook and reflection context
- **Stateless Roles**: Generator/Reflector/Curator have no state
- **Fail-Fast**: Errors stop execution (configurable fallback)
- **Centralized**: Single process (scalable to distributed later)

### Scalability
- **Current**: 10-1,000 samples, single machine
- **Bottleneck**: LLM API rate limits
- **Scaling Path**: Batch API calls → Async processing → Distributed architecture
- **Production**: Separate inference (fast) and training (background) tiers

### Security
- **PII Risk**: Samples may contain sensitive data sent to LLM API
- **Mitigation**: Anonymize data, encrypt storage, use private LLM
- **Playbook Safety**: Stores patterns, not raw data (safe to share)

---

## Implementation Guidance

### For Developers

**To implement a new task**:
1. Create `TaskEnvironment` subclass with `evaluate()` method
2. Define how to compare output to ground truth
3. Provide samples with ground_truth field
4. Run `OfflineAdapter` for training
5. Use trained playbook with `Generator` for inference

**To add a new LLM provider**:
1. Subclass `LLMClient`
2. Implement `complete()` method
3. Handle API-specific authentication and formatting
4. Add to `create_llm_client_from_config()` factory

**To scale to production**:
1. Deploy inference tier with pre-trained playbook
2. Collect user feedback asynchronously
3. Run background training workers
4. Sync updated playbook to inference tier
5. Monitor API costs and performance

### For Researchers

**To experiment with ACE**:
1. Start with provided examples (QA, code generation)
2. Modify prompts in `ace/prompts.py`
3. Adjust hyperparameters in config YAML
4. Run multi-epoch training with different settings
5. Analyze checkpoints to understand learning

**To extend ACE**:
1. Add retrieval mechanism for large playbooks
2. Implement hierarchical playbook structure
3. Add multi-agent collaboration
4. Experiment with different reflection strategies
5. Integrate with external knowledge bases

---

## Conclusion

The ACE framework implements a clean, modular architecture for adaptive LLM agents. The Generator-Evaluator-Reflector pattern enables continuous learning through structured playbook updates. Ground truth is only required for training, not production inference. The current MVP uses simple file-based storage and synchronous processing, with a clear path to distributed, production-scale deployment.

**Key Takeaway**: ACE separates inference (fast, stateless) from learning (slow, stateful), enabling both real-time user-facing applications and continuous improvement through feedback.

