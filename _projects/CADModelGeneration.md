---
layout: page
title: 3D CAD Generation
description: CAD model generation using VLM
img: assets/img/CAD/redpot.png
importance: 2
category: etc
related_publications: false
---

# CAD Model Generation using Vision-Language Models (VLMs)

This project explores **CAD model generation with Vision-Language Models (VLMs)** by translating natural-language requirements into **parametric, editable CAD programs** rather than static meshes. A common limitation of generative 3D outputs is that they can look reasonable but fail practical design needs such as **dimension control, symmetry, manufacturability, and easy iteration**.


## Overview
We designed and implemented an end-to-end pipeline that turns a user’s prompt and target media (image/video) into a structured CAD program, then improves it through iterative evolution and feedback.

### Fitness / Scoring
We score each candidate using an **LLM-based visual critique** conditioned on the target media and **six canonical renders** (**top, bottom, front, back, left, right**).

- **Input:** target (image/video) + 6 rendered views (PNG) + a structured rubric prompt  
- **Output (strict JSON):**  
  - `keep`: what already matches the target  
  - `improve`: concrete, actionable edits to reduce mismatch  
  - `score` (1–10): overall resemblance using the full scale  

The **score** is used as the primary fitness signal, while `improve` provides targeted guidance for the next generation (e.g., proportions, feature placement, curvature/fillets, thickness/clearances).

### Evolutionary Refinement (Top-2 Guided Evolution)
We refine designs with a simple **top-2 loop** driven by critique scores.

1. **Select parents (Top-2):** pick the two highest-scoring candidates from critique JSONs.
2. **Build context:** provide the target media, original prompt, and both parents’ artifacts (**CadQuery code + 6-view renders**).
3. **Generate offspring:** the model outputs a new **executable CadQuery program** (`result`) per call; we sample multiple offspring in parallel for diversity.
4. **Iterate:** render and critique the new candidates, then repeat the loop for further improvements.
