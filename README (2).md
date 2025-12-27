
# Policy RAG Assistant

## Project Overview
This project is a simple **Retrieval-Augmented Generation (RAG)** based question answering system built on top of company policy documents like Refund, Cancellation and Shipping policies.

The main goal of this assignment is **not building a big system**, but to show my understanding of:
- how RAG works
- prompt design and iteration
- avoiding hallucinations
- evaluating LLM outputs in a simple way

The full project can be run in **Google Colab without any errors**.

---

## Setup Instructions

### Environment
- Python 3
- Google Colab (used for development)

### Install Required Libraries
Run the below command in Colab:

```bash
pip install langchain langchain-community langchain-text-splitters sentence-transformers chromadb transformers
```

### Data Files
All policy documents are stored as text files inside the `data/` folder.

```
data/
 ├── refund_policy.txt
 ├── cancellation_policy.txt
 └── shipping_policy.txt
```

### How to Run
1. Load the policy documents  
2. Split text into chunks  
3. Generate embeddings  
4. Store embeddings in ChromaDB  
5. Retrieve relevant chunks  
6. Generate answer using prompt  
7. Evaluate the outputs  

---

## Architecture Overview

The system follows a basic RAG flow:

```
User Question
   ↓
Vector Search (ChromaDB)
   ↓
Relevant Policy Chunks
   ↓
Prompt with Rules
   ↓
LLM (FLAN-T5)
   ↓
Answer or Safe Fallback
```

### Tools Used
- Document Loader: TextLoader
- Text Splitter: RecursiveCharacterTextSplitter
- Embeddings: all-MiniLM-L6-v2
- Vector Store: ChromaDB
- LLM: google/flan-t5-base

---

## Chunking Strategy

- Chunk size: 200 characters  
- Overlap: 40 characters  

Reason for this choice is policy documents are small, so smaller chunks helps in getting more accurate retrieval and reduces unnecessary context.

---

## Prompt Engineering

### Initial Prompt
```text
Answer the question using the context below.

Context:
{context}

Question:
{question}

Answer:
```

This prompt sometimes allowed vague answers when context was not fully available.

---

### Improved Prompt (Final)
```text
You are a company policy assistant.

Rules:
- Answer ONLY from the given context.
- If answer is not present, reply with:
  "The information is not available in the provided documents."
- Do not guess or add outside knowledge.
- Keep the answer short and clear.

Context:
{context}

Question:
{question}

Answer:
```

This improved prompt reduced hallucinations and made the answers more reliable.

---

## Evaluation Results

A small evaluation set was created with different types of questions.

| Question | Expected Behaviour | Result | Score |
|--------|-------------------|--------|-------|
| Refund time period | 14 days | Correct | ✅ |
| Cancel after shipping | Not allowed | Correct | ✅ |
| Shipping delay reason | Weather / operations | Correct | ✅ |
| Digital product refund | Not refundable | Correct | ✅ |
| International shipping | Not mentioned | Safe fallback | ❌ |
| Student discounts | Not mentioned | Safe fallback | ❌ |

Legend:
- ✅ Correct and grounded  
- ❌ Correctly declined (no hallucination)

---

## Edge Case Handling

### Question outside documents
```
Q: Do you offer international shipping?
A: The information is not available in the provided documents.
```

### Completely unrelated question
```
Q: Who is the CEO of the company?
A: The information is not available in the provided documents.
```

This ensures model does not make up answers.

---

## Key Trade-offs

- Used smaller chunk size for better precision instead of recall
- Manual evaluation instead of automated metrics due to small dataset
- Used open-source LLM to keep the project reproducible
- Focused more on prompt constraints than complex retrieval logic

---

## Future Improvements

If more time is available:
- Add reranking to improve retrieval quality
- Use structured JSON outputs
- Add simple logging for analysis
- Compare different prompt variations

---

## Final Reflection

**What I am most proud of:**  
Designing a prompt which clearly controls hallucinations while still answering correctly.

**One thing I would improve next:**  
Adding lightweight reranking and output validation to make system more robust.
