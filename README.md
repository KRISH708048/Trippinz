# TripPins

A travel discovery platform with semantic search capabilities powered by ChromaDB vector embeddings and Neo4j graph database.

## 🌟 Features

- 🏨 **Multi-City Hotel Discovery** - Delhi, Jaipur, and Rishikesh (~5,000 hotels)
- 🔍 **Semantic Search** - Natural language hotel search using vector embeddings
- 🗺️ **Interactive Map** - Explore hotels and POIs visually
- 💬 **AI Chat Assistant** - Conversational hotel recommendations
- 📊 **Graph Database** - Complex relationship queries with Neo4j
- 🎯 **POI Integration** - Hotels linked to nearby Points of Interest

## 🚀 Quick Start

### Option 1: Complete Setup (Recommended)

```bash
# 1. Backend setup
cd backend
npm install
./setup-chromadb.sh
./setup-neo4j.sh  # Follow prompts for Neo4j Desktop

# 2. Start ChromaDB (in separate terminal)
npm run chromadb:start

# 3. Ingest all city data (in main terminal)
npm run setup:all  # ~10-15 minutes

# 4. Start backend server
npm run dev

# 5. Frontend setup (in another terminal)
cd frontend
npm install
npm run dev
```

**Access the app:**
- Frontend: http://localhost:5173
- Backend API: http://localhost:3001
- Neo4j Browser: http://localhost:7474

### Option 2: Quick Setup with Single City

```bash
# Backend
cd backend
npm install
./setup-chromadb.sh
./setup-neo4j.sh

# Start ChromaDB (separate terminal)
npm run chromadb:start

# Setup Delhi only (~3-5 minutes)
npm run neo4j:setup
npm run chromadb:sync

# Start backend
npm run dev

# Frontend (another terminal)
cd frontend
npm install
npm run dev
```

## 📋 Prerequisites

