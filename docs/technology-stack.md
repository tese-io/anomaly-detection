# Technology Stack

## Google Document AI + Resistant AI Integration

This guide provides detailed implementation instructions for integrating Google Document AI and Resistant AI into your anomaly detection system. The combination of these two services provides:

- **Google Document AI**: High-accuracy content extraction from various document formats
- **Resistant AI**: Forensic-grade document tampering detection
- **Seamless Integration**: Both services work together through Google Cloud Marketplace

### Why This Combination?

Google officially recommends using Resistant AI with Document AI for enhanced fraud detection. This combination provides:

1. **Comprehensive Coverage**: Document AI extracts content, Resistant AI detects tampering
2. **Enterprise Integration**: Both available through Google Cloud Marketplace
3. **Scalable Pricing**: Pay-per-use model with volume discounts
4. **High Accuracy**: Specialized processors for different document types

## Google Document AI Setup

### Available Processors

| Processor | Purpose | Cost | Best For |
|-----------|---------|------|----------|
| **Form Parser** | General extraction | $30/1,000 pages | Most documents |
| **Utility Parser** | Utility bills | $0.10 per 10 pages | Utility bills |
| **Layout Parser** | Text and tables | $10/1,000 pages | Structured documents |
| **Enterprise OCR** | High-accuracy OCR | $1.50/1,000 pages | Image-only documents |

### Processor Selection Logic

```python
class ProcessorSelector:
    def __init__(self, project_id: str, location: str):
        self.project_id = project_id
        self.location = location
        
    def select_processor(self, file_info: FileInfo) -> str:
        """Select appropriate processor based on file characteristics"""
        
        # Check file type
        if file_info.file_type.lower() in ['utility_bill', 'electricity_bill', 'gas_bill']:
            return 'utility_parser'
        
        # Check if it's an image-only document
        if file_info.file_type.lower() in ['jpg', 'jpeg', 'png', 'tiff']:
            return 'enterprise_ocr'
        
        # Default to form parser
        return 'form_parser'
```

## Resistant AI Setup

### Marketplace Integration

Resistant AI is available through Google Cloud Marketplace and provides:

- **Document Forensics**: Forensic-grade tampering detection
- **Support for Multiple File Formats**: PDFs, images, Office documents
- **Real-time Analysis**: <20 seconds processing time
- **Detailed Tampering Indicators**: Specific evidence of manipulation

### Integration Architecture

The system uses a clean separation of concerns:

- **Resistant AI** = "Is this document authentic/unaltered?"
- **Document AI** = "What does this document say?"
- **Our Agents** = "Does what it says support the user's answer?"

## API Integration Details

### Resistant AI Response Format

```json
{
  "file_id": "f_001",
  "tamper_score": 78.4,
  "decision": "likely_tampered",
  "indicators": [
    "handwritten overlay in numeric cell",
    "texture mismatch vs surrounding digits",
    "incremental edit artifacts"
  ]
}
```

### Google Document AI Response Format

```json
{
  "file_id": "f_001",
  "pages": [
    {
      "page": 1,
      "kv": [
        {
          "key": "Usage (kWh)",
          "value": "7833",
          "bbox": [x1, y1, x2, y2],
          "confidence": 0.93
        }
      ],
      "tables": [
        {
          "name": "Usage history",
          "rows": [
            {"month": "Apr 2025", "kwh": "2410"},
            {"month": "May 2025", "kwh": "2613"},
            {"month": "Jun 2025", "kwh": "2810"}
          ],
          "bbox": [x1, y1, x2, y2],
          "confidence": 0.91
        }
      ],
      "text": "extracted text content"
    }
  ]
}
```

## Cost Optimization

### Processor Selection Strategy

1. **Use Form Parser by default**: Most cost-effective for general documents
2. **Apply for Utility Parser access**: Reduces cost by ~66% for eligible documents
3. **Optimize page counts**: Avoid unnecessary multi-page scans
4. **Cache common prompts**: Reduce LLM costs through prompt optimization
5. **Storage tiering**: Move older data to cheaper storage tiers

### Cost Examples

**Per Document (1-page utility bill):**
- **Form Parser path**: ~$0.031 + Resistant AI cost
- **Utility Parser path**: ~$0.101 + Resistant AI cost
- **LLM costs**: ~$0.001–0.002 per document
- **Storage**: ~$0.00002–0.00005 per month

**Monthly Examples:**
- 1,000 docs: ~$95 + Resistant AI
- 10,000 docs: ~$950 + Resistant AI
- 100,000 docs: ~$9,500 + Resistant AI

## Best Practices

### Performance Optimization

1. **Parallel processing**: Process multiple files simultaneously
2. **Caching**: Cache common document types and patterns
3. **Batch processing**: Group similar documents for efficiency
4. **Resource management**: Monitor and optimize API usage

### Security Considerations

1. **Data encryption**: Encrypt files in transit and at rest
2. **Access controls**: Implement proper authentication and authorization
3. **Audit trails**: Log all processing steps for compliance
4. **Data retention**: Implement proper data lifecycle management

### Quality Assurance

1. **Confidence thresholds**: Set minimum confidence levels for decisions
2. **Human review**: Escalate uncertain cases for manual review
3. **Continuous monitoring**: Track accuracy and performance metrics
4. **Feedback loops**: Use results to improve algorithms
