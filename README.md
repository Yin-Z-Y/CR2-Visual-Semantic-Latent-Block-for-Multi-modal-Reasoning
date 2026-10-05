<div align="center">

<h1>Construct, Replay, and Reason</h1>
<h3>Visual-Semantic Latent Block for Multi-modal Reasoning</h3>

<p><b>A dedicated latent reasoning state, constructed from visual evidence and semantic intent.</b></p>

<!-- <p>
  <a href="https://anonymous.4open.science/r/CR2"><img src="https://img.shields.io/badge/Project-CR%C2%B2-6366F1?style=flat-square" alt="Project"></a>
  <img src="https://img.shields.io/badge/Backbone-Qwen2.5--VL-0F766E?style=flat-square" alt="Qwen2.5-VL backbone">
  <img src="https://img.shields.io/badge/Model_Scale-3B_%7C_7B-475569?style=flat-square" alt="3B and 7B models">
  <img src="https://img.shields.io/badge/Benchmarks-6-D97706?style=flat-square" alt="Six benchmarks">
</p> -->


<p>
  <a href="#updates">🔥 Updates</a> &nbsp;·&nbsp;
  <a href="#abstract">📖 Abstract</a> &nbsp;·&nbsp;
  <a href="#motivation">🔍 Motivation</a> &nbsp;·&nbsp;
  <a href="#method">🧩 Method</a> &nbsp;·&nbsp;
  <a href="#benchmarks">🏆 Benchmarks</a>
</p>


</div>



---

<a id="updates"></a>

## 🔥 Updates

- **[2026-10-05]** Added this research overview, including the preliminary analysis, construct-and-replay framework, and benchmark results with 3B and 7B backbones.

<!-- Add future announcements here in reverse chronological order. Only announce code, checkpoints, or data after their release links are available. -->

<a id="abstract"></a>

## 📖 Abstract

Latent Visual Reasoning (LVR) performs intermediate computation in a visual latent space, allowing reasoning states to carry visual information without verbalizing it. It constructs this space by aligning hidden-state-derived latents with selected visual tokens. However, our analysis identifies two limitations: direct alignment overlooks the distinct representational roles of high-level hidden states and local visual tokens, while using ground-truth RoI visual tokens during training but self-generated latents during inference introduces a substantial training–inference gap. We propose **CR²**, a two-pass framework that **Constructs** and **Replays** a visual-semantic latent block for multi-modal **Reasoning**. In Pass-I, a latent head transforms sampled seeds into a multi-token latent block, with auxiliary supervision from RoI visual features and answer-related content. The constructed latent block serves as the reasoning state and is replayed in Pass-II for answer generation. CR² employs the same construct-and-replay procedure during training and inference, thereby reducing their discrepancy. Experiments with 3B and 7B backbones across six multimodal benchmarks demonstrate consistent improvements. At the 7B scale, our method outperforms the strongest compared baselines by **3.3**, **3.2**, **6.4**, and **3.5 percentage points** on RealWorldQA, OCRBench, BLINK Counting, and BLINK Spatial Relation, respectively.

<a id="motivation"></a>

## 🔍 Motivation: Revisiting Latent Visual Reasoning

**Can a local visual token serve as a contextual reasoning state?** LVR grounds latent reasoning through direct alignment with question-relevant RoI tokens. Our preliminary analysis reveals two limitations of this design.

<p align="center">
  <img src="assets/pre_study.png" alt="Preliminary LVR analysis: hidden-to-visual alignment, representation mismatch, and training–inference mismatch" width="100%">
</p>
<p align="center"><sub>Preliminary analysis from the paper: the LVR paradigm and its two central mismatches.</sub></p>

| Observation                      | Evidence                                                     | Implication for latent construction                          |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Representation-role mismatch** | Aligned hidden states and RoI visual tokens remain separated in the t-SNE visualization; increasing alignment strength degrades accuracy. | Reasoning states need to retain contextual semantics while incorporating localized visual evidence. |
| **Training–inference mismatch**  | Training uses ground-truth RoI tokens as intermediate inputs, whereas inference generates its own latents. The reported accuracy gap is at least **12.8 percentage points**. | The reasoning state should be constructed through the same procedure during training and inference. |

> 💡 **Our idea:** Construct a dedicated visual-semantic latent block for reasoning. Use RoI features and answer-related content to supervise its construction, then condition answer generation on the constructed block during both training and inference.

<a id="method"></a>

## 🧩 Method: Construct, Replay, and Reason

CR² uses two passes to construct a visual-semantic latent block and replay it for answer generation.

<p align="center">
  <img src="assets/framework.png" alt="CR² framework: construct visual-semantic residual latents in Pass-I, then replay the latent block for answer generation in Pass-II" width="100%">
</p>
<p align="center"><sub>Construct visual-semantic latents in Pass-I, then replay them for reasoning in Pass-II.</sub></p>

### ① Construct · Pass-I

The model processes the image, question, and a block of sampled seeds in one causal forward pass. A latent head extracts **visual evidence** and **semantic intent** from the hidden states, fuses them into residual updates, and applies these updates to the seeds to construct the latent block. During training, RoI visual features guide the visual stream, while answer-related content guides the semantic stream.

