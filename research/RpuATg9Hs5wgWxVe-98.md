```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "MachineLearningModelOutput",
  "type": "object",
  "properties": {
    "modelVersion": {
      "type": "string",
      "description": "The version identifier of the model that produced the result.",
      "minLength": 1
    },
    "predictionProbability": {
      "type": "number",
      "description": "The probability assigned by the model to the predicted class.",
      "minimum": 0,
      "maximum": 1
    },
    "inputDataFeatures": {
      "type": "array",
      "description": "The list of input feature names that were supplied to the model.",
      "items": { "type": "string" },
      "minItems": 1
    },
    "inferenceTimestamp": {
      "type": "string",
      "description": "Timestamp (UTC) when the inference was executed.",
      "format": "date-time"
    },
    "confidenceScore": {
      "type": "number",
      "description": "Overall confidence score returned by the model.",
      "minimum": 0,
      "maximum": 1
    }
  },
  "required": [
    "modelVersion",
    "predictionProbability",
    "inputDataFeatures",
    "inferenceTimestamp",
    "confidenceScore"
  ],
  "additionalProperties": false
}
```

**Key points**

- The `$schema` keyword points to the Draft 7 specification so validators know how to interpret the rules.  
- `type` guarantees that each property is of the declared data type.  
- `minimum`/`maximum` for numeric fields enforce the 0‑to‑1 range.  
- `format: "date-time"` requires the timestamp to follow ISO 8601 (e.g., `"2025-10-27T13:45:00Z"`).  
- `additionalProperties: false` blocks any keys that are not listed in `properties`, preventing extraneous data.  
- `required` ensures that all five properties must be present in every valid output.