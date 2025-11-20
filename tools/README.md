# Development Tools

This directory contains development and testing utilities for the Feature Detection API.

## 📁 Contents

| Tool | Description | Type |
|------|-------------|------|
| `demo_api.py` | Command-line API demo script | Python Script |
| `streamlit_app.py` | Interactive web dashboard | Streamlit App |

---

##  Prerequisites

These tools require **development dependencies**. Install them first:

```bash
# From project root
pip install -r requirements-dev.txt
```

**Why?** These tools require packages not included in production:
- `httpx`: Async HTTP client for API calls
- `redis`: Direct Redis client for statistics
- `streamlit`: Dashboard framework

---

## Demo API Script (`demo_api.py`)

### Purpose
Command-line utility to demonstrate the API's caching capabilities and performance.

### Features
-  Automatic health check
-  Process multiple images across multiple runs
-  Display cache hit/miss status
-  Show performance metrics
-  Redis statistics (keys, memory, hit rate)
-  Calculate cache speedup

### Usage

#### Basic Demo
```bash
# Run with default settings (2 runs, all images in data/images/)
python tools/demo_api.py
```

#### Custom Runs
```bash
# Run 10 iterations
python tools/demo_api.py --runs 10
```

#### Single Image
```bash
# Test specific image
python tools/demo_api.py --image data/images/lena_color_256.tif --runs 5
```

#### Custom API URL
```bash
# Point to different API
python tools/demo_api.py --api http://api.example.com:8080 --runs 3
```

### Example Output

```
 Checking API health...
 API is healthy!

================================================================================
 Feature Detection API Demo
================================================================================

 Initial Redis Stats:
   Keys: 0 | Memory: 0.12MB | Hit Rate: 0.0%

────────────────────────────────────────────────────────────────────────────────
 Run #1 - Cache MISS expected
────────────────────────────────────────────────────────────────────────────────
 lena_color_256.tif        | Keypoints: 1247 |  PROCESSED  | Time: 11245.3ms
 cameraman.tif             | Keypoints:  512 |  PROCESSED  | Time: 10987.2ms

────────────────────────────────────────────────────────────────────────────────
 Run #2 - Cache HIT expected
────────────────────────────────────────────────────────────────────────────────
 lena_color_256.tif        | Keypoints: 1247 |  CACHED    | Time:    12.5ms
 cameraman.tif             | Keypoints:  512 |  CACHED    | Time:     9.3ms

================================================================================
 Final Results
================================================================================

 Redis Stats:
   Cached Keys: 2
   Memory Used: 0.15MB
   Cache Hits: 2
   Cache Misses: 2
   Hit Rate: 50.0%

 Performance:
   Cache Miss Avg: 11116.3ms
   Cache Hit Avg:     10.9ms
   Speedup: 1019.8x faster with cache! 

================================================================================
 Demo Complete! Processed 4 requests
================================================================================
```

### Key Metrics Shown
- **Keypoints**: Number of SIFT features detected
- **Cache Status**: HIT () or MISS ()
- **Processing Time**: API processing latency
- **Redis Stats**: Cache size, memory, hit rate
- **Speedup**: Performance improvement with caching

---

## Streamlit Dashboard (`streamlit_app.py`)

### Purpose
Interactive web dashboard for monitoring and testing the Feature Detection API.

### Features
-  **Single Image Upload & Test**
-  **Batch Processing** (run demo script from UI)
-  **Redis Browser** (view/delete keys, monitor stats)
-  **Logs & Health** (Docker logs, API health, metrics)

### Usage

#### Start Dashboard
```bash
# From project root
streamlit run tools/streamlit_app.py
```

Dashboard opens automatically at `http://localhost:8501`

### Tabs Overview

#### Tab 1:  Single Image Detection
- Upload any image (TIF, PNG, JPG, BMP)
- Click "Detect Features"
- View results instantly:
  - Keypoints count
  - Descriptors shape
  - Cache status (HIT/MISS)
  - Processing time
  - Full JSON response

**Use Case**: Quick testing of individual images

#### Tab 2:  Batch Demo
- Configure number of images
- Set runs per image (1-20)
- Click "Run Batch Demo"
- View real-time output

**Use Case**: Performance benchmarking and load testing

#### Tab 3:  Redis Browser
- View Redis statistics:
  - Total keys
  - Memory used
  - Cache hit rate
  - Hits/misses count
