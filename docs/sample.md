# Sample Usage

## Overview

This section provides practical examples of how to use the anomaly detection system, including API calls, response formats, and common use cases.

## API Examples

### Basic Anomaly Detection Request

#### Request
```bash
curl -X POST "https://api.anomaly-detection.com/v1/anomaly-detection" \
  -H "Content-Type: multipart/form-data" \
  -F "question_id=q_001" \
  -F "answer=7832.67" \
  -F "files=@utility_bill_april.pdf" \
  -F "files=@utility_bill_may.pdf" \
  -F "files=@utility_bill_june.pdf"
```

#### Response
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
    "confidence": 0.85
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
  "final_decision": "validated"
}
```

## Use Cases

### 1. ESG Reporting Validation
A company needs to validate their carbon footprint data for ESG reporting.

### 2. Financial Document Verification
A bank needs to verify income statements for loan applications.

### 3. Compliance Auditing
An auditor needs to verify expense claims for tax purposes.

## Integration Examples

### Python SDK Usage
```python
from anomaly_detection import AnomalyDetectionClient

# Initialize client
client = AnomalyDetectionClient(
    api_key="your-api-key",
    base_url="https://api.anomaly-detection.com"
)

# Analyze documents
result = client.analyze(
    question_id="electricity_consumption_q3_2025",
    answer="7832.67",
    files=["april_bill.pdf", "may_bill.pdf", "june_bill.pdf"]
)

# Check results
if result.final_decision == "validated":
    print("✅ Document validation successful")
else:
    print("❌ Document validation failed")
```

## Best Practices

### 1. File Preparation
- Use PDF format when possible for best results
- Ensure documents are clear and readable
- Keep files under 10MB for optimal processing

### 2. Answer Formatting
- Include appropriate decimal places
- Specify units clearly (kWh, $, etc.)
- Use consistent formatting across submissions

### 3. Error Handling
- Implement retry logic for transient failures
- Set appropriate timeouts for API calls
- Have fallback validation methods