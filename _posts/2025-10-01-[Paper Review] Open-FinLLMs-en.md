---
title: "[Paper Review] A Review of the FinLLMs Paper"
date: 2025-10-01 18:30:00 +09:00
categories: [Paper, VLM]
tags: [Paper Review, Open-FinLLMs, Financial LLM, VLM, LLM-Instruct]
description: "A detailed review of the Open-FinLLMs paper, which introduces a new suite of open multimodal large language models for financial applications."
language: en
postid: 8
---

### Open-FinLLM
This post reviews the following paper: [Open-FinLLMs: Open Multimodal Large Language Models for Financial Applications](https://arxiv.org/abs/2408.11878).

---
### 1. Background and Motivation

The paper addresses the key limitations of existing LLMs in the financial sector, namely their difficulty in understanding non-textual data and the **unique nuances** (`특유의`) of financial terminology.

Previous attempts to overcome these limitations have faced their own set of challenges:

* **Lack of Domain-Specific Corpora:** Models were often trained or tuned on limited financial corpora.
    > restricting their ability to fully capture the complexity of financial knowledge, language, and data types. For example, FinTral uses only 20 billion tokens for continual pre-training.
    * The paper suggests that 20B tokens are insufficient for deep financial understanding, although it does not propose a specific standard for what is "enough."

* **Inability to Understand Multimodal Data:** Previous models lacked the ability to process diverse financial data types.
    > they show limited multimodal capabilities, lacking support for tabular, time-series, and chart data. missing critical aspects of real-world tasks such as portfolio optimization and trend analysis.

* **Limited Evaluation Scenarios:** The benchmarks used to evaluate these models were often too narrow.
    > narrow scenarios / Zero/fewshot performance, multimodal reasoning, and financial decision-making tasks remain underexplored, limiting real-world applicability

---
### 2. The Paper's Proposed Method

To address these issues, the authors introduce a new suite of models built on a significantly larger and more diverse dataset. The core contributions are:

* **A massive 52-billion-token corpus** was created for pre-training.
    > comprising text, tabular, and timeseries data from high-quality financial sources such as reports, papers, and market data for the first time. This innovative pre-training strategy incorporating extensive data modalities equips FinLLaMA with deep financial insights and analytical capabilities

* **FinLLaMA:** The foundational model, pre-trained on the 52B token corpus.
* **FinLLaMA-Instruct:** An instruction-tuned version of FinLLaMA, fine-tuned on 573K instruction samples.
* **FinLLaVA:** A multimodal version that incorporates 1.43 million financial multimodal instruction pairs, including images, text, charts, and tabular data.
    > Unlike traditional methods that primarily focus on standard image-text pairs, our fine-tuning process is the first to incorporate chart image-text pairs and images of tabular layouts.

#### Key Training Strategies

* **Corpus Size and Quality:** The authors used a 52B token corpus, a significant increase from the 20B used in some previous models. They prioritized quality by excluding noisy web data.
    > Unlike existing models, we chose not to use web data due to its higher noise level compared to other data sources.)
* **Preventing Catastrophic Forgetting:** To ensure the model retained its general knowledge while learning financial specifics, the training data included both the financial corpus and a large **general-domain dataset** (from the 15T token FineWeb dataset).

---
### 3. Training Process Details

#### Instruction Tuning Details

> For instruction tuning, we utilize FinLLaMA as the backbone model and conduct training on 8 NVIDIA A100 80GB GPUs for 6 hours. The model is optimized using Qlora (Dettmers et al., 2024) via AutoTrain 4 , configured with a block size and model maximum length of 4096. We train the model over 2 epochs with a batch size of 1 and a learning rate of 0.0002. Parameter-efficient tuning is achieved with LoRA settings of r = 64, α = 128, and no dropout, using INT4 quantization. All linear modules are targeted, with right-aligned padding. The AdamW optimizer (Loshchilov and Hutter), coupled with a cosine scheduler, is used for optimization, along with gradient accumulation set to 4.

#### Multimodal Instruction Fine-tuning

> Our approach follows the training framework established by LLaVA1.5 (Liu et al., 2024b), implementing a two-stage instruction-tuning process.

* **Multimodal Alignment Stage:**
    > The key objective is to train a two-layer MLP projector to bridge the gap between the vision encoder’s features and the LLM’s embedding. For each input, consisting of an image Xv, instructions Xinstruct that may involve single-turn or multi-turn conversations, and the target answer Xa, the vision encoder processes the image data to generate a vision feature: Zv = g(Xv).
* **Supervised Fine-tuning (SFT) Stage:**
    > **Training Details**: In the multi-modal alignment stage, we set the global batch size to 128, the learning rate to 1 × 10−3 , with a warm-up ratio of 0.03 and cosine decay. Training uses bf16 and tf32 precision for stability and acceleration. Weight decay is set to 0.0, and the model’s maximum length is 2048 tokens. We train on eight NVIDIA HGX H20 GPUs, completing one epoch in approximately 30 hours. In the SFT stage, we set the global batch size to 256, the learning rate to 2 × 10−5 , with a warmup ratio of 0.05 and cosine decay. The model’s maximum length is increased to 8192 tokens for longer sequences. Weight decay remains at 0.0, and training runs for one epoch with the same precision settings for efficiency and performance.

---
### 4. Limitations and Future Directions

The paper identifies several areas for future work:

* **Model Scale:** The current 8B model size leaves room for exploration into both smaller, more efficient models and larger, higher-performance models.
* **Multilingual Support:** The model is currently English-only and could be extended to support multiple languages.
* **Scenario Coverage:** The training and evaluation scenarios are still limited, which may not fully reflect real-world complexity.

---
### Appendix: Data Preparation

#### Data Cleaning and Preparation

> For tabular and time-series data, we first split them into rows into samples of approximately 2,048 tokens each, formatting each block in HTML and ensuring each includes the table header for context. We then combine all datasets and further chunk the entire dataset into 8,192 token blocks, readying the data for efficient processing by the model.

#### Data for Multimodal Training

> Our table data is sourced from the Fintabnet and Marketing categories of SynthTabNet, featuring real financial and marketing tables with diverse layouts. Each table includes parsed bounding boxes, allowing us to reconstruct structure-aware prompts, which are more accurate than image-based descriptions. To ensure OCR quality, we limit tables to a maximum size of 10 × 10 to avoid resolution-related cell blurring. In the SFT stage, we design seven financial-specific tasks (Appendix E.3), randomly sampled in the prompts. Figure 2 shows an example table and how we align and generate SFT data. C.2 Chart Our chart dataset is derived from Unichart, Chart2Text, and ChartQA, covering real financial data, marketing trends, and varied visual styles. We focus on numerical and financial charts to support robust quantitative analysis. An example questionanswer pair is shown in Figure 3. During SFT, we develop seven chart-specific tasks (Appendix E.3) to enhance the model’s ability to interpret financial charts and extract insights. Figure 4 illustrates an example chart used in SFT data generation.