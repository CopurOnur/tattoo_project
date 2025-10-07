# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a monorepo containing an AI-powered tattoo search engine with both backend and frontend components. Users upload tattoo images to find visually similar designs using state-of-the-art computer vision models (CLIP, DINOv2, SigLIP). The system combines image captioning with visual similarity search and provides advanced patch-level attention analysis.

**Repository Structure:**
- `tattoo_search_engine/` - Python FastAPI backend deployed on Hugging Face Spaces
- `tattoo_search_engine_frontend/` - Next.js TypeScript frontend deployed on Vercel

**Live Deployments:**
- Frontend: https://tattoo-search-frontend.vercel.app
- Backend API: https://onurcopur-tattoo-search-engine.hf.space

## Backend (`tattoo_search_engine/`)

### Commands

```bash
# Local development
cd tattoo_search_engine
python app.py

# Docker
docker build -t tattoo-search .
docker run -p 7860:7860 --env-file .env tattoo-search

# Testing endpoints
curl http://localhost:7860/health
curl http://localhost:7860/models
curl -X POST http://localhost:7860/search -F "file=@tattoo.jpg" -F "embedding_model=clip"
```

### Environment Setup

Required: `HF_TOKEN` (HuggingFace API token for GLM-4.5V captioning)

Create `.env` file:
```
HF_TOKEN=your_token_here
```

### Architecture

**Core Pipeline Flow:**
1. Image Upload → FastAPI `/search` endpoint
2. Caption Generation → GLM-4.5V via HuggingFace InferenceClient (Novita provider)
3. Multi-Platform Search → SearchEngineManager coordinates Pinterest, Reddit, Instagram searches
4. URL Validation → URLValidator filters accessible images
5. Embedding Extraction → CLIP/DINOv2/SigLIP encodes query + candidates
6. Similarity Computation → Cosine similarity ranking with parallel processing
7. Optional Patch Analysis → PatchAttentionAnalyzer for detailed visual correspondence

**Key Components:**

- `main.py - TattooSearchEngine`: Main orchestration class
  - `generate_caption()`: GLM-4.5V captioning via HuggingFace InferenceClient
  - `search_images()`: Delegates to SearchEngineManager with caching
  - `compute_similarity()`: Parallel processing with early stopping optimization

- `embeddings.py`: Model abstraction layer
  - `EmbeddingModel`: Abstract base class
  - `CLIPEmbedding`, `DINOv2Embedding`, `SigLIPEmbedding`: Model implementations
  - `EmbeddingModelFactory`: Factory pattern with fallback handling
  - All models support global embeddings and patch-level features

- `search_engines/`: Multi-platform search system
  - `SearchEngineManager`: Coordinates parallel searches with fallback strategies
  - Platform implementations: Pinterest, Reddit, Instagram
  - Tiered approach: primary platforms → additional platforms → query simplification

- `patch_attention.py`: Visual correspondence analysis
  - Computes patch-level attention matrices between images
  - Creates matplotlib visualizations as base64 PNG
  - Returns attention data showing which regions correspond best

- `utils/`: Supporting utilities
  - `SearchCache`: In-memory LRU cache with TTL (1hr, 1000 entries)
  - `URLValidator`: Concurrent URL validation

**Performance Optimizations:**
- GPU acceleration when available
- ThreadPoolExecutor for concurrent downloads (max 10 workers)
- Early stopping: stops at 20 results (5 with patch attention)
- Search result caching (1hr TTL)
- Future cancellation after targets met
- Global model instance reuse

**Deployment:**
- Dockerized for HuggingFace Spaces
- Port 7860 (HF Spaces default)
- Recommended: T4 Small GPU or higher
- Models auto-download and cache in `/app/cache`

## Frontend (`tattoo_search_engine_frontend/`)

### Commands

```bash
cd tattoo_search_engine_frontend

# Development
npm run dev          # http://localhost:3000

# Production
npm run build        # Build for production
npm run start        # Start production server
npm run export       # Build and export static site

# Code Quality
npm run lint         # Run ESLint
```

### Architecture

**Framework:** Next.js 14 (Pages Router) with TypeScript
**Node.js:** Requires 18+

**State Management Pattern (pages/index.tsx):**
- Search State: `selectedImage`, `results`, `caption`, `isLoading`, `error`
- Model Config: `selectedModel`, `usedModel`, `patchAttentionEnabled`
- Analysis State: `detailedAnalysis`, `analysisLoading`, `selectedResultForAnalysis`

