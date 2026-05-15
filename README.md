# 🎬 Movie Recommendation System 2.0

A **hybrid movie recommendation web application** that combines content-based filtering (TF-IDF) with genre-based recommendations. Search for any movie, explore recommendations, and discover new films with detailed information from **The Movie Database (TMDB)**.

---

## ✨ Features

- 🔍 **Smart Movie Search** - Search by movie title with autocomplete suggestions
- 🎯 **Dual Recommendation Engine** - Combines TF-IDF similarity and genre-based recommendations
- 🖼️ **Rich Visual Interface** - Movie posters, backdrops, ratings, and release dates
- 🏠 **Home Feed** - Browse trending, popular, top-rated, and upcoming movies
- 📱 **Responsive Design** - Works seamlessly on desktop and mobile devices
- ⚡ **Fast Performance** - Pre-computed TF-IDF matrices for instant local recommendations

---

## 🏗️ Project Architecture

```
┌─────────────────────────────────────────────────────────┐
│         Streamlit Frontend (app.py)                      │
│  - Movie Search & Browse                                │
│  - Movie Details & Recommendations Display              │
│  - Home Feed with Categories                            │
└────────────────┬────────────────────────────────────────┘
                 │ HTTP/REST API
                 ▼
┌─────────────────────────────────────────────────────────┐
│         FastAPI Backend (main.py)                        │
│  ┌──────────────────────────────────────────────────┐   │
│  │     Recommendation Engines                       │   │
│  │  • TF-IDF (Local Dataset)                        │   │
│  │  • Genre-Based (TMDB API)                        │   │
│  └──────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────┐   │
│  │     External APIs                               │   │
│  │  • TMDB API (Movie Data, Posters, Metadata)    │   │
│  └──────────────────────────────────────────────────┘   │
└────────────────┬────────────────────────────────────────┘
                 │ File System
                 ▼
┌─────────────────────────────────────────────────────────┐
│         ML Models (models/ directory)                    │
│  • df.pkl - Movie dataset (title, metadata)            │
│  • tfidf_matrix.pkl - TF-IDF sparse matrix             │
│  • tfidf.pkl - TF-IDF vectorizer                       │
│  • indices.pkl - Title to index mapping                │
└─────────────────────────────────────────────────────────┘
```

---

## 🧠 Recommendation Algorithms

### 1. **TF-IDF Content-Based Filtering**

**What is TF-IDF?**
- **Term Frequency-Inverse Document Frequency** is a text analysis technique that measures the importance of words in documents
- TF-IDF identifies unique and relevant terms in movie descriptions/metadata
- Movies with similar text descriptions get similar TF-IDF vectors

**How It Works:**
1. Each movie's description is converted into a TF-IDF vector
2. When you search for a movie, we compute cosine similarity between:
   - The selected movie's TF-IDF vector
   - All other movies' TF-IDF vectors
3. Movies with highest cosine similarity scores are recommended

**Mathematical Formula:**
```
TF-IDF(term) = Term Frequency × Inverse Document Frequency
Similarity = (Vector A · Vector B) / (||A|| × ||B||)  [Cosine Similarity]
```

**Advantages:**
- ✅ Fast (pre-computed matrices)
- ✅ No user history needed
- ✅ Finds movies with similar plots/themes
- ✅ Works with any movie in the dataset

**Limitations:**
- ❌ Only considers textual similarity
- ❌ Can miss movies with different descriptions but similar themes
- ❌ Doesn't account for genres or user preferences

---

### 2. **Genre-Based Collaborative Discovery**

**How It Works:**
1. Fetches the primary genre of the selected movie from TMDB
2. Queries TMDB's Discover API for popular movies in that genre
3. Filters out the original movie from results
4. Sorts by popularity and returns top recommendations

**Data Flow:**
```
Selected Movie → TMDB API → Get Genres → Discover API → 
Popular Movies with Same Genre → Filter & Return
```

**Advantages:**
- ✅ Explores related categories
- ✅ Discovers trending movies in similar genres
- ✅ Uses real-time TMDB data
- ✅ Complements TF-IDF recommendations

**Limitations:**
- ❌ Limited to one genre per movie
- ❌ Relies on external API (slower)
- ❌ Popular != Similar quality or style

---



### 3. **Hybrid Recommendation Strategy**

The system combines both methods to provide comprehensive recommendations:

```
User selects Movie A
        ↓
┌───────┴───────┐
│               │
▼               ▼
TF-IDF        Genre-Based
(12 Results)  (12 Results)
│               │
└───────┬───────┘
        ▼
    Display Both
  (Labeled sections)
```

---

## 🔧 Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | Streamlit | Web UI, Real-time interactions |
| **Backend** | FastAPI | REST API, Async operations |
| **ML/Data** | scikit-learn, pandas, numpy | TF-IDF, data processing |
| **External API** | TMDB API | Movie metadata, images, ratings |
| **Server** | Uvicorn | ASGI server for FastAPI |
| **Environment** | Python 3.12 | Runtime & virtual environment |

---

## 📋 API Endpoints

### Core Endpoints

| Endpoint | Method | Parameters | Description |
|----------|--------|-----------|-------------|
| `/health` | GET | - | Health check |
| `/home` | GET | `category`, `limit` | Browse home feed |
| `/tmdb/search` | GET | `query`, `page` | Search TMDB by keyword |
| `/movie/id/{tmdb_id}` | GET | `tmdb_id` | Get movie details |
| `/recommend/tfidf` | GET | `title`, `top_n` | TF-IDF recommendations only |
| `/recommend/genre` | GET | `tmdb_id`, `limit` | Genre-based recommendations |
| `/movie/search` | GET | `query`, `tfidf_top_n`, `genre_limit` | Bundle: Details + Both recommendations |

