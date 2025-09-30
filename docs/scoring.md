# Scoring & Final Output

## Overview

The anomaly detection system produces three comprehensive scores that evaluate different aspects of document validation and answer consistency. Each score includes detailed explanations and confidence measures to provide transparent, auditable results.

## Scoring Components

### 1. Text Anomaly Score
**Purpose**: Evaluate the consistency of the provided answer against historical data for the same question.

**Range**: 0-100 (higher = more anomalous)

**Calculation Method**:
- Statistical analysis of historical answers (mean, standard deviation, outliers)
- Pattern recognition and semantic similarity analysis
- Weighted combination of statistical and pattern-based scores

**Labels**:
- **Normal** (0-30): Answer falls within expected range
- **Suspicious** (30-60): Answer shows some deviation from historical patterns
- **Anomalous** (60-100): Answer significantly deviates from historical data

### 2. Tampering Score
**Purpose**: Assess the likelihood that attached documents have been manipulated or tampered with.

**Range**: 0-100 (higher = more likely tampered)

**Calculation Method**:
- Forensic analysis using Resistant AI
- Detection of digital manipulation indicators
- Analysis of document structure, metadata, and content consistency
- Aggregation across multiple files

**Labels**:
- **Clean** (0-30): No forensic anomalies detected
- **Suspicious** (30-60): Some indicators of potential tampering
- **Likely Tampered** (60-100): Strong evidence of document manipulation

### 3. Support Score
**Purpose**: Evaluate how well the extracted document content supports the provided answer.

**Range**: 0-100 (higher = better supported)

**Calculation Method**:
- Content extraction and normalization
- Comparison of extracted data with provided answer
- Coverage analysis (completeness of evidence)
- Tolerance-based validation (±2% or ±50 kWh for electricity)

**Labels**:
- **Not Supported** (0-30): Evidence contradicts the answer
- **Weak Support** (30-60): Limited evidence supporting the answer
- **Supported** (60-80): Good evidence supporting the answer
- **Strongly Supported** (80-100): Strong evidence supporting the answer

## Final Output Format

### Complete Response Structure
```json
{
  "run_id": "run_12345",
  "question_id": "q_001",
  "timestamp": "2025-01-15T10:30:00Z",
  "processing_time_ms": 2500,
  "text_anomaly": {
    "score": 22.0,
    "label": "normal",
    "explanation": "Answer falls within expected range based on historical data",
    "confidence": 0.85,
    "statistical_analysis": {
      "mean_length": 150,
      "std_length": 25,
      "z_score": 0.8,
      "percentile": 75
    },
    "pattern_analysis": {
      "semantic_similarity": 0.82,
      "vocab_overlap": 0.75,
      "sentiment_deviation": 0.1
    }
  },
  "tampering": {
    "aggregate_score": 15.0,
    "label": "clean",
    "explanation": "No forensic anomalies detected; digital signatures valid",
    "files": [
      {
        "file_id": "f_001",
        "score": 15.0,
        "decision": "clean",
        "indicators": [],
        "confidence": 0.92
      }
    ]
  },
  "support": {
    "coverage": 1.0,
    "score": 84.0,
    "label": "strongly_supported",
    "explanation": "Sum(Apr–Jun)=7,833 kWh; within 0.02% of submitted 7,832.67 after rounding",
    "citations": [
      {
        "file_id": "f_001",
        "page": 1,
        "span": "Usage history Apr–Jun 2025",
        "extracted_value": 7833,
        "confidence": 0.93
      }
    ]
  },
  "final_decision": "validated",
  "quality_metrics": {
    "overall_confidence": 0.87,
    "data_completeness": 1.0,
    "processing_quality": "high"
  }
}
```

## Score Aggregation Logic

### Final Decision Rules
The system combines all three scores to determine the final decision:

```python
def determine_final_decision(text_anomaly, tampering, support):
    """Determine final decision based on all scores"""
    
    # Rule 1: High tampering score overrides everything
    if tampering.score > 60:
        return "not_validated"
    
    # Rule 2: Low support score indicates insufficient evidence
    if support.score < 30:
        return "not_supported"
    
    # Rule 3: High text anomaly with low support
    if text_anomaly.score > 60 and support.score < 60:
        return "suspicious"
    
    # Rule 4: Good scores across all dimensions
    if (text_anomaly.score < 30 and 
        tampering.score < 30 and 
        support.score > 80):
        return "validated"
    
    # Rule 5: Moderate scores
    if (text_anomaly.score < 60 and 
        tampering.score < 60 and 
        support.score > 60):
        return "conditionally_validated"
    
    # Default: Requires review
    return "requires_review"
```

