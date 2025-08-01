[![PyPI - Downloads](https://img.shields.io/pypi/dm/llama-cloud-services)](https://pypi.org/project/llama-cloud-services/)
[![GitHub contributors](https://img.shields.io/github/contributors/run-llama/llama_cloud_services)](https://github.com/run-llama/llama_cloud_services/graphs/contributors)
[![Discord](https://img.shields.io/discord/1059199217496772688)](https://discord.gg/dGcwcsnxhU)

# Llama Cloud Services

This repository contains the code for hand-written SDKs and clients for interacting with LlamaCloud.

This includes:

- [LlamaParse](./parse.md) - A GenAI-native document parser that can parse complex document data for any downstream LLM use case (Agents, RAG, data processing, etc.).
- [LlamaReport (beta/invite-only)](./report.md) - A prebuilt agentic report builder that can be used to build reports from a variety of data sources.
- [LlamaExtract](./extract.md) - A prebuilt agentic data extractor that can be used to transform data into a structured JSON representation.
- [LlamaCloud Index](./index.md) - A widely customizable and fully automated document ingestion pipeline that also serves retrieval purposes.

## Getting Started

Install the package:

```bash
pip install llama-cloud-services
```

Then, get your API key from [LlamaCloud](https://cloud.llamaindex.ai/).

Then, you can use the services in your code:

```python
from llama_cloud_services import (
    LlamaParse,
    LlamaReport,
    LlamaExtract,
    LlamaCloudIndex,
)

parser = LlamaParse(api_key="YOUR_API_KEY")
report = LlamaReport(api_key="YOUR_API_KEY")
extract = LlamaExtract(api_key="YOUR_API_KEY")
index = LlamaCloudIndex(
    "my_first_index", project_name="default", api_key="YOUR_API_KEY"
)
```

See the quickstart guides for each service for more information:

- [LlamaParse](./parse.md)
- [LlamaReport (beta/invite-only)](./report.md)
- [LlamaExtract](./extract.md)
- [LlamaCloud Index](./index.md)

## Switch to EU SaaS 🇪🇺

If you are interested in using LlamaCloud services in the EU, you can adjust your base URL to `https://api.cloud.eu.llamaindex.ai`.

You can also create your API key in the EU region [here](https://cloud.eu.llamaindex.ai).

```python
from llama_cloud_services import (
    LlamaParse,
    LlamaReport,
    LlamaExtract,
    EU_BASE_URL,
)

parser = LlamaParse(api_key="YOUR_API_KEY", base_url=EU_BASE_URL)
report = LlamaReport(api_key="YOUR_API_KEY", base_url=EU_BASE_URL)
extract = LlamaExtract(api_key="YOUR_API_KEY", base_url=EU_BASE_URL)
index = LlamaCloudIndex(
    "my_first_index",
    project_name="default",
    api_key="YOUR_API_KEY",
    base_url=EU_BASE_URL,
)
```

## Documentation

You can see complete SDK and API documentation for each service on [our official docs](https://docs.cloud.llamaindex.ai/).

## Terms of Service

See the [Terms of Service Here](./TOS.pdf).

## Get in Touch (LlamaCloud)

You can get in touch with us by following our [contact link](https://www.llamaindex.ai/contact).
