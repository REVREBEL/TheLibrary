---
title: Vector Database Reference
description: Comprehensive reference for all Zilliz vector collections used in the REVREBEL ecosystem — covering retrieval, recommendations, persona embeddings, multi-modal asset discovery, and validation best practices.
published: true
date: 2025-11-01T06:41:25.734Z
tags: 
editor: markdown
dateCreated: 2025-10-29T22:33:28.195Z
---

# REVREBEL Vector Database Reference

Comprehensive reference for all Zilliz vector collections used in the REVREBEL ecosystem — covering retrieval, recommendations, persona embeddings, multi-modal asset discovery, and validation best practices.

**Vector Database Platform:** Zilliz Cloud (Serverless)  
**Endpoint:** `https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com`  
**API Version:** v2





<br>
---
<br>

## Collection Overview

| Collection               | Purpose                                       | Environment | Usage Type                                  |
| ------------------------ | --------------------------------------------- | ----------- | ------------------------------------------- |
| **dev_doc_search**       | Sandbox for document retrieval                | Dev         | RAG testing, embedding, metadata validation |
| **prod_doc_search**      | Canonical production document retrieval       | Prod        | Live factual lookup                         |
| **dev_recommendations**  | Sandbox for similarity and suggestion testing | Dev         | Creative exploration                        |
| **prod_recommendations** | Production recommendation engine              | Prod        | Live content suggestion system              |
| **dev_asset_embeddings** | Sandbox for visual/audio embedding testing    | Dev         | Multi-modal search, metadata validation     |
| **prod_asset_embeddings** | Production semantic asset discovery          | Prod        | Visual similarity, cross-modal search       |
| **persona_revrebel**     | REVREBEL brand personality & tone embeddings  | Core        | Stylistic memory for GPTs                   |

**Note:** Zilliz collection names use underscores instead of hyphens.

<br>
---
<br>

## 1. `dev-doc-search`

**Purpose:**
Sandbox for document/text retrieval. Safe for experiments with chunking, metadata schemas, and embedding models.

**Use Cases:**

* Validate new embedding pipelines
* Test metadata filters (`domain`, `program`, `artifact_type`)
* Debug query recall and ranking

**Notes:**

* Can be wiped anytime
* Use to test index dimensions before production rollout

<br>
---
<br>

## 2. `prod-doc-search`

**Purpose:**
Production-grade RAG (Retrieve-Augment-Generate) index. The backbone for contextual GPT lookups.

**Use Cases:**

* Fetch SOPs, briefs, or guides
* Surface relevant policies in real-time workflows
* Feed canonical docs into GPT completions

**Data Examples:**
Finalized documentation, approved playbooks, style guides.

**Notes:**

* Must include `source_url`, `doc_id`, `version`
* Immutable except for version upgrades
* All content must be validated and approved

<br>
---
<br>

## 3. `dev-recommendations`

**Purpose:**
Experimentation zone for similarity-based retrieval (creative matching).

**Use Cases:**

* Related content testing
* Headline or concept similarity
* Recommendation model validation

**Notes:**

* Freeform experimental data
* Can test multimodal (image/text) embeddings

<br>
---
<br>

## 4. `prod-recommendations`

**Purpose:**
Live recommendation index powering production GPTs and automation logic.

**Use Cases:**

* Suggest related case studies, visuals, or CTAs
* "You might also like" results for websites and dashboards
* Cross-sell and upsell logic for hotels or campaigns

**Notes:**

* Only curated data allowed
* Maintain structured metadata (`domain`, `artifact_type`, `tags`)
* Version-lock embedding models between prod indexes

<br>
---
<br>

## 5. `dev-asset-embeddings`

**Purpose:**
Sandbox for testing multi-modal embeddings (visual, audio) before production deployment. Safe environment for validating embedding models, fusion strategies, and metadata schemas for brand assets.

**Use Cases:**

* Test visual embedding models (CLIP, ImageBind)
* Validate audio embedding pipelines (CLAP, Whisper)
* Experiment with embedding fusion strategies
* Debug visual similarity queries
* Test metadata extraction (color palettes, visual tags)

**Notes:**

* Can be wiped anytime
* Test different embedding dimensions (512, 768, 1024)
* Validate URL references to CDN assets
* Prototype cross-modal search patterns

<br>
---
<br>

## 6. `prod-asset-embeddings`

**Purpose:**
Production-grade multi-modal index enabling semantic discovery of visual and audio brand assets through lightweight vector embeddings while maintaining actual assets on CDN.

**Use Cases:**

* "Find logos with retro-tech aesthetic" (text → visual search)
* "Show photos similar to this reference image" (visual → visual search)
* "Find all assets matching our arcade-era clarity vibe" (cross-modal search)
* "Locate brand audio with upbeat energy" (text → audio search)
* Style transfer queries ("photos matching logo aesthetic")

**Data Examples:**
Logos, brand photography, graphics, video keyframes, audio signatures.

