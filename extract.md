# LlamaExtract

LlamaExtract provides a simple API for extracting structured data from unstructured documents like PDFs, text files and images.

## Quick Start

```python
from llama_cloud_services import LlamaExtract
from pydantic import BaseModel, Field

# Initialize client
extractor = LlamaExtract()


# Define schema using Pydantic
class Resume(BaseModel):
    name: str = Field(description="Full name of candidate")
    email: str = Field(description="Email address")
    skills: list[str] = Field(description="Technical skills and technologies")


# Create extraction agent
agent = extractor.create_agent(name="resume-parser", data_schema=Resume)

# Extract data from document
result = agent.extract("resume.pdf")
print(result.data)
```

## Core Concepts

- **Extraction Agents**: Reusable extractors configured with a specific schema and extraction settings.
- **Data Schema**: Structure definition for the data you want to extract in the form of a JSON schema or a Pydantic model.
- **Extraction Jobs**: Asynchronous extraction tasks that can be monitored.

## Defining Schemas

Schemas can be defined using either Pydantic models or JSON Schema:

### Using Pydantic (Recommended)

```python
from pydantic import BaseModel, Field
from typing import List, Optional


class Experience(BaseModel):
    company: str = Field(description="Company name")
    title: str = Field(description="Job title")
    start_date: Optional[str] = Field(description="Start date of employment")
    end_date: Optional[str] = Field(description="End date of employment")


class Resume(BaseModel):
    name: str = Field(description="Candidate name")
    experience: List[Experience] = Field(description="Work history")
```

### Using JSON Schema

```python
schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string", "description": "Candidate name"},
        "experience": {
            "type": "array",
            "description": "Work history",
            "items": {
                "type": "object",
                "properties": {
                    "company": {
                        "type": "string",
                        "description": "Company name",
                    },
                    "title": {"type": "string", "description": "Job title"},
                    "start_date": {
                        "anyOf": [{"type": "string"}, {"type": "null"}],
                        "description": "Start date of employment",
                    },
                    "end_date": {
                        "anyOf": [{"type": "string"}, {"type": "null"}],
                        "description": "End date of employment",
                    },
                },
            },
        },
    },
}

agent = extractor.create_agent(name="resume-parser", data_schema=schema)
```

### Important restrictions on JSON/Pydantic Schema

_LlamaExtract only supports a subset of the JSON Schema specification._ While limited, it should
be sufficient for a wide variety of use-cases.

- All fields are required by default. Nullable fields must be explicitly marked as such,
  using `anyOf` with a `null` type. See `"start_date"` field above.
- Root node must be of type `object`.
- Schema nesting must be limited to within 5 levels.
- The important fields are key names/titles, type and description. Fields for
  formatting, default values, etc. are **not supported**. If you need these, you can add the
  restrictions to your field description and/or use a post-processing step. e.g. default values can be supported by making a field optional and then setting `"null"` values from the extraction result to the default value.
- There are other restrictions on number of keys, size of the schema, etc. that you may
  hit for complex extraction use cases. In such cases, it is worth thinking how to restructure
  your extraction workflow to fit within these constraints, e.g. by extracting subset of fields
  and later merging them together.

## Other Extraction APIs

### Extraction over bytes or text

You can use the `SourceText` class to extract from bytes or text directly without using a file. If passing the file bytes,
you will need to pass the filename to the `SourceText` class.

```python
with open("resume.pdf", "rb") as f:
    file_bytes = f.read()
result = test_agent.extract(SourceText(file=file_bytes, filename="resume.pdf"))
```

```python
result = test_agent.extract(
    SourceText(text_content="Candidate Name: Jane Doe")
)
```

### Batch Processing

Process multiple files asynchronously:

```python
# Queue multiple files for extraction
jobs = await agent.queue_extraction(["resume1.pdf", "resume2.pdf"])

# Check job status
for job in jobs:
    status = agent.get_extraction_job(job.id).status
    print(f"Job {job.id}: {status}")

# Get results when complete
results = [agent.get_extraction_run_for_job(job.id) for job in jobs]
```

### Updating Schemas

Schemas can be modified and updated after creation:

```python
# Update schema
agent.data_schema = new_schema

# Save changes
agent.save()
```

### Managing Agents

```python
# List all agents
agents = extractor.list_agents()

# Get specific agent
agent = extractor.get_agent(name="resume-parser")

# Delete agent
extractor.delete_agent(agent.id)
```

## Installation

```bash
pip install llama-extract==0.1.0
```

## Tips & Best Practices

At the core of LlamaExtract is the schema, which defines the structure of the data you want to extract from your documents.

1. **Schema Design**:

   - Try to limit schema nesting to 3-4 levels.
   - Make fields optional when data might not always be present. Having required fields may force the model
     to hallucinate when these fields are not present in the documents.
   - When you want to extract a variable number of entities, use an `array` type. However, note that you cannot use
     an `array` type for the root node.
   - Use descriptive field names and detailed descriptions. Use descriptions to pass formatting
     instructions or few-shot examples.
   - Above all, start simple and iteratively build your schema to incorporate requirements.

2. **Running Extractions**:
   - Note that resetting `agent.schema` will not save the schema to the database,
     until you call `agent.save`, but it will be used for running extractions.
   - Check job status prior to accessing results. Any extraction error should be available as
     part of `job.error` or `extraction_run.error` fields for debugging.
   - Consider async operations (`queue_extraction`) for large-scale extraction once you have finalized your schema.

### Hitting "The response was too long to be processed" Error

This implies that the extraction response is hitting output token limits of the LLM. In such cases, it is worth rethinking the design of your schema to enable a more efficient/scalable extraction. e.g.

- Instead of one field that extracts a complex object, you can use multiple fields to distribute the extraction logic.
- You can also use multiple schemas to extract different subsets of fields from the same document and merge them later.

Another option (orthogonal to the above) is to break the document into smaller sections and extract from each section individually, when possible. LlamaExtract will in most cases be able to handle both document and schema chunking automatically, but there are cases where you may need to do this manually.

## Additional Resources

- [Example Notebook](docs/examples-py/extract/resume_screening.ipynb) - Detailed walkthrough of resume parsing
- [Discord Community](https://discord.com/invite/eN6D2HQ4aX) - Get help and share feedback