- **Node.js** v16 or higher
- **npm** (comes with Node.js)
- **Python 3.8+** (required for ChromaDB)
- **Neo4j Desktop** (recommended) - [Download here](https://neo4j.com/download/)
  - Alternative: Docker with Neo4j image

## 📂 Project Structure

```
TripPins/
├── frontend/                 # React + Vite frontend
│   ├── src/
│   │   ├── components/       # React components
│   │   │   ├── ChatSidebar.jsx       # AI chat interface
│   │   │   ├── ContentPanel.jsx      # Hotel list panel
│   │   │   └── map/                  # Map components
│   │   ├── context/          # React context providers
│   │   └── App.jsx           # Main app component
│   └── package.json
│
├── backend/                  # Node.js + Express backend
│   ├── scripts/
│   │   ├── ingest-hotels-to-neo4j.js  # Data ingestion script
│   │   ├── sync-hotels-to-chromadb.js # ChromaDB sync script
│   │   └── seed-poi-taxonomy.js       # POI categories seeding
│   ├── services/
│   │   ├── neo4j.js          # Neo4j service
│   │   └── chromadb.js       # ChromaDB service
│   ├── hotel_data/           # City hotel data files
│   │   ├── delhi.txt         # ~1,700 hotels
│   │   ├── Jaipur.txt        # ~1,200 hotels
│   │   └── Rishikesh.txt     # ~2,000 hotels
│   ├── SETUP.md              # Detailed setup guide
│   ├── DATA_INGESTION_GUIDE.md  # Data ingestion guide
│   ├── QUICK_SETUP.md        # Quick reference
│   └── package.json
│
└── README.md                 # This file
```

## 🎯 Available Cities

- **Delhi** - Capital city with ~1,700 hotels
- **Jaipur** - Pink City with ~1,200 hotels  
- **Rishikesh** - Yoga capital with ~2,000 hotels

Total: **~5,000 hotels** with POIs and amenities

## 📦 NPM Scripts

### Backend Commands

```bash
# Data ingestion
npm run setup:all              # Complete setup (wipe + seed + ingest all + sync)
npm run neo4j:setup-all        # Neo4j setup for all cities
npm run neo4j:ingest-all       # Ingest all cities
npm run neo4j:ingest-delhi     # Ingest Delhi only
npm run neo4j:ingest-jaipur    # Ingest Jaipur only
npm run neo4j:ingest-rishikesh # Ingest Rishikesh only
npm run chromadb:sync          # Sync Neo4j hotels to ChromaDB

# Database management
npm run neo4j:wipe             # Clear all Neo4j data
npm run neo4j:seed-taxonomy    # Seed POI categories

# Development
npm run dev                    # Start backend server (with auto-reload)
npm run chromadb:start         # Start ChromaDB server

# Examples
npm run example:neo4j          # Run Neo4j examples
npm run example                # Run ChromaDB examples
```

### Frontend Commands

```bash
npm run dev                    # Start dev server (Vite)
npm run build                  # Build for production
npm run preview                # Preview production build
```

## 🏗️ Architecture

### Tech Stack

**Frontend:**
- React 18 with Vite
- Mapbox GL JS for interactive maps
- Context API for state management
- IndexedDB caching (via idb library)

**Backend:**
- Node.js + Express.js
- Neo4j graph database (hotels, POIs, relationships)
- ChromaDB vector database (semantic search) - **MANDATORY**
- CopilotKit for AI chat integration
- SearchOrchestrator for ChromaDB-first search

### Search Architecture: ChromaDB-First

All hotel searches use a **ChromaDB-first** approach for optimal results:

```
User Query → CopilotKit (NER) → SearchOrchestrator
                                    ↓
                            ChromaDB (Semantic Search)
                                    ↓
                            Neo4j (Enrich + Filter)
                                    ↓
                            Merge & Rank (60% semantic + 40% entity)
                                    ↓
                            Final Results
```

**Benefits:**
- Natural language queries ("peaceful spa retreat") work perfectly
- Synonyms matched automatically ("spa" finds "wellness center")
- Entity extraction extracts city, amenities, POI, rating
- Combined scoring provides best-of-both-worlds results

### Database Schema

**Neo4j Graph:**
```
(City)-[:HAS_AREA]->(Area)-[:HAS_HOTEL]->(Hotel)-[:NEAR_POI]->(POI)-[:OF_CATEGORY]->(POICategory)
```

**Key Features:**
- Hotels store amenities as list properties (optimized)
- Top 10 POIs per hotel marked as `is_embedded` for quick access
- City highlights auto-detected and flagged
- Fulltext search index on POI names

See [backend/NEO4J_SCHEMA.md](./backend/NEO4J_SCHEMA.md) for details.
See [docs/CHROMADB_FIRST_SEARCH.md](./docs/CHROMADB_FIRST_SEARCH.md) for search architecture.

## 🔧 Configuration

### Backend Environment Variables

Create `backend/.env`:

```env
# Server
PORT=3001
NODE_ENV=development

# Neo4j
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your-password

# ChromaDB
CHROMADB_HOST=localhost
CHROMADB_PORT=8000

# Azure OpenAI (for AI chat - optional)
AZURE_OPENAI_API_KEY=your-key
AZURE_OPENAI_ENDPOINT=your-endpoint
AZURE_OPENAI_DEPLOYMENT=your-deployment
```

### Frontend Environment Variables

Create `frontend/.env`:

```env
VITE_API_URL=http://localhost:3001
VITE_MAPBOX_TOKEN=your-mapbox-token
```

## 🧪 Testing & Verification

### Health Checks

```bash
# Overall health
curl http://localhost:3001/api/health

# Neo4j status
curl http://localhost:3001/api/neo4j/status

# ChromaDB status  
curl http://localhost:3001/api/chromadb/status
```

### Sample Neo4j Queries

Open Neo4j Browser (http://localhost:7474):

```cypher
// Count all hotels
MATCH (h:Hotel) RETURN count(h)

// Hotels with spa in Jaipur
MATCH (h:Hotel) 
WHERE h.city = 'Jaipur' AND 'Spa' IN h.amenities
RETURN h.name, h.star_rating LIMIT 10;

// Hotels near metro stations
MATCH (h:Hotel)-[r:NEAR_POI]->(p:POI {category_id: 'METRO_STATION'})
WHERE r.distance_km < 2
RETURN h.name, h.city, p.name, r.distance_km;

// City highlights
MATCH (p:POI) WHERE p.is_city_highlight = true 
RETURN p.name, p.category_name;
```

### Test Semantic Search

```bash
curl -X POST http://localhost:3001/api/search \
  -H "Content-Type: application/json" \
  -d '{"query": "luxury spa hotel near airport", "limit": 5}'
```

## 📚 Documentation

- **[QUICK_SETUP.md](./backend/QUICK_SETUP.md)** - Get started in 15 minutes
- **[SETUP.md](./backend/SETUP.md)** - Complete setup guide with troubleshooting
- **[DATA_INGESTION_GUIDE.md](./backend/DATA_INGESTION_GUIDE.md)** - Detailed ingestion guide
- **[NEO4J_SCHEMA.md](./backend/NEO4J_SCHEMA.md)** - Database schema documentation
- **[CHROMADB_FIRST_SEARCH.md](./docs/CHROMADB_FIRST_SEARCH.md)** - Search architecture guide
- **[VIEWPORT_CACHE_SERVICE.md](./docs/VIEWPORT_CACHE_SERVICE.md)** - Frontend caching documentation
- **[CACHING_ARCHITECTURE_GUIDE.md](./docs/CACHING_ARCHITECTURE_GUIDE.md)** - Caching learning guide

## 🐛 Troubleshooting

### Common Issues

**ChromaDB connection failed:**
```bash
# Make sure ChromaDB server is running
npm run chromadb:start
```

**Neo4j connection failed:**
1. Open Neo4j Desktop
2. Ensure database is Active (green status)
3. Verify password in `.env` matches Neo4j Desktop

**Port already in use:**
```bash
# Change port in backend/.env
PORT=3002
```

**Data ingestion stuck:**
```bash
# Reduce batch size for lower memory systems
node scripts/ingest-hotels-to-neo4j.js --all --batch 25
```

**Reset everything:**
```bash
# Backend
cd backend
npm run neo4j:wipe -- --confirm
rm -rf chroma_db && mkdir chroma_db
npm run setup:all
```

See [backend/SETUP.md](./backend/SETUP.md) for detailed troubleshooting.

## 🚦 Development Workflow

1. **Start services** (3 terminals):
   ```bash
   # Terminal 1: ChromaDB
   cd backend && npm run chromadb:start
   
   # Terminal 2: Backend
   cd backend && npm run dev
   
   # Terminal 3: Frontend
   cd frontend && npm run dev
   ```

2. **Make changes** - Files auto-reload

3. **Test changes**:
   - Frontend: http://localhost:5173
   - Backend API: http://localhost:3001/api/health
   - Neo4j Browser: http://localhost:7474

4. **Query Neo4j** directly via Browser to test data

5. **Test semantic search** via chat interface or API

## 🎓 Learning Resources

- [Neo4j Cypher Manual](https://neo4j.com/docs/cypher-manual/current/)
- [ChromaDB Documentation](https://docs.trychroma.com/)
- [CopilotKit Docs](https://docs.copilotkit.ai/)
- [Mapbox GL JS](https://docs.mapbox.com/mapbox-gl-js/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

ISC

---

**Built with ❤️ for travelers**
