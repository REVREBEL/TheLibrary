---
title: Vector Database Schemas
description: This document defines the schema specifications for all REVREBEL vector database collections hosted on Zilliz Cloud. Each collection is optimized for specific content types and retrieval patterns.
published: true
date: 2025-11-01T06:41:26.553Z
tags: vector database, schema, zilliz
editor: markdown
dateCreated: 2025-10-29T22:34:51.220Z
---

# REVREBEL Vector Database Schemas
## Zilliz Cloud Platform

**Platform:** Zilliz Cloud (Managed Milvus)  
**SDK:** PyMilvus / Milvus SDK  
**Vector Dimensions:** 1536 (OpenAI text-embedding-3-small) for text, 512 (CLIP) for images

This document defines the schema specifications for all REVREBEL vector database collections hosted on Zilliz Cloud. Each collection is optimized for specific content types and retrieval patterns.

---

## persona_revrebel Schema 

**Purpose:** Brand personality, voice, and tone guidance for AI-powered content generation

<br>

| Field Name       | Field Type              | Nullable | Default Value | Description                                                       | Index |
| ---------------- | ----------------------- | :------: | ------------- | ----------------------------------------------------------------- | :---: |
| primary_key (PK) | VARCHAR (128)           |    No    | —             | The Primary Key                                                   |       |
| vector           | FLOAT_VECTOR (1536)     |    No    | —             | Text embedding vector (OpenAI text-embedding-3-small)             |   ✅   |
| persona_id       | VARCHAR (32)            |    No    | —             | Unique persona identifier (e.g., revrebel_core, revrebel_founder) |       |
| persona_version  | VARCHAR (32)            |    No    | —             | Semantic version (e.g., v2.0, v2.1-experimental)                  |   ✅   |
| module           | VARCHAR (64)            |    No    | —             | Top-level module (Brand Core, Brand Voice & Style, etc.)          |   ✅   |
| submodule        | VARCHAR (64)            |    No    | —             | Specific submodule within module                                  |   ✅   |
| status           | VARCHAR (14)            |    No    | —             | active, deprecated, or draft                                      |   ✅   |
| tone             | ARRAY<VARCHAR (24)>[10] |    Yes   | —             | Tone descriptors (e.g., witty, confident, rebellious)             |   ✅   |
| use_case         | VARCHAR (64)            |    Yes   | —             | Primary application context                                       |   ✅   |
| audience         | ARRAY<VARCHAR (32)>[10] |    Yes   | —             | Target audience                                                   |   ✅   |
| platform         | ARRAY<VARCHAR (24)>[8]  |    Yes   | —             | Sub-platform (e.g., linkedin under social)                        |   ✅   |
| channel          | ARRAY<VARCHAR (24)>[8]  |    Yes   | —             | Specific channel (LinkedIn, Blog, Email, etc.)                    |   ✅   |
| severity         | VARCHAR (10)            |    Yes   | —             | Rule importance (must, should, prefer, avoid, never)              |       |
| author           | VARCHAR (32)            |    Yes   | REVREBEL      | Creator identifier                                                |       |
| language         | VARCHAR (2)             |    Yes   | en            | Content language (default: en)                                    |       |
| created_ym       | VARCHAR (8)             |    No    | —             | Creation date (YYYY-MM format)                                    |       |
| confidence_score | FLOAT                   |    Yes   | —             | Certainty level (0.0–1.0)                                         |   ✅   |
| weight           | FLOAT                   |    Yes   | —             | Importance weighting (0.0–1.0)                                    |   ✅   |
| dynamic_field    | JSON                    |    Yes   | —             | Flexible fields for evolving metadata                             |       |

<br> 
<br> 

## prod\_asset_embeddings Schema 

**Purpose:** Visual and audio asset semantic search with multimodal embeddings

<br>

| Field Name       | Field Type              | Nullable | Default Value | Description                                               | Index |
| ---------------- | ----------------------- | :------: | ------------- | --------------------------------------------------------- | :---: |
| primary_key (PK) | VARCHAR (128)           |    No    | —             | The Primary Key                                           |       |
| vector           | FLOAT_VECTOR (1536)      |    No    | —             | Visual/audio embedding (CLIP/CLAP models)                 |   ✅   |
| domain           | VARCHAR (32)            |    No    | —             | Primary content category (e.g., coding, design, strategy) |   ✅   |
| asset_type       | VARCHAR (32)            |    Yes   | —             | Type of asset (logo, photo, graphic, video, audio)        |   ✅   |
| format           | VARCHAR (4)             |    Yes   | —             | File format (png, svg, jpg, ttf, mp4, wav, pdf)           |   ✅   |
| visual_tags      | ARRAY<VARCHAR (16)>[10] |    Yes   | —             | Visual characteristics (e.g., ["geometric","bold"])       |   ✅   |
| color_palette    | ARRAY<VARCHAR (7)>[20]  |    Yes   | —             | Dominant colors (e.g., ["#163666","#047C97"])             |       |
| style_era        | ARRAY<VARCHAR (32)>[6]  |    Yes   | —             | Design era (e.g., "arcade","retro-future")                |       |
| variant          | VARCHAR (32)            |    Yes   | —             | Asset variant (primary, secondary, icon, stacked, circle) |   ✅   |
| approval_status  | VARCHAR (12)            |    Yes   | —             | Approval state (approved, review, draft)                  |   ✅   |
| last_updated     | VARCHAR (11)            |    No    | —             | Last date updated in yyyy-MM-dd format                    |       |
| created_ym       | VARCHAR (8)             |    No    | —             | Creation month in yyyy-MM format                          |   ✅   |
| dynamic_field    | JSON                    |    Yes   | —             | Flexible fields for evolving metadata                     |       |

