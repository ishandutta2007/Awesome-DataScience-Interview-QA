# 📝 NLP

[← Back to main README](../README.md)

---

### Q: Explain the difference between stemming and lemmatization.

**Answer:**
**Stemming** crudely chops word endings using heuristic rules (e.g., "running" → "run", but also "universe" → "univers") without understanding grammar — fast but can produce non-words. **Lemmatization** uses vocabulary and morphological analysis (often with a POS tag) to reduce a word to its true dictionary base form (lemma) — e.g., "better" → "good" — producing valid words but requiring more computation and linguistic resources (dictionaries/POS taggers). Lemmatization is generally preferred when interpretability of the reduced form matters; stemming is fine for quick, resource-constrained preprocessing like search indexing.

---

### Q: What is TF-IDF, and what problem does it solve compared to raw word counts?

**Answer:**
**TF-IDF (Term Frequency–Inverse Document Frequency)** weights a word's importance in a document by how often it appears there (**TF**) discounted by how common it is **across all documents** (**IDF** = log(N / documents containing the term)). This solves the problem where raw word counts overweight extremely common words ("the", "is") that appear everywhere and carry little distinguishing information, while giving more weight to words that are frequent in a specific document but rare across the corpus (which tend to be more topically meaningful).

---

### Q: What is word embedding, and how does Word2Vec learn embeddings?

**Answer:**
Word embeddings represent words as dense, low-dimensional vectors that capture semantic relationships (words with similar meaning/usage end up close together in vector space), unlike sparse one-hot encodings that carry no notion of similarity. **Word2Vec** learns these embeddings via a shallow neural network trained on one of two tasks: **CBOW** (predict a target word from its surrounding context words) or **Skip-gram** (predict surrounding context words from a target word). The learned weight matrix (mapping words to hidden-layer activations) becomes the embedding — words used in similar contexts end up with similar vectors, famously enabling analogies like `king - man + woman ≈ queen`.

---

### Q: How is the Transformer's self-attention different from the attention used in earlier seq2seq (RNN + attention) models?

**Answer:**
Earlier seq2seq attention (e.g., Bahdanau/Luong attention) let a **decoder** attend to encoder hidden states, computed sequentially since the underlying encoder/decoder were RNNs — attention was an add-on to an inherently sequential architecture. **Self-attention** in Transformers lets **every token attend to every other token in the same sequence** (including itself), computed **in parallel** rather than sequentially, since there's no recurrence — this parallelism is a major reason Transformers train much faster on modern hardware (GPUs/TPUs) and can capture long-range dependencies more directly without the information bottleneck of a single RNN hidden state.

---

### Q: What is the difference between BERT and GPT at an architectural level, and why does it affect what tasks each is naturally good at?

**Answer:**
**BERT** uses only the Transformer **encoder**, trained with a **bidirectional masked language modeling** objective (predict a randomly masked token using context from both left and right) — well-suited for understanding tasks (classification, NER, extractive QA) where you have the full input available at once. **GPT** uses only the Transformer **decoder**, trained with a **left-to-right (causal/autoregressive) language modeling** objective (predict the next token given only preceding tokens) — well-suited for generation tasks, since it naturally produces text one token at a time without "seeing the future," which BERT's bidirectional context would otherwise leak during generation.

---

### Q: What is tokenization, and why do modern LLMs use subword tokenization (like BPE) instead of word-level tokenization?

**Answer:**
Tokenization splits text into units (tokens) the model processes. **Word-level tokenization** requires a huge vocabulary to cover all words and still fails on unseen/rare words (out-of-vocabulary problem), especially for morphologically rich languages. **Subword tokenization** (e.g., **Byte-Pair Encoding (BPE)**, WordPiece, SentencePiece) breaks words into smaller frequent sub-units (e.g., "unhappiness" → "un" + "happi" + "ness"), learned from corpus statistics — this keeps vocabulary size manageable, handles rare/unseen words gracefully by falling back to smaller known pieces (even down to individual characters/bytes in the worst case), and lets the model share statistical strength across morphologically related words.

---

### Q: Explain what perplexity measures for a language model, and its limitations as an evaluation metric.

**Answer:**
Perplexity is the exponential of the average negative log-likelihood the model assigns to a held-out test set — intuitively, it measures how "surprised" the model is by real text, with **lower perplexity indicating the model assigns higher probability to (predicts better) the actual next tokens**. Limitations: perplexity is not directly comparable across models with **different tokenizers/vocabularies** (since it's computed per-token, and token granularity differs), and a low perplexity doesn't guarantee the model is good at **downstream tasks** or produces **factually correct or genuinely useful** text — it purely measures how well the model models the statistical distribution of the training-like text, not task usefulness.

---

### Q: How would you build a spam classifier from scratch, from data to deployment?

**Answer:**
1. **Data**: collect labeled spam/ham examples, being mindful of class imbalance (spam is usually the minority class).
2. **Preprocessing**: lowercase, remove/handle punctuation and stopwords, tokenize; consider keeping some "noisy" signals (excessive caps, exclamation marks) since they're often genuinely predictive of spam.
3. **Feature representation**: start simple with **TF-IDF or bag-of-words** with a Naive Bayes or logistic regression baseline (surprisingly strong and fast for spam detection) before reaching for embeddings/deep learning.
4. **Evaluation**: prioritize **precision** if false positives (blocking legitimate email) are more costly than false negatives, which is often true for spam filters, and use PR-AUC given class imbalance.
5. **Deployment**: serve as a lightweight, low-latency model (spam filtering needs to be fast at scale); monitor for **concept drift**, since spam patterns evolve quickly as spammers adapt, requiring periodic retraining on fresh labeled data.

---

### Q: What is Named Entity Recognition (NER), and what makes it a harder problem than simple keyword matching?

**Answer:**
NER identifies and classifies spans of text into predefined categories (person, organization, location, date, etc.). It's harder than keyword matching because entity mentions are **context-dependent and ambiguous** — "Washington" could be a person, a state, or a city depending on context, and entity boundaries aren't always obvious (e.g., "Bank of America" is one entity, not two). Modern NER models use **sequence labeling** (e.g., BIO tagging with a Transformer-based encoder) that considers the surrounding context of each token, rather than matching against a static list of known entity names, which fails on new/unseen entities and ambiguous cases.

---

### Q: What is Retrieval-Augmented Generation (RAG), and why is it useful compared to relying purely on an LLM's parametric knowledge?

**Answer:**
RAG combines a **retrieval system** (e.g., a vector database performing semantic search over a document corpus) with a **generative LLM** — at query time, relevant documents/passages are retrieved and inserted into the model's context, and the LLM generates its answer grounded in that retrieved content. This is useful because it lets the model access **up-to-date or proprietary information** it wasn't trained on (without expensive retraining), and it substantially reduces **hallucination** by grounding responses in verifiable source text, while also enabling **citation** of sources for the generated answer — a pure LLM has no way to cite its "knowledge" or update it without retraining.

---

### Q: How would you evaluate the quality of a text summarization model?

**Answer:**
Automated metrics like **ROUGE** (measures n-gram overlap between generated and reference summaries) are the standard baseline but correlate imperfectly with human judgment of quality — they can reward summaries that copy phrasing without necessarily being coherent or capturing the most important content. Better practice combines ROUGE with **human evaluation** on dimensions like **faithfulness/factual consistency** (does the summary avoid hallucinating facts not in the source), **coherence**, and **relevance/coverage** (are the most important points captured). Increasingly, **LLM-as-judge** evaluation (using a strong LLM to rate summaries against a rubric) is used as a scalable proxy for human evaluation, though it has its own biases worth being aware of.

---
