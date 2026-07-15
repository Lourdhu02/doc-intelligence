# InduOCRBench — HuggingFace Dataset

**URL:** https://huggingface.co/datasets/qihoo360/InduOCRBench
**Paper:** arXiv 2605.00911 (ACL 2026 Industry Track)

## Overview
OCR benchmark for industrial RAG systems. 570 PDFs / 3,402 pages across 11 challenge types.

## Challenge Types
ComplexBackground, HighPixel, UltraLong, MultiColumn, UltraWide, Watermark, CrossPageTable, VisualStyle, MultiFont, Long, Normal

## Dual Evaluation Tracks
1. OCR Fidelity — character/structure-level metrics
2. RAG Impact — end-to-end retrieval + generation (2,071 QA pairs)

## Key Finding
High OCR accuracy ≠ strong RAG performance. VisualStyle: 82.9% OCR → 52.8% RAG (30.1pt gap)