<br> 
<br> 

## prod_doc_search Schema

**Purpose:** Full-text search and retrieval of documentation, guides, and knowledge base content

<br> 

| Field Name       | Field Type          | Nullable | Default Value | Description                                               | Index |
| ---------------- | ------------------- | :------: | ------------- | --------------------------------------------------------- | :---: |
| primary_key (PK) | VARCHAR (128)       |    No    | —             | Primary Key                                               |       |
| vector           | FLOAT_VECTOR (1536) |    No    | —             | Text embedding (OpenAI text-embedding-3-small)            |   ✅   |
| domain           | VARCHAR (32)        |    No    | —             | Primary content category (e.g., coding, design, strategy) |   ✅   |
| program          | VARCHAR (64)        |    Yes   | —             | System or app area (e.g., webflow, budibase, n8n)        |   ✅   |
| artifact_type    | VARCHAR (64)        |    Yes   | —             | Content type (e.g., howto, checklist, prompt)             |   ✅   |
| created_ym       | VARCHAR (8)         |    No    | —             | Creation month (YYYY-MM)                                  |   ✅   |
| dynamic_field    | JSON                |    Yes   | —             | Flexible fields for evolving metadata                     |       |

<br> 
<br> 

## prod_recommendations Schema

**Purpose:** Personalized content recommendations and contextual suggestions

<br> 

| Field Name       | Field Type              | Nullable | Default Value | Description                                                       | Index |
| ---------------- | ----------------------- | :------: | ------------- | ----------------------------------------------------------------- | :---: |
| primary_key (PK) | VARCHAR (128)           |    No    | —             | The Primary Key                                                   |       |
| vector           | FLOAT_VECTOR (1536)     |    No    | —             | Text embedding (OpenAI text-embedding-3-small)                    |   ✅   |
| persona_id       | VARCHAR (32)            |    No    | —             | Unique persona identifier (e.g., revrebel_core, revrebel_founder) |       |
| persona_version  | VARCHAR (32)            |    No    | —             | Semantic version (e.g., v2.0, v2.1-experimental)                  |   ✅   |
| module           | VARCHAR (64)            |    No    | —             | Top-level module (Brand Core, Brand Voice & Style, etc.)          |   ✅   |
| submodule        | VARCHAR (64)            |    No    | —             | Specific submodule within module                                  |   ✅   |
| status           | VARCHAR (14)            |    No    | —             | active, deprecated, or draft                                      |   ✅   |
| tone             | ARRAY<VARCHAR (24)>[10] |    Yes   | —             | Tone descriptors (e.g., witty, confident, rebellious)             |   ✅   |
| use_case         | VARCHAR (64)            |    Yes   | —             | Primary application context                                       |   ✅   |
| audience         | ARRAY<VARCHAR (32)>[10] |    Yes   | —             | Target audience                                                   |   ✅   |
| platform         | ARRAY<VARCHAR (24)>[8]  |    Yes   | —             | Sub-platform (e.g., linkedin under social)                        |   ✅   |
| channel          | ARRAY<VARCHAR (24)>[8]  |    Yes   | —             | Specific channel (LinkedIn, Blog, Email, etc.)                    |   ✅   |
| severity         | VARCHAR (10)            |    Yes   | —             | Rule importance (must, should, prefer, avoid, never)              |       |
| author           | VARCHAR (32)            |    Yes   | REVREBEL      | Creator identifier                                                |       |
| language         | VARCHAR (2)             |    Yes   | en            | Content language (default: en)                                    |       |
| created_ym       | VARCHAR (8)             |    No    | —             | Creation date (YYYY-MM format)                                    |       |
| confidence_score | FLOAT                   |    Yes   | —             | Certainty level (0.0–1.0)                                         |   ✅   |
| weight           | FLOAT                   |    Yes   | —             | Importance weighting (0.0–1.0)                                    |   ✅   |
| dynamic_field    | JSON                    |    Yes   | —             | Flexible fields for evolving metadata                             |       |

---

## Zilliz Cloud Configuration Notes

### Index Configuration
- **Metric Type:** COSINE (for semantic similarity)
- **Index Type:** HNSW (Hierarchical Navigable Small World)
- **HNSW Parameters:**
  - `M`: 16-32 (higher M = better recall, more memory)
  - `efConstruction`: 100-500 (higher = better index quality, slower build)

### Query Parameters
- **ef:** 64-256 (higher = better recall, slower search)
- **topK:** Typically 5-50 results per query

### Collection Management
```python
from pymilvus import connections, Collection

# Connect to Zilliz Cloud
connections.connect(
    alias="default",
    uri=ZILLIZ_CLOUD_URI,
    token=ZILLIZ_CLOUD_TOKEN
)

# Access collection
collection = Collection("persona_revrebel")
collection.load()  # Load into memory for queries
```

### Best Practices
1. **Batch Operations:** Insert/update in batches of 100-1000 for optimal performance
2. **Field Indexing:** Index frequently filtered fields (status, module, persona_version, etc.)
3. **Dynamic Fields:** Use JSON dynamic fields for evolving schema without migration
4. **Versioning:** Always include `persona_version` and `status` for safe updates

---

**Last Updated:** 2025-10-28  
**Platform:** Zilliz Cloud (Managed Milvus)  
**Maintained by:** REVREBEL Brand & Data Engineering  
**Version:** 2.1 (Migrated from Cloudflare Vectorize to Zilliz)