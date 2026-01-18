---
name: openai-extraction
description: OpenAI integration patterns for structured data extraction using Structured Outputs. Use when implementing AI-powered extraction, parsing structured data from text, or working with GPT models. Triggers on OpenAI, extraction, GPT, AI parsing keywords.
---

# OpenAI Extraction Patterns

> **Note:** OpenAI pricing changes frequently. Always check the [official OpenAI pricing page](https://openai.com/pricing) for current rates before estimating costs.

## Client Setup

```python
from openai import AsyncOpenAI
import os

# Singleton client with retry
client = AsyncOpenAI(
    api_key=os.environ["OPENAI_API_KEY"],
    max_retries=3,
    timeout=60.0
)
```

## Structured Outputs (Recommended)

OpenAI's Structured Outputs feature guarantees responses match your schema.

### With Pydantic Models

```python
from pydantic import BaseModel, Field
from openai import AsyncOpenAI

class ExtractedItem(BaseModel):
    name: str = Field(description="Item name")
    value: float | None = Field(description="Numeric value if present")
    category: str = Field(description="Item category")

class ExtractionResult(BaseModel):
    items: list[ExtractedItem]
    summary: str = Field(description="Brief summary of extracted data")

async def extract_structured(text: str) -> ExtractionResult:
    response = await client.beta.chat.completions.parse(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": "Extract structured data from the provided text."
            },
            {"role": "user", "content": text}
        ],
        response_format=ExtractionResult,
        temperature=0.1  # Low for consistency
    )

    return response.choices[0].message.parsed
```

### Complex Nested Structures

```python
from pydantic import BaseModel, Field
from enum import Enum

class ContactType(str, Enum):
    EMAIL = "email"
    PHONE = "phone"
    ADDRESS = "address"

class Contact(BaseModel):
    type: ContactType
    value: str

class Person(BaseModel):
    name: str
    title: str | None = None
    contacts: list[Contact] = Field(default_factory=list)

class Organization(BaseModel):
    name: str
    people: list[Person]
    website: str | None = None

class ExtractionResponse(BaseModel):
    organizations: list[Organization]
    confidence: float = Field(ge=0, le=1, description="Extraction confidence 0-1")

async def extract_organizations(html_content: str) -> ExtractionResponse:
    response = await client.beta.chat.completions.parse(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": """Extract organization and staff information from the HTML content.
                Be thorough - extract all people and their contact information.
                Set confidence based on data quality (1.0 = clear data, 0.5 = some guessing)."""
            },
            {"role": "user", "content": html_content}
        ],
        response_format=ExtractionResponse,
        temperature=0.1
    )

    return response.choices[0].message.parsed
```

## JSON Mode (Legacy)

For simpler cases or when you need more control:

```python
import json

SYSTEM_PROMPT = """
Extract structured data from the provided text.
Return ONLY valid JSON matching this schema:
{
  "items": [
    {"name": "string", "value": "number or null"}
  ]
}
"""

async def extract_json(text: str) -> dict:
    response = await client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": text}
        ],
        response_format={"type": "json_object"},
        temperature=0.1
    )

    return json.loads(response.choices[0].message.content)
```

## Handling Long Content

### Chunking

```python
import tiktoken

def chunk_text(text: str, max_tokens: int = 100000) -> list[str]:
    encoder = tiktoken.encoding_for_model("gpt-4o-mini")
    tokens = encoder.encode(text)

    chunks = []
    for i in range(0, len(tokens), max_tokens):
        chunk_tokens = tokens[i:i + max_tokens]
        chunks.append(encoder.decode(chunk_tokens))

    return chunks

async def extract_from_long_text(text: str) -> list[ExtractedItem]:
    chunks = chunk_text(text)
    all_results = []

    for chunk in chunks:
        result = await extract_structured(chunk)
        all_results.extend(result.items)

    return all_results
```

### Streaming for Progress

```python
async def extract_with_streaming(text: str):
    """Stream extraction for real-time progress updates."""
    async with client.beta.chat.completions.stream(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Extract data..."},
            {"role": "user", "content": text}
        ],
        response_format=ExtractionResult,
    ) as stream:
        async for event in stream:
            if event.type == "content.delta":
                # Partial content available
                yield event.delta
```

## Error Handling

```python
from openai import RateLimitError, APIError, APITimeoutError
import asyncio

class ExtractionError(Exception):
    """Custom exception for extraction failures."""
    pass

async def extract_with_retry(
    text: str,
    max_retries: int = 3
) -> ExtractionResult:
    last_error = None

    for attempt in range(max_retries):
        try:
            return await extract_structured(text)

        except RateLimitError as e:
            # Exponential backoff for rate limits
            wait_time = 2 ** attempt * 10
            await asyncio.sleep(wait_time)
            last_error = e

        except APITimeoutError as e:
            if attempt == max_retries - 1:
                raise ExtractionError(f"Timeout after {max_retries} attempts") from e
            await asyncio.sleep(5)
            last_error = e

        except APIError as e:
            if e.status_code and e.status_code >= 500:
                # Server error, retry
                await asyncio.sleep(5)
                last_error = e
            else:
                # Client error, don't retry
                raise ExtractionError(f"API error: {e.message}") from e

    raise ExtractionError(f"Max retries exceeded: {last_error}")

# Handle parsing failures
async def safe_extract(text: str) -> ExtractionResult | None:
    try:
        response = await client.beta.chat.completions.parse(
            model="gpt-4o-mini",
            messages=[...],
            response_format=ExtractionResult,
        )

        # Check if parsing succeeded
        if response.choices[0].message.refusal:
            logging.warning(f"Model refused: {response.choices[0].message.refusal}")
            return None

        return response.choices[0].message.parsed

    except Exception as e:
        logging.error(f"Extraction failed: {e}")
        return None
```

## Vision (Images & PDFs)

```python
import base64

async def extract_from_image(image_bytes: bytes) -> ExtractionResult:
    base64_image = base64.b64encode(image_bytes).decode("utf-8")

    response = await client.beta.chat.completions.parse(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": "Extract all structured data from this image."
                    },
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": f"data:image/png;base64,{base64_image}"
                        }
                    }
                ]
            }
        ],
        response_format=ExtractionResult,
        temperature=0.1
    )

    return response.choices[0].message.parsed

# For PDFs, convert pages to images first
async def extract_from_pdf(pdf_bytes: bytes) -> list[ExtractionResult]:
    from pdf2image import convert_from_bytes

    images = convert_from_bytes(pdf_bytes)
    results = []

    for image in images:
        # Convert PIL Image to bytes
        import io
        buffer = io.BytesIO()
        image.save(buffer, format="PNG")
        image_bytes = buffer.getvalue()

        result = await extract_from_image(image_bytes)
        results.append(result)

    return results
```

## Batch Processing

```python
import asyncio
from typing import Sequence

async def batch_extract(
    texts: Sequence[str],
    concurrency: int = 5
) -> list[ExtractionResult]:
    """Extract from multiple texts with concurrency control."""
    semaphore = asyncio.Semaphore(concurrency)

    async def extract_one(text: str) -> ExtractionResult | None:
        async with semaphore:
            try:
                return await extract_with_retry(text)
            except ExtractionError as e:
                logging.error(f"Failed: {e}")
                return None

    results = await asyncio.gather(
        *[extract_one(text) for text in texts],
        return_exceptions=False
    )

    return [r for r in results if r is not None]
```

## Usage Tracking

```python
async def extract_with_tracking(text: str) -> tuple[ExtractionResult, dict]:
    """Extract with usage metadata (without hardcoded prices)."""
    response = await client.beta.chat.completions.parse(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Extract data..."},
            {"role": "user", "content": text}
        ],
        response_format=ExtractionResult,
    )

    usage = {
        "model": "gpt-4o-mini",
        "prompt_tokens": response.usage.prompt_tokens,
        "completion_tokens": response.usage.completion_tokens,
        "total_tokens": response.usage.total_tokens,
        # Note: Check OpenAI pricing page for current rates
    }

    return response.choices[0].message.parsed, usage
```

## Best Practices

1. **Use Structured Outputs** - Guarantees valid schema, no JSON parsing errors
2. **Pydantic models** - Type safety and validation built-in
3. **Low temperature** (0.1-0.3) - More consistent extractions
4. **Field descriptions** - Help the model understand expected data
5. **Handle refusals** - Check `message.refusal` for declined requests
6. **Retry with backoff** - Handle rate limits gracefully
7. **Validate output** - Even with Structured Outputs, validate business logic
8. **Chunk large inputs** - Stay within context limits

## Model Selection

| Model | Best For | Context |
|-------|----------|---------|
| `gpt-4o-mini` | Fast, cost-effective extractions | 128K |
| `gpt-4o` | Complex reasoning, high accuracy | 128K |
| `gpt-4-turbo` | Balance of speed and capability | 128K |

> Always check [OpenAI's model documentation](https://platform.openai.com/docs/models) for the latest capabilities and limits.
