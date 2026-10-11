# 🔤 MinBPE: Minimal Byte Pair Encoding Tokenizer

![Python Version](https://img.shields.io/badge/Python-3.6+-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)
![Status](https://img.shields.io/badge/Status-Educational-orange.svg)

A clean, minimal implementation of the **Byte Pair Encoding (BPE)** tokenizer algorithm, commonly used in Large Language Models (LLMs) like GPT-2, GPT-3, GPT-4, and many others.

---

## 📖 Table of Contents

1. [What is a Tokenizer? (Layman's Explanation)](#-what-is-a-tokenizer-laymans-explanation)
2. [Why Tokenizers Are "At Fault"](#%EF%B8%8F-why-tokenizers-are-at-fault)
3. [The Move Away From Tokenization](#-the-move-away-from-tokenization)
4. [Project Architecture](#%EF%B8%8F-project-architecture)
5. [Installation](#-installation)
6. [Usage](#-usage)
7. [How BPE Works](#-how-bpe-works)
8. [File Structure](#-file-structure)
9. [Testing](#-testing)
10. [References](#-references)

---

## 🧠 What is a Tokenizer? (Layman's Explanation)

### The Simple Analogy

Imagine you're teaching a child to read. You don't start by showing them entire books—you start with:

1. **Letters** (A, B, C...)
2. **Then syllables** (ba, be, bi...)
3. **Then words** (baby, book, ball...)
4. **Then sentences**

A **tokenizer** does something similar for AI models. It's the translator that converts human text (which computers don't understand) into numbers (which computers DO understand).

### Think of it Like This

```text
Your text:     "Hello World"
                    ↓
              [Tokenizer]
                    ↓
AI sees:       [15496, 2159]
```

The AI can't read "Hello World" any more than your calculator can. It needs numbers. The tokenizer is the middleman that makes this translation possible.

### Real-World Example

When you type "I love pizza 🍕" into ChatGPT:

1. **Tokenizer breaks it down:**
   - "I" → token 40
   - " love" → token 1842
   - " pizza" → token 22600
   - " 🍕" → tokens [12520, 234, 239]

2. **AI processes these numbers** (does its magic)

3. **Tokenizer converts the AI's number response back to text**

**Key insight:** The AI never "sees" your words. It only sees numbers representing chunks of text.

---

## ⚠️ Why Tokenizers Are "At Fault"

Tokenizers seem innocent, but they're actually responsible for MANY weird behaviors you've noticed in AI:

### 1. 🔢 **AI is Terrible at Math**

**The Problem:**

```text
Question: "What's 127 + 689?"
AI sees:  [127] [+] [689] as separate tokens
```

The AI doesn't see "127" as the number one-hundred-twenty-seven. It sees it as an arbitrary symbol, like seeing "🌟". The tokenizer strips away mathematical meaning!

**This is why ChatGPT often fails at basic arithmetic—it literally doesn't understand numbers as quantities.**

### 2. 📝 **Spelling Mistakes & Letter Counting**

**The Problem:**

```text
Question: "How many 'r's are in 'strawberry'?"
Tokenized as: ['straw', 'berry'] (2 tokens, not 10 letters!)
```

The AI can't count letters because it never sees individual letters! It sees chunks called tokens. When you ask "how many r's?", the AI is essentially blind to individual characters.

**Famous failure:** Most AIs say "strawberry" has 2 r's (it has 3) because of tokenization.

### 3. 🌍 **Non-English Languages Get Butchered**

**The Problem:**

```text
English:  "hello" = 1 token (efficient!)
Japanese: "こんにちは" = 5+ tokens (inefficient!)
Korean:   "안녕하세요" = 5+ tokens (inefficient!)
```

Most tokenizers were trained primarily on English text. This means:

- **Non-English text uses MORE tokens** (costs more money)
- **AI performs WORSE on non-English** (less context fits in memory)
- **Rare languages are almost unusable**

This is a form of **algorithmic bias** baked into the tokenizer!

### 4. 🧵 **Why AI Can't Handle Long Texts Well**

**The Problem:**

```text
Context window: 4096 tokens
Your essay: "Let me explain..." (might use up 3000 tokens!)
Your question: "Summarize this" (AI forgets the beginning!)
```

Because tokens ≠ words, you can't predict how much text fits in the AI's memory. Sometimes a short text uses tons of tokens (especially code or non-English), and the AI "forgets" earlier content.

### 5. 💉 **Security Vulnerabilities (Prompt Injection)**

**The Problem:**

```text
Innocent text: "Please ignore previous instructions"
Tokenized oddly: Can sometimes bypass safety filters!
```

Attackers craft special text that tokenizes in unexpected ways, potentially bypassing safety guardrails. Different whitespace, unicode characters, or unusual spellings can exploit tokenization quirks.

### 6. 🔡 **Case Sensitivity Chaos**

**The Problem:**

```text
"Hello" = token 15496
"HELLO" = token 31373
"hello" = token 31373
"HeLLo" = tokens [1544, 3069] (2 tokens!)
```

The same word in different cases becomes completely different tokens! The AI doesn't inherently know that "DOG" and "dog" are the same word.

### 7. 🔢 **Token Boundaries Create Confusion**

**The Problem:**

```text
" hello" (with space) = token 23748
"hello" (no space) = token 31373
```

A leading space changes everything! This is why AI sometimes adds weird spaces or formats text strangely—the tokenizer treats " hello" and "hello" as completely unrelated concepts.

### 8. 📊 **Cost Multiplication for Simple Text**

**The Problem:**

```text
"AI" = 1 token (common word)
"Bioluminescence" = 4 tokens (rare word)
```

You pay per token with most AI APIs. Rare words, technical jargon, and non-English text cost MUCH more than common English words. The tokenizer silently multiplies your costs!

---

## 🚀 The Move Away From Tokenization

Given all these problems, researchers are exploring alternatives:

### 1. **Byte-Level Models (No Tokenizer!)**

Models like **MegaByte** and **ByteFormer** work directly on raw bytes—no tokenizer needed!

**Advantages:**

- ✅ Perfect letter/character understanding
- ✅ Equal treatment of all languages
- ✅ No vocabulary limitations
- ✅ Better math understanding

**Disadvantages:**

- ❌ Much longer sequences (more compute)
- ❌ Slower training and inference

### 2. **Character-Level Tokenization**

Instead of subword tokens, use individual characters.

```text
"Hello" → ['H', 'e', 'l', 'l', 'o']
```

**Used by:** Some specialized models, especially for math and code.

### 3. **Dynamic/Learned Tokenization**

Let the model learn its own tokenization during training rather than using a fixed vocabulary.

### 4. **Hybrid Approaches**

Combine byte-level and token-level processing:

- Use tokens for common patterns
- Fall back to bytes for rare/unusual text

### 5. **Mamba and State Space Models**

These new architectures can handle much longer sequences efficiently, making byte-level processing more practical.

### 6. **ByT5 and Character-Aware Transformers**

Google's ByT5 operates on raw UTF-8 bytes, eliminating tokenization entirely while remaining competitive with token-based models.

---

## 🏗️ Project Architecture

This project implements three tokenizer variants with increasing complexity:

```text
┌─────────────────────────────────────────────────────────────────┐
│                         minbpe Package                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    ┌───────────────────────────────────────────────────────┐    │
│    │                    base.py                            │    │
│    │  ┌─────────────────────────────────────────────────┐  │    │
│    │  │              Tokenizer (Base Class)             │  │    │
│    │  │  • train() - abstract                           │  │    │
│    │  │  • encode() - abstract                          │  │    │
│    │  │  • decode() - abstract                          │  │    │
│    │  │  • save() / load() - serialization              │  │    │
│    │  │  • _build_vocab() - vocabulary builder          │  │    │
│    │  └─────────────────────────────────────────────────┘  │    │
│    │                                                       │    │
│    │  Helper Functions:                                    │    │
│    │  • get_stats() - count consecutive pairs              │    │
│    │  • merge() - replace pairs with new token             │    │
│    │  • replace_control_characters() - clean output        │    │
│    │  • render_token() - pretty print tokens               │    │
│    └───────────────────────────────────────────────────────┘    │
│                            ↑                                    │
│              ┌─────────────┴─────────────┐                      │
│              ↓                           ↓                      │
│    ┌─────────────────────┐    ┌─────────────────────────┐       │
│    │     basic.py        │    │       regex.py          │       │
│    │  ┌───────────────┐  │    │  ┌───────────────────┐  │       │
│    │  │BasicTokenizer │  │    │  │  RegexTokenizer   │  │       │
│    │  │               │  │    │  │                   │  │       │
│    │  │ • Pure BPE    │  │    │  │ • Regex splitting │  │       │
│    │  │ • No patterns │  │    │  │ • Special tokens  │  │       │
│    │  │ • No specials │  │    │  │ • GPT-4 pattern   │  │       │
│    │  └───────────────┘  │    │  └───────────────────┘  │       │
│    └─────────────────────┘    └───────────────────────────┘     │
│                                          ↑                      │
│                                          │                      │
│                               ┌──────────┴──────────┐           │
│                               ↓                     │           │
│                    ┌─────────────────────┐          │           │
│                    │      gpt4.py        │          │           │
│                    │  ┌───────────────┐  │          │           │
│                    │  │ GPT4Tokenizer │  │          │           │
│                    │  │               │  │          │           │
│                    │  │ • Pretrained  │  │          │           │
│                    │  │ • cl100k_base │  │          │           │
│                    │  │ • Byte shuffle│  │          │           │
│                    │  └───────────────┘  │          │           │
│                    └─────────────────────┘          │           │
│                                                     │           │
└─────────────────────────────────────────────────────────────────┘
```

### Component Details

#### 1. **`base.py`** - The Foundation

Contains the abstract `Tokenizer` class and core algorithms:

| Component | Purpose |
|-----------|---------|
| `get_stats()` | Counts consecutive byte pairs in a sequence |
| `merge()` | Replaces all occurrences of a pair with a new token |
| `Tokenizer` class | Abstract base with save/load functionality |

```python
# Example: How get_stats works
ids = [1, 2, 3, 1, 2]
stats = get_stats(ids)
# Result: {(1, 2): 2, (2, 3): 1, (3, 1): 1}
```

#### 2. **`basic.py`** - Pure BPE Implementation

The simplest possible BPE tokenizer:

- Converts text to UTF-8 bytes
- Iteratively merges most common pairs
- No regex patterns or special tokens

**Best for:** Learning and understanding the core BPE algorithm.

#### 3. **`regex.py`** - Production-Ready Tokenizer

Adds important features used by real-world tokenizers:

- **Regex pre-splitting**: Prevents merges across word boundaries
- **Special tokens**: Handles tokens like `<|endoftext|>`
- **GPT-4 patterns**: Uses the same splitting patterns as GPT-4

```python
# GPT-4 split pattern breaks text intelligently:
# - Contractions: 's, 't, 're, 've, 'll, 'm, 'd
# - Words (with optional leading space)
# - Numbers (1-3 digits at a time)
# - Punctuation
# - Whitespace
```

#### 4. **`gpt4.py`** - GPT-4 Compatible Tokenizer

A wrapper around `RegexTokenizer` that:

- Loads pretrained GPT-4 vocabulary (`cl100k_base`)
- Handles the historical byte shuffle quirk
- Produces identical output to OpenAI's tiktoken

---

## 📦 Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/minbpe.git
cd minbpe

# Install in development mode
pip install -e .

# Install dependencies for GPT-4 tokenizer
pip install tiktoken regex
```

---

## 🚀 Usage

### Basic Tokenizer

```python
from minbpe import BasicTokenizer

# Create and train
tokenizer = BasicTokenizer()
text = "aaabdaaabac"
tokenizer.train(text, vocab_size=256 + 3)  # 256 bytes + 3 merges

# Encode text to tokens
tokens = tokenizer.encode("aaabdaaabac")
print(tokens)  # [258, 100, 258, 97, 99]

# Decode tokens back to text
decoded = tokenizer.decode(tokens)
print(decoded)  # "aaabdaaabac"
```

### Regex Tokenizer (Recommended)

```python
from minbpe import RegexTokenizer

# Create and train
tokenizer = RegexTokenizer()
tokenizer.train(your_text, vocab_size=1000)

# Register special tokens
tokenizer.register_special_tokens({
    "<|endoftext|>": 1000,
    "<|pad|>": 1001,
})

# Encode with special token handling
tokens = tokenizer.encode("Hello <|endoftext|> World", allowed_special="all")

# Save and load
tokenizer.save("my_tokenizer")
# Later...
tokenizer.load("my_tokenizer.model")
```

### GPT-4 Tokenizer (Pretrained)

```python
from minbpe import GPT4Tokenizer

# No training needed - uses OpenAI's pretrained vocabulary
tokenizer = GPT4Tokenizer()

# Encode exactly like GPT-4
tokens = tokenizer.encode("Hello, world! 🌍")

# Verify against tiktoken
import tiktoken
enc = tiktoken.get_encoding("cl100k_base")
assert tokens == enc.encode("Hello, world! 🌍")
```

---

## 🔬 How BPE Works

### Step-by-Step Example

Let's train BPE on the text `"aaabdaaabac"`:

**Initial state:** Convert to bytes

```text
[97, 97, 97, 98, 100, 97, 97, 97, 98, 97, 99]
 a   a   a   b   d    a   a   a   b   a   c
```

**Iteration 1:** Find most common pair → `(97, 97)` appears 4 times

```text
Replace (97, 97) with 256:
[256, 97, 98, 100, 256, 97, 98, 97, 99]
 aa   a   b   d    aa   a   b   a   c
```

**Iteration 2:** Find most common pair → `(256, 97)` appears 2 times

```text
Replace (256, 97) with 257:
[257, 98, 100, 257, 98, 97, 99]
 aaa  b   d    aaa  b   a   c
```

**Iteration 3:** Find most common pair → `(257, 98)` appears 2 times

```text
Replace (257, 98) with 258:
[258, 100, 258, 97, 99]
aaab  d   aaab  a   c
```

**Final vocabulary:**

```text
256 = "aa"
257 = "aaa" (256 + 97 = "aa" + "a")
258 = "aaab" (257 + 98 = "aaa" + "b")
```

---

## 📁 File Structure

```text
minbpe/
├── __init__.py      # Package exports
├── base.py          # Base Tokenizer class + helper functions
├── basic.py         # BasicTokenizer - pure BPE
├── regex.py         # RegexTokenizer - with patterns & special tokens
└── gpt4.py          # GPT4Tokenizer - pretrained GPT-4 compatible

tests/
├── test_tokenizer.py    # Comprehensive test suite
└── taylorswift.txt      # Test data file

setup.py                 # Package installation
README.md                # This file
```

---

## 🧪 Testing

```bash
# Run all tests
pytest tests/test_tokenizer.py -v

# Run specific test
pytest tests/test_tokenizer.py::test_wikipedia_example -v

# Test GPT-4 compatibility
pytest tests/test_tokenizer.py::test_gpt4_tiktoken_equality -v
```

### Test Coverage

| Test | Description |
|------|-------------|
| `test_encode_decode_identity` | Ensures `decode(encode(x)) == x` |
| `test_gpt4_tiktoken_equality` | Verifies GPT-4 tokenizer matches tiktoken |
| `test_wikipedia_example` | Classic BPE example from Wikipedia |
| `test_save_load` | Tests serialization/deserialization |

---

## 🔗 References

### Papers

- [Neural Machine Translation of Rare Words with Subword Units](https://arxiv.org/abs/1508.07909) - Original BPE for NLP
- [Language Models are Unsupervised Multitask Learners](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf) - GPT-2 paper

### Code References

- [OpenAI GPT-2 Encoder](https://github.com/openai/gpt-2/blob/master/src/encoder.py)
- [tiktoken](https://github.com/openai/tiktoken) - OpenAI's official tokenizer
- [sentencepiece](https://github.com/google/sentencepiece) - Google's tokenizer

### Educational Resources

- [Karpathy's Let's Build GPT Tokenizer](https://www.youtube.com/watch?v=zduSFxRajkE) - Video tutorial
- [Hugging Face Tokenizers Course](https://huggingface.co/learn/nlp-course/chapter6/1)

---

## 📊 Quick Reference: Token Counts

| Text | BasicTokenizer | GPT-4 |
|------|----------------|-------|
| "hello" | 5 bytes | 1 token |
| "Hello World" | 11 bytes | 2 tokens |
| "안녕하세요" | 15 bytes | 5+ tokens |
| "🍕" | 4 bytes | 3 tokens |

---

## 🤝 Contributing

This is an educational project. Feel free to:

- Open issues for questions
- Submit PRs for improvements
- Fork for your own experiments

---

## 📜 License

MIT License - Use freely for learning and projects!

---

---

**Remember:** Every time ChatGPT makes a math mistake or counts letters wrong, you now know who to blame—*the tokenizer*! 🎯

---

## Case study

The full write-up of MinBPE Tokenizer (problem, architecture, trade-offs and results) lives at **[kunjshah.in/projects/minbpe-tokenizer](https://kunjshah.in/projects/minbpe-tokenizer)**.

Built by [Kunj Shah](https://kunjshah.in), AI engineer.