**Key Principle:**
**Store URLs, not binaries.** Assets remain on CDN (Cloudinary, Google Drive, R2). Only lightweight embeddings (~2KB) and metadata are stored in the vector database.

**Notes:**

* All assets must include `url` pointing to CDN location
* Embeddings generated from asset content (CLIP for images, CLAP for audio)
* Hybrid embeddings combine visual + text descriptions
* Requires `embedding_model` and `embedding_dimension` in metadata
* Only approved, on-brand assets allowed

<br>
---
<br>

## 7. `persona:revrebel`



**Purpose:** Personality & Voice Embeddings
This index is dedicated to storing REVREBEL's brand personality traits including **brand voice** , **tone examples** and **stylistic rules** like writing techniques. Enables modular injection of stylistic guidance into GPT responses — separate from factual knowledge or recommendation content.

**Use Cases:**

* Style-aware completions
* Personality preloading in multi-step prompt chains
* Tone tuning (e.g., witty vs authoritative)
* Experimental testing of tone or voice shifts (e.g., witty vs dry)
* Persona evolution tracking (`persona_version`)

<br>

**Collection role:**

- persona_revrebel holds canonical persona vectors and metadata.
- No propagation

<br>

Other collections (prod_doc_search, prod_asset_embeddings, prod_recommendations) query it at generation or retrieval time, this avoids duplication, ensures consistency, and allows semantic lookups.


<br>
---
<br>



## Production Insert & Query Examples

The snippets below show how to **write to** and **read from** the production collections using the Zilliz REST API.

**Authentication:** All requests require a Bearer token in the Authorization header.  
**Base URL:** `https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com`

<br>

### A) `prod-doc-search` (Production RAG)

**Zilliz REST API Insert:**

```bash
# INSERT a finalized doc chunk
curl --request POST \
  --url https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com/v2/vectordb/entities/insert \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer YOUR_ZILLIZ_API_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "collectionName": "prod_doc_search",
    "data": [{
      "primary_key": "doc-wf-auth-01",
      "vector": [0.123, 0.456, ...],
      "domain": "coding",
      "program": "webflow",
      "artifact_type": "howto",
      "created_ym": "2025-10",
      "source_url": "https://drive.google.com/...",
      "doc_id": "wf-auth-01",
      "version": "v3"
    }]
  }'
```

**Zilliz REST API Search:**

```bash
# SEARCH for similar docs
curl --request POST \
  --url https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com/v2/vectordb/entities/search \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer YOUR_ZILLIZ_API_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "collectionName": "prod_doc_search",
    "data": [[0.123, 0.456, ...]],
    "limit": 10,
    "filter": "domain == \"coding\" && program == \"webflow\" && artifact_type == \"howto\"",
    "outputFields": ["domain", "program", "artifact_type", "source_url", "doc_id"]
  }'
```

**Node.js Example:**

```ts
// INSERT a doc chunk
const ZILLIZ_ENDPOINT = 'https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com';
const ZILLIZ_API_KEY = process.env.ZILLIZ_API_KEY;

const insertResponse = await fetch(
  `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/insert`,
  {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      collectionName: 'prod_doc_search',
      data: [{
        primary_key: crypto.randomUUID(),
        vector: embedding384,
        domain: 'coding',
        program: 'webflow',
        artifact_type: 'howto',
        created_ym: '2025-10',
        source_url: 'https://drive.google.com/...',
        doc_id: 'wf-auth-01',
        version: 'v3'
      }]
    })
  }
);

// SEARCH for similar docs
const searchResponse = await fetch(
  `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/search`,
  {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      collectionName: 'prod_doc_search',
      data: [embedding384],
      limit: 10,
      filter: 'domain == "coding" && program == "webflow" && artifact_type == "howto"',
      outputFields: ['domain', 'program', 'artifact_type', 'source_url', 'doc_id']
    })
  }
);

const results = await searchResponse.json();
```

<br>

### B) `prod-recommendations` (Production rec engine)

**Zilliz REST API Insert:**

```bash
# INSERT an approved recommendation candidate
curl --request POST \
  --url https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com/v2/vectordb/entities/insert \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer YOUR_ZILLIZ_API_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "collectionName": "prod_recommendations",
    "data": [{
      "primary_key": "rec-hlx-042",
      "vector": [0.123, 0.456, ...],
      "domain": "design",
      "artifact_type": "headline",
      "created_ym": "2025-10",
      "tags": ["luxury", "retro-tech"],
      "source_url": "https://drive.google.com/...",
      "doc_id": "hlx-042",
      "version": "v1"
    }]
  }'
```

**Zilliz REST API Search:**

```bash
# SEARCH for similar recommendations
curl --request POST \
  --url https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com/v2/vectordb/entities/search \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer YOUR_ZILLIZ_API_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "collectionName": "prod_recommendations",
    "data": [[0.123, 0.456, ...]],
    "limit": 5,
    "filter": "domain == \"design\" && artifact_type == \"headline\"",
    "outputFields": ["domain", "artifact_type", "tags", "source_url", "doc_id"]
  }'
```

