# 🎨 AI-Powered Tattoo Search Engine

An intelligent tattoo search engine that uses state-of-the-art computer vision models to find visually similar tattoo designs. Upload an image and discover matching tattoos from across the web with advanced patch-level attention analysis.

[![Live Demo](https://img.shields.io/badge/Live-Demo-blue)](https://tattoo-search-frontend.vercel.app)
[![Backend API](https://img.shields.io/badge/API-Hugging%20Face-yellow)](https://onurcopur-tattoo-search-engine.hf.space)

## ✨ Features

- **🖼️ Visual Search**: Upload any tattoo image to find similar designs
- **🤖 Multiple AI Models**: Choose between CLIP, DINOv2, and SigLIP embeddings
- **💬 AI-Powered Captions**: Automatic tattoo description generation using GLM-4.5V
- **🔍 Multi-Platform Search**: Aggregates results from Pinterest, Reddit, and Instagram
- **📊 Patch-Level Analysis**: Deep visual correspondence analysis between query and results
- **⚡ High Performance**: GPU-accelerated with parallel processing and intelligent caching

## 🚀 Live Demo

- **Frontend**: [https://tattoo-search-frontend.vercel.app](https://tattoo-search-frontend.vercel.app)
- **Backend API**: [https://onurcopur-tattoo-search-engine.hf.space](https://onurcopur-tattoo-search-engine.hf.space)

## 🏗️ Architecture

This is a monorepo containing:

### Backend (`tattoo_search_engine/`)
- **Framework**: Python FastAPI
- **Deployment**: Hugging Face Spaces (Dockerized)
- **GPU**: Optimized for T4 Small or higher
- **Key Features**:
  - Image captioning with GLM-4.5V
  - Multi-model embedding extraction (CLIP, DINOv2, SigLIP)
  - Parallel image search across multiple platforms
  - Cosine similarity ranking with early stopping
  - Patch-level attention analysis
  - LRU caching with 1-hour TTL

### Frontend (`tattoo_search_engine_frontend/`)
- **Framework**: Next.js 14 (TypeScript)
- **Deployment**: Vercel
- **Key Features**:
  - Drag-and-drop image upload
  - Real-time search results
  - Interactive attention visualizations
  - Responsive design
  - Two-tier analysis (quick + detailed)

## 📋 Prerequisites

### Backend
- Python 3.9+
- Docker (optional)
- HuggingFace API token
- GPU recommended for optimal performance

### Frontend
- Node.js 18+
- npm or yarn

## 🛠️ Local Development

### Backend Setup

```bash
cd tattoo_search_engine

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create .env file
echo "HF_TOKEN=your_huggingface_token" > .env

# Run the server
python app.py
# Server runs at http://localhost:7860
```

### Frontend Setup

```bash
cd tattoo_search_engine_frontend

# Install dependencies
npm install

# Create .env.local file
echo "NEXT_PUBLIC_BACKEND_URL=http://localhost:7860" > .env.local

# Run development server
npm run dev
# Frontend runs at http://localhost:3000
```

### Docker Setup (Backend)

```bash
cd tattoo_search_engine

# Build image
docker build -t tattoo-search .

# Run container
docker run -p 7860:7860 --env-file .env tattoo-search
```

## 📡 API Endpoints

### Health Check
```bash
GET /health
```

### List Available Models
```bash
GET /models
```

### Search Tattoos
```bash
POST /search
Form Data:
  - file: Image file
  - embedding_model: "clip" | "dinov2" | "siglip"
  - include_patch_attention: "true" | "false"
```

### Detailed Attention Analysis
```bash
POST /analyze-attention
Query Params:
  - candidate_url: URL of result image
  - embedding_model: Model to use
  - include_visualizations: "true" | "false"
```

## 🎯 How It Works

1. **Image Upload**: User uploads a tattoo image
2. **Caption Generation**: GLM-4.5V generates a descriptive caption
3. **Multi-Platform Search**: Searches Pinterest, Reddit, Instagram using the caption
4. **URL Validation**: Filters out inaccessible images
5. **Embedding Extraction**: Selected model (CLIP/DINOv2/SigLIP) encodes images
6. **Similarity Computation**: Ranks results by cosine similarity
7. **Patch Analysis** (Optional): Computes fine-grained visual correspondences

## 🔧 Configuration

### Environment Variables

**Backend** (`.env`):
```env
HF_TOKEN=your_huggingface_token_here
```

**Frontend** (`.env.local`):
```env
NEXT_PUBLIC_BACKEND_URL=http://localhost:7860
```

### Performance Tuning

- **Cache Settings**: Adjust TTL and size in `SearchCache` class
- **Parallel Downloads**: Configure worker count in `ThreadPoolExecutor`
- **Early Stopping**: Modify target result counts in `compute_similarity()`
- **GPU Memory**: Adjust batch sizes in embedding models

## 🧪 Testing

### Backend
```bash
# Health check
curl http://localhost:7860/health

# List models
curl http://localhost:7860/models

# Search with image
curl -X POST http://localhost:7860/search \
  -F "file=@path/to/tattoo.jpg" \
  -F "embedding_model=clip"
```

### Frontend
```bash
cd tattoo_search_engine_frontend

# Run linter
npm run lint

# Build for production
npm run build
```

## 📦 Deployment

### Backend (Hugging Face Spaces)
1. Create a new Space on Hugging Face
2. Select Docker as the SDK
3. Push code to the Space repository
4. Add `HF_TOKEN` secret in Space settings
5. Select GPU hardware (T4 Small recommended)

### Frontend (Vercel)
1. Import project to Vercel
2. Set environment variable: `NEXT_PUBLIC_BACKEND_URL`
3. Deploy automatically on push to main

## 🤝 Contributing

Contributions are welcome! Here's how to add new features:

### Adding a New Embedding Model
1. Create class in `tattoo_search_engine/embeddings.py` inheriting from `EmbeddingModel`
2. Implement required methods
3. Register in `EmbeddingModelFactory`
4. Update frontend `ModelSelector.tsx`

### Adding a New Search Platform
1. Create engine in `tattoo_search_engine/search_engines/`
2. Inherit from `BaseSearchEngine`
3. Register in `SearchEngineManager`
4. Update platform prioritization

## 📄 License

This project is open source and available under the MIT License.

## 🙏 Acknowledgments

- **Models**: OpenAI CLIP, Meta DINOv2, Google SigLIP, THUDM GLM-4.5V
- **Deployment**: Hugging Face Spaces, Vercel
- **Frameworks**: FastAPI, Next.js

## 📧 Contact

For questions or feedback, please open an issue on GitHub.

---

Built with ❤️ using state-of-the-art AI models
