# 🧠 Deep Learning

[← Back to main README](../README.md)

---

### Q: Explain backpropagation in your own words.

**Answer:**
Backpropagation computes the gradient of the loss function with respect to every weight in the network using the **chain rule**, propagating error signals backward from the output layer to the input layer. Each layer's gradient is computed from the gradient of the layer after it, multiplied by the local derivative — this avoids recomputing shared sub-expressions and makes training deep networks computationally tractable. These gradients are then used by an optimizer (e.g., SGD, Adam) to update weights in the direction that reduces loss.

---

### Q: What is the vanishing/exploding gradient problem, and how do modern architectures address it?

**Answer:**
In deep networks, gradients are products of many partial derivatives during backpropagation; if these are consistently < 1 (e.g., with sigmoid/tanh saturating), gradients shrink toward zero in early layers (**vanishing**), stalling learning. If consistently > 1, gradients grow uncontrollably (**exploding**), destabilizing training. Fixes include: **ReLU-family activations** (non-saturating for positive inputs), **residual/skip connections** (ResNets, which give gradients a direct path), **batch/layer normalization**, careful **weight initialization** (Xavier/He), and **gradient clipping** for exploding gradients.

---

### Q: What is the difference between batch, mini-batch, and stochastic gradient descent?

**Answer:**
**Batch GD** computes the gradient using the entire training set per update — accurate but slow and memory-heavy. **Stochastic GD (SGD)** updates using a single example at a time — fast and noisy, which can help escape local minima but makes convergence erratic. **Mini-batch GD** (the standard in practice) uses small batches (e.g., 32–256 examples) — balances computational efficiency (via vectorization/GPU parallelism) with a reasonable gradient estimate, and the noise acts as a mild regularizer.

---

### Q: Explain what Dropout does and why it helps prevent overfitting.

**Answer:**
Dropout randomly "drops" (zeroes out) a fraction of neurons during each training forward pass, forcing the network to not rely too heavily on any single neuron or co-adapted group of neurons. This acts like training an ensemble of many thinned sub-networks that share weights, and at test time the full network is used (with weights scaled, or using "inverted dropout" during training) to approximate averaging over that ensemble — reducing overfitting, especially in large fully-connected layers.

---

### Q: What is Batch Normalization and why does it accelerate training?

