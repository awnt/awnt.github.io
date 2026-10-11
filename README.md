---
layout: single
author_profile: true
---

# AWNT — Portfolio & Curiosity Space

Welcome to my personal workspace and portfolio. This space documents my projects, experiments, and technical notes as a Data Science student.

🌐 **Live Portfolio:** [awnt.github.io](https://awnt.github.io)

---

### 🧠 Deep Learning & LLM Roadmap
A documentation of my journey exploring and building neural network architectures from scratch up to Large Language Models.
> **Status:** 🚧 Work in Progress (Active Learning Plan)
> 
#### Foundations & Autograd Engine
*Objective: Master the underlying mathematics and computation graph without frameworks.*
- [ ] **Scalar Autograd Engine:** Build a computational DAG with reverse-mode automatic differentiation (Chain Rule) in pure Python.
- [ ] **Manual MLP:** Implement a Multi-Layer Perceptron and manual forward/backward passes using NumPy.
- [ ] **Loss & Optimization:** Understand MSE, Cross-Entropy, and weight updates via Stochastic Gradient Descent (SGD).

#### Deep Learning & PyTorch Core
*Objective: Transition to production standards and understand modern neural architectural components.*
- [ ] **Tensor Autograd:** Re-implement computation graphs efficiently using PyTorch tensors and autograd.
- [ ] **Architecture Essentials:** Deep dive into Normalization techniques (LayerNorm) and modern activations (GELU, SwiGLU).
- [ ] **Modern Optimizers:** Analyze the mathematics and mechanics of Adam and AdamW.

#### Attention & Sequence Modeling
*Objective: Understand tokenization, vector spaces, and self-attention dynamics.*
- [ ] **Tokenization & Embeddings:** Implement subword tokenization, vocabulary mapping, and positional encodings.
- [ ] **Self-Attention Mechanism:** Dissect Query, Key, Value mappings and Scaled Dot-Product Attention from scratch.
- [ ] **Multi-Head Attention:** Vectorize multi-head representations to capture parallel contextual subspaces.

#### Generative Pre-trained Transformers (GPT)
*Objective: Assemble a decoder-only Transformer from scratch for autoregressive text generation.*
- [ ] **Causal Masking:** Enforce autoregressive constraints using lower-triangular causal masks.
- [ ] **Transformer Block Assembly:** Connect residual streams, pre-LayerNorm blocks, and feed-forward networks.
- [ ] **Training Mini-GPT:** Train a lightweight character/BPE-level GPT model on a custom text corpus.
- [ ] **Text Generation & Sampling:** Implement inference pipelines with Temperature, Top-K, and Nucleus (Top-P) sampling.

#### Large Language Models & Fine-Tuning
*Objective: Run and fine-tune open-weight models at an operational level.*
- [ ] **Open-Source LLMs:** Run open-weight architectures (LLaMA, Mistral, Gemma) across local and cloud environments.
- [ ] **Parameter-Efficient Fine-Tuning (PEFT):** Implement low-rank adaptation techniques (LoRA & QLoRA).
- [ ] **Instruction Tuning (SFT):** Format instruction datasets and conduct Supervised Fine-Tuning for task completion.

---

### Featured Repositories
- **[portspace](https://github.com/awnt/portspace)** — Source code, notebooks, and implementations supporting this roadmap.