---
model:
  id: "429b9e8b-d99e-44de-91ad-706cf8183658"
  source: 1
  name: "@cf/baai/bge-base-en-v1.5"
  description: "BAAI general embedding (bge) models transform any given text into a compact vector"
  task:
    id: "0137cdcf-162a-4108-94f2-1ca59e8c65ee"
    name: "Text Embeddings"
    description: "Feature extraction models transform raw data into numerical features that can be processed while preserving the information in the original dataset. These models are ideal as part of building vector search applications or Retrieval Augmented Generation workflows with Large Language Models (LLM)."
  tags:
    - "baai"
    - "text-embeddings"
  properties:
    - property_id: "info"
      value: "https://huggingface.co/BAAI/bge-base-en-v1.5"
    - property_id: "max_input_tokens"
      value: "512"
    - property_id: "output_dimensions"
      value: "768"
    - property_id: "constellation_config"
      value: "infer_response_cache: in_memory\n\nmax_requests_per_min:\n  default: 180\n  accounts:\n    32118455: 2160 # ai.cloudflare.com staging\n    50147400: 2160 # ai.cloudflare.com\n\nneurons:\n  metrics:\n    - name: input_tokens\n      neuron_cost: 0.0060575\nmax_concurrent_requests: 100"
task_type: "text-embeddings"
model_display_name: "bge-base-en-v1.5"
layout: "model"
title: "bge-base-en-v1.5"
json_schema:
  input: "{\n  \"type\": \"object\",\n  \"properties\": {\n    \"text\": {\n      \"oneOf\": [\n        {\n          \"type\": \"string\"\n        },\n        {\n          \"type\": \"array\",\n          \"items\": {\n            \"type\": \"string\"\n          },\n          \"maxItems\": 100\n        }\n      ]\n    }\n  },\n  \"required\": [\n    \"text\"\n  ]\n}"
  output: "{\n  \"type\": \"object\",\n  \"contentType\": \"application/json\",\n  \"properties\": {\n    \"shape\": {\n      \"type\": \"array\",\n      \"items\": {\n        \"type\": \"number\"\n      }\n    },\n    \"data\": {\n      \"type\": \"array\",\n      \"items\": {\n        \"type\": \"array\",\n        \"items\": {\n          \"type\": \"number\"\n        }\n      }\n    }\n  }\n}"

---
