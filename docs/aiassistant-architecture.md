# MOLT AI Assistant — Architecture

This page documents the MOLT AI Assistant architecture with multiple diagram styles suitable for repository docs, review discussions, and presentations.

## Overview

The MOLT AI Assistant ingests schema metadata and database connection context, analyzes compatibility and migration complexity, applies embedded migration knowledge, invokes an LLM backend for planning and UDF conversion, and produces executable migration artifacts.

## High-Level Block Diagram

```mermaid
flowchart LR
    A[Schema JSON] --> D
    B[Source DB URL] --> N
    C[Target DB URL] --> N

    D[Schema Analysis<br/>- feature detection<br/>- schema context<br/>- compatibility analysis] --> E[Prompt Builder]

    S[Embedded Skills / Knowledge Base<br/>- molt-fetch<br/>- molt-convert<br/>- migration-planning<br/>- UDF guides] --> E

    P[Tokenizer / Privacy Layer<br/>- tokenize identifiers<br/>- maintain reverse map] --> E

    G[Configuration<br/>env vars + config loader] --> H[LLM Backend<br/>Vertex AI / Claude]

    E --> H

    D --> I{Schema larger<br/>than 100KB?}
    I -- No --> J[Generate Migration Plan]
    I -- Yes --> K[Chunk Schema]
    K --> L[Generate Plan per Chunk]
    L --> M[Merge Chunked Plans]
    M --> J

    A --> Q[DDL Conversion Pipeline<br/>parse -> preprocess -> sanitize -> validate]
    D --> R[UDF Conversion Pipeline<br/>detect -> convert in parallel -> review]

    H --> J
    H --> R
    H --> D

    J --> T[Post-Processing<br/>sanitize -> normalize -> defer FKs -> apply config -> synthesize steps]
    N[Connection Utilities<br/>URL parsing / local DB detection] --> T

    T --> U[Executable Migration Plan]
    Q --> V[Converted CockroachDB DDL]
    R --> W[UDF Migration Results]
    D --> X[Analysis Report]
```

## Presentation-Style Architecture View

```mermaid
flowchart TB
    subgraph Inputs["Inputs"]
        IN1[Schema JSON]
        IN2[Source DB URL]
        IN3[Target DB URL]
    end

    subgraph Knowledge["Knowledge + Configuration"]
        K1[Embedded Skills]
        K2[Prompt Templates]
        K3[Environment / Config]
    end

    subgraph Privacy["Privacy Layer"]
        P1[Schema Tokenizer]
        P2[Reverse Mapping]
    end

    subgraph Intelligence["AI Intelligence Layer"]
        A1[Schema Analysis]
        A2[Prompt Builder]
        A3[LLM Backend<br/>Vertex AI / Claude]
    end

    subgraph Pipelines["Execution Pipelines"]
        E1[DDL Conversion]
        E2[UDF Conversion]
        E3[Migration Plan Generation]
        E4[Chunking + Merge]
        E5[Plan Post-Processing]
    end

    subgraph Support["Support Services"]
        S1[Connection Utilities]
        S2[SQL Utilities]
        S3[Progress Tracking]
    end

    subgraph Outputs["Outputs"]
        OUT1[Analysis Report]
        OUT2[Converted DDL]
        OUT3[UDF Migration Results]
        OUT4[Executable Migration Plan]
    end

    IN1 --> P1
    P1 --> A1
    P1 --> A2

    K1 --> A2
    K2 --> A2
    K3 --> A3

    A1 --> A2
    A2 --> A3

    IN1 --> E1
    A1 --> E2
    A1 --> E3
    A3 --> E2
    A3 --> E3

    IN1 --> E4
    E4 --> E3
    E3 --> E5

    IN2 --> S1
    IN3 --> S1
    S1 --> E5
    S2 --> E1
    A3 --> S3

    A1 --> OUT1
    E1 --> OUT2
    E2 --> OUT3
    E5 --> OUT4
    P2 --> OUT4
```

## Runtime Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    participant U as User / CLI
    participant C as Config Loader
    participant T as Tokenizer
    participant A as Schema Analyzer
    participant P as Prompt Builder
    participant S as Embedded Skills
    participant L as LLM Backend (Vertex AI / Claude)
    participant D as DDL Pipeline
    participant UF as UDF Pipeline
    participant CH as Chunking Engine
    participant PG as Plan Generator
    participant PP as Post-Processor
    participant O as Output

    U->>C: Load environment + AI config
    C-->>U: Config ready

    U->>T: Submit schema JSON
    T-->>U: Tokenized schema + reverse map

    U->>A: Analyze schema
    A->>A: Detect schema features
    A->>A: Extract schema context
    A->>P: Request analysis / planning prompts
    P->>S: Load relevant migration + UDF skills
    S-->>P: Embedded guidance
    P->>L: Send enriched analysis prompt
    L-->>A: Analysis result

    par DDL conversion
        U->>D: Convert DDL from schema
        D->>D: Parse PostgreSQL dump
        D->>D: Preprocess statements
        D->>D: Strip/extract UDFs
        D->>D: Validate generated SQL
        D-->>O: Converted CockroachDB DDL
    and UDF conversion
        A->>UF: Detect UDF features
        UF->>P: Build UDF conversion prompts
        P->>L: Send conversion prompt(s)
        L-->>UF: Converted UDFs
        UF->>P: Build review prompts
        P->>L: Send review prompt(s)
        L-->>UF: Reviewed UDF results
        UF-->>O: UDF migration results
    and plan generation
        A->>CH: Check schema size
        alt Schema > 100KB
            CH->>CH: Split schema into chunks
            loop For each chunk
                CH->>P: Build chunk plan prompt
                P->>L: Send chunk prompt
                L-->>PG: Chunk migration plan
            end
            PG->>CH: Merge chunked plans
        else Schema <= 100KB
            A->>P: Build migration plan prompt
            P->>L: Send planning prompt
            L-->>PG: Migration plan
        end

        PG->>PP: Sanitize + normalize plan
        PP->>PP: Defer foreign keys
        PP->>PP: Apply migration mode / TLS / bucket config
        PP->>PP: Synthesize validation + execution steps
        PP-->>O: Executable migration plan
    end

    A-->>O: Analysis report
    O-->>U: Final outputs
```

## Notes

- The tokenizer protects sensitive schema identifiers before prompt submission.
- Embedded markdown skills and templates shape the prompts sent to the LLM backend.
- Large schemas are chunked to stay within prompt and processing constraints.
- Post-processing converts model output into a safer, executable migration plan.
