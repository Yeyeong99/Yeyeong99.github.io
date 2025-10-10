---
title: "[Paper Review] FinLLMs 논문 정리"
date: 2025-10-01 18:30:00 +09:00
categories: [Paper, VLM]
tags: [Paper Review, Open-FinLLMs, Financial LLM, VLM, LLM-Instruct]     # TAG names should always be lowercase
description: 금융 특화 생성형 모델 논문 정리
---

# Open-FinLLM
[Open-FinLLMs: Open Multimodal Large Language Models for Financial Applications](https://arxiv.org/abs/2408.11878)
- 해당 논문 참고해 인용 및 정리

## 설계 배경

- 기존 LLM: non text 데이터 이해 어려움 + 금융이라는 특수 분야가 가진 뉘앙스, 용어 이해 어려움
- 극복을 위한 기존의 방법론들
    - 도메인 특화 corpora가 제한된 채로 continual training, instruction tuning에 사용됨(도메인 특화 corpora가 충분하지 않다): 금융의 복잡성을 이해하는 데에 스스로 제한을 둠: restricting their ability to fully capture the complexity of financial knowledge, language, and data types. For example, FinTral uses only 20 billion tokens for continual pre-training. ⇒ **20b token은 부족 (부족하다고 판단한 기준에 대한 명시는 없음)**
    - 멀티모달 능력이 부족해 non text data 이해 못함: they show limited multimodal capabilities, lacking support for tabular, time-series, and chart data. missing critical aspects of real-world tasks such as portfolio optimization and trend analysis.
    - 평가 시 시나리오가 제한됨 ⇒ narrow scenarios / Zero/fewshot performance, multimodal reasoning, and financial decision-making tasks remain underexplored, limiting real-world applicability

## Open FinLLM의 시도

**52-billion-token corpus** comprising text, tabular, and timeseries data from high-quality financial sources such as reports, papers, and market data for the first time. This innovative pre-training strategy incorporating extensive data modalities equips FinLLaMA with deep financial insights and analytical capabilities

- FinLLaMA: 52b 토큰으로 pre trained

- FinLLaMA-Instruct: 573K 데이터 (instruct + financial)
- FinLLaVA:  1.43b ⇒ 1,430K financial multimodal instruction pairs, including images, text, charts, and tabular data / Unlike traditional methods that primarily focus on standard image-text pairs, our fine-tuning process is the first to incorporate chart image-text pairs and images of tabular layouts.

### Method

- 20billion보다 높임 ⇒ 52 billion
    - 양 늘리고 + 질 확보 ⇒ 노이즈가 많은 웹 데이터 제외  (Unlike existing models, we chose not to use web data due to its higher noise level compared to other data sources.)
- catastrophic forgetting 방지
    - **general domain data** + financial domain corpus
        - general: Fineweb dataset: 15 trillion, cleaned & deduplicated

## 각 단계

### Instruction Tuning Details

- For instruction tuning, we utilize FinLLaMA as the backbone model and conduct training on 8 NVIDIA A100 80GB GPUs for 6 hours. The model is optimized using Qlora (Dettmers et al., 2024) via AutoTrain 4 , configured with a block size and model maximum length of 4096. We train the model over 2 epochs with a batch size of 1 and a learning rate of 0.0002. Parameter-efficient tuning is achieved with LoRA settings of r = 64, α = 128, and no dropout, using INT4 quantization. All linear modules are targeted, with right-aligned padding. The AdamW optimizer (Loshchilov and Hutter), coupled with a cosine scheduler, is used for optimization, along with gradient accumulation set to 4.

### Multimodal Instruction Finetuning

- Our approach follows the training framework established by LLaVA1.5 (Liu et al., 2024b), implementing a two-stage instruction-tuning process.
    - Multimodal Alignment
        - The key objective is to train a two-layer MLP projector to bridge the gap between the vision encoder’s features and the LLM’s embedding.
        - For each input, consisting of an image Xv, instructions Xinstruct that may involve single-turn or multi-turn conversations, and the target answer Xa, the vision encoder processes the image data to generate a vision feature: Zv = g(Xv).
    - Supervised Fine-tuning
- **Training Details**: In the multi-modal alignment stage, we set the global batch size to 128, the learning rate to 1 × 10−3 , with a warm-up ratio of 0.03 and cosine decay. Training uses bf16 and tf32 precision for stability and acceleration. Weight decay is set to 0.0, and the model’s maximum length is 2048 tokens. We train on eight NVIDIA HGX H20 GPUs, completing one epoch in approximately 30 hours. In the SFT stage, we set the global batch size to 256, the learning rate to 2 × 10−5 , with a warmup ratio of 0.05 and cosine decay. The model’s maximum length is increased to 8192 tokens for longer sequences. Weight decay remains at 0.0, and training runs for one epoch with the same precision settings for efficiency and performance.

## 한계

- 8B에 제한됨 ⇒ 작게해서 효율 높이거나 / 크게 해서 성능 높이거나
- 영어에 제한 ⇒ multilingual 확장 가능성
- 시나리오 제한 ⇒ 현실의 일부 사항만 반영

## 데이터 정제 방법

For tabular and time-series data, we first split them into rows into samples of approximately 2,048 tokens each, formatting each block in HTML and ensuring each includes the table header for context. We then combine all datasets and further chunk the entire dataset into 8,192 token blocks, readying the data for efficient processing by the model.

### 멀티모달에서 사용한 데이터

Our table data is sourced from the Fintabnet and Marketing categories of SynthTabNet, featuring real financial and marketing tables with diverse layouts. Each table includes parsed bounding boxes, allowing us to reconstruct structure-aware prompts, which are more accurate than image-based descriptions. To ensure OCR quality, we limit tables to a maximum size of 10 × 10 to avoid resolution-related cell blurring. In the SFT stage, we design seven financial-specific tasks (Appendix E.3), randomly sampled in the prompts. Figure 2 shows an example table and how we align and generate SFT data. C.2 Chart Our chart dataset is derived from Unichart, Chart2Text, and ChartQA, covering real financial data, marketing trends, and varied visual styles. We focus on numerical and financial charts to support robust quantitative analysis. An example questionanswer pair is shown in Figure 3. During SFT, we develop seven chart-specific tasks (Appendix E.3) to enhance the model’s ability to interpret financial charts and extract insights. Figure 4 illustrates an example chart used in SFT data generation.