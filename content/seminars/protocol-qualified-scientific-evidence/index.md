---
title: "Protocol-Qualified Scientific Evidence for Trustworthy Cross-Paper Comparison"
date: 2026-10-30
draft: false
tags: ["Research", "LLM"]
author: "Harish Varma Siravuri"
description: "Evidence-linked, calibrated, comparability-checked result records for comparing methods across papers"
---

> 🎬 **Video coming soon.** The recording of this talk will be posted here.

## Overview

**Speaker:** Harish Varma Siravuri  
**Date:** Friday, October 30, 2026, 2:30 PM  
**Room:** PM103

Scientific papers are being published far faster than anyone can curate them by hand. So researchers increasingly depend on automatically extracted results (leaderboards, knowledge graphs, and LLM-based extraction) to compare methods across papers. These records are hard to trust, for three reasons. An extractor may copy a value incorrectly. Two values that were extracted correctly may come from different experimental conditions, such as a test split versus a validation split. And a system may name a "winner" that the available evidence does not actually support.

This talk presents a thesis built on protocol-qualified result records. Each record carries three things: a link to the text or table cell that supports it, a calibrated confidence score, and explicit checks on whether it can be compared with other results. The talk covers preliminary results from the published studies and outlines the proposed work ahead.

## Research Questions

- **Evidence-linked extraction:** extracting results from text and tables. A multi-agent framework with a Critic model gives better-calibrated confidence (ICMLA 2026).
- **Review ranking:** ordering records so human reviewers catch the most errors within a limited budget (JCDL 2026).
- **Comparability:** deciding when reported results can be compared. A census of cross-paper disagreements finds that most genuine disagreements come from differences in protocol (AI4SciSci 2026).
- **Answering:** answering comparison questions with answers that are robust, conditional, or flagged as "insufficient evidence."
- **Tracking change:** detecting when new results change a conclusion, and explaining why.
