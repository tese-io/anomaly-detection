# Document Analysis Flow

**Goal:** Validate document authenticity and consistency; flag anomalies.

**Inputs:** PDFs/images; **Outputs:** extracted fields, tamper indicators, support evidence.

## Overview

The Document Analysis Flow processes attached files to detect tampering and extract relevant information. This flow combines Google Document AI for content extraction and Resistant AI for tampering detection to provide comprehensive document validation.

## Components

### 1. File Format Detection
- **Purpose**: Identify file type and validate format
- **Supported Formats**: PDF, DOC, DOCX, XLSX, images (JPG, PNG, TIFF)
- **Validation**: MIME type checking, file signature verification
- **Output**: File type classification with confidence

### 2. Tampering Detection Engine
- **Purpose**: Detect document manipulation and fraud
- **Technology**: Resistant AI integration
- **Analysis**: Forensic examination of document structure, metadata, and content
- **Output**: Tampering score with detailed indicators

### 3. Content Extraction Engine
- **Purpose**: Extract structured data from documents
- **Technology**: Google Document AI
- **Processors**: Form Parser, Utility Parser, Layout Parser, Enterprise OCR
- **Output**: Structured data with confidence scores

### 4. Multi-file Correlation
- **Purpose**: Correlate information across multiple files
- **Analysis**: Cross-reference data, detect inconsistencies
- **Validation**: Verify data consistency and completeness
- **Output**: Correlation analysis with conflict detection

## Process Flow

### Step 1: File Processing
```python
async def process_files(self, files: List[FileInfo]) -> List[ProcessingResult]:
    """Process multiple files in parallel"""
    
    results = []
    for file_info in files:
        # Validate file format
        validation_result = await self.validate_file(file_info)
        
        # Process file
        if validation_result.is_valid:
            result = await self.process_single_file(file_info)
            results.append(result)
        else:
            results.append(ProcessingResult(
                file_id=file_info.file_id,
                status="invalid",
                error=validation_result.error
            ))
    
    return results
```

### Step 2: Tampering Detection
```python
async def detect_tampering(self, file_info: FileInfo) -> TamperingResult:
    """Detect tampering using Resistant AI"""
    
    try:
        # Call Resistant AI API
        response = await self.resistant_ai_client.analyze_document(
            file_path=file_info.file_path,
            file_type=file_info.file_type
        )
        
        return TamperingResult(
            file_id=file_info.file_id,
            score=response.tamper_score,
            decision=response.decision,
            indicators=response.indicators,
            confidence=response.confidence
        )
        
    except Exception as e:
        logger.error(f"Tampering detection failed for {file_info.file_id}: {str(e)}")
        return TamperingResult(
            file_id=file_info.file_id,
            score=0.0,
            decision="error",
            indicators=["Processing error occurred"],
            confidence=0.0
        )
```

### Step 3: Content Extraction
```python
async def extract_content(self, file_info: FileInfo) -> ContentExtractionResult:
    """Extract content using Google Document AI"""
    
    try:
        # Select appropriate processor
        processor = self._select_processor(file_info.file_type)
        
        # Extract content
        extraction_result = await self.docai_client.process_document(
            file_path=file_info.file_path,
            processor=processor
        )
        
        return ContentExtractionResult(
            file_id=file_info.file_id,
            pages=extraction_result.pages,
            confidence=extraction_result.confidence,
            extracted_data=self._normalize_data(extraction_result)
        )
        
    except Exception as e:
        logger.error(f"Content extraction failed for {file_info.file_id}: {str(e)}")
        raise ContentExtractionError(f"Extraction failed: {str(e)}")
```

### Step 4: Multi-file Correlation
```python
def correlate_files(self, extraction_results: List[ContentExtractionResult]) -> CorrelationResult:
    """Correlate information across multiple files"""
    
    correlations = []
    conflicts = []
    
    for i, result1 in enumerate(extraction_results):
        for j, result2 in enumerate(extraction_results[i+1:], i+1):
            # Compare extracted data
            correlation = self._compare_extracted_data(result1, result2)
            correlations.append(correlation)
            
            # Detect conflicts
            if correlation.conflict_score > 0.7:
                conflicts.append(Conflict(
                    file1_id=result1.file_id,
                    file2_id=result2.file_id,
                    conflict_type=correlation.conflict_type,
                    description=correlation.conflict_description
                ))
    
    return CorrelationResult(
        correlations=correlations,
        conflicts=conflicts,
        overall_consistency=1.0 - (len(conflicts) / len(correlations)) if correlations else 1.0
    )
```

