---
name: openai-extraction
description: >
  OpenAI structured extraction patterns for Python data pipelines.
  Trigger: OpenAI, Responses API, structured outputs, PDF extraction, HTML extraction, AI parsing, chunking, retries, or model output validation.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "2.0"
---

## Critical Patterns

- Prefer the **Responses API** for new text/extraction workflows.
- Prefer **Structured Outputs** over JSON mode when schema adherence matters.
- In Responses API, structured output configuration belongs under `text.format`.
- Validate parsed output with Pydantic models and handle refusals/empty outputs explicitly.
- Chunk large PDFs/HTML/text into bounded inputs; persist prompt/model/version metadata.
- Rate-limit requests and retry transient 429/5xx errors with exponential backoff.
- Do not send secrets or unnecessary PII to OpenAI.
- Tests must mock OpenAI; CI should not depend on live model calls.

## Example Shape

```python
from pydantic import BaseModel, Field
from openai import OpenAI

client = OpenAI()

class ExtractedSchool(BaseModel):
    name: str
    state: str | None = None
    county: str | None = None

class SchoolExtraction(BaseModel):
    schools: list[ExtractedSchool] = Field(default_factory=list)

# Use the project-approved SDK helper or JSON schema path.
# Keep model name configurable rather than hardcoded in business logic.
response = client.responses.parse(
    model=settings.OPENAI_EXTRACTION_MODEL,
    instructions="Extract schools from the input. Return only schema-valid data.",
    input=document_chunk,
    text_format=SchoolExtraction,
)

result: SchoolExtraction = response.output_parsed
```

## Commands

```bash
pytest app/tests/test_*openai* tests/test_*openai* -q
ruff check app services tests
```
