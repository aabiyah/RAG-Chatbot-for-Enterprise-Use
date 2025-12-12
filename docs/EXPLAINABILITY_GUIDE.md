# Explainability Analysis Guide

This guide explains how to run the explainability analysis notebook and interpret the results.

## Prerequisites

Ensure you have:
1. ✅ Built the FAISS index (`python scripts/build_index.py`)
2. ✅ Ollama running with a model loaded (`ollama serve`)
3. ✅ Jupyter and visualization libraries installed

## Running the Notebook

### Option 1: JupyterLab (Recommended)

```bash
cd "/Users/aabiyahzehra/Desktop/Tucows Interview Assignment/aabiyah-interview-exercise-ai-tucows"
source .venv/bin/activate
jupyter lab
```

Then navigate to `notebooks/explainability_analysis.ipynb` and run all cells.

### Option 2: Jupyter Notebook

```bash
cd "/Users/aabiyahzehra/Desktop/Tucows Interview Assignment/aabiyah-interview-exercise-ai-tucows"
source .venv/bin/activate
jupyter notebook
```

Open `notebooks/explainability_analysis.ipynb` and run cells sequentially.

### Option 3: VS Code / PyCharm

Open the `.ipynb` file directly in your IDE and run cells using the built-in Jupyter support.

## What the Notebook Analyzes

### 1. **Confidence Score Distribution**
- Histogram and box plot of confidence scores across test queries
- Identifies queries below the confidence threshold (0.6)
- Shows mean, median, and standard deviation

### 2. **Document Relevance Heatmap**
- Visual representation of similarity scores between queries and retrieved FAQs
- Shows which queries have strong matches vs. weak matches
- Helps identify FAQ coverage gaps

### 3. **Action Escalation Patterns**
- Bar chart and pie chart showing distribution of action types
- Tracks how many queries need human review, escalation, or can be auto-resolved
- Key metric for system automation rate

### 4. **Confidence vs Similarity Correlation**
- Scatter plot showing relationship between top-1 similarity and final confidence
- Includes trend line and correlation coefficient
- Validates that the confidence scoring algorithm is working correctly

### 5. **Query-by-Query Breakdown**
- Detailed table showing each query's performance
- Includes confidence, similarity, action, and reference count
- Useful for debugging specific query failures

### 6. **Reasoning Trace Validation**
- Samples of the LLM's internal reasoning
- Only shown when `debug=True` is passed to the API
- Helps understand why certain decisions were made

### 7. **Retrieval Quality Metrics**
- Average similarity scores (top-1 and top-3)
- Count of high/low confidence queries
- Average number of references cited per query

### 8. **Summary and Recommendations**
- Automated analysis of strengths and weaknesses
- Actionable recommendations based on the data
- Flags areas needing improvement (low confidence, high escalation rate, etc.)

### 9. **Export Results**
- Saves detailed analysis to `docs/explainability_results.json`
- Can be used for reporting or further analysis

## Test Queries Analyzed

The notebook tests 8 sample queries covering:
- GDPR and data privacy
- DNS and nameserver management
- Domain redemption periods
- Domain transfers
- Refund policies
- WHOIS visibility
- Consent request timeouts
- Ownership changes

## Expected Runtime

- **With Ollama (local)**: 2-5 minutes (depends on model speed)
- **With OpenAI API**: 30-60 seconds

⚠️ **Note**: The notebook makes 8 LLM calls, so Ollama responses may be slow. Consider reducing `test_queries` to 3-4 items for faster testing.

## Interpreting Results

### Good System Performance
- ✅ Average confidence > 0.7
- ✅ Top-1 similarity > 0.6
- ✅ Correlation coefficient > 0.7
- ✅ < 20% queries need human review

### Needs Improvement
- ⚠️ Average confidence < 0.5
- ⚠️ Top-1 similarity < 0.4
- ⚠️ Weak correlation (< 0.5)
- ⚠️ > 40% queries need human review

### Red Flags
- 🚨 Any query with confidence < 0.3
- 🚨 Top-1 similarity consistently < 0.3
- 🚨 No correlation between similarity and confidence
- 🚨 > 60% queries escalated to human review

## Troubleshooting

### "No module named 'embeddings'"
```bash
# Make sure you're in the project root
cd "/Users/aabiyahzehra/Desktop/Tucows Interview Assignment/aabiyah-interview-exercise-ai-tucows"
# Run jupyter from there
jupyter lab
```

### "FAISS index not found"
```bash
# Build the index first
python scripts/build_index.py
```

### "Ollama timeout"
- Ensure Ollama is running: `ollama serve`
- Test the model: `ollama run mistral "test"`
- Consider using a smaller model: `ollama pull phi3:mini`

### Visualizations not showing
```bash
# Install matplotlib backend
pip install pyqt5
# Or for notebook
%matplotlib inline  # Add to first code cell
```

## Output Files

After running the notebook, you'll have:
- **Visualizations**: Displayed inline in the notebook
- **JSON export**: `docs/explainability_results.json` (detailed query results)
- **Summary statistics**: Printed in the final cells

## Next Steps

1. **Review low-confidence queries** → Add more FAQs for those topics
2. **Check escalation patterns** → Tune the confidence threshold
3. **Analyze similarity scores** → Improve embedding quality or FAQ phrasing
4. **Export results** → Share with stakeholders or integrate into CI/CD

## Advanced Usage

### Running with Different Models

Edit cell 2 to use a specific model:
```python
llm_client = TucowsSupportLLM(model="phi3:mini")  # Faster
# or
llm_client = TucowsSupportLLM(model="llama3.2")   # Better quality
```

### Adding Custom Test Queries

Edit the `test_queries` list in cell 3:
```python
test_queries = [
    "Your custom query here",
    "Another test question",
    # ... add more
]
```

### Adjusting Confidence Threshold

The notebook uses 0.6 as the threshold. To test different values:
```python
# In the relevant cells, change:
action = should_escalate(confidence, llm_response.get('action_required', 'none'), 0.7)  # Higher threshold
```

## Integration with CI/CD

You can run the notebook headlessly:
```bash
jupyter nbconvert --to notebook --execute notebooks/explainability_analysis.ipynb --output executed_analysis.ipynb
```

Then parse `docs/explainability_results.json` for quality metrics.

---

**Questions?** Check the main README or examine the code comments in the notebook cells.