**Node.js Example:**

```ts
// INSERT a recommendation candidate
const insertResponse = await fetch(
  `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/insert`,
  {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      collectionName: 'prod_recommendations',
      data: [{
        primary_key: crypto.randomUUID(),
        vector: embedding384,
        domain: 'design',
        artifact_type: 'headline',
        created_ym: '2025-10',
        tags: ['luxury', 'retro-tech'],
        source_url: 'https://drive.google.com/...',
        doc_id: 'hlx-042',
        version: 'v1'
      }]
    })
  }
);

// SEARCH for similar recommendations
const searchResponse = await fetch(
  `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/search`,
  {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      collectionName: 'prod_recommendations',
      data: [embedding384],
      limit: 5,
      filter: 'domain == "design" && artifact_type == "headline"',
      outputFields: ['domain', 'artifact_type', 'tags', 'source_url', 'doc_id']
    })
  }
);

const results = await searchResponse.json();
```

<br>

### C) `persona_revrebel` (Personality & Voice Embeddings RAG)

**Zilliz REST API Insert:**

```bash
curl --request POST \
  --url https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com/v2/vectordb/entities/insert \
  --header 'Accept: application/json' \
  --header 'Authorization: Bearer YOUR_ZILLIZ_API_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "collectionName": "persona_revrebel",
    "data": [{
      "primary_key": "persona-voice-core-01",
      "vector": [0.123, 0.456, ...],
      "persona_id": "revrebel_core",
      "persona_version": "v1",
      "module": "Brand Voice & Style",
      "submodule": "Voice Principles",
      "status": "active",
      "tone": ["witty", "rebellious", "punchy"],
      "use_case": "style_scaffold",
      "created_ym": "2025-10",
      "author": "Gizmo",
      "language": "en"
    }]
  }'
```

<br>
---
<br>

### Best Practices

* Store **voice scaffolds**, **stylistic examples**, and **writing rules** here
* Keep separate from factual indexes (`prod-doc-search`)
* Version persona updates with `persona_version` (e.g., `v2`, `v2.1-beta`)
* Use prefix-based retrieval (e.g., `persona:revrebel`) for tone-consistent completions

<br>
---
<br>

## Multi-Modal Asset Embeddings: Deep Dive

### Architecture Philosophy

**Core Principle:** Separate asset storage from semantic search.

```
┌─────────────────────────────────────────────┐
│          CDN Storage Layer                  │
│  (Cloudinary, Google Drive, R2)             │
│  • Actual image/video/audio files           │
│  • Optimized delivery                       │
│  • 2MB - 50MB per asset                     │
└─────────────────────────────────────────────┘
                    ↑
                    │ URL Reference
                    │
┌─────────────────────────────────────────────┐
│     Vector Database (Vectorize)             │
│  • Lightweight embeddings (~2KB)            │
│  • Metadata + URL references                │
│  • Semantic search capability               │
└─────────────────────────────────────────────┘
```

### Asset Embedding Schema

```json
{
  "id": "asset-{uuid}",
  "values": [...],  // 512 or 768-dim embedding
  "text": "Primary REVREBEL logo - horizontal lockup with retro-tech aesthetic",
  "metadata": {
    // Asset Identity
    "asset_type": "logo",
    "variant": "primary",
    "asset_id": "logo-primary-001",
    
    // Storage Location (CRITICAL)
    "url": "https://res.cloudinary.com/revrebel/image/upload/v1761516148/RR/Logos/revrebel-logo.png",
    "cdn_provider": "cloudinary",
    "backup_url": "https://backup.cdn.example.com/...",
    
    // File Properties
    "format": "png",
    "dimensions": "2400x800",
    "file_size_kb": 145,
    "color_depth": "24bit",
    
    // Brand Context
    "domain": "visual-identity",
    "submodule": "logos",
    "use_case": "website-print-social",
    "channel": "web",
    
    // Visual Characteristics
    "visual_tags": ["geometric", "bold", "minimal", "retro-tech"],
    "color_palette": ["#163666", "#047C97", "#71C9C5"],
    "mood": "professional-rebellious",
    "style_era": "arcade",
    
    // Embedding Metadata
    "embedding_model": "clip-vit-b-32",
    "embedding_dimension": 512,
    "text_embedding_model": "text-embedding-3-small",
    "fusion_method": "weighted_average",
    "visual_weight": 0.7,
    "text_weight": 0.3,
    
    // Standard Fields
    "created_ym": "2025-10",
    "version": "v2.1",
    "author": "design-team",
    "approval_status": "approved",
    "status": "active",
    "last_updated": "2025-10-28"
  }
}
```

### Decision Tree: Should This Asset Be Embedded?