### Quality Metrics
The system also provides overall quality metrics:

- **Overall Confidence**: Weighted average of all confidence scores
- **Data Completeness**: Percentage of required data successfully extracted
- **Processing Quality**: Assessment of processing success and reliability

## Explanation Generation

### Text Anomaly Explanations
```python
def generate_text_explanation(score, stats, patterns):
    """Generate human-readable explanation for text anomaly score"""
    
    if score < 30:
        return "Answer falls within expected range based on historical data"
    elif score < 60:
        return f"Answer shows moderate deviation from historical patterns (z-score: {stats.z_score:.2f})"
    else:
        return f"Answer significantly deviates from historical data (percentile: {stats.percentile:.1f}%)"
```

### Tampering Explanations
```python
def generate_tampering_explanation(score, indicators):
    """Generate explanation for tampering score"""
    
    if score < 30:
        return "No forensic anomalies detected; digital signatures valid"
    elif score < 60:
        return f"Some indicators of potential tampering: {', '.join(indicators[:2])}"
    else:
        return f"Strong evidence of document manipulation: {', '.join(indicators)}"
```

### Support Explanations
```python
def generate_support_explanation(score, citations, coverage):
    """Generate explanation for support score"""
    
    if score > 80:
        return f"Strong evidence supporting the answer with {coverage:.1%} coverage"
    elif score > 60:
        return f"Good evidence supporting the answer with {coverage:.1%} coverage"
    elif score > 30:
        return f"Limited evidence supporting the answer with {coverage:.1%} coverage"
    else:
        return "Insufficient evidence to support the answer"
```

## Confidence Scoring

### Confidence Calculation
Each score includes a confidence measure based on:

- **Data Quality**: Completeness and reliability of input data
- **Processing Success**: Success rate of analysis components
- **Model Confidence**: Confidence from machine learning models
- **Historical Performance**: Accuracy of similar analyses

### Confidence Thresholds
- **High Confidence**: > 0.8
- **Medium Confidence**: 0.5 - 0.8
- **Low Confidence**: < 0.5

## Audit Trail

### Processing Log
Every analysis includes a detailed audit trail:

```json
{
  "audit_trail": {
    "processing_steps": [
      {
        "step": "file_validation",
        "timestamp": "2025-01-15T10:30:00Z",
        "status": "success",
        "details": "3 files validated successfully"
      },
      {
        "step": "tampering_detection",
        "timestamp": "2025-01-15T10:30:15Z",
        "status": "success",
        "details": "Resistant AI analysis completed"
      },
      {
        "step": "content_extraction",
        "timestamp": "2025-01-15T10:30:30Z",
        "status": "success",
        "details": "Document AI extraction completed"
      }
    ],
    "api_calls": [
      {
        "service": "resistant_ai",
        "endpoint": "/analyze",
        "response_time_ms": 1200,
        "status_code": 200
      }
    ],
    "cost_breakdown": {
      "document_ai": 0.09,
      "resistant_ai": 0.50,
      "llm": 0.002,
      "total": 0.592
    }
  }
}
```

## Quality Assurance

### Validation Checks
- **Score Consistency**: Verify scores are within expected ranges
- **Explanation Quality**: Ensure explanations are clear and accurate
- **Data Integrity**: Validate all extracted data and citations
- **Processing Completeness**: Confirm all required steps completed

### Human Review Triggers
- **Low Confidence**: Scores with confidence < 0.5
- **High Anomaly**: Text anomaly score > 80
- **Tampering Detected**: Tampering score > 60
- **Insufficient Support**: Support score < 30
- **Processing Errors**: Any component failures

## Performance Metrics

### Key Performance Indicators
- **Processing Time**: Average time per analysis
- **Accuracy Rate**: Comparison with human review
- **False Positive Rate**: Incorrect anomaly detections
- **False Negative Rate**: Missed anomalies
- **Cost per Analysis**: Total cost per document

### Monitoring and Alerting
- **Score Distribution**: Monitor score distributions for anomalies
- **Processing Delays**: Alert when processing time exceeds SLA
- **Error Rates**: Alert when error rates increase
- **Cost Thresholds**: Alert when costs exceed budget

## Future Enhancements

### Advanced Scoring
- **Machine Learning Models**: Train custom models for specific domains
- **Ensemble Methods**: Combine multiple scoring approaches
- **Adaptive Thresholds**: Adjust thresholds based on historical performance
- **Real-time Learning**: Update models with new data

### Enhanced Explanations
- **Natural Language Generation**: Generate more natural explanations
- **Visual Explanations**: Include charts and graphs
- **Interactive Explanations**: Allow users to explore details
- **Multilingual Support**: Support multiple languages