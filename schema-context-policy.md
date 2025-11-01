---
title: Schema Context Policy
description: This page defines how vector payloads adapt their schema conventions and key naming to align with the correct transport protocol (REST vs SDK/gRPC) to ensure successful Zilliz insert and upsert operations.
published: true
date: 2025-11-01T06:41:21.576Z
tags: vector database, schema
editor: markdown
dateCreated: 2025-10-29T22:16:03.613Z
---

# **REVREBEL Schema Context Policy**

**Version:** 1.0
**Author:** REVREBEL Vector Data Manager
**Created:** 2025-10-29
**Purpose:** Define how vector payloads adapt their schema conventions and key naming to align with the correct transport protocol (REST vs SDK/gRPC) to ensure successful Zilliz insert and upsert operations.

---

## **1. Purpose and Scope**

This policy defines the adaptive schema rules for the **REVREBEL Vector Data Manager** when performing vector insert, upsert, or search operations across all Zilliz-based collections.

Its primary goal is to guarantee **schema compatibility and operational reliability** by dynamically aligning payload formats with the connection method being used — **REST API** or **SDK/gRPC API**.

---

## **2. Context-Aware Schema Selection**

The Vector Data Manager must always inspect the **target transport type** before constructing or sending any vector payload.

| Transport Method                | Expected Root Key | Primary Key Field | Casing Style   | Usage Context                                                           |
| ------------------------------- | ----------------- | ----------------- | -------------- | ----------------------------------------------------------------------- |
| **Zilliz REST API (HTTP/JSON)** | `collectionName`  | `primary_key`     | **camelCase**  | Used for public RESTful insert routes and Zilliz Cloud UI integrations. |
| **Zilliz SDK / gRPC API**       | `collection_name` | `id`              | **snake_case** | Used for internal or system-level SDK clients and automated pipelines.  |

---

## **3. Protocol-Aware Behavior Rules**

* Detect the **endpoint type** dynamically:

  * If endpoint URL includes `/rest/`, `/v1/vector/insert`, or a Zilliz Cloud HTTPS route → assume **REST schema**.
  * If communication occurs over gRPC or SDK connection → assume **SDK schema**.
* The system must **log or echo** the selected schema mode before every write or search operation for transparent auditing.
* REST payloads must adhere strictly to Zilliz REST specification casing; SDK payloads follow Milvus client schema.

---

## **4. Priority Hierarchy**

When discrepancies occur between:

1. System-level configuration
2. Knowledge-base schema documentation
3. Runtime transport context

**Runtime context takes precedence.**
The payload must always match the schema required by the active API method.

> **Rule of Thumb:** *“The transport defines the truth.”*

---

## **5. Embedding and Data Handling Rules**

| Behavior                 | Description                                                                                                    |
| ------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **Embedding Generation** | If `vector` is null and embedding generation is permitted, use OpenAI’s `text-embedding-3-small` (1536 dims).  |
| **Upsert Policy**        | Deterministic updates: same ID overwrites existing vector. Reject null vectors unless flagged as placeholders. |
| **Field Validation**     | Enforce all required fields: `persona_id`, `persona_version`, `status`, `created_ym`.                          |
| **Error Handling**       | Return concise Zilliz error summaries with `request_id`. Do not expose credentials.                            |
| **Provenance Metadata**  | Every record includes `author`, `persona_version`, `created_ym`.                                               |

---


## **6. Implementation Pseudocode**

```yaml
if transport == "REST":
    payload:
        collectionName: string
        data:
          - primary_key: string
            vector: [float, float, ...]
            ...
else if transport == "SDK":
    payload:
        collection_name: string
        data:
          - id: string
            vector: [float, float, ...]
            ...
```

---

## **7. Design Philosophy**

> “The Vector Manager is not just a data handler — it’s a context-aware bridge between schema design and database execution.”

Its purpose is not only to deliver valid data but to ensure the **semantic and structural integrity** of every vector operation, across any protocol layer.

---

## **8. Outcomes of Policy Alignment**

* No “no Route matched…” upload errors.
* Automatic schema casing adaptation for all future operations.
* Consistent, human-readable inserts matching documentation examples.
* Traceable, self-describing insert logs with declared mode and target collection.

---

## **9. Document Metadata**

| Field                    | Value                                                                                  |
| ------------------------ | -------------------------------------------------------------------------------------- |
| **Document Type**        | Configuration Policy                                                                   |
| **Applies To**           | REVREBEL Vector Data Manager                                                           |
| **Collections Affected** | `persona_revrebel`, `prod_doc_search`, `prod_recommendations`, `prod_asset_embeddings` |
| **Last Updated**         | 2025-10-29                                                                             |
| **Maintained By**        | REVREBEL Core System                                                                   |