```
[Asset Upload/Registration]
        |
        +-- Is it a binary asset? (image/video/audio)
        |       |
        |       YES --> Continue to embedding decision
        |       NO  --> Route to text indexes (prod-doc-search)
        |
        +-- Will users search for it semantically?
        |       |
        |       YES --> "Find logos with retro vibe"
        |       |       "Photos similar to this mood"
        |       |       --> CREATE EMBEDDING
        |       |
        |       NO  --> "Get the primary logo" (exact match)
        |               "Download brand colors"
        |               --> METADATA ONLY (no embedding)
        |
        +-- Asset Type Routing:
                |
                +-- Logo/Graphic --> ✅ CLIP embedding
                +-- Photo/Image --> ✅ CLIP embedding
                +-- Video --> ✅ CLIP (keyframes) or VideoMAE
                +-- Audio --> ✅ CLAP or Whisper embedding
                +-- Font File --> ⚠️  Text description embedding only
                +-- Color Palette --> ❌ Exact match, no embedding
                +-- PDF/Doc --> Route to prod-doc-search
```

### Asset Type Decision Matrix

| Asset Type | Store URL? | Store Embedding? | Searchable By | Primary Use Case |
|------------|------------|------------------|---------------|------------------|
| **Logo** | ✅ Always | ✅ Yes (CLIP) | Text, Visual | "Find retro logos" |
| **Photo** | ✅ Always | ✅ Yes (CLIP) | Text, Visual | "Similar mood photos" |
| **Brand Colors** | ✅ Always | ❌ No | Exact match | "Get primary color" |
| **Font Files** | ✅ Always | ⚠️ Text only | Text description | "Find bold fonts" |
| **Video** | ✅ Always | ✅ Yes (CLIP keyframes) | Text, Visual | "Videos with energy" |
| **Audio** | ✅ Always | ✅ Yes (CLAP) | Text, Audio | "Upbeat brand audio" |
| **Documents** | ✅ Always | ✅ Yes (Text) | Text content | Use prod-doc-search |
| **Graphics** | ✅ Always | ✅ Yes (CLIP) | Text, Visual | "Icon style match" |

### Embedding Models by Asset Type

**Visual Assets:**

| Asset Type | Recommended Model | Dimension | Use Case |
|------------|-------------------|-----------|----------|
| **Logo** | CLIP ViT-B/32 | 512 | Style matching, semantic search |
| **Photo** | CLIP ViT-L/14 | 768 | High-quality visual similarity |
| **Graphic** | CLIP ViT-B/32 | 512 | Design element search |
| **Video** | CLIP (keyframes) | 512 | Scene/mood matching |

**Audio Assets:**

| Asset Type | Recommended Model | Dimension | Use Case |
|------------|-------------------|-----------|----------|
| **Brand Audio** | CLAP | 512 | Sonic branding similarity |
| **Voice** | Whisper (encoder) | 768 | Voice tone matching |

**Hybrid (Text + Visual):**

| Combination | Fusion Method | Visual Weight | Text Weight |
|-------------|---------------|---------------|-------------|
| **Logo + Description** | Weighted Average | 70% | 30% |
| **Photo + Caption** | Weighted Average | 80% | 20% |
| **Video + Transcript** | Concatenation | 60% | 40% |

### Storage Impact Analysis

```
Per Asset Storage:
- Embedding vector (512 dims): ~2KB
- Metadata JSON: ~1KB
- Total per asset: ~3KB

Scale Comparison:
1,000 assets:
  - Actual images: 1,000 × 2MB = 2GB
  - Embeddings only: 1,000 × 3KB = 3MB (667x smaller)

10,000 assets:
  - Actual images: 10,000 × 2MB = 20GB  
  - Embeddings only: 10,000 × 3KB = 30MB (667x smaller)
```

### Query Performance Benchmarks

Expected latency for Cloudflare Vectorize:

- **Text → Asset search:** 50-100ms
- **Asset → Asset search:** 30-80ms  
- **Hybrid search:** 100-150ms
- **Cross-modal batch:** 150-250ms

### Asset Ingestion Workflow