**Backend Integration (60s timeout):**

1. `POST /search` (line 59 in pages/index.tsx)
   - Query params: `embedding_model`, `include_patch_attention`
   - Returns: `SearchResponse` with results, caption, model info
   - Basic patch attention included when enabled

2. `POST /analyze-attention` (line 127 in pages/index.tsx)
   - Query params: `candidate_url`, `embedding_model`, `include_visualizations`
   - Returns: `DetailedAttentionAnalysis` with full matrix, visualizations
   - Called on-demand when user clicks "Analyze" on a result

Backend URL: `NEXT_PUBLIC_BACKEND_URL` environment variable (see `vercel.json`)

**Type System (types/search.ts):**
- `SearchResult`: Basic result with score, url, optional patch attention
- `SearchResponse`: API response from `/search`
- `DetailedAttentionAnalysis`: Full analysis from `/analyze-attention`
- `PatchCorrespondence`: Query-to-candidate patch mappings

**Two-Tier Attention Analysis:**

1. Quick Attention (SearchResults.tsx)
   - Lightweight summary in search results
   - Basic similarity metrics in cards
   - Minimal overhead

2. Detailed Analysis Modal (AttentionAnalysisPanel.tsx + PatchCorrespondenceViewer.tsx)
   - Full patch-to-patch attention matrix
   - Interactive heatmaps and correspondence maps
   - Triggered on-demand
   - Tabbed interface: Overview, Statistics, Visualizations

**Component Organization:**
- `ImageUpload.tsx`: Drag-and-drop with preview
- `SearchResults.tsx`: Results grid
- `ModelSelector.tsx`: Embedding model selection
- `RobustImage.tsx`: Fallback for broken URLs
- `AttentionAnalysisPanel.tsx`: Detailed analysis modal
- `PatchCorrespondenceViewer.tsx`: Patch correspondence visualization

**Deployment (Vercel):**
- `vercel.json` defines env vars, max duration (30s), security headers
- Update `NEXT_PUBLIC_BACKEND_URL` for new environments
- Uses Next.js Image with wildcard remote patterns (next.config.js)

## Cross-Cutting Concerns

### Model Selection
Both backend and frontend support dynamic switching between CLIP, DINOv2, and SigLIP. Backend uses global singleton pattern with lazy initialization. Frontend passes model selection via query params.

### Error Handling
- Backend: Fallback strategies for captioning, search, and validation
- Frontend: AbortController for timeouts (60s), user-friendly error messages

### Testing the Full Stack

1. Start backend: `cd tattoo_search_engine && python app.py`
2. Start frontend: `cd tattoo_search_engine_frontend && npm run dev`
3. Navigate to: http://localhost:3000
4. Upload image, select model, search

### Adding New Features

**New Embedding Model:**
1. Add class in `tattoo_search_engine/embeddings.py` inheriting from `EmbeddingModel`
2. Implement required methods: `load_model()`, `encode_image()`, `encode_image_patches()`, `get_model_name()`
3. Register in `EmbeddingModelFactory.AVAILABLE_MODELS`
4. Add config to `get_default_model_configs()`
5. Update frontend `ModelSelector.tsx` if UI changes needed

**New Search Platform:**
1. Create engine in `tattoo_search_engine/search_engines/` inheriting from `BaseSearchEngine`
2. Add platform to `SearchPlatform` enum in `base.py`
3. Implement `search()` and `is_valid_url()` methods
4. Register in `SearchEngineManager.engines`
5. Update platform prioritization in `search_with_fallback()`

### Common Development Patterns

**Backend:**
- Models are singletons - reused globally to avoid reloading
- Always use ThreadPoolExecutor for I/O-bound operations
- Implement early stopping for expensive operations
- Add comprehensive logging for debugging

**Frontend:**
- Separate loading states for independent operations
- Use AbortController for cancellable requests
- Type all API responses in types/search.ts
- Handle image loading failures with RobustImage component

### Important Notes

- Backend requires GPU for optimal performance (T4 Small recommended)
- Frontend build requires backend URL at build time for static optimization
- Both projects have individual CLAUDE.md files with more detailed information
- Search cache significantly reduces API load - respect TTL settings
- Patch attention is computationally expensive - only enable when needed
