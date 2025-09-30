# Anomaly Detection Architecture (Open Docs)

This site documents our anomaly-detection system: high-level architecture, flows, scoring, and implementation notes. It is maintained by Tese.io and open to community contributions.

## Overview

This document outlines the architecture for an anomaly detection system designed to validate metric catalog records against supporting documents. The system processes two primary inputs:

1. **Metric Catalog Record**: Contains question information (ID, question, answer, metadata)
2. **Supporting Files**: Documents in various formats (PDF, DOC, XLSX, images, etc.)

### System Objectives

The architecture aims to detect three types of anomalies:

1. **Text-based Anomaly Detection**: Analyze historical answers for the same question to identify if the current answer is anomalous
2. **Document Tampering Detection**: Identify if attached files have been modified or tampered with
3. **Answer-Document Consistency**: Verify if the provided answer is supported by the content in the attached files

### Output Metrics

Each anomaly detection produces three scores:
- **Text Anomaly Score**: 0-100 (higher = more anomalous)
- **Tampering Score**: 0-100 (higher = more likely tampered)
- **Support Score**: 0-100 (higher = better supported by documents)

Each score includes a detailed text description explaining the reasoning.

## Quick Links

- **Architecture** → System components and data flow
- **Technology Stack** → Google Document AI + Resistant AI integration
- **Implementation Guide** → Technical implementation details
- **Cost Analysis** → Pricing and optimization strategies
- **API Reference** → Endpoints and response formats

## Getting Started

1. **Architecture Overview** - Understand the system components and data flow
2. **Technology Stack** - Learn about Google Document AI and Resistant AI integration
3. **Implementation Guide** - Follow technical implementation steps
4. **Cost Analysis** - Plan your budget and optimization strategy

## Contributing

This documentation is open to community contributions. Please see our [Contributing Guide](CONTRIBUTING.md) for details on how to contribute to the project.