```typescript
// Asset ingestion pipeline for Zilliz
interface AssetUpload {
  file: File | URL;
  type: 'logo' | 'photo' | 'graphic' | 'video' | 'audio';
  description: string;
  metadata: Record<string, any>;
}

const ZILLIZ_ENDPOINT = 'https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com';
const ZILLIZ_API_KEY = process.env.ZILLIZ_API_KEY;

async function ingestAsset(
  upload: AssetUpload
): Promise<string> {
  
  // STEP 1: Upload to CDN
  const cdnUrl = await uploadToCDN(upload.file, {
    folder: `RR/${upload.type}s`,
    transformation: upload.type === 'logo' 
      ? { format: 'auto', quality: 'auto' }
      : undefined
  });
  
  // STEP 2: Generate visual embedding
  let visualEmbedding: number[] | null = null;
  
  if (['logo', 'photo', 'graphic', 'video'].includes(upload.type)) {
    visualEmbedding = await generateVisualEmbedding(upload.file, {
      model: 'clip-vit-b-32',
      dimension: 512
    });
  } else if (upload.type === 'audio') {
    visualEmbedding = await generateAudioEmbedding(upload.file, {
      model: 'clap-large',
      dimension: 512
    });
  }
  
  // STEP 3: Generate text embedding from description
  const textEmbedding = await generateTextEmbedding(upload.description, {
    model: 'text-embedding-3-small',
    dimension: 512
  });
  
  // STEP 4: Fuse embeddings (if visual exists)
  const finalEmbedding = visualEmbedding 
    ? fuseEmbeddings(visualEmbedding, textEmbedding, {
        visualWeight: 0.7,
        textWeight: 0.3,
        method: 'weighted_average'
      })
    : textEmbedding;
  
  // STEP 5: Extract visual characteristics
  const visualTags = await extractVisualTags(upload.file);
  const colorPalette = await extractDominantColors(upload.file, { count: 5 });
  
  // STEP 6: Insert into Zilliz vector database
  const assetId = `asset-${crypto.randomUUID()}`;
  
  const response = await fetch(
    `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/insert`,
    {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        collectionName: 'prod_asset_embeddings',
        data: [{
          primary_key: assetId,
          vector: finalEmbedding,
          asset_type: upload.type,
          url: cdnUrl,
          cdn_provider: 'cloudinary',
          format: upload.type === 'logo' ? 'png' : 'jpg',
          visual_tags: visualTags,
          color_palette: colorPalette,
          created_ym: new Date().toISOString().slice(0, 7),
          status: 'active',
          ...upload.metadata
        }]
      })
    }
  );
  
  if (!response.ok) {
    throw new Error(`Failed to insert asset: ${await response.text()}`);
  }
  
  return assetId;
}
```

### Query Pattern Examples

**1. Text → Asset Search**
```typescript
// "Find logos with retro-tech aesthetic"
async function textToAssetSearch(
  query: string,
  assetType: string
) {
  const queryEmbedding = await generateTextEmbedding(query);
  
  const response = await fetch(
    `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/search`,
    {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        collectionName: 'prod_asset_embeddings',
        data: [queryEmbedding],
        limit: 10,
        filter: `asset_type == "${assetType}" && status == "active"`,
        outputFields: ['url', 'visual_tags', 'color_palette']
      })
    }
  );
  
  const results = await response.json();
  
  return results.data.map(item => ({
    assetId: item.id,
    url: item.url,
    similarity: item.distance,
    visualTags: item.visual_tags
  }));
}
```

**2. Visual Similarity Search**
```typescript
// "Find photos similar to this reference image"
async function visualSimilaritySearch(
  referenceAssetId: string,
  targetAssetType: string
) {
  // First, get the reference asset's vector
  const getResponse = await fetch(
    `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/get`,
    {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        collectionName: 'prod_asset_embeddings',
        id: [referenceAssetId],
        outputFields: ['vector']
      })
    }
  );
  
  const refData = await getResponse.json();
  if (!refData.data || refData.data.length === 0) {
    throw new Error('Reference asset not found');
  }
  
  const refVector = refData.data[0].vector;
  
  // Now search for similar assets
  const searchResponse = await fetch(
    `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/search`,
    {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        collectionName: 'prod_asset_embeddings',
        data: [refVector],
        limit: 11,  // +1 to account for self-match
        filter: `asset_type == "${targetAssetType}" && status == "active"`,
        outputFields: ['url', 'visual_tags']
      })
    }
  );
  
  const results = await searchResponse.json();
  
  return results.data
    .filter(item => item.id !== referenceAssetId)
    .map(item => ({
      assetId: item.id,
      url: item.url,
      similarity: item.distance,
      visualTags: item.visual_tags
    }));
}
```

**3. Cross-Modal Discovery**
```typescript
// "Find all assets matching arcade-era clarity vibe"
async function crossModalSearch(
  vibe: string,
  env: Env
) {
  const vibeEmbedding = await generateTextEmbedding(vibe);
  
  const results = await env.PROD_ASSETS.query(vibeEmbedding, {
    topK: 20,
    filter: {
      status: 'active',
      asset_type: ['logo', 'photo', 'graphic', 'video']
    }
  });
  
  // Group by asset type
  const grouped = results.matches.reduce((acc, match) => {
    const type = match.metadata.asset_type;
    if (!acc[type]) acc[type] = [];
    acc[type].push({
      url: match.metadata.url,
      description: match.metadata.description,
      similarity: match.score
    });
    return acc;
  }, {} as Record<string, any[]>);
  
  return grouped;
}
```

**4. Style Transfer Query**
```typescript
// "Find photos that match our primary logo's aesthetic"
async function styleTransferQuery(
  sourceAssetId: string,
  targetAssetType: string,
  env: Env
) {
  const sourceVector = await env.PROD_ASSETS.getByIds([sourceAssetId]);
  
  const results = await env.PROD_ASSETS.query(sourceVector[0].values, {
    topK: 15,
    filter: {
      asset_type: targetAssetType,
      status: 'active'
    }
  });
  
  return results.matches.map(m => ({
    url: m.metadata.url,
    visualTags: m.metadata.visual_tags,
    colorPalette: m.metadata.color_palette,
    styleSimilarity: m.score
  }));
}
```

