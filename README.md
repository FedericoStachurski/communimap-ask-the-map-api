# CommuniMap Ask the Map API

A lightweight Python API for querying CommuniMap multimodal embeddings (text + image) using natural language.

This service powers the **"Ask the Map"** feature by:

* accepting a precomputed embedding bundle (`.zip`)
* running multimodal similarity search (text + image)
* returning structured results (JSON) for frontend apps (e.g. Shiny)

---

## Features

* Multimodal search using **text + image embeddings**
* Supports **CLIP** and **SigLIP** models
* FAISS-based fast similarity search
* Accepts zipped embedding bundles
* Returns clean JSON for easy frontend integration

---

## Expected Embedding Bundle

Upload a `.zip` file containing:

```
<prefix>_text.npy
<prefix>_image.npy
<prefix>_meta.json
```

Where:

* `*_text.npy` → text embeddings
* `*_image.npy` → image embeddings
* `*_meta.json` → metadata (id, text, lat, lon, image URL)

---

## Installation

Create a virtual environment and install dependencies:

```bash
python3 -m venv venv
source venv/bin/activate

pip install -r requirements.txt
```

---

## Running the API

Start the server with:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

Then open:

```
http://localhost:8000/docs
```

to access the interactive API documentation.

---

## API Endpoints

### Health check

```
GET /health
```

Returns:

```json
{ "status": "ok" }
```

---

### Query embedding bundle

```
POST /query-bundle
```

#### Form data:

* `file` → embedding bundle (`.zip`)
* `query` → search string
* `k` → number of results (default: 50)
* `threshold` → similarity threshold (default: 0.3)

#### Example response:

```json
[
  {
    "id": "123",
    "score": 0.82,
    "score_text": 0.75,
    "score_img": 0.88,
    "lat": 55.87,
    "lon": -4.28,
    "text": "Flooding near the road",
    "image": "https://..."
  }
]
```

---

## How it works

1. Extract `.zip` bundle
2. Load embeddings + metadata
3. Normalize vectors
4. Build FAISS indices (text + image)
5. Encode query using the same VLM
6. Perform similarity search
7. Fuse scores:

   ```
   score = w_text * score_text + w_img * score_img
   ```
8. Return filtered results as JSON

---

## Integration (Shiny / Web Apps)

This API is designed to be called from external apps.

Example flow:

```
Frontend (Shiny / JS)
        ↓
POST /query-bundle
        ↓
JSON results
        ↓
Render map + table
```

---

## Project Structure (suggested)

```
.
├── main.py
├── requirements.txt
├── README.md
└── .gitignore
```

---

## Notes

* Requires FAISS (`faiss-cpu` or `faiss-gpu`)
* Make sure the query model matches the embedding model
* Large embedding bundles may impact performance
* GPU recommended for large-scale use

---

## Future Improvements

* Persistent FAISS index (avoid rebuilding each query)
* Streaming results
* Batch queries
* Caching frequent queries
* Hosted embedding storage

---

## Author

CommuniMap / GALLANT Project
University of Glasgow

---

## License

MIT License (or specify your preferred license)
