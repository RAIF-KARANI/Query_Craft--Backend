# QueryCraft Backend

## Overview

QueryCraft Backend is a Node.js and Express-based REST API that powers the QueryCraft application. It acts as an AI-driven database querying assistant, enabling users to connect to various external database engines (SQL, NoSQL, Graph) or upload raw data files (CSV, JSON, SQL) to generate, execute, and analyze database queries using natural language. 

The backend handles authentication, conversation memory, dynamic database connections, and intelligent query routing to multiple Large Language Model (LLM) providers, including Google GenAI (Gemini), OpenRouter, and local local models.

> **Note**: This repository contains the backend code. The frontend application can be found here: [QueryCraft Frontend](../querycraft-frontend).

## Key Features

* **Multi-Model AI Integration**: Dynamically routes prompts to Google GenAI, OpenRouter, or local models based on query type, latency, and cost preferences.
* **Broad Database Support**: Executes generated queries against PostgreSQL, MySQL, MariaDB, MongoDB, Neo4j, and SQLite.
* **File-to-Database Import**: Upload CSV, JSON, or SQL files. The system automatically converts them into temporary in-memory/on-disk SQLite databases for immediate AI querying.
* **Conversation Memory**: Maintains chat context across sessions and automatically summarizes older interactions in the background to optimize LLM context windows.
* **Robust Output Parsing**: Extracts executable code blocks (SQL, Cypher, GraphQL, MongoDB shell commands) from raw LLM text outputs using resilient heuristics.
* **Secure & Rate-Limited**: Implements JWT-based authentication, password hashing (bcrypt), request rate limiting (express-rate-limit), and HTTP header security (Helmet).

## Architecture / System Design

The system follows a scalable MVC-like REST API architecture written in JavaScript/Node.js, chosen for its non-blocking I/O efficiency when handling multiple concurrent database streams and external LLM API requests.

* **App State Database**: MongoDB (via Mongoose) stores `Users`, `Chats`, and `Queries`.
* **API Routing**: Express router directs traffic to specific domains (`auth`, `chat`, `db`, `query`).
* **LLM Processing Unit**: `utils/llm.js` and `utils/responseParser.js` manage prompt generation, API communication with retry/exponential backoff logic, and text extraction.
* **Dynamic Database Controller**: `controllers/dbController.js` manages ad-hoc connections to user-specified external databases or temporary SQLite databases generated from uploaded files.

## Tech Stack

* **Runtime**: Node.js
* **Framework**: Express.js
* **Primary Database**: MongoDB (Mongoose)
* **Database Drivers**: `pg` (PostgreSQL), `mysql2` (MySQL/MariaDB), `mongodb` (MongoDB), `neo4j-driver` (Neo4j), `better-sqlite3` (SQLite)
* **AI/LLM SDKs**: `@google/genai`, `axios` (for OpenRouter and local LLM endpoints)
* **Security & Auth**: `jsonwebtoken`, `bcryptjs`, `helmet`, `cors`, `express-rate-limit`
* **File Handling**: `multer`, `csv-parser`, `tmp`, `jsonfile`

## Project Structure

```text
querycraft-backend/
├── Dockerfile                  # Containerization configuration
├── index.js                    # Application entry point & server setup
├── package.json                # Dependencies and NPM scripts
├── controllers/
│   └── dbController.js         # Handles dynamic DB connections and file imports
├── middleware/
│   └── auth.js                 # JWT verification middleware
├── models/
│   ├── Chat.js                 # Mongoose schema for user chat sessions
│   ├── Query.js                # Mongoose schema for individual AI queries
│   └── User.js                 # Mongoose schema for users and password hashing
├── routes/
│   ├── auth.js                 # Authentication endpoints (signup, login, me)
│   ├── chat.js                 # Chat session management endpoints
│   ├── db.js                   # File upload and query execution endpoints
│   └── query.js                # LLM interaction and query generation endpoints
└── utils/
    ├── conversationMemory.js   # Background LLM summarization of long chats
    ├── llm.js                  # LLM provider routing and execution logic
    └── responseParser.js       # Extracts executable code blocks from LLM output
```

## Installation

Run the following commands to clone the repository, navigate to the directory, and install the necessary dependencies.