### Integration with Existing Indexes

**Hybrid Knowledge + Asset Query:**
```typescript
// Combined documentation + visual asset retrieval
async function hybridKnowledgeAssetQuery(
  userQuery: string,
  env: Env
) {
  const embedding = await generateTextEmbedding(userQuery);
  
  // Query 1: Knowledge base
  const knowledge = await env.PROD_DOCS.query(embedding, {
    topK: 5,
    filter: { domain: 'visual-identity' }
  });
  
  // Query 2: Visual assets
  const assets = await env.PROD_ASSETS.query(embedding, {
    topK: 5,
    filter: { asset_type: 'logo' }
  });
  
  // Query 3: Persona voice
  const voice = await env.PERSONA_REVREBEL.query(embedding, {
    topK: 2,
    filter: { submodule: 'Visual Identity' }
  });
  
  return {
    documentation: knowledge.matches,
    relatedAssets: assets.matches,
    voiceGuidance: voice.matches
  };
}
```

### Implementation Phases

**Phase 1: Pilot (Logos Only)**
- Embed existing logo library from Cloudinary
- Validate search quality and performance
- Establish baseline metadata schema
- Test text → logo and logo → logo queries

**Phase 2: Expansion (Photos & Graphics)**
- Extend to brand photography
- Add graphic elements and icons
- Implement cross-modal search
- Validate style transfer queries

**Phase 3: Advanced (Video & Audio)**
- Add video keyframe embeddings
- Implement audio signature search
- Build multi-modal recommendation engine
- Enable complex fusion strategies

### Validation Checklist

