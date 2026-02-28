# LearnMate — AI Study Assistant powered by RAG

An interactive AI study assistant that lets students upload any subject book or document and learn from it conversationally. Ask questions, get summaries, request quizzes, generate flashcards, and make sure nothing is missed — all from a single chat interface. Built from scratch using LangChain, FAISS, HuggingFace Embeddings, and Groq LLM.

## Overview

LearnMate is built around a simple frustration: reading a textbook passively is inefficient. Students highlight, re-read, and still walk away unsure if they have actually understood the material.

LearnMate changes that. Upload any subject book or study material as a PDF and the system becomes an active learning companion. Ask it to explain a concept, quiz you on a chapter, generate flashcards, identify the most important points, or summarise a topic in simpler terms. Every response is grounded strictly in the uploaded document — no guessing, no hallucination, no content from outside the book.

The entire pipeline runs in Google Colab with no local setup or paid infrastructure required.

## The Problem

Large Language Models are trained on general internet data. Ask one about a privately written textbook, a subject guide, or any document it has never encountered, and it will either refuse to answer or, more dangerously, generate a confident but completely wrong response.

This is a real problem in an educational setting. A student asking a general-purpose chatbot about their specific course material will get answers that sound correct but may contradict what is actually in the book.

LearnMate solves this by retrieving the relevant sections of the document before generating any response. The model only speaks from what is in the book. Nothing is invented.

## What Students Can Do

- Ask conceptual questions and get explanations in plain language
- Request a quiz on any chapter or topic to test their understanding
- Generate flashcards from a section for quick revision
- Ask for a summary of a topic before reading it in full
- Identify key points and definitions they should not miss
- Ask follow-up questions to go deeper on anything they did not understand

## Architecture

```
User Question
     |
     v
+------------------+
|  Query Encoder   |  <- HuggingFace all-MiniLM-L6-v2
+--------+---------+
         | embedding vector
         v
+------------------+
|  FAISS Index     |  <- Similarity search across all chunks
+--------+---------+
         | top-k relevant chunks
         v
+------------------+
|  Prompt Builder  |  <- Injects retrieved chunks as context
+--------+---------+
         | grounded prompt
         v
+------------------+
|  Groq LLM        |  <- LLaMA 3.1 8B via Groq API
+--------+---------+
         | answer
         v
     Chat UI
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Document Loading | LangChain PyPDFLoader |
| Text Splitting | RecursiveCharacterTextSplitter |
| Embeddings | HuggingFace all-MiniLM-L6-v2 |
| Vector Store | FAISS |
| LLM | Groq API / LLaMA 3.1 8B Instant |
| UI | ipywidgets |
| Environment | Google Colab |

## How It Works

**1. Document Loading**
The uploaded PDF is read using LangChain's `PyPDFLoader`, which extracts text page by page and returns structured document objects.

**2. Chunking**
The document is split into overlapping chunks of approximately 500 characters using `RecursiveCharacterTextSplitter`. The overlap ensures content at chunk boundaries is not lost.

**3. Embedding**
Each chunk is converted into a 384-dimensional vector using `sentence-transformers/all-MiniLM-L6-v2`. This captures the semantic meaning of the text in a format that can be compared mathematically.

**4. Indexing**
All vectors are stored in a FAISS index, which organises them so that semantically similar chunks are spatially close. This allows fast nearest-neighbour search at query time.

**5. Retrieval**
When a question is asked, it is embedded using the same model. FAISS returns the top-k most similar chunks (k=3 by default) based on cosine similarity.

**6. Generation**
The retrieved chunks are injected into a prompt alongside the question and sent to LLaMA 3.1 8B via the Groq API. The model generates a response grounded in the retrieved content.

**7. Chat Interface**
An ipywidgets UI provides an input box, submit button, clear button, and scrollable chat history inside the Colab notebook.

## Getting Started

You will need a Google account to run the notebook in Colab and a free Groq API key from [console.groq.com](https://console.groq.com).

1. Open the notebook file in Google Colab
2. Run the first cell to install dependencies
3. Add your Groq API key where indicated
4. Upload a PDF when prompted
5. Ask questions in the chat UI

Dependencies are installed automatically inside the notebook:

```
langchain
langchain-community
langchain-text-splitters
langchain-huggingface
pypdf
sentence-transformers
faiss-cpu
ipywidgets
groq
```

## Example

Document used: *Operating Systems — Process Management and Scheduling* (Second Year CS)

```
User: What is the difference between a process and a thread?
Bot:  A process is an independent program in execution with its own
      memory space, file handles, and system resources. A thread is a
      lighter unit of execution that runs within a process and shares
      the same memory and resources as other threads in that process.
      Threads are faster to create and switch between, but a crash in
      one thread can affect the entire process.

User: Quiz me on CPU scheduling algorithms.
Bot:  Sure. Here are three questions based on the chapter:
      1. What is the key difference between preemptive and
         non-preemptive scheduling?
      2. In Round Robin scheduling, what happens if a process does
         not finish within its time quantum?
      3. Which scheduling algorithm can lead to starvation, and why?

User: Give me flashcards for the key terms in this chapter.
Bot:  Here are five flashcards:
      - Deadlock: A state where two or more processes are waiting
        on each other indefinitely and none can proceed.
      - Context Switch: The process of saving the state of a running
        process and loading the state of the next scheduled process.
      - Starvation: A condition where a low-priority process is
        perpetually delayed because higher-priority processes keep
        arriving.
      - Throughput: The number of processes completed per unit time.
      - Turnaround Time: Total time from process submission to
        completion.

User: What causes a race condition?
Bot:  Not found in document.
```

The last response is intentional. LearnMate only answers from the uploaded book. If a topic is not covered in the document, it says so rather than pulling from outside knowledge.
