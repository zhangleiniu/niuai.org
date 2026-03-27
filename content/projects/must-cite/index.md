---
title: "Rider: A Curated Search Engine for Research Papers"
date: 2026-03-25
draft: false
author: "Lei Zhang"
tags: ["Search Engine", "Academic Search", "Machine Learning", "Information Retrieval", "Research Tools"]
cover:
    image: "title.png"
    relative: false
---

# Rider

**Rider** is a search engine for research papers that I built to support my own research workflow.

🔗 Live Demo: http://rider.cs.niu.edu/

---

## Motivation

Existing academic search platforms such as Google Scholar and Semantic Scholar are extremely useful, but they often introduce friction in daily research workflows:

- Noisy or inconsistent metadata
- Limited control over conference-level filtering
- Difficulty focusing on high-quality venues
- Mixed archival vs. non-archival content
- Ranking not always aligned with research needs

To address these issues, Rider focuses on **clean, curated, and conference-aware search for research papers**.

---

## Key Features

Rider is designed around practical research usage:

### Conference-based search
Users can filter papers by specific venues, including top AI/ML conferences, making it easier to focus on high-quality research output.

### Clean and curated dataset
The system emphasizes data quality and consistency, aiming to reduce noise compared to general-purpose academic search engines.

### Fast keyword-based retrieval
The current backend is powered by **Meilisearch**, providing efficient BM25-style ranking for keyword search.

### Ongoing semantic search integration
We are currently working on integrating semantic search capabilities with a student collaborator to improve retrieval beyond keyword matching.

---

## System Design (Current)

- Backend search engine: **Meilisearch**
- Ranking model: BM25-style lexical retrieval
- Data model: curated conference-based paper collection
- Planned extension: semantic embeddings for hybrid search

---

## Future Work

Rider is actively evolving. Planned improvements include:

- Semantic search for paper retrieval
- Better ranking signals for research relevance
- Expanded conference coverage
- Improved metadata normalization
- Potential recommendation features

---

## Status

Active development. Feedback and suggestions are welcome.