### Request/Response Examples

**Search Bundle (most used):**
```bash
GET /movie/search?query=Inception&tfidf_top_n=12&genre_limit=12
```

## 🚀 Installation & Setup

### Prerequisites
- **Python 3.12+**
- **pip** (Python package manager)
- **TMDB API Key** (free from [TMDB](https://www.themoviedb.org/settings/api))

### Step 1: Clone Repository
```bash
git clone https://github.com/yourusername/web-movie-recommendation.git
cd web-movie-recommendation
```

### Step 2: Create Virtual Environment
```bash
# Windows
python -m venv myenv
myenv\Scripts\activate

# macOS/Linux
python3 -m venv myenv
source myenv/bin/activate
```

### Step 3: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 4: Setup Environment Variables
Create a `.env` file in the project root:
```env
TMDB_api_key=your_api_key_here
```

Get your free API key:
1. Visit [TMDB API](https://www.themoviedb.org/settings/api)
2. Sign up for a free account
3. Request an API key
4. Copy your API key to `.env`

### Step 5: Run the Application

**Start Backend (FastAPI):**
```bash
# Development with auto-reload
uvicorn main:app --reload

# Production
uvicorn main:app --host 0.0.0.0 --port 8000
```

**Start Frontend (Streamlit) - In a new terminal:**
```bash
streamlit run app.py
```

The application will be available at:
- **Frontend:** `http://localhost:8501`
- **Backend API:** `http://localhost:8000`
- **API Docs:** `http://localhost:8000/docs`


## 🎯 How to Use

### 1. **Browse Home Feed**
- Open the app
- Select a category (Trending, Popular, Top Rated, etc.)
- Adjust grid columns as needed
- Click on any poster to view details

### 2. **Search for Movies**
- Type in the search box (minimum 2 characters)
- See autocomplete suggestions
- View matching results in grid
- Click to open details and recommendations

### 3. **View Recommendations**
- On the details page, scroll down to see:
  - **Similar Movies (TF-IDF)** - Based on plot similarity
  - **More Like This (Genre)** - Based on genre

### 4. **Navigate**
- Use "Back to Home" button to return
- Click "Open" on any poster to view details
- Use query parameters for direct linking:
  ```
  ?view=details&id=27205  # Open Inception details
  ```

---

## 🔐 Security & Best Practices

- ✅ **CORS Enabled** - Allows cross-origin requests for integration
- ✅ **Error Handling** - Graceful fallbacks when APIs fail
- ✅ **Input Validation** - FastAPI validates all parameters
- ✅ **Timeout Protection** - 20-second timeout on external API calls
- ✅ **Rate Limiting** - Cache layer (30s) on autocomplete queries
- ⚠️ **API Key** - Keep `.env` file private, never commit to git

---

## 📊 Performance Optimization

| Component | Optimization |
|-----------|--------------|
| TF-IDF Recommendations | Pre-computed sparse matrices (instant) |
| TMDB Search | API call with cache |
| Genre Recommendations | Lazy-loaded only when needed |
| Images | CDN-hosted (image.tmdb.org) |
| Frontend | Streamlit built-in caching |

**Typical Response Times:**
- Search suggestions: ~100ms (cached)
- Movie details: ~500ms (TMDB API)
- TF-IDF recommendations: ~50ms (local)
- Genre recommendations: ~800ms (TMDB API)

---

## 🌐 Deployment

### Deploy Backend to Render/Heroku
```bash
# Create Procfile
echo "web: uvicorn main:app --host 0.0.0.0 --port $PORT" > Procfile

# Push to Render or Heroku
git push heroku main
```

### Deploy Frontend to Streamlit Cloud
1. Push code to GitHub
2. Connect repository to [Streamlit Cloud](https://streamlit.io/cloud)
3. Set `TMDB_api_key` secret in deployment settings
4. Deploy!

**Live Demo:** https://movie-rec-466x.onrender.com

---

## 🐛 Troubleshooting

### Issue: "TMDB_api_key is not set"
**Solution:** 
- Create `.env` file with `TMDB_api_key=your_key`
- Restart both servers

### Issue: "Title not found in local dataset"
**Reason:** Movie exists in TMDB but not in training dataset
**Solution:** 
- TF-IDF only works for movies in the pre-trained dataset
- Genre recommendations still work (fallback)

### Issue: Slow responses
**Solution:**
- Clear Streamlit cache: Menu → Clear cache
- Check TMDB API rate limits
- Verify internet connection

---

## 📈 Future Enhancements

- [ ] 🤖 Collaborative Filtering (user ratings/preferences)
- [ ] 📊 Machine Learning Model (neural networks)
- [ ] 👤 User Authentication & Personalized Recommendations
- [ ] ❤️ Watchlist & Rating System
- [ ] 🎬 Actor/Director Recommendations
- [ ] 📱 Mobile App (React Native)
- [ ] 🔔 Notification System (new movie in followed genres)
- [ ] 📊 Analytics Dashboard

---

## 📝 License

This project is open source. Please include proper attribution if used.

---

## 🤝 Contributing

Contributions are welcome! Here's how:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📧 Contact & Support

- **Issues:** Open an issue on GitHub
- **Questions:** Check the [FAQ](#) or contact the maintainer
- **Feature Requests:** Submit as a GitHub issue with `[FEATURE]` tag

---

## 🙏 Acknowledgments

- **TMDB** - Movie data, posters, and metadata
- **Streamlit** - Interactive web framework
- **FastAPI** - Modern, fast web framework
- **scikit-learn** - Machine learning library

---


