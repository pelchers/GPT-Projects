# pgvector Integration in BetterGPT

## Overview

pgvector provides our semantic search capabilities, enabling AI-powered code and content search through vector embeddings. This integration is crucial for our "Ask the Project" feature and semantic code navigation.

## Core Features

| Feature | Description | Priority |
|---------|-------------|----------|
| Semantic Search | Content-aware search across codebase | P0 |
| Code Navigation | Symbol and function finding | P0 |
| Similar Content | Find related documents and code | P1 |
| Vector Clustering | Group similar content | P2 |

## Implementation

### 1. Database Setup

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create tables for embeddings
CREATE TABLE document_embeddings (
  id SERIAL PRIMARY KEY,
  content_id UUID REFERENCES documents(id),
  embedding vector(1536),  -- OpenAI embedding dimension
  metadata JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE code_embeddings (
  id SERIAL PRIMARY KEY,
  file_path TEXT,
  symbol_name TEXT,
  embedding vector(1536),
  symbol_type TEXT,
  language TEXT,
  metadata JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Create indexes
CREATE INDEX ON document_embeddings USING ivfflat (embedding vector_cosine_ops);
CREATE INDEX ON code_embeddings USING ivfflat (embedding vector_cosine_ops);
```

### 2. Drizzle Schema

```typescript
// src/db/schema/embeddings.ts
import { pgTable, serial, uuid, text, timestamp, jsonb } from 'drizzle-orm/pg-core';
import { documents } from './documents';
import { vec } from 'drizzle-orm/pg-core';

export const documentEmbeddings = pgTable('document_embeddings', {
  id: serial('id').primaryKey(),
  contentId: uuid('content_id').references(() => documents.id),
  embedding: vec('embedding', { dimensions: 1536 }),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at').defaultNow()
});

export const codeEmbeddings = pgTable('code_embeddings', {
  id: serial('id').primaryKey(),
  filePath: text('file_path').notNull(),
  symbolName: text('symbol_name'),
  embedding: vec('embedding', { dimensions: 1536 }),
  symbolType: text('symbol_type'),
  language: text('language'),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at').defaultNow()
});
```

### 3. Search Service

```typescript
// src/services/SearchService.ts
import { db } from '../db';
import { eq, sql } from 'drizzle-orm';
import { documentEmbeddings, codeEmbeddings } from '../db/schema/embeddings';
import { OpenAIService } from './OpenAIService';

export class SearchService {
  private openai: OpenAIService;

  constructor() {
    this.openai = new OpenAIService();
  }

  async searchCode(query: string, options: SearchOptions = {}): Promise<SearchResult[]> {
    const embedding = await this.openai.createEmbedding(query);
    
    const results = await db
      .select({
        filePath: codeEmbeddings.filePath,
        symbolName: codeEmbeddings.symbolName,
        similarity: sql`1 - (embedding <=> ${embedding})`
      })
      .from(codeEmbeddings)
      .where(options.language ? eq(codeEmbeddings.language, options.language) : undefined)
      .orderBy(sql`similarity DESC`)
      .limit(options.limit || 10);

    return this.processResults(results);
  }

  async searchDocuments(query: string): Promise<SearchResult[]> {
    const embedding = await this.openai.createEmbedding(query);
    
    return db
      .select({
        contentId: documentEmbeddings.contentId,
        similarity: sql`1 - (embedding <=> ${embedding})`
      })
      .from(documentEmbeddings)
      .orderBy(sql`similarity DESC`)
      .limit(10);
  }
}
```

### 4. Embedding Generation

```typescript
// src/services/EmbeddingService.ts
export class EmbeddingService {
  private openai: OpenAIService;
  
  async generateCodeEmbeddings(
    code: string,
    metadata: CodeMetadata
  ): Promise<void> {
    const embedding = await this.openai.createEmbedding(code);
    
    await db.insert(codeEmbeddings).values({
      filePath: metadata.filePath,
      symbolName: metadata.symbolName,
      embedding,
      symbolType: metadata.type,
      language: metadata.language,
      metadata: metadata
    });
  }

  async updateEmbeddings(
    filePattern: string
  ): Promise<void> {
    const files = await this.getFiles(filePattern);
    for (const file of files) {
      const ast = await this.parseCode(file);
      const symbols = this.extractSymbols(ast);
      
      for (const symbol of symbols) {
        await this.generateCodeEmbeddings(
          symbol.code,
          {
            filePath: file.path,
            symbolName: symbol.name,
            type: symbol.type,
            language: file.language
          }
        );
      }
    }
  }
}
```

## Integration with UI

```typescript
// src/components/SearchBar.tsx
export const SearchBar = () => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<SearchResult[]>([]);
  const searchService = new SearchService();

  const handleSearch = async () => {
    const codeResults = await searchService.searchCode(query);
    const docResults = await searchService.searchDocuments(query);
    
    setResults([...codeResults, ...docResults]);
  };

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search code and documents..."
      />
      <SearchResults results={results} />
    </div>
  );
};
```

## Required Dependencies

```json
{
  "dependencies": {
    "drizzle-orm": "^0.28.0",
    "drizzle-kit": "^0.19.0",
    "postgres": "^3.4.0",
    "@neondatabase/serverless": "^0.6.0"
  }
}
```

## Performance Optimization

1. **Indexing Strategy**
   ```sql
   -- Optimize for different search patterns
   CREATE INDEX idx_code_embeddings_language 
   ON code_embeddings(language, embedding vector_cosine_ops);
   
   CREATE INDEX idx_code_embeddings_symbol_type 
   ON code_embeddings(symbol_type, embedding vector_cosine_ops);
   ```

2. **Batch Processing**
   ```typescript
   async function batchEmbeddingUpdate(
     symbols: CodeSymbol[],
     batchSize = 100
   ) {
     for (let i = 0; i < symbols.length; i += batchSize) {
       const batch = symbols.slice(i, i + batchSize);
       await Promise.all(
         batch.map(symbol => 
           generateCodeEmbeddings(symbol.code, symbol.metadata)
         )
       );
     }
   }
   ```

3. **Caching**
   ```typescript
   class EmbeddingCache {
     private cache = new LRUCache<string, Float32Array>(1000);
     
     async getEmbedding(content: string): Promise<Float32Array> {
       const key = this.hashContent(content);
       if (this.cache.has(key)) {
         return this.cache.get(key)!;
       }
       
       const embedding = await openai.createEmbedding(content);
       this.cache.set(key, embedding);
       return embedding;
     }
   }
   ```

## Best Practices

1. **Embedding Management**
   - Regular reindexing schedule
   - Version control for embeddings
   - Cleanup of stale embeddings

2. **Search Quality**
   - Tune similarity thresholds
   - Implement feedback loop
   - Monitor search metrics

3. **Resource Usage**
   - Monitor index sizes
   - Optimize query patterns
   - Implement connection pooling

## Monitoring

```typescript
// src/monitoring/SearchMetrics.ts
class SearchMetrics {
  async trackSearch(query: string, results: SearchResult[]) {
    await db.insert(searchLogs).values({
      query,
      resultCount: results.length,
      topSimilarity: results[0]?.similarity,
      timestamp: new Date()
    });
  }

  async getSearchStats(): Promise<SearchStats> {
    return db
      .select({
        avgResultCount: sql`avg(result_count)`,
        avgSimilarity: sql`avg(top_similarity)`,
        totalSearches: sql`count(*)`
      })
      .from(searchLogs);
  }
}
```

## Future Improvements

1. **Advanced Features**
   - Cross-language semantic search
   - Code similarity detection
   - Automated code grouping

2. **Performance**
   - Distributed embedding generation
   - Real-time index updates
   - Advanced caching strategies

3. **Integration**
   - IDE plugin support
   - CI/CD pipeline integration
   - External API access 