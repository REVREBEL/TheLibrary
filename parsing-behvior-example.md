---
title: Parsing Behavior Example
description: A detailed look at a real world example and the decision tree for making data decisions on classification and recall structure.
published: true
date: 2025-11-01T06:41:28.178Z
tags: 
editor: markdown
dateCreated: 2025-10-29T22:54:22.164Z
---


# Brandscape: Logic Parsing + Decision Tree 

<br>

A detailed look at a real world example and the decision tree for making data decisions on classification and recall structure.





| Stage                   | Behavior                                                                                                                          | Effect                                                                                  |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Parsing**             | Detects any hint of instructive phrasing (should, avoid, never, aim to, must, encourage, etc.).                                   | Creates **Matrix** entries automatically, even if not explicitly formatted as Do/Don’t. |
| **Dual Mode**           | When a narrative contains both philosophy *and* guidance, it always creates **both Canonical + Matrix** entries (never just one). | Maximum tokenization for search and downstream application.                             |
| **Granularity**         | Breaks long sentences into smaller logical rules (“Aim to write clearly. Avoid jargon.” → 2 rules).                               | Each rule gets its own embedding + metadata object.                                     |
| **Linkage**             | Every rule record links back to its Canonical parent via `source_id`.                                                             | Preserves context hierarchy for retrieval.                                              |
| **Metadata enrichment** | Automatically assigns `rule_type` (do/dont/suggest), `severity`, and contextual tone fields.                                      | Keeps precision and context balanced.                                                   |




| Type                          | Purpose                                  | Zilliz effect                           | Vector weight |
| ----------------------------- | ---------------------------------------- | --------------------------------------- | ------------- |
| **Main BRANDSCAPE entry**     | Holds the full narrative + deep metadata | Anchor for semantic search, full recall | **1.0**       |
| **Quick reference entry**     | Answers “What’s REVREBEL like?”          | Short, high-precision search return     | ~0.5          |
| **Cultural references entry** | Surface cultural lookup / analogies      | Reinforces conceptual associations      | ~0.4          |


<br>
<br>


##⚙️ How REVREBEL should execute
When you submit a new Brandscape:

#### 1. Extract full text → generate native embedding (text-embedding-3-small).

#### 2. Create deterministic IDs

* revrebel_core_personality_brandscape_v1
* revrebel_quick_ref_personality
* revrebel_cultural_refs_personality

#### 3. Assemble the three JSON payloads as you outlined:

* one “rich” object (full content + metadata + dynamic_field)
* two “supporting search entries” (lightweight, different submodules)

#### 4. POST all three to

```bash
https://YOUR_ZILLIZ_ENDPOINT/v1/vector/insert
```
as a batch:

```json
{
  "collection_name": "persona_revrebel",
  "data": [
    { ...main entry... },
    { ...quick_ref entry... },
    { ...cultural_refs entry... }
  ]
}
```


###✅ What the GPT will now be able to do
* Accept a single rich input (your BRANDSCAPE narrative).
* Auto-generate all supporting derivative entries with proper metadata links (source field).
* Maintain schema compliance (created_ym, enums, deterministic IDs).
* Assign weighted confidence values (main = 1.0, support < 1).
* Return the complete JSON payload ready for immediate REST submission.

<br>

## ⚙️ How REVREBEL Should Behave (Integrated Logic)
### 1️⃣ Step One — Classify the Input
When a new piece of text arrives:

* **If it describes strategy, philosophy, tone, or brand identity → classify as a Canonical Principle.**
 		
	→ e.g., your BRANDSCAPE main narrative.

* **If it contains executable rules, “Do/Don’t”, or behavior logic → classify as a Matrix Entry.**

* **If both → create both types:**

	* 	Canonical record for conceptual context.
	* 	Matrix entries for the specific rules.

This decision is now automated using keywords and structural cues (e.g., detection of imperative phrasing, “Do/Don’t”, “should”, “avoid”).

<br>

### 2️⃣ Step Two — Apply the Brandscape Multi-Entry Pattern (when Canonical)
If the classification = Canonical, REVREBEL performs:

1. Create one Main entry (rich, full narrative, weight = 1.0).
2. Create supporting entries:

	* Quick Reference (summarized answer form)
	* Cultural References (metadata focus)

All three share the same persona_id, persona_version, and source linkage, ensuring that Zilliz searches can resolve conceptual and atomic data together.

<br>

### 3️⃣ Step Three — Matrix Handling
If classification = Matrix or Dual:

* Parse individual rule statements.
* Create atomic entries per rule with:

```json
{
  "module": "Brand Voice & Style",
  "submodule": "Voice Matrix",
  "attribute": "Transparency",
  "rule_type": "do" | "dont",
  "severity": "must" | "should",
  ...
}
```

<br>

### 4️⃣ Step Four — Vector + Metadata Construction
Every record (Canonical or Matrix) goes through:

