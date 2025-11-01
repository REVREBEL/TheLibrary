---
title: Vector Database: Retrieval & Generation Guide
description: This guide teaches you how to **query and retrieve** from the REVREBEL brand vector database to generate consistently on-brand content.
published: true
date: 2025-11-01T23:43:17.128Z
tags: 
editor: markdown
dateCreated: 2025-11-01T23:32:18.561Z
---

# REVREBEL Brand Vector Database: Retrieval & Generation Guide

## Executive Overview

This guide teaches you how to **query and retrieve** from the REVREBEL brand vector database to generate consistently on-brand content. Whether you're building marketing automation, content generators, or AI assistants, this documentation provides practical examples across multiple platforms and languages.

---

## Table of Contents

1. [Core Retrieval Concepts](#core-retrieval-concepts)
2. [Query Strategy Framework](#query-strategy-framework)
3. [Multi-Language SDK Examples](#multi-language-sdk-examples)
4. [Use Case Patterns](#use-case-patterns)
5. [Brand Synthesis Workflows](#brand-synthesis-workflows)
6. [Quality Assurance & Validation](#quality-assurance--validation)
7. [Advanced Techniques](#advanced-techniques)
8. [Performance Optimization](#performance-optimization)
9. [Error Handling](#error-handling)
10. [Complete End-to-End Example](#complete-end-to-end-example)

---

## Core Retrieval Concepts

### Understanding the REVREBEL Collections

The brand database consists of four primary collections:

| Collection | Purpose | Embedding Dimensions | Primary Use |
|------------|---------|---------------------|-------------|
| `persona_revrebel` | Brand voice, tone, personality traits | 1536 (text) | Copy generation, voice consistency |
| `prod_asset_embeddings` | Visual assets (logos, images, design elements) | 512 (CLIP) | Visual consistency, asset selection |
| `prod_doc_search` | Brand guidelines, examples, discoveries | 1536 (text) | Reference retrieval, context building |
| `prod_recommendations` | Best practices, do's/don'ts, strategic guidance | 1536 (text) | Decision support, strategy validation |

### Retrieval Principles

**1. Semantic Search First**
- Use `search_vectors` for meaning-based retrieval
- Queries understand intent, not just keywords
- Example: "rebellious but professional tone" finds voice guidelines even without exact matches

**2. Metadata Filtering Second**
- Use `query_vectors` for categorical browsing
- Filter by domain, artifact_type, status, timestamps
- Example: Find all approved email templates from Q4 2025

**3. Hybrid When Appropriate**
- Combine semantic similarity with metadata filters
- `search_vectors` with `filter_expr` parameter
- Example: "Email subject lines" (semantic) + `artifact_type == 'email_template'` (filter)

---

## Query Strategy Framework

### Strategy 1: Voice & Tone Retrieval

**When to use:** Generating copy, writing content, maintaining brand voice

```
Collection: persona_revrebel
Query Type: Semantic search
Embedding: Text-based (1536-dim)
```

**Example Query Pattern:**
```python
query = "How should REVREBEL sound when addressing hotel executives in email?"
# Embeds to vector, searches persona_revrebel
# Returns: Voice guidelines, tone examples, linguistic patterns
```

### Strategy 2: Visual Asset Discovery

**When to use:** Selecting images, finding logos, visual consistency checks

```
Collection: prod_asset_embeddings
Query Type: Semantic search (CLIP embeddings)
Embedding: Visual-based (512-dim)
```

**Example Query Pattern:**
```python
query_image = <uploaded_design_mockup>
# CLIP embeds image to 512-dim vector
# Returns: Similar approved brand assets, style-matched visuals
```

### Strategy 3: Example Retrieval

**When to use:** Finding reference content, studying patterns, learning conventions

```
Collection: prod_doc_search
Query Type: Hybrid (semantic + filters)
Embedding: Text-based (1536-dim)
```

**Example Query Pattern:**
```python
semantic_query = "subject lines for contract confirmations"
filter_expr = "artifact_type == 'email_template' && status == 'approved'"
# Returns: Real REVREBEL email examples with that function
```

### Strategy 4: Strategic Guidance

**When to use:** Decision-making, validating approaches, getting recommendations

```
Collection: prod_recommendations
Query Type: Semantic search
Embedding: Text-based (1536-dim)
```

**Example Query Pattern:**
```python
query = "Should we use gaming metaphors for enterprise clients?"
# Returns: Strategic recommendations, do's/don'ts, context-specific guidance
```

---

## Multi-Language SDK Examples

### Python (Primary - Full Feature Set)

#### Installation
```bash
pip install pymilvus openai pillow
```

#### Complete Retrieval Example

```python
from pymilvus import MilvusClient
import openai
from typing import List, Dict, Any
import os

class REVREBELBrandRetriever:
    """
    Production-ready REVREBEL brand knowledge retrieval system.
    Handles text and image queries across all brand collections.
    """
    
    def __init__(self, 
                 zilliz_uri: str = os.getenv('ZILLIZ_CLOUD_URI'),
                 zilliz_token: str = os.getenv('ZILLIZ_CLOUD_TOKEN'),
                 openai_api_key: str = os.getenv('OPENAI_API_KEY')):
        """Initialize connections to Zilliz and OpenAI"""
        self.client = MilvusClient(uri=zilliz_uri, token=zilliz_token)
        openai.api_key = openai_api_key
        
    def embed_text_query(self, query: str) -> List[float]:
        """Convert text query to 1536-dim embedding"""
        response = openai.embeddings.create(
            model="text-embedding-3-small",
            input=query
        )
        return response.data[0].embedding
    
    def search_brand_voice(self, 
                          query: str, 
                          limit: int = 5,
                          filters: str = None) -> List[Dict]:
        """
        Search brand voice and personality guidelines.
        
        Args:
            query: Natural language description of what you need
            limit: Number of results (default: 5)
            filters: Optional metadata filters
            
        Returns:
            List of voice guidelines with similarity scores
            
        Example:
            results = retriever.search_brand_voice(
                "How to write email subject lines that feel energetic but professional"
            )
        """
        query_vector = self.embed_text_query(query)
        
        search_params = {
            "collection_name": "persona_revrebel",
            "data": [query_vector],
            "limit": limit,
            "output_fields": ["*"]
        }
        
        if filters:
            search_params["filter"] = filters
            
        results = self.client.search(**search_params)
        return self._format_results(results[0])
    
    def search_examples(self,
                       query: str,
                       artifact_type: str = None,
                       limit: int = 5) -> List[Dict]:
        """
        Find real REVREBEL content examples.
        
        Args:
            query: What kind of example you're looking for
            artifact_type: Filter by type (email_template, landing_page, etc.)
            limit: Number of examples to return
            
        Returns:
            List of example content with context
            
        Example:
            examples = retriever.search_examples(
                "partnership confirmation emails",
                artifact_type="email_template"
            )
        """
        query_vector = self.embed_text_query(query)
        
        filter_expr = None
        if artifact_type:
            filter_expr = f"artifact_type == '{artifact_type}'"
        
        search_params = {
            "collection_name": "prod_doc_search",
            "data": [query_vector],
            "limit": limit,
            "output_fields": ["*"],
        }
        
        if filter_expr:
            search_params["filter"] = filter_expr
            
        results = self.client.search(**search_params)
        return self._format_results(results[0])
    
    def get_recommendations(self,
                           decision_context: str,
                           limit: int = 3) -> List[Dict]:
        """
        Get strategic recommendations for brand decisions.
        
        Args:
            decision_context: Describe the decision you're making
            limit: Number of recommendations
            
        Returns:
            Strategic guidance from brand knowledge base
            
        Example:
            recs = retriever.get_recommendations(
                "Should we use technical jargon when writing for hotel GMs?"
            )
        """
        query_vector = self.embed_text_query(decision_context)
        
        results = self.client.search(
            collection_name="prod_recommendations",
            data=[query_vector],
            limit=limit,
            output_fields=["*"]
        )
        
        return self._format_results(results[0])
    
    def browse_by_metadata(self,
                          collection: str,
                          filter_expr: str,
                          limit: int = 10) -> List[Dict]:
        """
        Browse content by metadata without semantic search.
        
        Args:
            collection: Which collection to browse
            filter_expr: Metadata filter expression
            limit: Max results
            
        Returns:
            Filtered entities
            
        Example:
            # Get all email templates from October 2025
            emails = retriever.browse_by_metadata(
                "prod_doc_search",
                "artifact_type == 'email_template' && created_ym == '2025-10'",
                limit=20
            )
        """
        results = self.client.query(
            collection_name=collection,
            filter=filter_expr,
            limit=limit,
            output_fields=["*"]
        )
        
        return results
    
    def _format_results(self, raw_results: List) -> List[Dict]:
        """Clean and format search results for readability"""
        formatted = []
        for hit in raw_results:
            formatted.append({
                'id': hit.get('id'),
                'similarity_score': hit.get('distance'),  # Cosine similarity
                'content': hit.get('entity', {}),
                'metadata': {
                    k: v for k, v in hit.get('entity', {}).items() 
                    if k not in ['vector', 'primary_key']
                }
            })
        return formatted


# ============================================
# USAGE EXAMPLES
# ============================================

if __name__ == "__main__":
    retriever = REVREBELBrandRetriever()
    
    # Example 1: Generate email subject line
    print("=" * 60)
    print("USE CASE: Generate contract confirmation email subject")
    print("=" * 60)
    
    voice_results = retriever.search_brand_voice(
        "energetic, confident email subject lines for signed contracts"
    )
    
    examples = retriever.search_examples(
        "signed agreement confirmation emails",
        artifact_type="email_template"
    )
    
    print("\n📝 Brand Voice Guidelines:")
    for result in voice_results[:2]:
        print(f"  Similarity: {result['similarity_score']:.3f}")
        print(f"  Guideline: {result['metadata'].get('content_preview', 'N/A')[:100]}...")
        print()
    
    print("\n📧 Example Subject Lines from Brand:")
    for example in examples[:3]:
        print(f"  • {example['metadata'].get('subject_line', 'N/A')}")
    
    # Example 2: Check if approach aligns with brand
    print("\n" + "=" * 60)
    print("USE CASE: Validate approach before creating content")
    print("=" * 60)
    
    recommendations = retriever.get_recommendations(
        "Is it on-brand to use 'Player Two Has Entered the Game' for B2B emails?"
    )
    
    print("\n💡 Strategic Guidance:")
    for rec in recommendations:
        print(f"  Relevance: {rec['similarity_score']:.3f}")
        print(f"  Guidance: {rec['metadata'].get('recommendation', 'N/A')[:200]}...")
        print()
    
    # Example 3: Browse all email templates
    print("\n" + "=" * 60)
    print("USE CASE: Browse all email templates in database")
    print("=" * 60)
    
    all_emails = retriever.browse_by_metadata(
        "prod_doc_search",
        "artifact_type == 'email_template'",
        limit=10
    )
    
    print(f"\n📬 Found {len(all_emails)} email templates:")
    for email in all_emails[:5]:
        print(f"  • {email.get('title', 'Untitled')} (ID: {email.get('primary_key')})")
```

#### Output Example
```
============================================================
USE CASE: Generate contract confirmation email subject
============================================================

📝 Brand Voice Guidelines:
  Similarity: 0.892
  Guideline: REVREBEL uses energetic, confident language with subtle gaming/tech references. Subject lines sho...
  
  Similarity: 0.867
  Guideline: For business communications, maintain professionalism while injecting personality. Use active voi...

📧 Example Subject Lines from Brand:
  • RACERS, REV YOUR ENGINES — WE'VE GOT A NEW HOTEL TO OPTIMIZE!
  • Player Two Has Entered the Game
  • Access Granted

============================================================
USE CASE: Validate approach before creating content
============================================================

💡 Strategic Guidance:
  Relevance: 0.901
  Guidance: Gaming metaphors are acceptable for B2B when balanced with professionalism. 'Player Two' reference works because it's immediately understandable and conveys partnership without being too casual. Best...

============================================================
USE CASE: Browse all email templates in database
============================================================

📬 Found 8 email templates:
  • Signed Agreement Confirmation (ID: email_contract_confirm_001)
  • Meeting Cancellation (ID: email_meeting_cancel_001)
  • Meeting Rescheduled (ID: email_meeting_reschedule_001)
  • Welcome Onboarding (ID: email_welcome_001)
  • Quarterly Review Reminder (ID: email_qtr_review_001)
```

---

### JavaScript/TypeScript (Node.js)

#### Installation
```bash
npm install @zilliz/milvus2-sdk-node openai
```

#### Complete Retrieval Example

```typescript
import { MilvusClient } from '@zilliz/milvus2-sdk-node';
import OpenAI from 'openai';

interface SearchResult {
  id: string;
  similarity: number;
  metadata: Record<string, any>;
}

class REVREBELRetriever {
  private milvusClient: MilvusClient;
  private openai: OpenAI;

  constructor(
    zillizUri: string = process.env.ZILLIZ_CLOUD_URI!,
    zillizToken: string = process.env.ZILLIZ_CLOUD_TOKEN!,
    openaiKey: string = process.env.OPENAI_API_KEY!
  ) {
    this.milvusClient = new MilvusClient({
      address: zillizUri,
      token: zillizToken
    });
    
    this.openai = new OpenAI({ apiKey: openaiKey });
  }

  /**
   * Convert text query to embedding vector
   */
  async embedText(query: string): Promise<number[]> {
    const response = await this.openai.embeddings.create({
      model: 'text-embedding-3-small',
      input: query
    });
    
    return response.data[0].embedding;
  }

  /**
   * Search brand voice guidelines
   * 
   * @example
   * const results = await retriever.searchBrandVoice(
   *   "How to write subject lines that sound rebellious but professional"
   * );
   */
  async searchBrandVoice(
    query: string,
    limit: number = 5,
    filter?: string
  ): Promise<SearchResult[]> {
    const queryVector = await this.embedText(query);
    
    const searchParams: any = {
      collection_name: 'persona_revrebel',
      vectors: [queryVector],
      limit: limit,
      output_fields: ['*']
    };
    
    if (filter) {
      searchParams.filter = filter;
    }
    
    const results = await this.milvusClient.search(searchParams);
    return this.formatResults(results.results);
  }

  /**
   * Find content examples from the brand library
   * 
   * @example
   * const examples = await retriever.searchExamples(
   *   "meeting cancellation emails",
   *   "email_template"
   * );
   */
  async searchExamples(
    query: string,
    artifactType?: string,
    limit: number = 5
  ): Promise<SearchResult[]> {
    const queryVector = await this.embedText(query);
    
    let filter = undefined;
    if (artifactType) {
      filter = `artifact_type == "${artifactType}"`;
    }
    
    const searchParams: any = {
      collection_name: 'prod_doc_search',
      vectors: [queryVector],
      limit: limit,
      output_fields: ['*']
    };
    
    if (filter) {
      searchParams.filter = filter;
    }
    
    const results = await this.milvusClient.search(searchParams);
    return this.formatResults(results.results);
  }

  /**
   * Get strategic recommendations
   * 
   * @example
   * const recs = await retriever.getRecommendations(
   *   "Should I use technical jargon for hotel executives?"
   * );
   */
  async getRecommendations(
    decisionContext: string,
    limit: number = 3
  ): Promise<SearchResult[]> {
    const queryVector = await this.embedText(decisionContext);
    
    const results = await this.milvusClient.search({
      collection_name: 'prod_recommendations',
      vectors: [queryVector],
      limit: limit,
      output_fields: ['*']
    });
    
    return this.formatResults(results.results);
  }

  /**
   * Browse by metadata filters without semantic search
   * 
   * @example
   * const emails = await retriever.browseByMetadata(
   *   "prod_doc_search",
   *   "artifact_type == 'email_template' && created_ym == '2025-10'"
   * );
   */
  async browseByMetadata(
    collection: string,
    filter: string,
    limit: number = 10
  ): Promise<any[]> {
    const results = await this.milvusClient.query({
      collection_name: collection,
      filter: filter,
      limit: limit,
      output_fields: ['*']
    });
    
    return results.data;
  }

  private formatResults(rawResults: any[]): SearchResult[] {
    return rawResults.map(hit => ({
      id: hit.id,
      similarity: hit.score,
      metadata: hit
    }));
  }
}

// ============================================
// USAGE EXAMPLES
// ============================================

async function demonstrateUsage() {
  const retriever = new REVREBELRetriever();
  
  // Use Case 1: Generate email content
  console.log('='.repeat(60));
  console.log('USE CASE: Writing a meeting reschedule email');
  console.log('='.repeat(60));
  
  const voiceGuidelines = await retriever.searchBrandVoice(
    'casual but professional tone for scheduling changes'
  );
  
  const examples = await retriever.searchExamples(
    'meeting reschedule emails',
    'email_template'
  );
  
  console.log('\n📝 Voice Guidelines:');
  voiceGuidelines.slice(0, 2).forEach(result => {
    console.log(`  Similarity: ${result.similarity.toFixed(3)}`);
    console.log(`  Content: ${result.metadata.content?.substring(0, 100)}...`);
  });
  
  console.log('\n📧 Example Templates:');
  examples.forEach(ex => {
    console.log(`  • ${ex.metadata.title || 'Untitled'}`);
  });
  
  // Use Case 2: Validate brand alignment
  console.log('\n' + '='.repeat(60));
  console.log('USE CASE: Check if copy is on-brand');
  console.log('='.repeat(60));
  
  const recs = await retriever.getRecommendations(
    'Is "Plot Twist: Your Meeting Has Moved" too casual for B2B?'
  );
  
  console.log('\n💡 Brand Guidance:');
  recs.forEach(rec => {
    console.log(`  Relevance: ${rec.similarity.toFixed(3)}`);
    console.log(`  Advice: ${rec.metadata.recommendation?.substring(0, 150)}...`);
  });
}

// Run demonstration
demonstrateUsage().catch(console.error);
```

---

### REST API (cURL Examples)

For languages without official SDKs, use Zilliz Cloud's REST API directly.

#### Authentication
```bash
export ZILLIZ_URI="https://your-cluster.api.gcp-us-west1.zillizcloud.com"
export ZILLIZ_TOKEN="your-api-token"
export OPENAI_KEY="your-openai-key"
```

#### Step 1: Generate Embedding (via OpenAI)
```bash
# Generate embedding for your query
curl -X POST "https://api.openai.com/v1/embeddings" \
  -H "Authorization: Bearer $OPENAI_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "text-embedding-3-small",
    "input": "How to write REVREBEL email subject lines"
  }' | jq '.data[0].embedding' > query_vector.json
```

#### Step 2: Search Brand Voice
```bash
curl -X POST "$ZILLIZ_URI/v2/vectordb/entities/search" \
  -H "Authorization: Bearer $ZILLIZ_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"collectionName\": \"persona_revrebel\",
    \"vector\": $(cat query_vector.json),
    \"limit\": 5,
    \"outputFields\": [\"*\"]
  }" | jq '.data'
```

#### Step 3: Search Examples with Filter
```bash
curl -X POST "$ZILLIZ_URI/v2/vectordb/entities/search" \
  -H "Authorization: Bearer $ZILLIZ_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"collectionName\": \"prod_doc_search\",
    \"vector\": $(cat query_vector.json),
    \"limit\": 5,
    \"filter\": \"artifact_type == 'email_template'\",
    \"outputFields\": [\"*\"]
  }" | jq '.data'
```

#### Step 4: Browse by Metadata (No Semantic Search)
```bash
curl -X POST "$ZILLIZ_URI/v2/vectordb/entities/query" \
  -H "Authorization: Bearer $ZILLIZ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "collectionName": "prod_doc_search",
    "filter": "artifact_type == \"email_template\" && created_ym == \"2025-10\"",
    "limit": 10,
    "outputFields": ["*"]
  }' | jq '.data'
```

---

### Go Example

```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"
    
    "github.com/milvus-io/milvus-sdk-go/v2/client"
    "github.com/sashabaranov/go-openai"
)

type REVREBELRetriever struct {
    milvusClient client.Client
    openaiClient *openai.Client
}

func NewREVREBELRetriever() (*REVREBELRetriever, error) {
    // Initialize Milvus connection
    milvusClient, err := client.NewClient(context.Background(), client.Config{
        Address: os.Getenv("ZILLIZ_CLOUD_URI"),
        APIKey:  os.Getenv("ZILLIZ_CLOUD_TOKEN"),
    })
    if err != nil {
        return nil, err
    }
    
    // Initialize OpenAI client
    openaiClient := openai.NewClient(os.Getenv("OPENAI_API_KEY"))
    
    return &REVREBELRetriever{
        milvusClient: milvusClient,
        openaiClient: openaiClient,
    }, nil
}

func (r *REVREBELRetriever) EmbedText(query string) ([]float32, error) {
    resp, err := r.openaiClient.CreateEmbeddings(
        context.Background(),
        openai.EmbeddingRequest{
            Model: openai.SmallEmbedding3,
            Input: []string{query},
        },
    )
    if err != nil {
        return nil, err
    }
    
    return resp.Data[0].Embedding, nil
}

func (r *REVREBELRetriever) SearchBrandVoice(query string, limit int) ([]client.SearchResult, error) {
    queryVector, err := r.EmbedText(query)
    if err != nil {
        return nil, err
    }
    
    searchResult, err := r.milvusClient.Search(
        context.Background(),
        "persona_revrebel",
        []string{},
        "",
        []string{"*"},
        []entity.Vector{entity.FloatVector(queryVector)},
        "vector",
        entity.COSINE,
        limit,
        nil,
    )
    
    return searchResult, err
}

func main() {
    retriever, err := NewREVREBELRetriever()
    if err != nil {
        log.Fatal(err)
    }
    defer retriever.milvusClient.Close()
    
    results, err := retriever.SearchBrandVoice(
        "How to write energetic email subject lines",
        5,
    )
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Println("Brand Voice Results:")
    for _, result := range results {
        fmt.Printf("Similarity: %.3f\n", result.Score)
        fmt.Printf("Content: %v\n\n", result.Fields)
    }
}
```

---

## Use Case Patterns

### Pattern 1: Content Generation Pipeline

**Scenario:** Generate a new email template on-brand

```python
def generate_branded_email(retriever, email_purpose: str) -> Dict:
    """
    Complete pipeline: Query brand → Synthesize → Generate
    """
    
    # Step 1: Get voice guidelines
    voice = retriever.search_brand_voice(
        f"tone and style for {email_purpose} emails",
        limit=3
    )
    
    # Step 2: Get similar examples
    examples = retriever.search_examples(
        f"{email_purpose} email templates",
        artifact_type="email_template",
        limit=3
    )
    
    # Step 3: Get strategic recommendations
    recs = retriever.get_recommendations(
        f"best practices for {email_purpose} communication",
        limit=2
    )
    
    # Step 4: Synthesize context for LLM
    context = {
        'voice_guidelines': [v['metadata'] for v in voice],
        'example_templates': [e['metadata'] for e in examples],
        'recommendations': [r['metadata'] for r in recs]
    }
    
    # Step 5: Generate with context
    prompt = f"""
    Using the REVREBEL brand voice guidelines and examples provided,
    generate a {email_purpose} email template.
    
    Brand Voice: {context['voice_guidelines']}
    Examples: {context['example_templates']}
    Best Practices: {context['recommendations']}
    
    Create subject line and body copy that matches this brand.
    """
    
    # Send to LLM (Claude, GPT, etc.)
    generated_email = your_llm_call(prompt)
    
    return {
        'generated': generated_email,
        'brand_context': context,
        'confidence_score': calculate_confidence(voice, examples)
    }

# Usage
retriever = REVREBELBrandRetriever()
result = generate_branded_email(retriever, "project kickoff")
print(result['generated'])
```

### Pattern 2: Brand Validation System

**Scenario:** Check if proposed content matches brand standards

```python
def validate_brand_alignment(retriever, 
                            content: str,
                            content_type: str) -> Dict:
    """
    Score content against brand standards
    """
    
    # Query for similar approved content
    similar_approved = retriever.search_examples(
        content,
        artifact_type=content_type,
        limit=5
    )
    
    # Get voice guidelines for this content type
    voice_standards = retriever.search_brand_voice(
        f"{content_type} voice and tone requirements",
        limit=3
    )
    
    # Calculate alignment scores
    avg_similarity = sum(r['similarity_score'] for r in similar_approved) / len(similar_approved)
    
    # Determine if on-brand
    is_on_brand = avg_similarity > 0.75  # Threshold
    
    feedback = []
    if not is_on_brand:
        feedback.append(f"Low similarity to approved {content_type}s")
        feedback.append("Review these guidelines:")
        feedback.extend([v['metadata'].get('guideline') for v in voice_standards])
    
    return {
        'is_on_brand': is_on_brand,
        'similarity_score': avg_similarity,
        'feedback': feedback,
        'similar_approved_content': similar_approved
    }

# Usage
proposed_subject = "Hey there! Quick update about your account"
validation = validate_brand_alignment(
    retriever,
    proposed_subject,
    "email_subject"
)

if not validation['is_on_brand']:
    print("⚠️ Content may not be on-brand")
    print("\n".join(validation['feedback']))
```

### Pattern 3: Contextual Asset Selection

**Scenario:** Find the right visual asset for a campaign

```python
from PIL import Image
import torch
from transformers import CLIPProcessor, CLIPModel

def find_matching_visual_asset(retriever,
                               description: str = None,
                               reference_image: Image = None) -> List[Dict]:
    """
    Find brand-approved visual assets matching description or image
    """
    
    if reference_image:
        # Use CLIP to embed image
        model = CLIPModel.from_pretrained("openai/clip-vit-base-patch32")
        processor = CLIPProcessor.from_pretrained("openai/clip-vit-base-patch32")
        
        inputs = processor(images=reference_image, return_tensors="pt")
        image_embedding = model.get_image_features(**inputs)
        query_vector = image_embedding.detach().numpy()[0].tolist()
        
    elif description:
        # Use CLIP to embed text description
        model = CLIPModel.from_pretrained("openai/clip-vit-base-patch32")
        processor = CLIPProcessor.from_pretrained("openai/clip-vit-base-patch32")
        
        inputs = processor(text=[description], return_tensors="pt")
        text_embedding = model.get_text_features(**inputs)
        query_vector = text_embedding.detach().numpy()[0].tolist()
    
    # Search visual asset collection
    results = retriever.client.search(
        collection_name="prod_asset_embeddings",
        data=[query_vector],
        limit=10,
        output_fields=["*"],
        filter="approval_status == 'approved'"
    )
    
    return retriever._format_results(results[0])

# Usage Example 1: Text description
assets = find_matching_visual_asset(
    retriever,
    description="modern hotel lobby with technology elements"
)

print("Matching brand assets:")
for asset in assets[:3]:
    print(f"  {asset['metadata']['asset_name']} (similarity: {asset['similarity_score']:.3f})")

# Usage Example 2: Reference image
reference = Image.open("competitor_ad.jpg")
similar_assets = find_matching_visual_asset(
    retriever,
    reference_image=reference
)
```

### Pattern 4: Time-Based Content Evolution

**Scenario:** Track how brand voice has evolved

```python
def analyze_brand_evolution(retriever,
                           topic: str,
                           time_periods: List[str]) -> Dict:
    """
    Compare brand content across time periods
    """
    
    evolution_data = {}
    
    for period in time_periods:
        # Query content from specific time period
        content = retriever.browse_by_metadata(
            "prod_doc_search",
            f"created_ym == '{period}' && domain == '{topic}'",
            limit=20
        )
        
        evolution_data[period] = {
            'content_count': len(content),
            'artifact_types': set(c.get('artifact_type') for c in content),
            'sample_content': content[:3]
        }
    
    return evolution_data

# Usage
evolution = analyze_brand_evolution(
    retriever,
    topic="email_communication",
    time_periods=["2025-08", "2025-09", "2025-10"]
)

print("Brand Evolution Analysis:")
for period, data in evolution.items():
    print(f"\n{period}:")
    print(f"  Content pieces: {data['content_count']}")
    print(f"  Types: {', '.join(data['artifact_types'])}")
```

---

## Brand Synthesis Workflows

### Workflow 1: Multi-Modal Brand Package Generation

**Goal:** Create a complete branded asset package (copy + visuals)

```python
class BrandPackageGenerator:
    """
    Orchestrates retrieval from multiple collections to generate
    cohesive brand packages
    """
    
    def __init__(self, retriever):
        self.retriever = retriever
        
    def generate_campaign_package(self,
                                  campaign_type: str,
                                  target_audience: str) -> Dict:
        """
        Generate complete branded campaign with copy and visuals
        """
        
        # 1. Voice & Tone
        voice = self.retriever.search_brand_voice(
            f"{campaign_type} tone for {target_audience}",
            limit=3
        )
        
        # 2. Copy Examples
        copy_examples = self.retriever.search_examples(
            f"{campaign_type} campaigns targeting {target_audience}",
            limit=5
        )
        
        # 3. Strategic Guidance
        strategy = self.retriever.get_recommendations(
            f"best practices for {campaign_type} aimed at {target_audience}",
            limit=3
        )
        
        # 4. Visual Assets (if applicable)
        # Use CLIP embedding for campaign_type
        visual_query = f"{campaign_type} visuals for {target_audience}"
        # ... CLIP embedding logic here ...
        
        # 5. Synthesize into prompt
        brand_package = {
            'voice_guidelines': self._extract_guidelines(voice),
            'copy_examples': self._extract_examples(copy_examples),
            'strategic_direction': self._extract_strategy(strategy),
            'visual_assets': [],  # populated from CLIP search
            'generation_prompt': self._build_generation_prompt(
                voice, copy_examples, strategy
            )
        }
        
        return brand_package
    
    def _extract_guidelines(self, voice_results):
        return [
            {
                'guideline': r['metadata'].get('guideline'),
                'examples': r['metadata'].get('examples'),
                'confidence': r['similarity_score']
            }
            for r in voice_results
        ]
    
    def _build_generation_prompt(self, voice, examples, strategy):
        return f"""
        BRAND VOICE PARAMETERS:
        {self._format_voice(voice)}
        
        PROVEN EXAMPLES:
        {self._format_examples(examples)}
        
        STRATEGIC DIRECTION:
        {self._format_strategy(strategy)}
        
        Generate campaign materials matching these brand standards.
        """

# Usage
generator = BrandPackageGenerator(retriever)
package = generator.generate_campaign_package(
    campaign_type="email_nurture_sequence",
    target_audience="hotel_revenue_managers"
)

# Feed package to LLM for generation
final_campaign = your_llm.generate(package['generation_prompt'])
```

### Workflow 2: Interactive Brand Q&A System

**Goal:** Build a chatbot that answers questions using brand knowledge

```python
class BrandKnowledgeAssistant:
    """
    RAG system that answers questions by retrieving from brand database
    """
    
    def __init__(self, retriever, llm_client):
        self.retriever = retriever
        self.llm = llm_client
        
    def answer_brand_question(self, question: str) -> str:
        """
        Use RAG pattern: Retrieve → Augment → Generate
        """
        
        # Determine which collection(s) to query
        collections_to_search = self._classify_question(question)
        
        # Retrieve relevant context
        context_chunks = []
        
        for collection in collections_to_search:
            if collection == 'persona':
                results = self.retriever.search_brand_voice(question, limit=3)
            elif collection == 'examples':
                results = self.retriever.search_examples(question, limit=3)
            elif collection == 'recommendations':
                results = self.retriever.get_recommendations(question, limit=3)
            
            context_chunks.extend(results)
        
        # Build augmented prompt
        context_text = self._format_context(context_chunks)
        
        augmented_prompt = f"""
        You are a REVREBEL brand expert. Answer this question using ONLY
        the provided brand documentation. Do not invent information.
        
        QUESTION: {question}
        
        BRAND DOCUMENTATION:
        {context_text}
        
        ANSWER:
        """
        
        # Generate response
        answer = self.llm.generate(augmented_prompt)
        
        return {
            'answer': answer,
            'sources': [c['id'] for c in context_chunks],
            'confidence': self._calculate_confidence(context_chunks)
        }
    
    def _classify_question(self, question: str) -> List[str]:
        """
        Determine which collections are relevant for this question
        """
        q_lower = question.lower()
        
        collections = []
        
        if any(word in q_lower for word in ['voice', 'tone', 'sound', 'personality']):
            collections.append('persona')
        
        if any(word in q_lower for word in ['example', 'sample', 'show me']):
            collections.append('examples')
        
        if any(word in q_lower for word in ['should', 'best practice', 'recommend']):
            collections.append('recommendations')
        
        # Default: search all
        if not collections:
            collections = ['persona', 'examples', 'recommendations']
        
        return collections

# Usage
assistant = BrandKnowledgeAssistant(retriever, your_llm_client)

response = assistant.answer_brand_question(
    "Should I use gaming metaphors when writing to hotel CFOs?"
)

print(response['answer'])
print(f"\nSources: {', '.join(response['sources'])}")
print(f"Confidence: {response['confidence']:.2%}")
```

---

## Quality Assurance & Validation

### QA Pattern 1: Similarity Threshold Validation

```python
def validate_retrieval_quality(results: List[Dict],
                              min_similarity: float = 0.70) -> Dict:
    """
    Ensure retrieved content is sufficiently relevant
    """
    
    high_quality = [r for r in results if r['similarity_score'] >= min_similarity]
    low_quality = [r for r in results if r['similarity_score'] < min_similarity]
    
    return {
        'passed_quality_check': len(low_quality) == 0,
        'high_quality_count': len(high_quality),
        'low_quality_count': len(low_quality),
        'avg_similarity': sum(r['similarity_score'] for r in results) / len(results),
        'recommendation': 'Use results' if len(low_quality) == 0 else 'Refine query'
    }

# Usage
results = retriever.search_brand_voice("email subject lines")
qa = validate_retrieval_quality(results)

if not qa['passed_quality_check']:
    print(f"⚠️ Low relevance detected. Avg similarity: {qa['avg_similarity']:.3f}")
    print("Consider rephrasing your query.")
```

### QA Pattern 2: Cross-Collection Consistency Check

```python
def check_brand_consistency(retriever, content_theme: str) -> Dict:
    """
    Verify that guidance is consistent across collections
    """
    
    # Query same theme across collections
    voice_results = retriever.search_brand_voice(content_theme, limit=3)
    example_results = retriever.search_examples(content_theme, limit=3)
    rec_results = retriever.get_recommendations(content_theme, limit=3)
    
    # Extract key attributes from each
    voice_attributes = extract_attributes(voice_results)
    example_attributes = extract_attributes(example_results)
    rec_attributes = extract_attributes(rec_results)
    
    # Calculate consistency score
    consistency = calculate_attribute_overlap(
        voice_attributes,
        example_attributes,
        rec_attributes
    )
    
    return {
        'consistency_score': consistency,
        'is_consistent': consistency > 0.75,
        'conflicting_guidance': identify_conflicts(
            voice_results, example_results, rec_results
        )
    }
```

### QA Pattern 3: Temporal Relevance Check

```python
def ensure_current_guidance(retriever,
                           query: str,
                           max_age_months: int = 6) -> List[Dict]:
    """
    Filter results to recent content only
    """
    from datetime import datetime, timedelta
    
    # Calculate cutoff date
    cutoff = datetime.now() - timedelta(days=max_age_months * 30)
    cutoff_ym = cutoff.strftime("%Y-%m")
    
    # Search with time filter
    results = retriever.client.search(
        collection_name="prod_doc_search",
        data=[retriever.embed_text_query(query)],
        limit=10,
        output_fields=["*"],
        filter=f"created_ym >= '{cutoff_ym}'"
    )
    
    formatted = retriever._format_results(results[0])
    
    if len(formatted) < 3:
        print(f"⚠️ Warning: Only {len(formatted)} recent results found.")
        print("Consider expanding time window or updating database.")
    
    return formatted
```

---

## Advanced Techniques

### Technique 1: Hybrid Search (Dense + Sparse)

```python
def hybrid_search(retriever,
                 semantic_query: str,
                 metadata_filters: List[str],
                 weights: Dict[str, float] = {'semantic': 0.7, 'metadata': 0.3}):
    """
    Combine semantic similarity with metadata matching
    """
    
    # Semantic search
    semantic_results = retriever.search_examples(semantic_query, limit=20)
    
    # Metadata-based retrieval
    metadata_results = []
    for filter_expr in metadata_filters:
        results = retriever.browse_by_metadata(
            "prod_doc_search",
            filter_expr,
            limit=20
        )
        metadata_results.extend(results)
    
    # Merge and re-rank
    merged = merge_and_rerank(
        semantic_results,
        metadata_results,
        weights
    )
    
    return merged[:10]  # Top 10 final results
```

### Technique 2: Query Expansion

```python
def expanded_retrieval(retriever, original_query: str):
    """
    Generate query variations for broader coverage
    """
    
    # Generate query expansions
    expansions = [
        original_query,
        f"examples of {original_query}",
        f"best practices for {original_query}",
        f"{original_query} templates",
        f"{original_query} guidelines"
    ]
    
    all_results = []
    for query in expansions:
        results = retriever.search_examples(query, limit=3)
        all_results.extend(results)
    
    # Deduplicate and rank
    deduplicated = deduplicate_by_id(all_results)
    ranked = rank_by_max_similarity(deduplicated)
    
    return ranked[:10]
```

---

## Performance Optimization

### Optimization 1: Batch Embeddings

When generating multiple embeddings, batch them:

```python
# Good: Batch processing
queries = ["query1", "query2", "query3"]
embeddings = openai.embeddings.create(
    model="text-embedding-3-small",
    input=queries  # Send all at once
)
```

### Optimization 2: Cache Embeddings

Store frequently-used query embeddings:

```python
query_cache = {}

def get_or_embed(query: str):
    if query not in query_cache:
        query_cache[query] = embed_text(query)
    return query_cache[query]
```

### Optimization 3: Limit Output Fields

Only request fields you need:

```python
# Good: Specific fields
output_fields=["primary_key", "title", "content_preview"]

# Avoid: Everything
output_fields=["*"]  # Slower, more data transfer
```

### Optimization 4: Use Pagination

For browsing large result sets:

```python
offset = 0
limit = 100

while True:
    results = retriever.browse_by_metadata(
        "prod_doc_search",
        "artifact_type == 'email_template'",
        limit=limit,
        offset=offset
    )
    
    if not results:
        break
    
    process_results(results)
    offset += limit
```

---

## Error Handling

### Production-Ready Error Handling

```python
from typing import Optional, List, Dict
import logging

class RobustREVREBELRetriever:
    """Production-ready retriever with error handling"""
    
    def search_with_fallback(self,
                            query: str,
                            collection: str = "prod_doc_search",
                            limit: int = 5) -> Optional[List[Dict]]:
        """
        Search with automatic fallback strategies
        """
        try:
            # Primary search
            results = self.search_examples(query, limit=limit)
            
            # Validate quality
            if not results or all(r['similarity_score'] < 0.6 for r in results):
                logging.warning(f"Low quality results for: {query}")
                
                # Fallback: Try broader query
                broader_query = self._broaden_query(query)
                results = self.search_examples(broader_query, limit=limit)
            
            return results
            
        except Exception as e:
            logging.error(f"Search failed for {query}: {e}")
            
            # Fallback: Return default branded content
            return self._get_default_content(collection)
    
    def _broaden_query(self, query: str) -> str:
        """Remove specific terms to broaden search"""
        # Implementation: Remove adjectives, keep nouns
        return query
    
    def _get_default_content(self, collection: str) -> List[Dict]:
        """Return safe default content when search fails"""
        return self.browse_by_metadata(
            collection,
            "status == 'approved' && artifact_type == 'template'",
            limit=5
        )
```

---

## Complete End-to-End Example

```python
#!/usr/bin/env python3
"""
Complete workflow: Generate branded email from scratch using vector retrieval
"""

from revrebel_retriever import REVREBELBrandRetriever
import anthropic  # Or openai, etc.

def generate_branded_email_complete(
    email_purpose: str,
    recipient_context: str,
    tone_preference: str = "professional but energetic"
):
    """
    Full pipeline: Query → Retrieve → Synthesize → Generate → Validate
    """
    
    # Initialize
    retriever = REVREBELBrandRetriever()
    llm = anthropic.Anthropic()
    
    print(f"🔍 Retrieving brand knowledge for: {email_purpose}")
    
    # Step 1: Voice Guidelines
    voice_query = f"{tone_preference} tone for {email_purpose} emails to {recipient_context}"
    voice_results = retriever.search_brand_voice(voice_query, limit=3)
    
    print(f"  ✓ Found {len(voice_results)} voice guidelines")
    
    # Step 2: Example Templates
    example_query = f"{email_purpose} email examples for {recipient_context}"
    examples = retriever.search_examples(example_query, artifact_type="email_template", limit=3)
    
    print(f"  ✓ Found {len(examples)} example templates")
    
    # Step 3: Strategic Recommendations
    strategy_query = f"best practices for {email_purpose} communication with {recipient_context}"
    recommendations = retriever.get_recommendations(strategy_query, limit=2)
    
    print(f"  ✓ Retrieved {len(recommendations)} strategic recommendations")
    
    # Step 4: Build Context
    context = f"""
    BRAND VOICE GUIDELINES:
    {format_voice_guidelines(voice_results)}
    
    PROVEN EXAMPLES:
    {format_examples(examples)}
    
    STRATEGIC GUIDANCE:
    {format_recommendations(recommendations)}
    """
    
    # Step 5: Generate
    print(f"\n✨ Generating branded email...")
    
    prompt = f"""
    You are writing on behalf of REVREBEL. Using the brand voice guidelines,
    examples, and strategic direction provided, create a {email_purpose} email.
    
    Recipient Context: {recipient_context}
    Desired Tone: {tone_preference}
    
    {context}
    
    Create:
    1. Subject line (must be energetic and confident)
    2. Email body (maintain REVREBEL personality)
    
    Follow the brand examples closely.
    """
    
    response = llm.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1000,
        messages=[{"role": "user", "content": prompt}]
    )
    
    generated_email = response.content[0].text
    
    # Step 6: Validate
    print(f"\n🔍 Validating brand alignment...")
    
    validation = validate_brand_alignment(
        retriever,
        generated_email,
        "email_template"
    )
    
    print(f"  Brand Alignment Score: {validation['similarity_score']:.2%}")
    print(f"  On-Brand: {'✓ Yes' if validation['is_on_brand'] else '✗ No'}")
    
    if not validation['is_on_brand']:
        print("\n⚠️ Warning: Generated content may not be fully on-brand")
        print("Feedback:")
        for feedback in validation['feedback']:
            print(f"  • {feedback}")
    
    # Step 7: Return Complete Package
    return {
        'generated_email': generated_email,
        'brand_context_used': {
            'voice_guidelines': voice_results,
            'examples': examples,
            'recommendations': recommendations
        },
        'validation': validation,
        'metadata': {
            'purpose': email_purpose,
            'recipient_context': recipient_context,
            'tone_preference': tone_preference
        }
    }


# Helper formatting functions
def format_voice_guidelines(results):
    return "\n".join([
        f"- {r['metadata'].get('guideline', 'N/A')} (confidence: {r['similarity_score']:.2%})"
        for r in results
    ])

def format_examples(results):
    return "\n\n".join([
        f"Example {i+1}:\n{r['metadata'].get('content', 'N/A')}"
        for i, r in enumerate(results)
    ])

def format_recommendations(results):
    return "\n".join([
        f"- {r['metadata'].get('recommendation', 'N/A')}"
        for r in results
    ])


# ============================================
# RUN COMPLETE WORKFLOW
# ============================================

if __name__ == "__main__":
    result = generate_branded_email_complete(
        email_purpose="meeting reschedule notification",
        recipient_context="hotel revenue manager",
        tone_preference="casual but professional"
    )
    
    print("\n" + "=" * 60)
    print("FINAL GENERATED EMAIL")
    print("=" * 60)
    print(result['generated_email'])
    
    print("\n" + "=" * 60)
    print("BRAND VALIDATION REPORT")
    print("=" * 60)
    print(f"Alignment Score: {result['validation']['similarity_score']:.2%}")
    print(f"Status: {'✓ On-Brand' if result['validation']['is_on_brand'] else '✗ Needs Review'}")
```

---

## Summary of Key Concepts

| Concept | When to Use | Collection | Method |
|---------|-------------|-----------|---------|
| **Semantic Search** | Find content by meaning | Any | `search_vectors()` |
| **Metadata Filtering** | Browse by category/date | Any | `query_vectors()` |
| **Hybrid Query** | Meaning + filters | Any | `search_vectors()` with `filter_expr` |
| **Voice Retrieval** | Copy generation | `persona_revrebel` | Text embedding → search |
| **Example Lookup** | Reference content | `prod_doc_search` | Text embedding → search |
| **Visual Search** | Asset selection | `prod_asset_embeddings` | CLIP embedding → search |
| **Strategy Check** | Decision validation | `prod_recommendations` | Text embedding → search |

---

## Quick Reference: Common Queries

### Generate Email Copy
```python
voice = retriever.search_brand_voice("email subject lines energetic")
examples = retriever.search_examples("email templates", artifact_type="email_template")
```

### Validate Content
```python
similar = retriever.search_examples(your_content, limit=5)
avg_score = sum(r['similarity_score'] for r in similar) / len(similar)
is_on_brand = avg_score > 0.75
```

### Find Visual Assets
```python
# Text description
assets = find_matching_visual_asset(retriever, description="modern hotel lobby")

# Reference image
assets = find_matching_visual_asset(retriever, reference_image=your_image)
```

### Get Strategic Guidance
```python
recs = retriever.get_recommendations("Should I use gaming metaphors?")
```

### Browse by Date
```python
recent = retriever.browse_by_metadata(
    "prod_doc_search",
    "created_ym >= '2025-10'",
    limit=20
)
```

---

## Troubleshooting

### Low Similarity Scores

**Problem:** All search results have similarity < 0.60

**Solutions:**
1. Broaden your query (remove adjectives, keep nouns)
2. Try query expansion (multiple variations)
3. Check if content exists in database
4. Verify embedding model matches ingestion

### Empty Results

**Problem:** Search returns no results

**Solutions:**
1. Verify collection name is correct
2. Check filter syntax (common error: wrong operators)
3. Ensure embeddings are same dimension as collection
4. Test with no filters first

### Slow Performance

**Problem:** Queries take too long

**Solutions:**
1. Batch embed multiple queries at once
2. Cache frequently-used embeddings
3. Reduce `output_fields` to only what you need
4. Use pagination for large result sets
5. Add metadata filters to narrow search space

### Inconsistent Results

**Problem:** Same query returns different results

**Solutions:**
1. Check if collection has been updated
2. Verify similarity threshold is appropriate
3. Use hybrid search (semantic + metadata filters)
4. Cross-validate across multiple collections

---

## Best Practices Summary

1. **Start with semantic search** for meaning-based retrieval
2. **Add filters gradually** to narrow results
3. **Validate similarity scores** before using content
4. **Cache embeddings** for frequently-used queries
5. **Combine collections** for richer context
6. **Always validate** generated content against brand standards
7. **Handle errors gracefully** with fallback strategies
8. **Monitor quality** over time with metrics

---

## Next Steps

1. **Start Simple:** Begin with basic voice retrieval queries
2. **Build Context:** Combine multiple collections for richer context
3. **Validate Quality:** Always check similarity scores
4. **Iterate Queries:** Refine searches based on result quality
5. **Cache Intelligently:** Store frequently-used embeddings
6. **Monitor Performance:** Track retrieval quality over time

---

## Resources

**Documentation:**
- Zilliz Cloud: https://docs.zilliz.com
- OpenAI Embeddings: https://platform.openai.com/docs/guides/embeddings
- CLIP Model: https://github.com/openai/CLIP
- Milvus Python SDK: https://milvus.io/docs/install-pymilvus.md

**Support:**
- REVREBEL Brand Database Version: 1.0
- Last Updated: October 2025
- Technical Support: [Your contact info]

---

**Document Version:** 1.0  
**Last Updated:** October 31, 2025  
**Maintained by:** REVREBEL Brand Systems Team