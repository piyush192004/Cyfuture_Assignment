# AI Agent for Academic Performance Analysis

> **Assignment:** Design an AI agent that helps students and faculty understand academic performance.

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture Flow Diagram](#architecture-flow-diagram)
3. [Component Details](#component-details)
4. [Decision Gates](#decision-gates)
5. [Libraries & Frameworks](#libraries--frameworks)
6. [Justification of Key Choices](#justification-of-key-choices)

---

## Overview

This AI agent retrieves academic records and attendance data, identifies trends and weak performance areas, combines structured academic data with institutional policy documents, and generates **separate, role-specific summaries** for students and faculty — prioritising the most impactful recommendations first.

---

## Architecture Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        INPUT LAYER                          │
│                                                             │
│   ┌──────────────────┐        ┌──────────────────┐         │
│   │   Academic DB    │        │  Attendance DB   │         │
│   │  (Grades, GPA,   │        │  (Logs, Records) │         │
│   │   Courses)       │        │                  │         │
│   └────────┬─────────┘        └────────┬─────────┘         │
│            └──────────────┬────────────┘                    │
│                    ┌──────▼──────┐                          │
│                    │    Data     │◄──── Policy PDFs /       │
│                    │  Collector  │      Guidelines          │
│                    └──────┬──────┘                          │
└───────────────────────────┼─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     VALIDATION LAYER                        │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                  Data Validator                     │   │
│   │   • Check completeness of records                  │   │
│   │   • Check historical depth (min 2 semesters)       │   │
│   │   • Flag missing / corrupt entries                 │   │
│   └─────────────────────┬───────────────────────────────┘   │
│                         │                                   │
│             ┌───────────▼────────────┐                      │
│             │    DECISION GATE 1     │                      │
│             │  Sufficient Data?      │                      │
│             └────┬──────────┬────────┘                      │
│                YES          NO                              │
│                 │           └──► Flag record, request data  │
└─────────────────┼───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│                      ANALYSIS LAYER                         │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                  Trend Analyzer                     │   │
│   │   • Subject-wise performance over time             │   │
│   │   • Attendance vs. grade correlation               │   │
│   │   • Semester-over-semester trend lines             │   │
│   └─────────────────────┬───────────────────────────────┘   │
│                         │                                   │
│   ┌─────────────────────▼───────────────────────────────┐   │
│   │               Weak Area Identifier                  │   │
│   │   • Subjects scoring below threshold (<60%)        │   │
│   │   • At-risk student detection                      │   │
│   │   • Performance gap analysis                       │   │
│   └─────────────────────┬───────────────────────────────┘   │
│                         │                                   │
│             ┌───────────▼────────────┐                      │
│             │    DECISION GATE 2     │                      │
│             │  Which data sources    │                      │
│             │  are required?         │                      │
│             └────┬──────────┬────────┘                      │
│           Academic        Policy                            │
│           Data only       Docs too                         │
└─────────────┼─────────────┼───────────────────────────────--┘
              │             │
              ▼             ▼
┌─────────────────────────────────────────────────────────────┐
│                 INSIGHT GENERATION LAYER                    │
│                                                             │
│   ┌────────────────────┐    ┌───────────────────────────┐   │
│   │  Structured Data   │    │   Policy / Guidance       │   │
│   │  Insight Engine    │    │   RAG Module              │   │
│   │  (Statistics + ML) │    │   (LLM + Vector Store)    │   │
│   └─────────┬──────────┘    └──────────┬────────────────┘   │
│             └──────────────┬───────────┘                    │
│                            │                               │
│              ┌─────────────▼───────────┐                   │
│              │    DECISION GATE 3      │                   │
│              │  Student or Faculty?    │                   │
│              └──────┬─────────┬────────┘                   │
└─────────────────────┼─────────┼───────────────────────────-┘
                      │         │
                      ▼         ▼
┌─────────────────────────────────────────────────────────────┐
│               RECOMMENDATION & OUTPUT LAYER                 │
│                                                             │
│   ┌─────────────────────┐     ┌─────────────────────────┐   │
│   │   Student Summary   │     │    Faculty Summary      │   │
│   │                     │     │                         │   │
│   │  • My weak areas    │     │  • Class-level trends   │   │
│   │  • Personalised     │     │  • At-risk student list │   │
│   │    study tips       │     │  • Intervention         │   │
│   │  • Prioritised      │     │    suggestions          │   │
│   │    action items     │     │  • Policy-backed        │   │
│   └─────────────────────┘     │    recommendations      │   │
│                               └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## Component Details

### 1. Data Collection Module
- Connects to institutional databases via **REST APIs** or **SQL queries**
- Ingests: grades, GPA, course enrollment, attendance logs
- Also ingests: academic policy PDFs and faculty guidance documents (for RAG)

### 2. Data Validation Module
- Verifies record completeness and historical depth
- Flags missing semesters, null grades, or corrupt attendance entries
- Feeds into **Decision Gate 1**

### 3. Trend Analyzer
- Applies statistical methods (moving averages, linear regression) to track performance over time
- Detects declining trajectories, subject-level struggles, and attendance–grade correlations

### 4. Weak Area Identifier
- Flags subjects below a configurable performance threshold (default: 60%)
- Uses rule-based logic plus ML classifiers to detect at-risk students
- Performs performance gap analysis across cohorts

### 5. RAG Module *(Retrieval-Augmented Generation)*
- Embeds institutional policy documents into a **vector store** (FAISS or ChromaDB)
- Retrieves relevant policy context at query time using **semantic search**
- Combines retrieved context with structured insights before passing to the LLM

### 6. Persona Router *(Decision Gate 3)*
- Determines the user's role: **Student** or **Faculty**
- Routes the enriched insights to the appropriate summary generator
- Ensures role-appropriate granularity and tone

### 7. Recommendation Prioritiser
- Ranks improvement areas by **severity** and **expected impact**
- Ensures critical performance gaps are surfaced at the top of any summary

---

## Decision Gates

| Gate | Location | Question | Outcomes |
|------|----------|----------|----------|
| **Gate 1** | After Validation | Is there sufficient historical data? | YES → Continue; NO → Flag & request data |
| **Gate 2** | After Analysis | Which data sources are needed? | Academic data only OR Academic + Policy docs |
| **Gate 3** | Before Output | Who is the end user? | Student summary OR Faculty summary |

---

## Libraries & Frameworks

| Layer | Library / Tool | Justification |
|-------|---------------|---------------|
| **Data Ingestion** | `pandas`, `SQLAlchemy` | Efficient tabular data handling and DB connectivity |
| **ML & Analysis** | `scikit-learn`, `statsmodels` | Trend detection, regression, anomaly detection — interpretable outputs |
| **LLM Backbone** | `LangChain` + Claude / GPT API | Orchestrates multi-step reasoning and prompt chaining |
| **RAG / Vector Store** | `FAISS` or `ChromaDB` + `sentence-transformers` | Fast semantic search over policy documents |
| **Agent Framework** | `LangGraph` or `AutoGen` | Stateful multi-step decision-making with graph-based flow control |
| **API Layer** | `FastAPI` | Exposes the agent as a lightweight, async REST service |
| **Frontend** | `Streamlit` or `React` | Dashboard interface for students and faculty |
| **Database** | `PostgreSQL` | Reliable relational store for academic records |

---

## Justification of Key Choices

### LangChain + LangGraph
Enables building **stateful, multi-step agents** with explicit decision gates. LangGraph's graph-based execution model maps directly to the architecture above — each node is a processing step, and edges represent conditional routing (the decision gates).

### RAG with FAISS / ChromaDB
Academic policy documents are semi-structured text. RAG ensures the agent **grounds recommendations in actual institutional guidelines** rather than generating policy hallucinations. Semantic retrieval means the right policy sections are surfaced contextually.

### scikit-learn for Analysis
Lightweight and interpretable — critical in academic settings where **explainability** matters to both administrators and students. Provides auditable trend analysis without the opacity of deep learning models.

### Separate Student / Faculty Outputs
Different stakeholders need **different levels of granularity**:
- Students need personalised, micro-level action items
- Faculty need macro-level class analytics and intervention triggers

### PostgreSQL as the Data Store
Relational integrity is essential for academic data (referential keys between students, courses, semesters). PostgreSQL provides the reliability and query flexibility needed for complex analytical queries.

---

## Summary

This architecture satisfies all assignment requirements:

- ✅ Retrieves academic records and attendance data from institutional systems
- ✅ Identifies trends and weak performance areas via statistical and ML analysis
- ✅ Combines structured academic data with unstructured policy documents (RAG)
- ✅ Generates separate summaries for students and faculty
- ✅ Determines which data sources are needed via explicit decision gates
- ✅ Decides whether sufficient historical data exists before generating insights
- ✅ Prioritises improvement recommendations based on identified performance gaps
- ✅ Mentions and justifies all libraries and frameworks used
