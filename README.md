# Feature Detection API Service

A production-ready, scalable FastAPI service for detecting SIFT features in images with Redis caching.

## Quick Start

### Prerequisites
- Docker & Docker Compose
- Python 3.11+ (for local development)

### Run with Docker Compose

```bash
# Start all services
docker-compose up -d

# Check service health
curl http://localhost:8000/health

# Detect features in an image
curl -X POST http://localhost:8000/api/v1/features/detect \
  -F "image=@path/to/your/image.jpg"

# View API documentation
open http://localhost:8000/docs
```

### Run Demo

```bash
# Activate virtual environment
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows

# Install dependencies
pip install -r requirements-dev.txt

# Run API demo
python tools/demo_api.py --n_images 3 --runs 2

# Run Streamlit dashboard
streamlit run tools/streamlit_app.py
```

### Stop Services

```bash
docker-compose down
```

## Architecture

### System Overview

```
┌─────────────────┐
│   Client        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐      ┌──────────────┐
│  FastAPI        │◄─────┤  Redis       │
│  (N Workers)    │      │  Cache       │
└────────┬────────┘      └──────────────┘
         │
         ▼
┌─────────────────┐
│ Feature         │
│ Detector        │
└─────────────────┘
```

### Data Flow

1. Client uploads image → API validates format/size
2. API generates SHA256 hash → checks Redis cache
3. **Cache Hit**: Returns cached results
4. **Cache Miss**: 
   - Detects features
   - Caches result in Redis
   - Returns response




## Project Structure

```
features_api_service/
├── api/                          # API Layer
│   ├── main.py                   # FastAPI application
│   ├── dependencies.py           # Dependency injection
│   ├── middleware.py             # Request logging & error handling
│   └── routes/
│       ├── features.py           # Feature detection endpoint
│       └── health.py             # Health check endpoints
├── core/                         
│   ├── config.py                 # Settings management
│   ├── exceptions.py             # Custom exceptions
│   └── logging_config.py         # Structured logging setup
├── services/                     
│   ├── cache_service.py          # Redis cache operations
│   ├── feature_detector.py       # feature detection
│   ├── feature_service.py        # Feature detection orchestration
│   └── image_service.py          # Image validation & processing
├── models/                       # Data Models
│   └── responses.py              # Pydantic response models
├── tests/                        
│   ├── test_api.py               # API integration tests
│   └── test_feature_detector.py  # Unit tests
├── tools/                        
│   ├── demo_api.py               # demo script
│   └── streamlit_app.py          # Interactive demo
├── data/                         
│   └── images/                   # Sample images
├── docker-compose.yml            
├── Dockerfile                    
├── requirements.txt              # Production dependencies
└── requirements-dev.txt          # Development dependencies
```

## Testing

### Run All Tests

```bash
# Activate virtual environment
source venv/bin/activate

# Run tests
pytest tests/ -v
```

```

## Configuration

### Environment Variables

Configure the service by setting environment variables or creating a `.env` file:

```bash
# Redis Configuration
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_DB=0
REDIS_MAX_CONNECTIONS=50

# Cache Configuration
CACHE_TTL=3600
CACHE_ENABLED=true

# API Configuration
API_HOST=0.0.0.0
API_PORT=8000
API_WORKERS=4

# Image Processing
MAX_IMAGE_SIZE_MB=10
UPLOAD_DIR=/app/data/uploads

# Logging
LOG_LEVEL=INFO
LOG_FORMAT=json

# Application
APP_NAME=Feature Detection API
APP_VERSION=1.0.0
ENVIRONMENT=production
```