# Release Notes

<cite>
**Referenced Files in This Document**   
- [docs/release_notes.md](file://docs/release_notes.md)
- [README.md](file://README.md)
- [pyproject.toml](file://pyproject.toml)
- [docker/.env](file://docker/.env)
- [docker/service_conf.yaml.template](file://docker/service_conf.yaml.template)
- [docker/docker-compose.yml](file://docker/docker-compose.yml)
- [docs/guides/upgrade_ragflow.mdx](file://docs/guides/upgrade_ragflow.mdx)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Version History](#version-history)
3. [New Features](#new-features)
4. [Enhancements](#enhancements)
5. [Bug Fixes](#bug-fixes)
6. [Breaking Changes](#breaking-changes)
7. [Upgrade Instructions](#upgrade-instructions)
8. [Compatibility Considerations](#compatibility-considerations)
9. [Release Cycle and Support Policy](#release-cycle-and-support-policy)
10. [Obtaining the Latest Version](#obtaining-the-latest-version)
11. [Conclusion](#conclusion)

## Introduction

RAGFlow is an open-source Retrieval-Augmented Generation (RAG) engine that combines deep document understanding with Agent capabilities to create a superior context layer for Large Language Models (LLMs). This release notes document provides comprehensive information about the changes, improvements, and bug fixes included in each RAGFlow release. The document covers new features, enhancements, bug fixes, breaking changes, and upgrade instructions to help users understand the evolution of the platform and successfully migrate between versions.

RAGFlow offers a streamlined RAG workflow adaptable to enterprises of any scale, with a focus on quality input leading to quality output. The platform supports various document formats, complex data sources, and provides grounded citations with reduced hallucinations. This document will detail the release history, highlighting key developments and providing practical guidance for users upgrading from previous versions.

**Section sources**
- [README.md](file://README.md#L73-L75)
- [docs/release_notes.md](file://docs/release_notes.md#L6-L8)

## Version History

This section provides a comprehensive overview of RAGFlow's version history, detailing the key changes in each release from v0.5.0 to the latest version. The release history demonstrates the platform's rapid evolution, with regular updates introducing new features, improvements, and bug fixes.

The version history shows a clear progression of capabilities, starting with foundational features like streaming output and document chunk retrieval in v0.6.0, through the introduction of advanced capabilities like GraphRAG, RAPTOR, and Agentic RAG, to the recent focus on Agent capabilities, workflow orchestration, and multimodal processing. Each release builds upon the previous one, expanding the platform's functionality and improving user experience.

### v0.22.1 (November 19, 2025)
The latest release focuses on Agent improvements, data source integration, and framework upgrades. Key changes include support for exporting Agent outputs in Word or Markdown formats, adding List operations and Variable aggregator components to Agents, and supporting S3-compatible data sources and JIRA synchronization. The release also includes a major framework upgrade, migrating the Flask web framework from synchronous to asynchronous to increase concurrency and prevent blocking issues when requesting upstream LLM services.

### v0.22.0 (November 12, 2025)
This release introduces significant breaking changes by shipping only the slim edition Docker image without embedding models. Key new features include data synchronization from five online sources (AWS S3, Google Drive, Notion, Confluence, and Discord), support for Docling document parsing, and a new administrative Web UI dashboard. The release also enhances Agent capabilities with structured output support and metadata filtering.

### v0.21.1 (October 23, 2025)
This release adds experimental support for PDF document parsing using MinerU and enhances UI/UX for dataset and personal center pages. It also upgrades the Infinity document engine to v0.6.1.

### v0.21.0 (October 15, 2025)
A major release introducing orchestratable ingestion pipelines, optimized GraphRAG & RAPTOR write processes, long-context RAG with TOC extraction, video file parsing, and a new Admin CLI tool for system administration.

### v0.20.5 (September 10, 2025)
Focuses on Agent performance optimization, framework-level prompt customization, and SQL component enhancements. It also re-enables Reasoning and Cross-language search features in the Chat module.

### v0.20.4 (August 27, 2025)
Includes Chinese localization for Agent components, improved Markdown and HTML parsing, and introduces the ENABLE_TIMEOUT_ASSERTION environment variable for file parsing tasks.

### v0.20.3 (August 20, 2025)
Revamps the user interface for Datasets, Chat, and Search pages, introduces document-level metadata filtering, and adds support for comparing answer performance of up to three chat model settings.

### v0.20.1 (August 8, 2025)
Adds support for dynamic specification of dataset names using variables in the Retrieval component and introduces a French language option for the user interface.

### v0.20.0 (August 4, 2025)
A major release introducing unified orchestration of Agents and Workflows, a comprehensive Agent refactor with Multi-Agent configurations, planning and reflection capabilities, and full MCP functionality implementation.

### v0.19.1 (June 23, 2025)
Addresses several critical issues including memory leaks during high-concurrency requests, large file parsing freezes, and excessive CPU usage caused by Ollama.

### v0.19.0 (May 26, 2025)
Introduces cross-language search support, a new Code component for Python and JavaScript scripts, enhanced image display in Chat and Search, and support for Claude 4 and ChatGPT o3 models.

### v0.18.0 (April 23, 2025)
Removes built-in rerank models to improve retrieval time, adds MCP server access to datasets, supports VLM models for document layout recognition, and enables OpenAI-compatible APIs for Agents.

### v0.17.2 (March 13, 2025)
Removes the Max_tokens setting from Chat configuration and Agent components, adds OpenAI-compatible APIs, introduces a German user interface, and accelerates knowledge graph extraction.

### v0.17.1 (March 11, 2025)
Improves English tokenization quality, table extraction logic in Markdown parsing, and adds support for parsing XLS files with improved error handling.

### v0.17.0 (March 3, 2025)
Introduces Deep Research for agentic reasoning in AI chat, Tavily-based web search integration, support for starting chats without specifying datasets, and HTML file previewing.

### v0.16.0 (February 6, 2025)
Adds support for DeepSeek R1 and DeepSeek V3 models, refactors GraphRAG to build knowledge graphs on entire datasets, and introduces an Iteration agent component.

### v0.15.1 (December 25, 2024)
Upgrades the Infinity document engine to v0.5.2, enhances log display for document parsing status, and fixes several issues related to embedding model changes and retrieval performance.

### v0.15.0 (December 18, 2024)
Introduces additional Agent-specific APIs, supports page rank scoring for improved retrieval, adds a Helm chart for Kubernetes deployment, and supports dark mode in the UI.

### v0.14.1 (November 29, 2024)
Adds Infinity's configuration file to facilitate integration and customization, and fixes issues with chunk editing, Chinese text parsing, and compatibility with Polars.

### v0.14.0 (November 26, 2024)
Introduces support for Infinity or Elasticsearch as document engines, enhances Agent user experience with more variables and auto-saving, and adds new agent templates.

### v0.13.0 (October 31, 2024)
Adds team management functionality, updates the Agent UI for improved usability, and introduces HTTP and Python APIs for dataset and chat assistant management.

### v0.12.0 (September 30, 2024)
Introduces slim editions of Docker images without built-in embedding models, improves multi-round dialogue results, and adds support for OpenTTS and SparkTTS models.

### v0.11.0 (September 14, 2024)
Introduces an AI search interface, supports audio output via FishAudio or Tongyi Qwen TTS, allows Postgres for metadata storage, and adds finance-specific agent components.

### v0.10.0 (August 26, 2024)
Introduces a text-to-SQL template in the Agent UI, implements Agent APIs, and adds Agent tools for GitHub, DeepL, BaiduFanyi, QWeather, and GoogleScholar.

### v0.9.0 (August 6, 2024)
Supports GraphRAG as a chunking method, introduces the Keyword agent component and search tools, and supports speech-to-text recognition for audio files.

### v0.8.0 (July 8, 2024)
Introduces Agentic RAG for graph-based workflow construction and adds support for DOCX files in the MANUAL chunking method.

### v0.7.0 (May 31, 2024)
Adds support for reranker models, integrates BCE, BGE, and Jina embedding models, implements RAPTOR for improved text retrieval, and supports ARM64 platforms.

### v0.6.0 (May 21, 2024)
Introduces streaming output, provides APIs for retrieving document chunks, and adds support for disabling Layout Recognition to reduce file chunking time.

### v0.5.0 (Not specified)
The earliest version in the release history, establishing foundational capabilities for the platform.

**Section sources**
- [docs/release_notes.md](file://docs/release_notes.md#L10-L800)

## New Features

This section details the significant new features introduced in recent RAGFlow releases, highlighting the platform's expanding capabilities and functionality.

### Agent and Workflow Capabilities

RAGFlow has significantly enhanced its Agent and workflow capabilities in recent releases. The v0.22.1 release introduced the ability to export Agent outputs in Word or Markdown formats, making it easier to share and integrate Agent results into other workflows. The release also added two new components to the Agent framework: the List operations component for manipulating list data and the Variable aggregator component for combining and processing variables.

The v0.22.0 release marked a major advancement in Agent capabilities with the introduction of structured output support, allowing Agents to generate responses in predefined formats. This release also introduced metadata filtering in the Retrieval component, enabling more precise control over which documents are retrieved based on their metadata attributes.

v0.21.0 introduced orchestratable ingestion pipelines, allowing users to create customized data ingestion and cleansing workflows. This feature enables flexible design of data flows or direct application of official data flow templates on the canvas, significantly improving data preparation capabilities.

v0.20.0 represented a paradigm shift in Agent functionality with the introduction of unified orchestration of both Agents and Workflows. This release included a comprehensive refactor of the Agent system, greatly enhancing its capabilities and usability with support for Multi-Agent configurations, planning and reflection, and visual functionalities. The release also fully implemented MCP (Model Context Protocol) functionality, allowing RAGFlow to function as an MCP Server, with Agents operating as MCP Clients.

### Data Processing and Document Understanding

RAGFlow has made significant strides in data processing and document understanding capabilities. v0.22.0 introduced support for data synchronization from five online sources: AWS S3, Google Drive, Notion, Confluence, and Discord. This enhancement allows users to keep their knowledge bases automatically updated with content from these popular collaboration and storage platforms.

The v0.21.1 release added experimental support for PDF document parsing using MinerU, expanding the platform's document parsing options. v0.21.0 introduced video file parsing, expanding RAGFlow's multimodal data processing capabilities to include video content.

v0.18.0 introduced support for using VLM (Vision Language Model) models as a processing pipeline during document layout recognition, enabling in-depth analysis of images within PDF and DOCX files. This feature allows RAGFlow to extract meaningful information from visual content, not just text.

v0.17.0 introduced Deep Research capabilities for agentic reasoning, combining internal knowledge base retrieval with web search to provide comprehensive answers to complex queries. This feature leverages Tavily-based web search to enhance contexts in agentic reasoning.

### Integration and Deployment

RAGFlow has expanded its integration and deployment options in recent releases. v0.22.0 launched a new administrative Web UI dashboard for graphical user management and service status monitoring, providing administrators with better visibility into system operations.

v0.21.0 introduced an Admin CLI tool, allowing users to manage and monitor RAGFlow's service status via command line. This addition provides an alternative interface for system administration, particularly useful for automation and scripting.

v0.15.0 added a Helm chart for deploying RAGFlow on Kubernetes, making it easier to integrate the platform into containerized environments and manage deployments at scale.

v0.14.0 introduced support for OpenSearch as a document engine option alongside Elasticsearch and Infinity, providing users with more flexibility in their infrastructure choices.

**Section sources**
- [docs/release_notes.md](file://docs/release_notes.md#L14-L35)
- [docs/release_notes.md](file://docs/release_notes.md#L48-L57)
- [docs/release_notes.md](file://docs/release_notes.md#L95-L105)
- [docs/release_notes.md](file://docs/release_notes.md#L255-L264)
- [docs/release_notes.md](file://docs/release_notes.md#L333-L335)

## Enhancements

This section details the significant enhancements and improvements introduced in recent RAGFlow releases, focusing on performance, usability, and functionality improvements.

### Framework and Performance Improvements

RAGFlow has made substantial improvements to its underlying framework and performance characteristics. The v0.22.1 release upgraded the Flask web framework from synchronous to asynchronous, increasing concurrency and preventing blocking issues when requesting upstream LLM services. This change significantly improves the platform's ability to handle multiple concurrent requests efficiently.

v0.20.5 introduced Agent performance optimizations, improving planning and reflection speed for simple tasks and optimizing concurrent tool calls for parallelizable scenarios. These improvements significantly reduce overall response time, making Agents more efficient and responsive.

v0.18.0 removed built-in rerank models because they had minimal impact on retrieval rates but significantly increased retrieval time. This change improves overall system performance by eliminating an unnecessary processing step.

v0.14.0 optimized term weight calculations, reducing retrieval time by 50%. This enhancement makes searches faster and more efficient, improving the user experience when querying large datasets.

### User Interface and Experience

RAGFlow has consistently improved its user interface and overall user experience across multiple releases. v0.22.1 continued the redesign of the Profile page layouts, enhancing the visual consistency and usability of user management features.

v0.20.3 revamped the user interface for the Datasets, Chat, and Search pages, providing a more modern and intuitive user experience. The release also introduced document-level metadata filtering, allowing automatic or manual filtering during chats or searches.

v0.20.4 completed Chinese localization for the Agent component, making the platform more accessible to Chinese-speaking users. The release also improved Markdown file parsing with AST support to avoid unintended chunking and enhanced HTML parsing with bs4-based HTML tag traversal.

v0.15.0 introduced a dark mode to the UI, allowing users to toggle between light and dark themes. This enhancement improves usability in low-light environments and provides a more comfortable viewing experience for users who prefer dark interfaces.

### Model and Language Support

RAGFlow has expanded its support for various models and languages in recent releases. v0.22.0 added support for the Kimi-K2-Thinking model, while v0.21.0 added support for Tongyi Qwen 3 series, Claude Sonnet 4.5, and Meituan LongCat-Flash-Thinking models.

v0.20.5 expanded model support to include Meituan LongCat, Kimi (kimi-k2-turbo-preview and kimi-k2-0905-preview), Qwen (qwen3-max-preview), and SiliconFlow's DeepSeek V3.1.

v0.20.1 added support for the latest GPT-5 and Claude 4.1 models, ensuring compatibility with cutting-edge LLMs. v0.19.1 added support for Qwen 3 Embedding and Voyage Multimodal 3 models.

In terms of language support, v0.20.1 introduced a French language option, v0.17.2 introduced German, and v0.15.0 added Japanese. v0.14.0 added three new UI languages contributed by the community: Indonesian, Spanish, and Vietnamese.

### Data Processing and Retrieval

RAGFlow has made several enhancements to its data processing and retrieval capabilities. v0.20.5 introduced framework-level prompt blocks in the System prompt section, enabling customization and overriding of prompts at the framework level. This enhancement provides greater flexibility and control over Agent behavior.

v0.20.3 introduced the ability to compare answer performance of up to three chat model settings on a single Chat page, helping users evaluate different model configurations and select the optimal one for their use case.

v0.17.1 improved English tokenization quality and table extraction logic in Markdown document parsing, resulting in more accurate and reliable text processing.

v0.16.0 upgraded the Infinity document engine to v0.6.0.dev3 and added support for GPU acceleration for DeepDoc, significantly improving document processing speed and quality.

**Section sources**
- [docs/release_notes.md](file://docs/release_notes.md#L24-L25)
- [docs/release_notes.md](file://docs/release_notes.md#L123-L127)
- [docs/release_notes.md](file://docs/release_notes.md#L162-L167)
- [docs/release_notes.md](file://docs/release_notes.md#L193-L198)
- [docs/release_notes.md](file://docs/release_notes.md#L342-L346)

## Bug Fixes

This section details the significant bug fixes and issue resolutions included in recent RAGFlow releases, addressing stability, functionality, and usability issues.

### Data and File Management Issues

RAGFlow has addressed several critical issues related to data and file management. v0.22.1 fixed an issue from v0.22.0 where users failed to parse uploaded files or switch embedding models in a dataset containing parsed files using a built-in model from a -full RAGFlow edition.

v0.20.5 resolved an issue where deleted files remained searchable, ensuring that removed content is properly excluded from search results. The release also fixed an issue preventing chat with Ollama models.

v0.20.4 addressed problems with sharing resources with teams and inappropriate restrictions on the number and size of uploaded files. It also fixed an issue where citations infinitely increased in multi-turn conversations.

v0.19.1 fixed a memory leak issue during high-concurrency requests and resolved an issue where large file parsing would freeze when GraphRAG entity resolution was enabled.

v0.17.0 fixed issues with video parsing and resolved problems with API calling that were introduced in v0.17.1.

### User Interface and Display Issues

Several releases have focused on resolving user interface and display issues. v0.22.1 fixed an issue with images concatenated in Word documents and addressed problems with mixed images and text not being correctly displayed in chat history.

v0.20.4 resolved issues with previewing referenced files in responses and sending messages after file uploads. It also fixed an automatic line break issue in the prompt editor.

v0.17.0 fixed an issue where users could not preview diagrams or images in an AI chat, improving the visual experience when interacting with multimodal content.

v0.14.1 addressed an issue where users were unable to display or edit content of a chunk after clicking it, restoring full functionality to the chunk editing interface.

### Authentication and Security Issues

RAGFlow has addressed several authentication and security-related issues. v0.20.4 fixed an OAuth2 authentication failure, ensuring reliable third-party authentication.

v0.19.1 resolved an issue with role-based authentication for S3 bucket access, improving security when integrating with external storage services.

v0.17.1 fixed an issue with options missing in the PDF parser dropdown, ensuring users can properly configure document parsing methods.

### Performance and Stability Issues

Multiple releases have focused on improving performance and stability. v0.20.3 resolved an issue where the timeout mechanism introduced in v0.20.0 caused tasks like GraphRAG to halt, restoring proper task execution.

v0.19.1 fixed an excessive CPU usage issue caused by Ollama and resolved a context error occurring when using Sandbox in standalone mode.

v0.15.1 addressed slow response times in question-answering and AI search due to repetitive loading of the embedding model, significantly improving system responsiveness.

v0.14.1 resolved compatibility issues between Infinity and GraphRAG, ensuring stable operation when using these features together.

**Section sources**
- [docs/release_notes.md](file://docs/release_notes.md#L26-L31)
- [docs/release_notes.md](file://docs/release_notes.md#L137-L145)
- [docs/release_notes.md](file://docs/release_notes.md#L177-L187)
- [docs/release_notes.md](file://docs/release_notes.md#L207-L213)
- [docs/release_notes.md](file://docs/release_notes.md#L283-L292)

## Breaking Changes

This section details the significant breaking changes introduced in recent RAGFlow releases, which require special attention when upgrading from previous versions.

### v0.22.0: Slim Edition Docker Image

The most significant breaking change in v0.22.0 is the shift to shipping only the slim edition Docker image without embedding models. This change means that RAGFlow no longer includes built-in embedding models in its Docker images, and the -slim suffix has been removed from the image tag.

This change requires users to configure external embedding services or deploy their own embedding models. Users who previously relied on the built-in embedding models will need to update their configuration to point to external embedding services.

To migrate, users should:
1. Update their service_conf.yaml configuration to specify external embedding services
2. Ensure they have access to embedding models through supported providers
3. Test their configurations thoroughly before deploying to production

### v0.20.0: Agent Compatibility

v0.20.0 introduced a major breaking change by making Agents incompatible with earlier versions. All existing Agents from previous versions must be rebuilt following the upgrade to v0.20.0.

This change was necessary due to the comprehensive refactor of the Agent system, which introduced new capabilities like Multi-Agent configurations, planning and reflection, and visual functionalities. The internal structure of Agents was significantly modified to support these new features.

To migrate, users should:
1. Export existing Agents before upgrading
2. Upgrade to v0.20.0
3. Rebuild Agents using the new framework
4. Import any necessary configuration from the exported Agents

### v0.18.0: Removal of Built-in Rerank Models

v0.18.0 removed built-in rerank models because they had minimal impact on retrieval rates but significantly increased retrieval time. This change means that users can no longer rely on the built-in rerank models and must configure external reranking services if needed.

To migrate, users should:
1. Evaluate whether reranking is necessary for their use case
2. If reranking is required, configure external reranking services through supported providers
3. Test retrieval quality with and without reranking to determine the optimal configuration

### v0.17.2: Removal of Max_tokens Setting

v0.17.2 removed the Max_tokens setting from Chat configuration and several Agent components (Generate, Rewrite, Categorize, Keyword). This change means that response length is now controlled entirely by the model provider's settings rather than RAGFlow's configuration.

To migrate, users should:
1. Check the Max_tokens setting in their model provider configuration
2. Adjust these settings as needed to control response length
3. Update any documentation or processes that reference the Max_tokens setting in RAGFlow

### v0.14.0: Configuration File Change

v0.14.0 introduced a breaking change by replacing service_config.yaml with service_config.yaml.template for configuring backend services. This change requires users to update their configuration approach.

To migrate, users should:
1. Ensure both code and Docker image are upgraded to v0.14.0 or later
2. Update configurations in service_config.yaml.template instead of service_config.yaml
3. Restart RAGFlow to apply the changes

**Section sources**
- [docs/release_notes.md](file://docs/release_notes.md#L40-L45)
- [docs/release_notes.md](file://docs/release_notes.md#L251-L253)
- [docs/release_notes.md](file://docs/release_notes.md#L327-L330)
- [docs/release_notes.md](file://docs/release_notes.md#L360-L366)
- [docs/release_notes.md](file://docs/release_notes.md#L599-L607)

## Upgrade Instructions

This section provides detailed instructions for upgrading RAGFlow to the latest version, covering both online and offline environments.

### Standard Upgrade Process

To upgrade RAGFlow to the latest published release, follow these steps:

1. Stop the server:
```bash
docker compose -f docker/docker-compose.yml down
```

2. Update the local code:
```bash
git pull
```

3. Switch to the latest officially published release (e.g., v0.22.1):
```bash
git checkout -f v0.22.1
```

4. Update the RAGFLOW_IMAGE in ragflow/docker/.env:
```bash
RAGFLOW_IMAGE=infiniflow/ragflow:v0.22.1
```

5. Update the RAGFlow image and restart RAGFlow:
```bash
docker compose -f docker/docker-compose.yml pull
docker compose -f docker/docker-compose.yml up -d
```

It is crucial to upgrade both the code and Docker image to ensure compatibility. The entrypoint.sh file in the code must match the Docker image version to prevent issues.

### Upgrading to Nightly Build

To upgrade to the most recent tested Docker image (nightly build), follow the same process but use the nightly tag:

1. Stop the server:
```bash
docker compose -f docker/docker-compose.yml down
```

2. Update the local code:
```bash
git pull
```

3. Update the RAGFLOW_IMAGE in ragflow/docker/.env:
```bash
RAGFLOW_IMAGE=infiniflow/ragflow:nightly
```

4. Update the RAGFlow image and restart RAGFlow:
```bash
docker compose -f docker/docker-compose.yml pull
docker compose -f docker/docker-compose.yml up -d
```

Note that nightly builds are unstable and may contain experimental features or bugs. They should only be used in development or testing environments.

### Offline Upgrade Process

For environments without Internet access, follow these steps:

1. From an environment with Internet access, pull the required Docker image:
```bash
docker pull infiniflow/ragflow:v0.22.1
```

2. Save the Docker image to a .tar file:
```bash
docker save -o ragflow.v0.22.1.tar infiniflow/ragflow:v0.22.1
```

3. Copy the .tar file to the target server using a USB drive, network transfer, or other method.

4. Load the .tar file into Docker on the target server:
```bash
docker load -i ragflow.v0.22.1.tar
```

5. Follow the standard upgrade process, ensuring the RAGFLOW_IMAGE in .env matches the loaded image.

### Migration for Breaking Changes

When upgrading across versions with breaking changes, additional steps are required:

For v0.22.0+ (slim edition):
- Configure external embedding services in service_conf.yaml.template
- Update LLM factory settings with appropriate API keys
- Test document parsing and retrieval functionality

For v0.20.0+ (Agent compatibility):
- Export existing Agents before upgrading
- Rebuild Agents using the new framework after upgrading
- Test Agent workflows thoroughly

For v0.17.2+ (Max_tokens removal):
- Check and adjust Max_tokens settings in model provider configurations
- Update any scripts or automation that relied on RAGFlow's Max_tokens setting

### Verification and Testing

After upgrading, verify the installation by checking the server status:
```bash
docker logs -f docker-ragflow-cpu-1
```

Look for the successful launch message:
```
____   ___    ______ ______ __
/ __ \ /   |  / ____// ____// /____  _      __
/ /_/ // /| | / / __ / /_   / // __ \| | /| / /
/ _, _// ___ |/ /_/ // __/  / // /_/ /| |/ |/ /
/_/ |_|/_/  |_|\____//_/    /_/ \____/ |__/|__/

* Running on all addresses (0.0.0.0)
```

Test key functionality including:
- Creating and parsing documents in a dataset
- Running retrieval tests
- Starting a chat session with a knowledge base
- Testing Agent workflows (if applicable)

**Section sources**
- [docs/guides/upgrade_ragflow.mdx](file://docs/guides/upgrade_ragflow.mdx#L16-L100)
- [README.md](file://README.md#L190-L205)
- [docs/release_notes.md](file://docs/release_notes.md#L207-L213)

## Compatibility Considerations

This section addresses compatibility considerations for RAGFlow across different deployment environments, hardware platforms, and third-party dependencies.

### Hardware and Platform Compatibility

RAGFlow officially supports x86 CPU and Nvidia GPU platforms. The Docker images are built for x86 platforms, and ARM64 platform support is limited. While RAGFlow can be tested on ARM64 platforms, Docker images are not maintained for ARM architectures.

For ARM64 platforms, users must build their own Docker images following the guide in the documentation. This requires installing build dependencies and using the appropriate build commands to create a compatible image.

GPU acceleration is supported for DeepDoc tasks, but requires specific configuration. Users must set the DEVICE environment variable to 'gpu' and ensure their Docker environment has access to GPU resources through the NVIDIA Container Toolkit.

### Operating System Compatibility

RAGFlow can be deployed on various operating systems, but configuration requirements differ:

For Linux:
- Ensure vm.max_map_count >= 262144
- Install jemalloc if not present (Ubuntu: libjemalloc-dev, CentOS: jemalloc, OpenSUSE: jemalloc, macOS: brew install jemalloc)

For macOS:
- Use Docker Desktop with gVisor for code executor functionality
- Set vm.max_map_count using Docker commands or launch daemon configuration
- The docker-compose-macos.yml file is provided but not actively maintained

For Windows:
- Use Docker Desktop with WSL 2 backend for best performance
- Configure vm.max_map_count through WSL settings
- The docker-compose-windows.yml file is not provided, requiring manual configuration

### Third-Party Dependency Compatibility

RAGFlow integrates with various third-party services and dependencies, with specific compatibility considerations:

Document Engines:
- Elasticsearch (default): Requires vm.max_map_count >= 262144
- Infinity: Not officially supported on Linux/arm64
- OpenSearch: Supported as an alternative to Elasticsearch
- OceanBase: Supported for vector storage

Storage Services:
- MinIO: Supported as an S3-compatible storage option
- AWS S3: Fully supported with appropriate credentials
- Azure Blob: Supported for object storage
- Google Drive: Supported for data synchronization
- Aliyun OSS: Supported as a file storage option

LLM Providers:
- OpenAI: Supported including GPT-5 series
- Anthropic: Supported including Claude 4 series
- Google: Supported including Gemini models
- Moonshot: Supported
- Tongyi-Qianwen: Supported
- ZHIPU-AI: Supported
- Local models via Ollama, Xinference, or LocalAI

Authentication Services:
- OAuth2: Supported for third-party authentication
- OIDC: Supported for identity provider integration
- GitHub: Supported for authentication

### Browser and Client Compatibility

RAGFlow's web interface is compatible with modern browsers including Chrome, Firefox, Safari, and Edge. The interface supports multiple languages including English, Chinese (simplified and traditional), Japanese, Korean, Indonesian, Portuguese (Brazil), and others.

The Python SDK is compatible with Python 3.10-3.12, as specified in the pyproject.toml file. Users should ensure their Python environment meets these requirements when using the SDK.

### Version Compatibility

When upgrading RAGFlow, it is essential to ensure compatibility between the code version and Docker image version. The entrypoint.sh file in the code must match the Docker image version to prevent issues. Users should always upgrade both components simultaneously.

For integrations with external systems, users should verify compatibility with the specific RAGFlow version they are using, as APIs and interfaces may change between versions.

**Section sources**
- [README.md](file://README.md#L146-L150)
- [README.md](file://README.md#L187-L188)
- [README.md](file://README.md#L292-L293)
- [docker/README.md](file://docker/README.md#L21)
- [pyproject.toml](file://pyproject.toml#L8)

## Release Cycle and Support Policy

This section outlines RAGFlow's release cycle and support policy, providing guidance on versioning, release frequency, and support expectations.

### Release Cycle

RAGFlow follows a regular release cycle with both stable releases and nightly builds. Stable releases are published with version numbers in the format vX.Y.Z (e.g., v0.22.1), while nightly builds are available with the 'nightly' tag.

The release history shows a pattern of frequent updates, with major releases approximately every 1-2 months. The version numbering follows semantic versioning principles:
- Major version (X): Significant changes, potential breaking changes
- Minor version (Y): New features and enhancements
- Patch version (Z): Bug fixes and minor improvements

Stable releases are published on GitHub and Docker Hub, with release notes detailing changes, improvements, and bug fixes. The project maintains a roadmap that outlines future development plans and priorities.

### Support Policy

RAGFlow provides support for the latest stable release and recent previous versions. Users are encouraged to upgrade to the latest version to receive bug fixes, security updates, and new features.

The project offers multiple channels for support and community engagement:
- GitHub Issues: For reporting bugs and requesting features
- GitHub Discussions: For general questions and community discussions
- Discord: For real-time chat and community support
- Twitter: For announcements and updates

Documentation is maintained in the docs directory and published on the project website. The documentation includes quickstart guides, user guides, developer guides, and API references.

### Deprecation Policy

When features are deprecated, RAGFlow provides clear warnings in the logs and documentation. For example, the v0.17.2 release removed the Max_tokens setting with a clear notice that users should check the model provider's settings instead.

Breaking changes are introduced with clear documentation and migration guidance. The v0.22.0 release, which changed to only shipping slim edition Docker images, included detailed instructions for configuring external embedding services.

### Long-Term Support

While RAGFlow does not currently offer formal long-term support (LTS) versions, the project maintains backward compatibility where possible and provides migration guidance for breaking changes.

Users requiring stable, long-term deployments are advised to:
1. Pin to specific version tags rather than using 'latest' or 'nightly'
2. Test upgrades thoroughly in staging environments before deploying to production
3. Monitor the release notes for breaking changes and deprecations
4. Subscribe to release notifications to stay informed of updates

The project's active development and regular release cycle suggest a commitment to ongoing improvement and support, with a focus on maintaining a stable and reliable platform for users.

**Section sources**
- [docs/release_notes.md](file://docs/release_notes.md#L6-L800)
- [README.md](file://README.md#L387-L389)
- [README.md](file://README.md#L391-L395)

## Obtaining the Latest Version

This section provides instructions for obtaining the latest version of RAGFlow through various distribution channels and verifying its integrity.

### Docker Distribution

The primary method for obtaining RAGFlow is through Docker images published on Docker Hub. Users can pull the latest stable release or nightly build using the following commands:

For the latest stable release:
```bash
docker pull infiniflow/ragflow:v0.22.1
```

For the nightly build:
```bash
docker pull infiniflow/ragflow:nightly
```

Alternative mirrors are available for users with limited access to Docker Hub:
```bash
# Huawei Cloud mirror
docker pull swr.cn-north-4.myhuaweicloud.com/infiniflow/ragflow:nightly

# Alibaba Cloud mirror  
docker pull registry.cn-hangzhou.aliyuncs.com/infiniflow/ragflow:nightly
```

### Source Code Distribution

RAGFlow is open-source and available on GitHub. Users can obtain the latest version by cloning the repository:

```bash
git clone https://github.com/infiniflow/ragflow.git
cd ragflow
```

To check out a specific release:
```bash
git checkout v0.22.1
```

The source code includes the complete project with all components, documentation, and configuration files.

### Python SDK

The RAGFlow Python SDK is available through PyPI and can be installed using pip:

```bash
pip install ragflow-sdk==0.22.1
```

The SDK version should match the server version for optimal compatibility.

### Version Verification

To verify the version of a RAGFlow installation, users can check several sources:

1. The pyproject.toml file contains the version number:
```toml
[project]
name = "ragflow"
version = "0.22.1"
```

2. The Docker image tag indicates the version:
```bash
docker images | grep ragflow
```

3. The release notes file (docs/release_notes.md) lists the latest release at the top.

4. At runtime, the version can be retrieved through the API or by checking the server logs during startup.

### Integrity Verification

To ensure the integrity of downloaded components:

1. Verify Docker image signatures if available
2. Check the GitHub repository's integrity through HTTPS
3. Validate file hashes when available
4. Use trusted sources (official Docker Hub, GitHub repository)

For production deployments, users should:
1. Pin to specific version tags rather than using 'latest'
2. Verify checksums of downloaded files when available
3. Use trusted networks for downloading components
4. Scan Docker images for vulnerabilities before deployment

### Alternative Distribution Methods

For users with limited Internet access, RAGFlow can be distributed through offline methods:

1. Save Docker images to tar files:
```bash
docker save -o ragflow.v0.22.1.tar infiniflow/ragflow:v0.22.1
```

2. Transfer the tar file via USB drive or secure file transfer

3. Load the image on the target system:
```bash
docker load -i ragflow.v0.22.1.tar
```

The source code can also be distributed as a zip archive or through internal Git repositories.

**Section sources**
- [pyproject.toml](file://pyproject.toml#L3)
- [README.md](file://README.md#L24-L25)
- [docker/README.md](file://docker/README.md#L80)
- [docs/release_notes.md](file://docs/release_notes.md#L10-L13)

## Conclusion

RAGFlow has demonstrated rapid and consistent development through its regular release cycle, introducing significant new features, enhancements, and bug fixes with each version. The platform has evolved from a basic RAG engine to a sophisticated system with advanced Agent capabilities, multimodal processing, and comprehensive workflow orchestration.

The release history shows a clear focus on improving document understanding, expanding integration options, and enhancing user experience. Key developments include the introduction of orchestratable ingestion pipelines, Deep Research capabilities, MCP functionality, and support for various document and data sources.

The platform's architecture supports flexible deployment options, with Docker being the primary distribution method. The recent shift to slim Docker images without built-in embedding models reflects a strategic decision to focus on core RAG functionality while allowing users to integrate with their preferred embedding services.

For users upgrading between versions, careful attention should be paid to breaking changes, particularly the Agent compatibility break in v0.20.0 and the shift to slim Docker images in v0.22.0. The comprehensive upgrade instructions and migration guidance provided in this document should help ensure smooth transitions.

RAGFlow's active development, regular releases, and responsive issue resolution indicate a healthy open-source project with a clear roadmap and commitment to continuous improvement. The platform is well-positioned to meet the evolving needs of organizations implementing RAG solutions, with a strong foundation for future enhancements in areas like multimodal processing, workflow automation, and enterprise integration.

**Section sources**
- [docs/release_notes.md](file://docs/release_notes.md#L6-L800)
- [README.md](file://README.md#L73-L75)