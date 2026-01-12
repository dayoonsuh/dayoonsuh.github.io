---
layout: page
title: 3D CAD Generation
description: CAD model generation using VLM
img: assets/img/CAD/redpot.png
importance: 2
category: Robotics
related_publications: false
---

# CAD Model Generation using Vision-Language Models (VLMs)

This project explores **CAD model generation with Vision-Language Models (VLMs)** by translating natural-language requirements into **parametric, editable CAD programs** rather than static meshes. A common limitation of generative 3D outputs is that they can look reasonable but fail practical design needs such as **dimension control, symmetry, manufacturability, and easy iteration**.

## Overview

We designed and implemented an end-to-end pipeline that turns a user’s text prompt (and optionally reference images) into a structured CAD program, then improves it through evolution and iterative feedback.

## What We Built

- **Intent understanding:** Parses user requirements from text and images
- **Program generation:** Produces **parametric CAD code** (e.g., sketches, extrusions, fillets, hole patterns)
- **Preview & validation:** Renders previews and runs lightweight **constraint checks** (e.g., dimensions, symmetry, placement validity)
- **Iterative refinement:** Updates the CAD program based on detected issues and natural-language feedback

## Motivation

Instead of generating a single “finished” 3D mesh, this project focuses on generating **editable CAD** that supports real design workflows. By representing geometry as a parametric program, the output becomes easier to modify (e.g., “make the handle thicker,” “add four symmetric holes,” “increase height by 20%”) and more compatible with engineering constraints.

## Evolutionary Refinement (Evolution Algorithm)

Instead of generating one CAD program and hoping it satisfies all constraints, we treat CAD generation as a **search problem** over program space.

### Representation
Each candidate solution is a **parametric CAD program** (a sequence of operations + parameters), e.g., sketch primitives → extrude/revolve → boolean ops → fillets/chamfers → patterned features.

### Fitness / Scoring
We score candidates with a weighted objective that can include:
- **Requirement match:** alignment with prompt and target features
- **Constraint satisfaction:** dimension bounds, symmetry, feature placement validity, etc.
- **Geometric validity:** watertightness / non-degenerate operations / no obvious self-intersections
- **Optional visual similarity:** rendered-view alignment to a reference image or target silhouette

### Search Loop
At each iteration:
1. **Initialize** a population from VLM-generated programs (diverse samples)
2. **Evaluate** each candidate by rendering + running checks to compute fitness
3. **Select** top-performing candidates (and optionally keep an elite set)
4. **Mutate / recombine** programs:
   - parameter mutations (e.g., thickness, radius, offsets)
   - operator-level edits (add/remove/move features)
   - structure edits (swap order, change primitive type, adjust patterns)
5. **Repeat** until constraints are met or improvements plateau

This approach is especially effective when small parameter changes meaningfully affect validity (e.g., hole collisions, symmetry breaks, invalid booleans), and when the best result requires multiple coordinated edits.

## Motivation

Instead of generating a single “finished” 3D mesh, this project focuses on generating **editable CAD** that supports real design workflows. By representing geometry as a parametric program, the output becomes easier to modify and more compatible with engineering constraints—while the evolution algorithm provides a principled way to **search for constraint-satisfying designs**.

## Outcome

The resulting system produces CAD models that are:
- **Editable:** Changes can be applied by modifying parameters or operations
- **Structured:** Geometry is built from meaningful operations rather than implicit surfaces
- **Constraint-aware:** Validation + evolutionary search reduces common errors (wrong scale, broken symmetry, missing features)
- **Iterative:** Supports a natural “generate → evaluate → evolve” loop
<!-- 
<div class="row">
  <div class="col-12 mt-3">
    {% include figure.liquid loading="eager" path="assets/img/CAD/redpot.png" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="row">
  <div class="col-12 mt-3">
    {% include figure.liquid loading="eager" path="assets/img/CAD/chair.png" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

<div class="row">
  <div class="col-12 mt-3">
    {% include figure.liquid loading="eager" path="assets/img/CAD/scooter.png" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div> -->



