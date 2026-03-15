# Query Craft - Backend

This backend service powers the Query Craft application, enabling natural language to SQL translation and database querying.

## Table of Contents

- [About](#about)
- [Features](#features)
- [Technologies Used](#technologies-used)
- [Installation](#installation)
- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Contributing](#contributing)
- [License](#license)

## About

Query Craft aims to democratize data access by allowing users to query databases using natural language. This backend handles the core logic of understanding user queries, generating SQL, executing it against the database, and returning the results.

## Features

*   **Natural Language to SQL:** Translates user questions into executable SQL queries.
*   **Database Connectivity:** Supports various database systems (e.g., PostgreSQL, MySQL, SQLite).
*   **Query Execution:** Executes generated SQL queries safely and efficiently.
*   **Result Formatting:** Returns query results in a structured format.
*   **Scalability:** Designed to handle a growing number of requests and complexity.

## Technologies Used

*   **Language:** Python (or Node.js, Go, etc., depending on the project's implementation)
*   **Framework:** Flask / Django (Python), Express.js (Node.js), Gin (Go)
*   **Database:** PostgreSQL, MySQL, SQLite, etc.
*   **LLM Integration:** OpenAI API, Hugging Face, or other NLP models.
*   **Containerization (Optional):** Docker

## Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/RAIF-KARANI/Query_Craft--Backend.git
    cd Query_Craft--Backend
    ```

2.  **Set up a virtual environment (Python):**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Environment Variables:**
    Create a `.env` file in the root directory and populate it with your database credentials and API keys:
    ```dotenv
    DATABASE_URL=postgresql://user:password@host:port/database
    OPENAI_API_KEY=your_openai_api_key
    # Other necessary environment variables
    ```

5.  **Database Setup:**
    *   If using PostgreSQL/MySQL, ensure your database server is running and accessible.
    *   Run database migrations (if applicable):
        ```bash
        # Example for Django
        python manage.py migrate
        ```

## Usage

1.  **Run the development server:**
    ```bash
    python app.py  # Or the command to start your specific backend server
    ```
    The server will typically run on `http://localhost:5000` (or another specified port).

2.  **Integrate with the Frontend:**
    The frontend application will send requests to this backend API to process user queries.

## API Endpoints

*   `POST /query`: Accepts a natural language query and database details, returns SQL query and results.
    *   **Request Body:**
        ```json
        {
          "natural_language_query": "Show me all users from California",
          "database_connection": {
            "type": "postgresql",
            "host": "localhost",
            "port": 5432,
            "user": "db_user",
            "password": "db_password",
            "database": "db_name"
          }
        }
        ```
    *   **Response Body:**
        ```json
        {
          "sql_query": "SELECT * FROM users WHERE state = 'California';",
          "results": [
            {"id": 1, "name": "Alice", "state": "California"},
            {"id": 5, "name": "Bob", "state": "California"}
          ],
          "explanation": "This query selects all columns from the 'users' table where the 'state' column is 'California'."
        }
        ```
*   *(Add other relevant API endpoints here)*

## Contributing

Contributions are welcome! Please follow these steps:

1.  Fork the repository.
2.  Create a new branch (`git checkout -b feature/your-feature-name`).
3.  Make your changes and commit them (`git commit -am 'Add some feature'`).
4.  Push to the branch (`git push origin feature/your-feature-name`).
5.  Open a Pull Request.

Please ensure your code follows the project's coding standards and includes tests where appropriate.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