- Browse cached keys
- Inspect cached values (JSON)
- View TTL for keys
- Delete individual keys or flush all

**Use Case**: Cache management and debugging

#### Tab 4:  Logs & Stats
- Check API health status
- View Docker logs (last 50 lines)
- Monitor cache performance
- Real-time metrics

**Use Case**: Monitoring and troubleshooting

### Dashboard Features

 **Real-time updates**: Refresh button for logs and stats  
 **Interactive**: Upload, browse, delete, refresh  
 **Visual**: Metrics, charts, status indicators  
 **Minimal**: Single file, clean interface  
 **Production-ready**: Connects to real services  

---

##  Common Workflows

### Workflow 1: Test Cache Performance
```bash
# 1. Clear cache (via dashboard)
streamlit run tools/streamlit_app.py
# → Navigate to "Redis Browser"
# → Click "Clear All Cache"

# 2. Run demo to see cache improvement
python tools/demo_api.py --runs 5
```

### Workflow 2: Load Testing
```bash
# 1. Start with clean slate
python tools/demo_api.py --runs 1

# 2. Test cache performance
python tools/demo_api.py --runs 20

# 3. Monitor via dashboard
streamlit run tools/streamlit_app.py
# → Check "Logs & Stats" tab
```

### Workflow 3: Debug API Issues
```bash
# 1. Launch dashboard
streamlit run tools/streamlit_app.py

# 2. Check health status (Tab 4)
# 3. View Docker logs (Tab 4)
# 4. Inspect Redis keys (Tab 3)
# 5. Test single image (Tab 1)
```

---

##  Configuration

Both tools use default localhost configuration. To change:

### Demo Script
```bash
python tools/demo_api.py --api http://your-api:8000
```

### Streamlit Dashboard
Edit `tools/streamlit_app.py`:
```python
# Lines 17-19
API_URL = "http://your-api:8000"
REDIS_HOST = "your-redis-host"
REDIS_PORT = 6379
```

---

## 🐛 Troubleshooting

### Issue: ModuleNotFoundError (httpx, redis, streamlit)
**Solution**: Install dev dependencies
```bash
pip install -r requirements-dev.txt
```

### Issue: API not responding
**Solution**: Ensure services are running
```bash
docker-compose up -d
docker-compose ps
```

### Issue: Cannot connect to Redis
**Solution**: Check Redis container
```bash
docker-compose ps redis
docker-compose logs redis
```

### Issue: Demo script hangs
**Solution**: Check API is healthy
```bash
curl http://localhost:8000/health
```

### Issue: Streamlit port already in use
**Solution**: Specify different port
```bash
streamlit run tools/streamlit_app.py --server.port 8502
```

---

##  Performance Expectations

### Demo Script
- **First run (cache miss)**: 10-15 seconds per image
- **Subsequent runs (cache hit)**: 10-20 milliseconds
- **Speedup**: 500-1000x faster with cache

### Streamlit Dashboard
- **Single image**: Instant results
- **Batch demo**: Depends on number of images/runs
- **Redis browser**: Instant key lookup

---

## 🎓 For Your Interview

### Key Points to Emphasize

#### Demo Script:
- "Automated testing of cache effectiveness"
- "Demonstrates 500-1000x speedup with caching"
- "Production-ready monitoring of Redis stats"
- "Clear visualization of performance metrics"

#### Streamlit Dashboard:
- "Interactive interface for non-technical users"
- "Real-time monitoring and debugging"
- "Production observability tool"
- "Minimal code, maximum functionality"

#### Both Tools:
- "Development utilities, not in production Docker image"
- "Separated via requirements-dev.txt"
- "Demonstrate production-ready code quality"
- "Useful for demos and presentations"

---

## 📚 Related Documentation

- [Installation Guide](../docs/INSTALLATION.md) - Setup instructions
- [Deployment Guide](../docs/deployment.md) - Production deployment
- [Architecture](../docs/architecture.html) - System design
- [Main README](../README.md) - Project overview

---

##  Summary

These tools are **development utilities** that:
-  Demonstrate API capabilities
-  Monitor production services
-  Aid in debugging and testing
-  Provide interactive interfaces
-  Follow production code quality standards

**Not included in production Docker image** - they're for local development and demos only!

---

**Run from project root directory to ensure correct paths!**


