---
title: "Deep Learning and Foundation Models for Earth and Planetary Sciencess"
date: 2026-04-10
draft: false
tags: ["Research", "Foundation Model"]
author: "Jichao Fang"
description: "Geo-Foundation Models and Applications"
featured_image: "title.png"
cover:
    image: "title.png"
    relative: false
---


{{< youtube XBxPPooBhPU >}}


[Download the presentation slides](slides.pdf)

## Overview

On April 10, 2026, Jichao Fang, a Ph.D. candidate in the Department of Earth, Atmosphere and Environment at Northern Illinois University, presented his research on deep learning and foundation models for Earth and planetary sciences. Jichao is also a Master's student in Computer Science and is expected to graduate this year.

The talk opened with a motivation for applying machine learning to Earth observation data — a domain that archives petabyte-scale imagery with rich spatial, temporal, and spectral structure, yet comes almost entirely without labels. Jichao then introduced the concept of Geo-Foundation Models, highlighting two notable examples: Prithivi (NASA/IBM), built on a Masked AutoEncoder architecture, and AlphaEarth (Google DeepMind), which encodes Earth's land surface at 10-meter resolution into compact 64-dimensional embeddings released as an analysis-ready data product.

The core of the presentation demonstrated how these embeddings can power downstream scientific tasks with surprisingly lightweight pipelines. Jichao showed results on crop yield estimation across nearly 1,000 U.S. counties — where a tuned SVR model on AlphaEarth embeddings achieved R² = 0.825 — and on landslide susceptibility mapping in mountainous regions of China, where a simple classifier reached high accuracy using the same embedding features.

The talk closed by looking beyond Earth, covering a Moon foundation model from the Luxembourg Space Agency, and Jichao's own work on a global Mars image retrieval system indexing over 26 million CTX images, enabling localization, landform distribution analysis, and similarity-based crater search — all served from a single CPU-only server. Generative applications using VAE and diffusion models for Mars imagery were also briefly showcased.