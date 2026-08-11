# Smart Contract Vulnerability Detection & Auto-Repair using RAG

An end-to-end system that detects vulnerabilities in Solidity smart contracts and generates verified, auto-repaired patches, combining a machine learning classifier, Retrieval-Augmented Generation (RAG), and a large language model (Llama 3) in a closed feedback loop.

---

## 📌 Overview

Traditional smart contract auditing relies heavily on static analysis and formal verification, which suffer from high false-positive rates and poor scalability against novel or logic-level vulnerabilities. This project builds a Retrieval-Augmented Generation-based framework that:

- Detects vulnerabilities in smart contracts with higher accuracy than static rule-based tools
- Reduces dependency on hand-written detection rules
- Adapts to emerging blockchain security threats via a continuously updatable knowledge base
- Goes beyond detection to generate and **verify** an actual repaired contract, not just a report

## 🎯 Goals

- **Detection** — classify vulnerability types present in a given contract
- **Explanation** — generate human-readable explanations of each flagged issue
- **Repair** — produce a candidate patched contract
- **Verification** — confirm the patch actually resolves the vulnerability before returning it

## 🏗️ Architecture

```
Smart Contract Input ──┐
                        ├──▶ Orchestrator ──┬──▶ ML Classifier ──────┐
User Prompt/Instructions┘                   └──▶ RAG Retriever ──────┼──▶ Fusion / Meta-Learner
                                              (searches Vector DB)   │           │
                                                                     ▼           ▼
                                                              Vector DB    Fused Vulnerability
                                                           (SWC Registry,        Verdict
                                                          patched pairs,          │
                                                         remediation notes)       ▼
                                                                            LLM (Llama 3)
                                                                      generates explanation
                                                                         + candidate patch
                                                                                 │
                                                                                 ▼
                                                                        Verification Step
                                                                    (re-run ML Classifier /
                                                                      Slither on patch)
                                                                          │         │
                                                              vulnerability      resolved
                                                                remains             │
                                                                    │               ▼
                                                                    └──▶ Final Output: Report
                                                                     (loop)   + Verified Patch
```

**Key design decisions:**
- ML Classifier and RAG Retriever run **in parallel**, then their outputs are combined by a Fusion/Meta-Learner rather than chained linearly — improves accuracy over a single sequential pipeline.
- Every generated patch is **re-verified** against the classifier/static analyzer before being returned, with iterative retry if the vulnerability isn't actually resolved.
- External tools (Mythril, Echidna, Surya) are wired in as an **optional** manual cross-check, not a hard dependency.

## ⚙️ Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| Frontend | React, Bootstrap, Monaco Editor | UI + Solidity-aware code editor |
| Backend / Orchestration | FastAPI (Python) | Routes requests across all components |
| Static Analysis | Slither, Mythril, Echidna, Surya, `solc` | Contract parsing, feature extraction, manual cross-check |
| ML Classifier | scikit-learn / XGBoost, optionally fine-tuned CodeBERT | Multi-label vulnerability probability prediction |
| Embeddings | sentence-transformers, CodeBERT / GraphCodeBERT | Vectorizing contracts and knowledge base entries |
| Vector Database | ChromaDB (or FAISS) | Stores SWC Registry entries, vulnerable/patched pairs, remediation patterns |
| Retrieval | LangChain, rank_bm25 | RAG retrieval pipeline (dense + keyword hybrid) |
| LLM | Llama 3 via Ollama (local) or Groq API (cloud) | Explanation and patch generation |
| Database | MySQL | User accounts, scan history, reports |

## 📊 Datasets

- [SmartBugs Curated](https://github.com/smartbugs/smartbugs-curated) — labeled vulnerable Solidity contracts
- [SolidiFI-Benchmark](https://github.com/DependableSystemsLab/SolidiFI-benchmark) — injected-vulnerability benchmark for evaluation

## 🚀 Getting Started

### Prerequisites
- Python 3.10+
- Node.js 18+
- `solc` (Solidity compiler)
- Ollama (for local LLM inference) or a Groq API key

### Installation

```bash
# Clone the repository
git clone <repo-url>
cd smart-contract-vulnerability-rag

# Backend setup
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Frontend setup
cd ../frontend
npm install

# Pull the LLM locally (if using Ollama)
ollama pull llama3
```

### Running the project

```bash
# Start backend
cd backend
uvicorn main:app --reload

# Start frontend
cd frontend
npm start
```

## 🧪 Evaluation Metrics

- **Detection:** Precision, Recall, F1-score per vulnerability class
- **Repair:** Correct Patch Rate (CPR), Error Repair Rate (ERR), Overall Repair Rate (ORR)

## 👥 Team — Group G1327

| Roll No. Role | Name |
|---|---|
| Team Member | Siddharth Tallapally |
| Team Member | Chakrapanda Sathwik |
| Team Member | P Pavan Kumar |
| Team Member | Rentala Risheeth Preetham |
| Team Member | Tirukoti Vinay |

**Mentor:** Shanker

## 📚 References

- ParaVul — Parallel LLM and Retrieval-Augmented Framework for Smart Contract Vulnerability Detection
- RAG-SmartVuln — Enhancing Smart Contract Vulnerability Detection via Retrieval-Augmented LLMs
- SCPatcher — Automated Smart Contract Code Repair via RAG and Knowledge Graph
- RAVEN — Agentic RAG for Automated Vulnerability Repair
- LLM-BSCVM — LLM-Based Blockchain Smart Contract Vulnerability Management Framework
- ContractTinker — LLM-Empowered Vulnerability Repair for Real-World Smart Contracts
- [SWC Registry](https://swcregistry.io/) — Smart Contract Weakness Classification

## 📄 License
 MIT.