```typescript
// Asset embedding health checks for Zilliz
async function validateAssetEmbeddings() {
  // Check 1: Get collection description
  const descResponse = await fetch(
    `${ZILLIZ_ENDPOINT}/v2/vectordb/collections/describe`,
    {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        collectionName: 'prod_asset_embeddings'
      })
    }
  );
  
  const collectionInfo = await descResponse.json();
  console.log('Collection info:', collectionInfo);
  
  // Check 2: Query sample to verify structure
  const sampleResponse = await fetch(
    `${ZILLIZ_ENDPOINT}/v2/vectordb/entities/query`,
    {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Authorization': `Bearer ${ZILLIZ_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        collectionName: 'prod_asset_embeddings',
        filter: 'status == "active"',
        limit: 10,
        outputFields: ['url', 'asset_type', 'cdn_provider']
      })
    }
  );
  
  const samples = await sampleResponse.json();
  
  // Check 3: Validate URLs exist
  const assetsWithoutUrl = samples.data.filter(item => !item.url);
  console.log(`Assets without URL: ${assetsWithoutUrl.length}`);
  
  // Check 4: Test CDN links (sample)
  for (const asset of samples.data.slice(0, 5)) {
    try {
      const response = await fetch(asset.url, { method: 'HEAD' });
      if (!response.ok) {
        console.error(`Broken URL: ${asset.url}`);
      }
    } catch (error) {
      console.error(`Error checking URL ${asset.url}:`, error);
    }
  }
  
  return samples;
}
```

### Code Utilities

**Embedding Fusion:**
```typescript
function fuseEmbeddings(
  visualEmb: number[],
  textEmb: number[],
  opts: { visualWeight: number; textWeight: number; method: string }
): number[] {
  if (opts.method === 'weighted_average') {
    return visualEmb.map((v, i) => 
      v * opts.visualWeight + textEmb[i] * opts.textWeight
    );
  }
  
  if (opts.method === 'concat') {
    return [...visualEmb, ...textEmb];
  }
  
  throw new Error(`Unknown fusion method: ${opts.method}`);
}
```

**Visual Tag Extraction:**
```typescript
async function extractVisualTags(imageUrl: string): Promise<string[]> {
  // Use GPT-4V for image analysis
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${OPENAI_KEY}` },
    body: JSON.stringify({
      model: 'gpt-4-vision-preview',
      messages: [{
        role: 'user',
        content: [
          { 
            type: 'text', 
            text: 'List 5 visual style tags for this image (e.g., geometric, bold, minimal).' 
          },
          { type: 'image_url', image_url: { url: imageUrl } }
        ]
      }],
      max_tokens: 100
    })
  });
  
  const tags = parseTagsFromResponse(await response.json());
  return tags;
}
```

**Color Palette Extraction:**
```typescript
async function extractDominantColors(
  imageUrl: string, 
  opts: { count: number }
): Promise<string[]> {
  // Use color quantization algorithm or service
  const image = await loadImage(imageUrl);
  const colors = await analyzeColorPalette(image, opts.count);
  return colors.map(c => rgbToHex(c));
}
```

<br>
---
<br>

## Index Validation & Health Checks

Regular validation ensures your dev and prod indexes remain performant, consistent, and aligned with the latest schema.

### Routine Checks

| Validation Task                | Description                                                                       | Frequency   |
| ------------------------------ | --------------------------------------------------------------------------------- | ----------- |
| **Vector Count Check**         | Confirm expected # of entries per index (`wrangler vectorize stats`)              | Weekly      |
| **Metadata Schema Validation** | Verify required fields exist (`domain`, `program`, `artifact_type`, `created_ym`) | Weekly      |
| **Embedding Dimensionality**   | Ensure vector dimensions match model output (e.g., 384 or 768)                    | Each deploy |
| **Drift Detection**            | Compare mean cosine similarity between dev and prod samples                       | Monthly     |
| **Duplicate Detection**        | Hash text and confirm no duplicates per `doc_id`                                  | Monthly     |
| **Embedding Latency Test**     | Measure average query latency (`vectorize query --benchmark`)                     | Monthly     |
| **Namespace Scan**             | Confirm each namespace exists and is accessible                                   | Quarterly   |
| **Backup Sync**                | Export embeddings to GCS or R2 for restore safety                                 | Quarterly   |

<br>
---
<br>

### Automated Validation Script (Example)

```bash
#!/bin/bash
# validate_collections.sh

collections=(dev_doc_search prod_doc_search dev_recommendations prod_recommendations dev_asset_embeddings prod_asset_embeddings persona_revrebel)

ZILLIZ_ENDPOINT="https://in03-9effbe09d86d9b7.serverless.gcp-us-west1.cloud.zilliz.com"
ZILLIZ_API_KEY="YOUR_ZILLIZ_API_TOKEN"

for collection in "${collections[@]}"; do
  echo "Validating $collection..."
  # Get collection stats
  curl --request GET \
    --url "${ZILLIZ_ENDPOINT}/v2/vectordb/collections/describe" \
    --header "Authorization: Bearer ${ZILLIZ_API_KEY}" \
    --header "Content-Type: application/json" \
    --data "{\"collectionName\":\"${collection}\"}"
  
  echo ""
done
```

<br>
---
<br>

### Health Metrics to Monitor

* **`avg_query_latency`** < 100ms (for prod)
* **`avg_cosine_similarity`** drift < ±0.03 between prod/dev
* **`vector_count_delta`** < 2% between weeks (unless reindexing)
* **`collection_size_growth`** < 15% month-over-month
* **`entity_count`** matches expected document ingestion rate

<br>
---
<br>

## Metadata Schema (Asset Fields)

| Field                 | Type   | Description                                                    |
| --------------------- | ------ | -------------------------------------------------------------- |
| `asset_type`          | string | Type of asset (logo, photo, graphic, video, audio)             |
| `variant`             | string | Asset variant (primary, secondary, icon, stacked, circle)      |
| `asset_id`            | string | Unique asset identifier                                        |
| `url`                 | string | **REQUIRED** - CDN URL to actual asset                         |
| `cdn_provider`        | string | CDN service (cloudinary, google-drive, r2, github)             |
| `backup_url`          | string | Secondary storage URL for redundancy                           |
| `format`              | string | File format (png, svg, jpg, ttf, mp4, wav, pdf)                |
| `dimensions`          | string | Asset dimensions (e.g., "2400x800")                            |
| `file_size_kb`        | number | File size in kilobytes                                         |
| `duration_sec`        | number | Duration for audio/video assets                                |
| `color_depth`         | string | Color depth (e.g., "24bit")                                    |
| `visual_tags`         | array  | Visual characteristics (e.g., ["geometric", "bold"])           |
| `color_palette`       | array  | Dominant colors (e.g., ["#163666", "#047C97"])                 |
| `mood`                | string | Emotional tone (e.g., "professional-rebellious")               |
| `style_era`           | string | Design era (e.g., "arcade", "retro-future")                    |
| `embedding_model`     | string | Model used for visual/audio embedding                          |
| `embedding_dimension` | number | Vector dimension (512, 768, 1024)                              |
| `text_embedding_model`| string | Model used for text embedding                                  |
| `fusion_method`       | string | Embedding combination method (weighted_average, concat)        |
| `visual_weight`       | number | Weight for visual embedding in fusion (0.0-1.0)                |
| `text_weight`         | number | Weight for text embedding in fusion (0.0-1.0)                  |
| `approval_status`     | string | Approval state (approved, review, draft)                       |

<br>
---
<br>

## Metadata Schema (Universal Fields)

| Field             | Type   | Description                                               |
| ----------------- | ------ | --------------------------------------------------------- |
| `domain`          | string | Primary content category (e.g., coding, design, strategy) |
| `program`         | string | System or app area (e.g., webflow, budibase, n8n)         |
| `artifact_type`   | string | Content type (e.g., howto, checklist, prompt)             |
| `created_ym`      | string | Creation month (`YYYY-MM`)                                |
| `source_url`      | string | Document origin (Drive, GitHub, etc.)                     |
| `doc_id`          | string | Unique document identifier                                |
| `version`         | string | Version of the document                                   |
| `property_code`   | string | Hotel or client identifier (e.g., DENPOP, SEAPOP)         |
| `data_kind`       | string | Defines the nature of the stored data helping segment content by data modality. <br> (e.g., `document`, `summary`, `image`, `metric`). |
| `lifecycle`       | string | Indicates the maturity or stage of the data, useful for retention rules and surfacing current material. <br> (`draft`, `review`, `published`, `archived`).  |
| `sensitivity`     | string | Flags data privacy/security classification and enables compliance filtering. <br> (`public`, `internal`, `confidential`, `restricted`). |
| `program_lang`    | string | Specifies the programming or query language relevant to the entry. This assists in filtering technical snippets. <br> (`js`, `ts`, `sql`, `python`, `none`). |
| `system_area`     | string | Defines the technical or business subsystem providing context for routing and categorization.  <br> (`webflow`, `budibase`, `n8n`, `cms`, `analytics`).  |
| `created_ym`      | string | Creation month in `yyyy-MM` format |                                    
| `last_updated `      | string | Last date updated in `yyyy-MM-dd` format |                                        

<br>
---
<br>

## Metadata Schema (Persona Fields)

<br>

See **REVREBEL_PersonaIndexReference.md** Document

<br>
---
<br>

## Summary

| Category        | Index                    | Focus                        | Environment |
| --------------- | ------------------------ | ---------------------------- | ----------- |
| Documents       | dev-doc-search           | Test/document retrieval      | Dev         |
| Documents       | prod-doc-search          | Live/document retrieval      | Prod        |
| Recommendations | dev-recommendations      | Experimental rec engine      | Dev         |
| Recommendations | prod-recommendations     | Live rec engine              | Prod        |
| Assets          | dev-asset-embeddings     | Test/visual-audio embeddings | Dev         |
| Assets          | prod-asset-embeddings    | Live/asset discovery         | Prod        |
| Persona         | persona:revrebel         | Voice and tone embeddings    | Core        |

<br>

### Why This Architecture Works

* Keeps experimental (dev) data isolated from live production
* Separates factual retrieval from creative similarity
* Maintains modular persona memory for consistent brand tone
* **Separates asset storage (CDN) from semantic search (vectors)**
* **Enables multi-modal discovery without storing binaries**
* Simplifies versioning and schema validation
* Enables automated health monitoring and regression detection

### Total System Footprint

```
Expected Vector Database Size:
- Documents (prod-doc-search): ~50MB (10K docs)
- Recommendations (prod-recommendations): ~20MB (5K items)
- Assets (prod-asset-embeddings): ~30MB (10K assets)
- Persona (persona:revrebel): ~5MB (1K voice records)

Total: ~105MB for 26K+ vectors

Compared to storing assets directly:
- 10K assets at 2MB average: 20GB
- Vector approach is 190x more efficient
```

<br>
---
<br>

## Zilliz API Endpoints Reference
<br>

### Insert Entities (Single or Batch)
```bash
POST /v2/vectordb/entities/insert
Authorization: Bearer YOUR_ZILLIZ_API_KEY
Content-Type: application/json

{
  "collectionName": "prod_doc_search",
  "data": [
    {
      "primary_key": "doc-001",
      "vector": [0.123, 0.456, ...],
      "domain": "coding",
      "program": "webflow"
    }
  ]
}
```

<br>

### Search Entities
```bash
POST /v2/vectordb/entities/search
Authorization: Bearer YOUR_ZILLIZ_API_KEY
Content-Type: application/json

{
  "collectionName": "prod_doc_search",
  "data": [[0.123, 0.456, ...]],
  "limit": 10,
  "filter": "domain == \"coding\" && program == \"webflow\"",
  "outputFields": ["domain", "program", "artifact_type"]
}
```

<br>

### Query Entities (Filter-based)
```bash
POST /v2/vectordb/entities/query
Authorization: Bearer YOUR_ZILLIZ_API_KEY
Content-Type: application/json

{
  "collectionName": "prod_doc_search",
  "filter": "domain == \"coding\"",
  "limit": 100,
  "outputFields": ["primary_key", "domain", "program"]
}
```

<br>

### Get Entities by ID
```bash
POST /v2/vectordb/entities/get
Authorization: Bearer YOUR_ZILLIZ_API_KEY
Content-Type: application/json

{
  "collectionName": "prod_doc_search",
  "id": ["doc-001", "doc-002"],
  "outputFields": ["vector", "domain", "program"]
}
```

<br>

### Describe Collection
```bash
POST /v2/vectordb/collections/describe
Authorization: Bearer YOUR_ZILLIZ_API_KEY
Content-Type: application/json

{
  "collectionName": "prod_doc_search"
}
```
<br>

**Last Updated:** 2025-10-28
**Maintained by:** REVREBEL Data Engineering
**File Name:** `revrebel_vectorize_reference.md`
**Version:** 2.0 (Added multi-modal asset embeddings)
