# Cost Analysis

## Overview

This document provides a comprehensive cost analysis for the anomaly detection system using Google Document AI and Resistant AI. The analysis covers per-document costs, monthly projections, and optimization strategies.

## Cost Components

### 1. Google Document AI (Extraction)

Document AI is billed **per page** and **by processor**:

- **Form Parser** (good default for utility bills, tables & key-values): **$30 / 1,000 pages** (= $0.03/page) for up to 1M pages/month; $20 / 1,000 thereafter
- **Enterprise Document OCR** (plain OCR text only): **$1.50 / 1,000 pages** (= $0.0015/page)
- **Pretrained "Utility parser"** (if Google grants access): **$0.10 per 10 pages** (= $0.01/page). Note: **limited-access** processor—you must request enablement
- **Custom processors** (only if you train your own): hosting **$0.05/hour** per deployed version (in addition to per-page prediction)

**What counts as a page**: PDF page = 1, image = 1; **DOCX up to 3,000 characters = 1 page; XLSX each sheet/tab = 1 page; PPTX each slide = 1 page.**

### 2. Resistant AI Document Forensics (Tamper/Fraud)

- **Pricing is not public**; sold via **Google Cloud Marketplace** / **AWS Marketplace** and typically **quote-based** by volume & deployment
- They provide an **ROI calculator** but not rates; marketing claims "<20 seconds" checks at scale
- Expect contract pricing, not self-serve

### 3. Your AI Agents (LLM Calls)

Using an efficient OpenAI model (e.g., **GPT-4o-mini**), public rates are: **$0.15 / 1M input tokens** and **$0.60 / 1M output tokens**. Your validation/orchestration prompts are small, so this cost is typically pennies per **thousands** of runs.

### 4. Object Storage

If you use **Google Cloud Storage (Standard)**, reference pricing is about **$0.020/GB-month** in common US regions (plus tiny request/egress fees).

## Per-Document Cost Examples

### Scenario A: 3 Separate Monthly Bills (3 pages total)

- **DocAI (Form Parser):** 3 × $0.03 = **$0.09**. If you get access to **Utility parser**: 3 × $0.01 = **$0.03**
- **Resistant AI:** **contract-based** (TBD with vendor)
- **Your agents (LLM):** example 16k input + 4k output tokens → (0.016×$0.15) + (0.004×$0.60) ≈ **$0.0048**
- **Storage:** ~3 MB kept → ≈ **$0.00006**/month at $0.020/GB

### Scenario B: Only 1–2 Bills Uploaded (1–2 pages)

- **DocAI (Form Parser):** $0.03–$0.06 (or $0.01–$0.02 via Utility parser)
- **Resistant AI:** contract-based
- **LLM + storage:** roughly proportional (tiny)

### Scenario C: One Bill with Usage History Table (1 page)

- **DocAI (Form Parser):** **$0.03** (or **$0.01** with Utility parser)
- **Resistant AI / LLM / storage:** as above

> If your bills average **2 pages**, double the DocAI portion. XLSX with **3 tabs** counts as **3 pages**; a DOCX over **3,000 chars** spills into multiple billable "pages."

## Monthly Cost Projections

Assume **each verification** has 3 one-page bills (3 pages), Form Parser pricing, and our earlier LLM estimate.

| Monthly Verifications | DocAI pages | DocAI cost (Form Parser) | LLM est. | Storage new (GB) | Storage cost est. |
| --------------------: | ----------: | -----------------------: | ---------: | ---------------: | ----------------: |
| 1,000 | 3,000 | **$90** | **$4.80** | ~3 GB | **$0.06** |
| 10,000 | 30,000 | **$900** | **$48** | ~30 GB | **$0.60** |
| 100,000 | 300,000 | **$9,000** | **$480** | ~300 GB | **$6.00** |

*(Resistant AI is additional and quote-based.)*

If you secure access to **Utility parser** ($0.01/page), the DocAI column drops by ~**66%**: e.g., 10k verifications would be **$300** instead of **$900**. (Utility parser is **limited access**; you must apply.)

## Cost Optimization Strategies

### 1. Processor Selection

- **Use Form Parser by default**: Most cost-effective for general documents
- **Apply for Utility Parser access**: Reduces cost by ~66% for eligible documents
- **Optimize page counts**: Avoid unnecessary multi-page scans
- **Cache common prompts**: Reduce LLM costs through prompt optimization
- **Storage tiering**: Move older data to cheaper storage tiers

### 2. Batch Processing

- **Group similar documents**: Process multiple documents in batches
- **Optimize file formats**: Use PDFs instead of images when possible
- **Compress files**: Reduce storage costs with appropriate compression

### 3. Volume Discounts

- **Google Document AI**: Volume discounts available for high usage
- **Resistant AI**: Contract-based pricing with volume tiers
- **OpenAI**: Usage-based pricing with potential enterprise discounts

## Cost Levers

### Immediate Optimizations

1. **Use Form Parser by default** for most documents
2. **Apply for Utility Parser access** to reduce costs by 66% for utility bills
3. **Optimize page counts** by trimming cover pages and unnecessary content
4. **Cache common agent prompts** to reduce LLM costs
5. **Use appropriate storage tiers** for different data types

### Long-term Optimizations

1. **Negotiate volume discounts** with Resistant AI through Google/AWS Marketplace
2. **Implement intelligent caching** for frequently processed documents
3. **Use custom processors** for specific document types if volume justifies the cost
4. **Optimize data retention policies** to minimize storage costs

## Budget Planning

### Startup Phase (1,000 documents/month)
- **Document AI**: $90
- **LLM**: $4.80
- **Storage**: $0.06
- **Resistant AI**: Contract-based (estimate $100-500/month)
- **Total**: ~$200-600/month

### Growth Phase (10,000 documents/month)
- **Document AI**: $900
- **LLM**: $48
- **Storage**: $0.60
- **Resistant AI**: Contract-based (estimate $1,000-5,000/month)
- **Total**: ~$2,000-6,000/month

### Scale Phase (100,000 documents/month)
- **Document AI**: $9,000
- **LLM**: $480
- **Storage**: $6.00
- **Resistant AI**: Contract-based (estimate $10,000-50,000/month)
- **Total**: ~$20,000-60,000/month

## ROI Considerations

### Cost per Document Analysis

- **Form Parser path**: ~$0.031 + Resistant AI cost
- **Utility Parser path**: ~$0.101 + Resistant AI cost
- **LLM costs**: ~$0.001–0.002 per document
- **Storage**: ~$0.00002–0.00005 per month

### Value Proposition

- **Automated fraud detection** reduces manual review costs
- **High accuracy** minimizes false positives and negatives
- **Scalable processing** handles high volumes efficiently
- **Audit trail** provides compliance and transparency

## Conclusion

The anomaly detection system provides cost-effective document validation with clear pricing models for most components. The main variable cost is Resistant AI, which requires contract negotiation but provides enterprise-grade fraud detection capabilities.

Key recommendations:
1. Start with Form Parser for cost efficiency
2. Apply for Utility Parser access to reduce costs by 66%
3. Negotiate volume discounts with Resistant AI
4. Implement caching and optimization strategies
5. Monitor costs and adjust strategies based on actual usage patterns
