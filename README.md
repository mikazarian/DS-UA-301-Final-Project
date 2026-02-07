# 🧠 Reducing LLM Hallucination in Political Fact-Checking with Tool Augmentation and Custom Scoring

**Michael Kazarian (NYU)**  
DS-UA 301 — Final Project

---

## 📌 Overview

Large Language Models (LLMs) often **hallucinate** when evaluating political claims, especially when those claims require historical or contextual knowledge. This project investigates whether **tool augmentation** (e.g., Wikipedia retrieval) and **structured evidence scoring** can reduce hallucinations and improve factual classification.

We compare three systems:

1. **Baseline LLM** — direct prompt, no tools
2. **Tool-Augmented ReACT Agent** — Wikipedia retrieval via LangChain
3. **Custom-Prompted Evidence Scoring System** — SERP API retrieval + GPT-4 scoring with quantitative trust metrics

We evaluate performance on the **LIAR political fact-checking dataset** and analyze why some approaches surprisingly underperform.

---

## 🎯 Key Insight

> Simply giving an LLM access to tools **does not** guarantee better factual reasoning.

| Model | Accuracy | Weighted F1 |
|-------|----------|-------------|
| Baseline (no tools) | **47.3%** | 0.408 |
| Tool-Augmented (ReACT + Wikipedia) | **43.3%** | 0.333 |
| **Custom Evidence Scoring (SERP + GPT-4)** | **69%** | **0.73** |

The best system was **not** the one with the most tools, but the one that **forced the LLM into a structured quantitative reasoning framework**.

---

## 🧪 Dataset

We use the **LIAR dataset** (12,836 political claims).

- Original labels: 6 truth categories
- Binarized into: **True / False**
- Evaluation set: first **300 claims** from official test split
- Metrics: Accuracy, F1, Confusion Matrices

---

## ⚙️ Methodology

### 1️⃣ Baseline Prompting (GPT-3.5)

The model receives only the claim and must answer:

```
Answer: True/False
Justification: ...
```

No external tools.

---

### 2️⃣ Tool-Augmented ReACT Agent (LangChain + Wikipedia)

- Agent performs up to 5 reasoning steps
- Uses Wikipedia API tool for evidence retrieval
- Integrates evidence before final decision

**Problem observed:**  
Agent over-relied on retrieval. If it failed to find relevant evidence, it defaulted to incorrect conclusions—even when the LLM “knew” the answer from training.

---

### 3️⃣ Custom Evidence Scoring System (SERP API + GPT-4)

This system mimics **human fact-checkers**.

For each claim, we retrieve 10 articles and compute:

#### ✔ Credibility Score
\[
\frac{N_{corroborating}}{N_{corroborating} + N_{contradicting}}
\]

#### ✔ Bias Score
Political leaning + emotional tone

#### ✔ Evidence Strength
\[
\frac{N_{strong}}{N_{strong} + N_{weak}}
\]

#### ✔ Trust Score
\[
w_1 \cdot credibility + w_2 \cdot (1 - |bias|) + w_3 \cdot evidence
\]

If trust score > 0.5 → **True**

The LLM outputs structured JSON, removing open-ended reasoning.

---

## 📊 Results

- Both baseline and tool models had **very poor recall on False claims**
- Tool model performed worse because of **retrieval failure**
- Custom scoring system significantly improved balance

(See report for confusion matrices and examples.)

---

## 🔍 Qualitative Findings

Example failure case:

> **Claim:** Obama’s NLRB sued Boeing over opening a plant in SC  
> **Ground Truth:** True  
> **Baseline:** True  
> **Tool Agent:** False  

The tool agent couldn’t retrieve the historical context, while the baseline LLM remembered it from training.

---

## 🧠 Why Custom Prompting Worked

The custom system:

- Reduced reliance on open-ended reasoning
- Leveraged LLM strengths: sentiment, semantic similarity, classification
- Forced **quantitative evidence aggregation**
- Minimized hallucination by avoiding narrative justification

---

## 🛠️ Tech Stack

- Python
- LangChain (ReACT agent)
- Wikipedia API
- SERP API
- GPT-3.5 / GPT-4
- sklearn for evaluation

---

## 📁 Project Structure

```
/data
/baseline
/react_agent
/custom_scoring
/evaluation
/prompts
report.pdf
```

---

## 🚧 Limitations

- LIAR dataset claims are not time-stamped
- Wikipedia retrieval is keyword-based and often irrelevant
- Binary labels oversimplify nuanced claims
- SERP API limited to 10 articles

---

## 🔮 Future Work

- Multi-class truth labels (partially true, etc.)
- ROC-based trust threshold tuning
- Improved retrieval beyond keyword search
- Tool usage that supports, not replaces, LLM reasoning

---

## 🏁 Takeaway

> The key to reducing LLM hallucination was **not** giving the model more information —  
> it was **forcing the model to reason in a structured, quantitative way**.

This project demonstrates how prompt design and evidence scoring can outperform naive tool augmentation in factual reasoning tasks.