## Processor Selection

### Form Parser (Default)
- **Use Case**: General documents, utility bills, forms
- **Cost**: $30/1,000 pages
- **Features**: Key-value extraction, table recognition, layout analysis
- **Best For**: Most document types

### Utility Parser (Specialized)
- **Use Case**: Utility bills specifically
- **Cost**: $0.10 per 10 pages
- **Features**: Utility-specific fields, usage history, billing periods
- **Best For**: Electricity, gas, water bills

### Layout Parser
- **Use Case**: Structured documents with complex layouts
- **Cost**: $10/1,000 pages
- **Features**: Text segmentation, table extraction, document structure
- **Best For**: Reports, statements, multi-column documents

### Enterprise OCR
- **Use Case**: Image-only documents
- **Cost**: $1.50/1,000 pages
- **Features**: High-accuracy OCR, text recognition
- **Best For**: Scanned documents, photos

## Output Formats

### Tampering Result
```json
{
  "file_id": "f_001",
  "score": 78.4,
  "decision": "likely_tampered",
  "indicators": [
    "handwritten overlay in numeric cell",
    "texture mismatch vs surrounding digits",
    "incremental edit artifacts"
  ],
  "confidence": 0.92
}
```

### Content Extraction Result
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
  ],
  "confidence": 0.89
}
```

### Correlation Result
```json
{
  "correlations": [
    {
      "file1_id": "f_001",
      "file2_id": "f_002",
      "similarity_score": 0.85,
      "conflict_score": 0.1,
      "matching_fields": ["account_number", "service_period"],
      "conflicting_fields": []
    }
  ],
  "conflicts": [],
  "overall_consistency": 0.95
}
```

## Quality Gates

### Tampering Thresholds
- **Clean**: Score < 30
- **Suspicious**: Score 30-60
- **Likely Tampered**: Score > 60

### Content Extraction Thresholds
- **High Confidence**: > 0.8
- **Medium Confidence**: 0.5-0.8
- **Low Confidence**: < 0.5

### Correlation Thresholds
- **High Consistency**: > 0.8
- **Medium Consistency**: 0.5-0.8
- **Low Consistency**: < 0.5

## Error Handling

### File Processing Errors
- **Invalid Format**: Skip file with error message
- **Corrupted File**: Attempt recovery or skip
- **Size Limits**: Reject oversized files
- **Security Issues**: Quarantine suspicious files

### API Errors
- **Resistant AI**: Fallback to basic validation
- **Document AI**: Retry with different processor
- **Network Issues**: Implement retry logic
- **Rate Limits**: Queue requests and retry

## Performance Optimization

### Parallel Processing
- **Multi-file**: Process multiple files simultaneously
- **Multi-page**: Process pages in parallel
- **API Calls**: Batch API requests when possible

### Caching Strategy
- **File Hashes**: Cache results by file hash
- **Processor Results**: Cache extraction results
- **Tampering Results**: Cache tampering analysis

### Resource Management
- **Memory Usage**: Monitor and limit memory consumption
- **CPU Usage**: Distribute processing across cores
- **Network Usage**: Optimize API call patterns

## Monitoring and Metrics

### Key Metrics
- **Processing Time**: Average time per file
- **Success Rate**: Percentage of successful extractions
- **Accuracy**: Comparison with manual validation
- **Cost per File**: Track processing costs

### Alerts
- **High Tampering Rate**: Alert when tampering rate exceeds threshold
- **Extraction Failures**: Alert when extraction success rate drops
- **API Errors**: Alert when API error rate increases
- **Processing Delays**: Alert when processing time exceeds SLA

## Integration Points

### Input Sources
- **File Upload Service**: Source of files to process
- **Validation Service**: File format validation
- **Storage Service**: File storage and retrieval

### Output Destinations
- **Scoring Engine**: Provides tampering and extraction scores
- **Audit Log**: Records processing results
- **Notification System**: Alerts for high tampering scores

## Future Enhancements

### Advanced Features
- **Custom Processors**: Train domain-specific models
- **Real-time Processing**: Stream processing for immediate results
- **Mobile Integration**: Support mobile document capture
- **Blockchain Verification**: Immutable document verification

### Performance Improvements
- **Edge Processing**: Process documents closer to users
- **GPU Acceleration**: Use GPUs for faster processing
- **Predictive Caching**: Pre-process likely documents
- **Smart Routing**: Route to optimal processors automatically