### ② Replay and Reason · Pass-II

The constructed latent block is replayed alongside the image and question to generate the answer. **Training and inference follow the same construct-and-replay procedure.** RoI features and answer-related annotations are used only as training supervision; inference requires the image and question.

<a id="benchmarks"></a>

## 🏆 Main Results

We evaluate CR² on **six multimodal benchmarks** with **Qwen2.5-VL-3B and Qwen2.5-VL-7B** backbones. Main results use a fixed latent budget of **$S=12$**. The SFT baselines are trained on the same Visual CoT data as CR² to control for the effect of training data.

### Benchmark Coverage

| Benchmark      | Evaluated capability                                         |
| :------------- | :----------------------------------------------------------- |
| HallusionBench | Multimodal hallucination                                     |
| MME            | General multimodal capabilities across 14 subtasks           |
| MathVista      | Mathematical reasoning in visual contexts                    |
| OCRBench       | Text recognition and understanding                           |
| RealWorldQA    | Visual understanding in real-world scenarios                 |
| BLINK          | Fine-grained perception and reasoning; Counting, Spatial Relation, and overall score |

### CR²-3B

| Model              | Hallusion |        MME | MathVista | OCRBench |     RWQA | BLINK Counting | BLINK Spatial | BLINK Total |
| :----------------- | --------: | ---------: | --------: | -------: | -------: | -------------: | ------------: | ----------: |
| GPT-4o · reference |      37.5 |     2282.0 |      60.2 |     82.3 |     64.4 |           60.8 |          80.4 |        67.2 |
| Qwen2.5-VL-3B      |      40.9 |     1759.1 |      47.9 |     73.4 |     50.8 |       **62.6** |          81.8 |        44.6 |
| SFT-3B             |      57.3 |     1938.8 |      52.2 |     77.4 |     52.9 |           61.5 |          82.5 |        45.4 |
| DMLR               |      41.4 |     1839.2 |      50.0 |     74.4 |     54.2 |           45.8 |          79.0 |        40.4 |
| LVR-3B             |      53.3 |     2142.6 |      57.5 |     80.3 |     57.6 |           60.5 |      **83.9** |        46.5 |
| LaViT-3B           |      51.1 |     2178.3 |      57.3 |     78.4 |     59.6 |           60.0 |          80.4 |        46.8 |
| LIVR-3B†           |         — |          — |         — |        — |        — |           63.6 |             — |        67.8 |
| **CR²-3B (Ours)**  |  **61.1** | **2189.3** |  **58.7** | **82.2** | **62.2** |           62.5 |          83.2 |    **47.7** |

**Highlights.** CR²-3B improves HallusionBench by **3.8 pp** and RealWorldQA by **2.6 pp** over the strongest compared general 3B baselines. It nearly matches GPT-4o on OCRBench (**82.2 vs. 82.3**) and exceeds its scores on BLINK Counting (**62.5 vs. 60.8**) and Spatial Relation (**83.2 vs. 80.4**).

### CR²-7B

| Model              | Hallusion |        MME | MathVista | OCRBench |     RWQA | BLINK Counting | BLINK Spatial | BLINK Total |
| :----------------- | --------: | ---------: | --------: | -------: | -------: | -------------: | ------------: | ----------: |
| GPT-4o · reference |      37.5 |     2282.0 |      60.2 |     82.3 |     64.4 |           60.8 |          80.4 |        67.2 |
| Qwen2.5-VL-7B      |      37.8 |     1961.4 |      57.3 |     82.0 |     60.4 |           50.4 |          88.8 |        48.5 |
| SFT-7B             |      54.2 |     2084.8 |      61.5 |     82.8 |     61.9 |           60.5 |          88.5 |        52.5 |
| LVR-7B             |  **58.6** |     2185.0 |      63.9 |     80.9 |     60.4 |           62.1 |          86.7 |        52.7 |
| **CR²-7B (Ours)**  |      58.3 | **2258.4** |  **64.9** | **86.0** | **65.2** |       **68.5** |      **92.3** |    **53.4** |

**Highlights.** CR²-7B leads the compared 7B models on **seven of eight reported metrics**, with gains of **3.3 pp** on RealWorldQA, **3.2 pp** on OCRBench, **6.4 pp** on BLINK Counting, and **3.5 pp** on BLINK Spatial Relation. It also exceeds GPT-4o on MathVista, OCRBench, and RealWorldQA.

<sub>All values are percentages except MME, which reports its original score. RWQA denotes RealWorldQA; BLINK Spatial denotes Spatial Relation. Bold marks the best result within each model-size group, excluding the GPT-4o reference and the task-specific LIVR-3B† row. † LIVR-3B uses task-specific fine-tuning, with results taken from its original paper. All results are transcribed from Table 1 of our paper.</sub>

---

<p align="center"><b>Construct a reasoning state. Replay it consistently. Reason with visual-semantic latents.</b><br><sub>CR² · Construct, Replay, and Reason</sub></p>