**Answer:**
Batch Norm normalizes the activations of a layer (zero mean, unit variance) across a mini-batch, then applies a learnable scale (γ) and shift (β). This reduces **internal covariate shift** (the distribution of layer inputs changing as earlier layers' weights update), allowing higher learning rates and faster, more stable convergence. It also has a mild regularizing effect due to the noise introduced by batch statistics, sometimes reducing the need for dropout.

---

### Q: Compare CNNs and RNNs — what problem is each architecturally suited for, and why?

**Answer:**
**CNNs** use convolutional filters with shared weights and local receptive fields, exploiting **spatial locality and translation invariance** — ideal for images, where nearby pixels are correlated and patterns (edges, textures) recur across positions. **RNNs** maintain a hidden state updated sequentially across time steps, exploiting **temporal/sequential dependencies** — suited for sequences like text or time series, though vanilla RNNs struggle with long-range dependencies (mitigated by LSTM/GRU gating, and largely superseded by Transformers for long sequences).

---

### Q: Explain the intuition behind the attention mechanism in Transformers.

**Answer:**
Attention lets each token in a sequence dynamically decide **how much to focus on every other token** when computing its representation, rather than relying on a fixed-size hidden state (as RNNs do) or a fixed local window (as CNNs do). Concretely, for each token a **Query** vector is compared (dot product) against **Key** vectors of all tokens to get attention weights (via softmax), which are used to compute a weighted sum of **Value** vectors. This allows the model to capture long-range dependencies in parallel (unlike RNNs' sequential processing), which is a major reason Transformers train faster and scale better.

**Follow-up:** Why do Transformers need positional encodings if attention itself is position-agnostic?

---

### Q: What is transfer learning, and when would you fine-tune vs. use a model as a fixed feature extractor?

**Answer:**
Transfer learning reuses a model pretrained on a large dataset (e.g., ImageNet, or a large text corpus) as a starting point for a new, often smaller-data task, since early/general layers tend to learn transferable features (edges, textures, or general language patterns). Use it as a **fixed feature extractor** (freeze pretrained layers, train only a new head) when your target dataset is small and similar to the source domain. **Fine-tune** (unfreeze some/all layers with a low learning rate) when you have more target data or the target domain differs meaningfully from the source, allowing the model to adapt its learned representations.

---

### Q: What's the difference between a loss function and a metric, and why can't you always optimize the metric directly?

**Answer:**
A loss function is what the optimizer directly minimizes during training and must be **differentiable** (for gradient-based methods); a metric is what you use to evaluate/report model quality and can be any measure relevant to the business (accuracy, F1, business revenue). Many useful metrics (accuracy, F1, AUC) are **non-differentiable or non-smooth**, so you optimize a differentiable proxy loss (cross-entropy, hinge loss) that correlates with the metric you actually care about, then evaluate using the real metric.

---

### Q: Explain the difference between L2 regularization and weight decay in the context of Adam optimizer.

**Answer:**
In plain SGD, L2 regularization (adding λ‖w‖² to the loss) and weight decay (directly shrinking weights by a factor each step) are mathematically equivalent. But with **adaptive optimizers like Adam**, they're **not equivalent** — L2 regularization gets folded into the gradient and then scaled by Adam's adaptive per-parameter learning rates, which weakens its regularizing effect inconsistently across parameters. **AdamW** decouples weight decay from the gradient-based update, applying it directly to the weights, which was shown empirically to generalize better than naive L2 regularization with Adam.

---

### Q: What is the exploding/vanishing gradient problem specific to vanilla RNNs, and how do LSTMs solve it?

**Answer:**
In vanilla RNNs, the same weight matrix is applied repeatedly across time steps, so gradients during backprop-through-time are products of many Jacobians — this compounds multiplicatively and tends to vanish or explode over long sequences, making it hard to learn long-range dependencies. **LSTMs** introduce a **cell state** with an additive update path controlled by gating mechanisms (forget, input, output gates), so gradients can flow through the cell state with much less multiplicative decay — the additive structure is the key architectural fix, similar in spirit to residual connections in CNNs.

---

### Q: What is the difference between generative adversarial networks (GANs) and variational autoencoders (VAEs) at a high level?

**Answer:**
**VAEs** learn a probabilistic latent space by encoding inputs into a distribution (mean/variance) and decoding samples from it, optimizing a reconstruction loss plus a KL-divergence term that regularizes the latent space toward a prior (usually standard normal) — this gives a smooth, interpretable latent space but often produces blurrier samples. **GANs** train a generator and discriminator adversarially (the generator tries to fool the discriminator into thinking generated samples are real) — this typically produces sharper, more realistic samples but training is less stable (mode collapse, oscillation) and there's no explicit likelihood being optimized.

---

### Q: How would you decide the right learning rate, and what does a learning rate schedule / warmup do?

**Answer:**
Too high a learning rate causes the loss to diverge or oscillate; too low causes painfully slow convergence and risk of getting stuck in poor local minima/plateaus. A common approach is a **learning rate range test** (increase LR exponentially over a few iterations, plot loss, pick the value just before divergence) combined with schedules like **cosine decay** or **step decay** to reduce LR as training progresses. **Warmup** (starting with a small LR and ramping up over the first few hundred/thousand steps) is especially important for Transformers, since large early updates with unstable gradient statistics (particularly with Adam) can destabilize training before the model has "settled."

---

### Q: What is the difference between a discriminative fine-tuning approach and prompting/in-context learning for large language models?

**Answer:**
**Fine-tuning** updates the model's weights on task-specific labeled data (fully or via parameter-efficient methods like LoRA), permanently specializing the model for that task — requires labeled data and compute but tends to yield strong, consistent task performance. **In-context learning/prompting** provides task instructions and/or a few examples directly in the input at inference time without updating any weights — cheap and flexible (no training needed), leveraging patterns the model already learned during pretraining, but performance can be less consistent and is sensitive to prompt phrasing and example selection.

---

### Q: Explain what "overfitting to the validation set" means in a deep learning context, and how to avoid it.

**Answer:**
If you repeatedly tune hyperparameters, architecture choices, or early-stopping points based on validation set performance, you effectively "leak" information about the validation set into your model selection process — your reported validation metric becomes an overly optimistic estimate of true generalization, even though the validation data was never directly used for gradient updates. Mitigate by keeping a **held-out test set** touched only once at the very end, using **cross-validation** for more robust hyperparameter selection, and limiting the number of tuning iterations/experiments run against the same validation split.

---