1. Embedding via OpenAI text-embedding-3-small → 1536-dim vector.
2. Metadata validation (created_ym, enums, etc.).
3. Deterministic ID creation (e.g., revrebel_core_personality_brandscape_v1, 

	vm_transparent_trustworthy_do).

4 Batch upsert to Zilliz Cloud in one payload.

<br>

### 5️⃣ Step Five — Result

One payload can contain mixed entry types:


```json
{
  "collection_name": "persona_revrebel",
  "data": [
    { ...Canonical main... },
    { ...Quick reference... },
    { ...Cultural references... },
    { ...Matrix rule A... },
    { ...Matrix rule B... }
  ]
}

```

Zilliz search results remain composable — you can query conceptually (“brand personality”) or operationally (“do rules for tone”).

<br>

## ✅ Why This Integration Is Powerful


| Layer                                 | Purpose                                                     | Benefit                            |
| ------------------------------------- | ----------------------------------------------------------- | ---------------------------------- |
| **Classification Overview (MD file)** | Decides whether content is Canonical, Matrix, or both       | Guarantees schema consistency      |
| **Brandscape Triple-Entry Model**     | Defines how Canonical narratives expand into search entries | Rich recall + precise retrieval    |
| **Zilliz Integration**                | Stores all entries with weighted embeddings                 | Scalable, query-ready architecture |
| **REVREBEL Behavior Context**         | Executes this logic automatically                           | Reliable ingestion and retrieval   |

<br>

## 🔹 1. Classification Layer Integration

* It will reference your Classification Overview.md as a rulebook.
* Before embedding anything, it will classify the input into:
	* Canonical Principle
	* Matrix Entry
	* Dual (both Canonical + Matrix)
	
This classification will trigger different vector generation behaviors.

<br>

## 🔹 2. Behavior by Class

| Type          | What It Produces                                                                   | Example Output                       |
| ------------- | ---------------------------------------------------------------------------------- | ------------------------------------ |
| **Canonical** | → Full **Brandscape triple-entry** (Main + Quick Reference + Cultural References). | Your JSON pattern above.             |
| **Matrix**    | → Multiple atomic “Do/Don’t” or “Rule” entries under `Voice Matrix`.               | Each with unique IDs and rule types. |
| **Dual**      | → One Canonical + Multiple Matrix entries.                                         | Combines both flows.                 |

<br>

## 🔹 3. Metadata Logic

* Canonical entries get: confidence_score=1.0, weight=1.0
* Supporting entries get: weight ~0.4–0.6, source link to main
* Matrix rules get: rule_type, severity, and weight according to the Classification document

<br>

## 🔹 4. Upsert Behavior
* It batches all entries (Canonical + Matrix) into a single Zilliz insert payload.
* Each payload contains fully validated, deterministic IDs.
* Outputs in ready-to-send JSON, optionally with cURL syntax for immediate execution.

<br>


## 🔹 5. Result
You’ll be able to drop any kind of content (from “BRANDSCAPE” to “Voice Rules”) into REVREBEL and it will:

* Classify it,
* Generate embeddings natively,
* Create all relevant entries with metadata,
* Return a single JSON ready for Zilliz.


<br>
<br>

## 🧠 What “Aggressive Classification” Actually Does

| Stage                   | Behavior                                                                                                                          | Effect                                                                                  |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Parsing**             | Detects any hint of instructive phrasing (should, avoid, never, aim to, must, encourage, etc.).                                   | Creates **Matrix** entries automatically, even if not explicitly formatted as Do/Don’t. |
| **Dual Mode**           | When a narrative contains both philosophy *and* guidance, it always creates **both Canonical + Matrix** entries (never just one). | Maximum tokenization for search and downstream application.                             |
| **Granularity**         | Breaks long sentences into smaller logical rules (“Aim to write clearly. Avoid jargon.” → 2 rules).                               | Each rule gets its own embedding + metadata object.                                     |
| **Linkage**             | Every rule record links back to its Canonical parent via `source_id`.                                                             | Preserves context hierarchy for retrieval.                                              |
| **Metadata enrichment** | Automatically assigns `rule_type` (do/dont/suggest), `severity`, and contextual tone fields.                                      | Keeps precision and context balanced.                                                   |

<br>


### ⚙️ Practical Result
When you paste a full brand section like a Voice & Tone Guide or BRANDSCAPE narrative, you’ll get something like this:

1. **Main Canonical Record**

	→ rich, full entry for context and embedding weight 1.0

2. **3 Support Canonical Records**

	→ Quick Ref / Cultural / Summary

3. **X Matrix Entries**

	→ all detected behavioral or directive statements, each a separate atomic vector
	→ weight 0.4–0.6

Everything is upserted together in one Zilliz batch payload.

<br>

### 🔍 Long-Term Advantage
This will make your vector database:

* Extremely searchable (many small, high-precision vectors)
* Cross-functional for future training, prompt grounding, and validation systems
* Composable — you can query by tone, rule_type, or context to dynamically generate or QA brand content.


