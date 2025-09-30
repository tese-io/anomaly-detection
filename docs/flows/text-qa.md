# Text Q&A Flow

**Goal:** Detect anomalies in long-form textual answers compared with historical distributions; provide an accurate answer + confidence.

**Inputs:** question, historical answers.  
**Outputs:** answer, confidence, evidence excerpts.

## Overview

The Text Q&A Flow analyzes the provided answer against historical data for the same question to identify if the current answer is anomalous. This flow is essential for detecting patterns and inconsistencies in textual responses over time.

## Components

### 1. Historical Data Retrieval
- **Purpose**: Fetch all previous answers for the question ID
- **Data Source**: Historical answers database
- **Retrieval Logic**: Get answers from the last 100 submissions for the same question
- **Filtering**: Apply time-based and quality filters

### 2. Statistical Analysis Engine
- **Purpose**: Calculate statistical measures from historical data
- **Metrics**: Mean, standard deviation, outliers, distribution analysis
- **Algorithms**: Z-score analysis, percentile ranking, trend analysis
- **Output**: Statistical summary with confidence intervals

### 3. Pattern Recognition
- **Purpose**: Detect patterns and anomalies in answer patterns
- **Techniques**: Machine learning models, NLP analysis, semantic similarity
- **Features**: Answer length, vocabulary, sentiment, structure
- **Output**: Pattern analysis with anomaly indicators

### 4. Anomaly Scoring
- **Purpose**: Generate a 0-100 score indicating anomaly level
- **Scoring Method**: Weighted combination of statistical and pattern scores
- **Thresholds**: 
  - 0-30: Normal
  - 30-60: Suspicious
  - 60-100: Anomalous
- **Output**: Anomaly score with explanation

## Process Flow

### Step 1: Data Retrieval
```python
async def _get_historical_answers(self, question_id: str) -> List[str]:
    """Retrieve historical answers for the question"""
    query = """
    SELECT answer, created_at, confidence_score 
    FROM historical_answers 
    WHERE question_id = %s 
    ORDER BY created_at DESC 
    LIMIT 100
    """
    return await self.db.fetch_all(query, (question_id,))
```

### Step 2: Statistical Analysis
```python
def analyze_statistical_patterns(self, current_answer: str, historical_answers: List[str]) -> StatisticalResult:
    """Perform statistical analysis on historical data"""
    
    # Calculate basic statistics
    answer_lengths = [len(answer) for answer in historical_answers]
    current_length = len(current_answer)
    
    # Z-score analysis
    mean_length = statistics.mean(answer_lengths)
    std_length = statistics.stdev(answer_lengths)
    z_score = (current_length - mean_length) / std_length if std_length > 0 else 0
    
    # Percentile ranking
    percentile = stats.percentileofscore(answer_lengths, current_length)
    
    return StatisticalResult(
        mean=mean_length,
        std=std_length,
        z_score=z_score,
        percentile=percentile,
        anomaly_score=abs(z_score) * 20  # Scale to 0-100
    )
```

### Step 3: Pattern Recognition
```python
def detect_patterns(self, current_answer: str, historical_answers: List[str]) -> PatternResult:
    """Detect patterns and anomalies in answer structure"""
    
    # Semantic similarity analysis
    similarities = []
    for hist_answer in historical_answers:
        similarity = self.calculate_semantic_similarity(current_answer, hist_answer)
        similarities.append(similarity)
    
    # Vocabulary analysis
    current_vocab = set(current_answer.lower().split())
    hist_vocab = set()
    for answer in historical_answers:
        hist_vocab.update(answer.lower().split())
    
    vocab_overlap = len(current_vocab.intersection(hist_vocab)) / len(current_vocab.union(hist_vocab))
    
    # Sentiment analysis
    current_sentiment = self.analyze_sentiment(current_answer)
    hist_sentiments = [self.analyze_sentiment(answer) for answer in historical_answers]
    sentiment_deviation = abs(current_sentiment - statistics.mean(hist_sentiments))
    
    return PatternResult(
        semantic_similarity=statistics.mean(similarities),
        vocab_overlap=vocab_overlap,
        sentiment_deviation=sentiment_deviation,
        anomaly_score=self.calculate_pattern_anomaly_score(similarities, vocab_overlap, sentiment_deviation)
    )
```

### Step 4: Score Calculation
```python
def _calculate_anomaly_score(self, stats: StatisticalResult, patterns: PatternResult) -> float:
    """Calculate anomaly score based on statistical and pattern analysis"""
    # Weighted combination of statistical and pattern scores
    statistical_weight = 0.7
    pattern_weight = 0.3
    
    return (stats.anomaly_score * statistical_weight + 
            patterns.anomaly_score * pattern_weight)
```

## Output Format

### Text Anomaly Result
```json
{
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
}
```

## Quality Gates

### Confidence Thresholds
- **High Confidence**: > 0.8
- **Medium Confidence**: 0.5 - 0.8
- **Low Confidence**: < 0.5

### Anomaly Thresholds
- **Normal**: Score < 30
- **Suspicious**: Score 30-60
- **Anomalous**: Score > 60

### Data Requirements
- **Minimum Historical Data**: 10 previous answers
- **Maximum Historical Data**: 100 previous answers
- **Time Window**: Last 12 months
- **Quality Filter**: Exclude answers with confidence < 0.5

## Error Handling

### Insufficient Historical Data
```python
if not historical_answers:
    return TextAnomalyResult(
        score=0.0,
        label="normal",
        explanation="No historical data available for comparison",
        confidence=0.0
    )
```

### Data Quality Issues
- **Missing Data**: Handle gracefully with default values
- **Corrupted Data**: Skip problematic entries
- **Outdated Data**: Apply time-based filtering

## Performance Optimization

### Caching Strategy
- **Historical Data Cache**: Cache recent historical data for 1 hour
- **Statistical Results Cache**: Cache calculated statistics for 30 minutes
- **Pattern Analysis Cache**: Cache pattern analysis for 15 minutes

### Batch Processing
- **Multiple Questions**: Process multiple questions in parallel
- **Bulk Updates**: Update historical data in batches
- **Async Processing**: Use async/await for database operations

## Monitoring and Metrics

### Key Metrics
- **Processing Time**: Average time per analysis
- **Accuracy**: Comparison with human review
- **False Positive Rate**: Incorrect anomaly detections
- **False Negative Rate**: Missed anomalies

### Alerts
- **High Anomaly Rate**: Alert when anomaly rate exceeds threshold
- **Processing Delays**: Alert when processing time exceeds SLA
- **Data Quality Issues**: Alert when historical data quality drops

## Integration Points

### Input Sources
- **Question Database**: Source of question IDs and metadata
- **Historical Answers**: Source of previous answers
- **User Submissions**: Current answer to analyze

### Output Destinations
- **Scoring Engine**: Provides text anomaly score
- **Audit Log**: Records analysis results
- **Notification System**: Alerts for high anomaly scores

## Future Enhancements

### Machine Learning Improvements
- **Custom Models**: Train domain-specific models
- **Ensemble Methods**: Combine multiple analysis techniques
- **Online Learning**: Update models with new data

### Advanced Analytics
- **Trend Analysis**: Detect long-term trends in answers
- **Seasonal Patterns**: Account for seasonal variations
- **Cross-Question Analysis**: Analyze patterns across related questions