```bash
# Clone the repository
git clone https://github.com/Rifaque/querycraft-backend.git

# Navigate into the project directory
cd querycraft-backend

# Install NPM dependencies
npm install
```

## Configuration

Create a `.env` file in the root directory. Copy and paste the following template, replacing the placeholder values with your actual credentials:

```bash
cat <<EOT >> .env
# Server
PORT=5001
NODE_ENV=development

# Application Database (MongoDB) - Required for user/chat state
MONGO_URI=mongodb://localhost:27017/querycraft

# Security
JWT_SECRET=your_super_secret_jwt_key_here

# AI / LLM Providers
GENAI_KEY=your_google_genai_api_key
OPENROUTER_KEY=your_openrouter_api_key
LLM_ENDPOINT=http://127.0.0.1:11434/api/generate
DEFAULT_MODEL=mistral:7b-instruct

# OpenRouter Metadata (Optional)
OPENROUTER_SITE_URL=https://localhost
OPENROUTER_APP_NAME=QueryCraft

# Summarization Settings
SUMMARY_MODEL=mistral:7b-instruct
SUMMARY_OLDEST_COUNT=15
SUMMARY_MAX_TOKENS=400
EOT
```

## Usage

### Running Locally

Development Mode (uses nodemon for auto-reloading):

```bash
npm run dev
```

Production Mode:

```bash
npm start
```

### Running via Docker

You can also build and run the application using the provided Dockerfile:

```bash
# Build the Docker image
docker build -t querycraft-backend .

# Run the container (maps port 5001 and passes the .env file)
docker run -p 5001:5001 --env-file .env querycraft-backend
```

## API Documentation

### Authentication (/api/auth)

* **POST /api/auth/signup**: Create a new user account. Requires name, email, password.
* **POST /api/auth/login**: Authenticate and receive a JWT. Requires email, password.
* **GET /api/auth/me**: Get current authenticated user profile (Requires Bearer Token).

### Chats (/api/chat) - Requires Bearer Token

* **GET /api/chat/**: List all chat sessions for the authenticated user.
* **POST /api/chat/**: Create a new chat session. Accepts title.
* **GET /api/chat/:id**: Get a specific chat and its associated queries.
* **DELETE /api/chat/:id**: Delete a specific chat session.
* **DELETE /api/chat/**: Delete all chat sessions for the user.

### Database Operations (/api/db)

* **POST /api/db/upload**: Upload a CSV, JSON, SQLite, or SQL file. Expects multipart/form-data with field file. Returns file metadata and internal id.
* **POST /api/db/execute**: Execute a query against an uploaded file or a provided connection string.

Body Example:

```json
{
  "sourceType": "connection",
  "connectionString": "postgres://user:pass@localhost:5432/mydb",
  "query": "SELECT * FROM users;"
}
```

### Query Generation (/api/query)

* **POST /api/query/**: Main endpoint to generate AI database queries based on natural language. Stores history in the database. (Requires Bearer Token).
  * Body Options: chatId, prompt, model, max_tokens, temperature.
* **POST /api/query/demo**: Rate-limited demo endpoint that does not require authentication or maintain chat history.

## Security Notes

* **Execution Isolation**: The backend executes queries strictly using the provided database drivers. No sanitization is applied to AI-generated queries before execution. Users are strongly advised to use read-only database credentials when connecting external databases to prevent destructive operations.
* **File Uploads**: Handled via multer and temporarily stored on disk. Limits are enforced (100MB), and uploaded CSV/JSON data is converted to isolated SQLite databases.
* **Rate Limiting**: Enforced via express-rate-limit globally and specifically on the unauthenticated /demo endpoint to prevent LLM abuse.

## Limitations

* Arbitrary execution of AI-generated queries on user databases carries inherent risks of data modification or deletion if write-access credentials are provided.
* The Neo4j bolt connection fallback logic relies on specific error messaging, which may break across different Neo4j driver versions.
* In-memory parsing of extremely large uploaded JSON files may cause Node.js heap memory exhaustion.

## Future Improvements

* Implement strict Read-Only enforcement options for standard SQL connections (e.g., wrapping transactions and rolling them back automatically).
* Add automated dataset schema extraction so the LLM has structural awareness before generating queries.
* Migrate to TypeScript for stronger type guarantees across database driver interactions.

## License

This project is licensed under the MIT License.
