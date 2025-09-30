# Architecture

## Components

### 1. Orchestrator
The central component that coordinates all processing flows and manages the overall workflow.

**Responsibilities:**
- Receive and validate input data
- Route processing to appropriate flows
- Aggregate results from all flows
- Generate final anomaly report
- Handle error recovery and retries

### 2. Text Analysis Flow
Analyzes the provided answer against historical data for the same question.

**Components:**
- Historical Data Retrieval
- Statistical Analysis Engine
- Pattern Recognition
- Anomaly Scoring

**Process:**
1. Retrieve all historical answers for the question ID
2. Perform statistical analysis (mean, standard deviation, outliers)
3. Apply pattern recognition algorithms
4. Generate anomaly score and explanation

### 3. Document Analysis Flow
Processes attached files to detect tampering and extract relevant information.

**Components:**
- File Format Detection
- Tampering Detection Engine
- Content Extraction Engine
- Multi-file Correlation

**Process:**
1. Detect file format and validate integrity
2. Perform tampering analysis
3. Extract relevant content and data
4. Correlate information across multiple files

## Data Flow

### Input Processing
```
Input → Orchestrator → Validation → Routing
```

### Text Analysis Flow
```
Question ID → Historical Data → Statistical Analysis → Anomaly Detection → Score + Description
```

### Document Analysis Flow
```
Files → Format Detection → Tampering Check → Content Extraction → Answer Validation → Scores + Descriptions
```

### Output Generation
```
All Scores → Aggregation → Final Report → Storage
```

## Technology Stack

### Primary Stack (Recommended)

#### 1. Google Document AI
**Purpose**: Content extraction and OCR
**Components**:
- **Form Parser**: $30/1,000 pages - General purpose extraction
- **Utility Parser**: $0.10 per 10 pages - Specialized for utility bills
- **Layout Parser**: $10/1,000 pages - Text and table extraction
- **Enterprise OCR**: $1.50/1,000 pages - High-accuracy OCR

**Advantages**:
- High accuracy across multiple file formats
- Specialized processors for different document types
- Integration with Google Cloud ecosystem
- Scalable pricing model

#### 2. Resistant AI
**Purpose**: Document tampering detection
**Integration**: Available through Google Cloud Marketplace
**Features**:
- Forensic-grade tampering detection
- Support for multiple file formats
- Real-time analysis (<20 seconds)
- Detailed tampering indicators

**Advantages**:
- Built specifically for document forensics
- Integrates seamlessly with Google Document AI
- Provides detailed tampering indicators
- Enterprise-grade security

#### 3. Custom AI Agents (LLM-based)
**Purpose**: Orchestration, validation, and reasoning
**Technology**: OpenAI GPT-4o-mini or similar
**Cost**: ~$0.15/1M input tokens, $0.60/1M output tokens

**Responsibilities**:
- Orchestrate the entire workflow
- Perform multi-file reasoning
- Generate explanations and citations
- Handle edge cases and conflicts

## Alternative Solutions

### Why Not Other Frameworks?

#### Taggun/Veryfi/Ocrolus Limitations
- **Narrow domain focus**: Optimized for receipts/invoices, not general documents
- **Limited file format support**: Primarily image-based, not PDF/Office files
- **Workflow constraints**: Built for specific use cases (expenses, lending)
- **Integration complexity**: Not designed for multi-file, multi-format workflows

#### Advantages of Resistant AI + Google Document AI
- **Breadth and depth**: Covers all file types with specialized processors
- **Clean separation of concerns**: Tampering vs. extraction vs. validation
- **Google Cloud integration**: Seamless deployment and scaling
- **Future-proof**: Designed for enterprise workflows