# AI Content Generator API

A production-ready FastAPI project that generates text using the Groq API.

## Project Structure

```text
ai-content-generator-api/
├── app/
│   ├── main.py
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── generate.py
│   │   └── health.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── groq_service.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── content.py
│   └── utils/
│       ├── __init__.py
│       ├── config.py
│       ├── exceptions.py
│       ├── logging_config.py
│       ├── middleware.py
│       └── rate_limiter.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_generate.py
│   ├── test_health.py
│   └── test_rate_limiter.py
├── .env
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```

## Features

- FastAPI application with automatic Swagger documentation.
- `POST /generate` endpoint for text generation.
- `GET /health` endpoint for health checks.
- Groq SDK integration isolated in a service layer.
- Pydantic request and response validation.
- Empty prompt validation.
- Centralized exception handling with meaningful error messages.
- Environment-based configuration through `.env`.
- Logging for requests and service failures.
- Response timing metric through the `X-Process-Time-MS` response header.
- In-memory rate limiting by client IP and path.
- Pytest unit tests with mocked Groq interactions.

## Setup

1. Create and activate a virtual environment:

```bash
python -m venv .venv
.venv\Scripts\activate
```

On macOS or Linux:

```bash
python -m venv .venv
source .venv/bin/activate
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Configure environment variables:

```bash
copy .env.example .env
```

On macOS or Linux:

```bash
cp .env.example .env
```

Then edit `.env` and set:

```env
GROQ_API_KEY=your-real-groq-api-key
```

## Run The Application

```bash
uvicorn app.main:app --reload
```

The API will run at:

```text
http://127.0.0.1:8000
```

Swagger documentation is available at:

```text
http://127.0.0.1:8000/docs
```

ReDoc documentation is available at:

```text
http://127.0.0.1:8000/redoc
```

## API Endpoints

### Health Check

Request:

```http
GET /health HTTP/1.1
Host: 127.0.0.1:8000
```

Response:

```json
{
  "status": "healthy"
}
```

### Generate Content

Request:

```http
POST /generate HTTP/1.1
Host: 127.0.0.1:8000
Content-Type: application/json

{
  "prompt": "Write a concise product description for an AI note-taking app."
}
```

Success response:

```json
{
  "generated_text": "Your generated content will appear here.",
  "status": "success"
}
```

Validation error response:

```json
{
  "status": "error",
  "message": "body.prompt: Value error, Prompt cannot be empty."
}
```

Rate limit response:

```json
{
  "status": "error",
  "message": "Rate limit exceeded. Please try again later."
}
```

## Test With Postman

1. Start the server with `uvicorn app.main:app --reload`.
2. Create a `GET` request to `http://127.0.0.1:8000/health`.
3. Click `Send` and confirm the response body is `{"status": "healthy"}`.
4. Create a `POST` request to `http://127.0.0.1:8000/generate`.
5. In the `Headers` tab, set `Content-Type` to `application/json`.
6. In the `Body` tab, choose `raw` and `JSON`.
7. Use this request body:

```json
{
  "prompt": "Write a short LinkedIn post about responsible AI adoption."
}
```

8. Click `Send`.
9. Confirm the response includes `generated_text`, `status`, and an `X-Process-Time-MS` response header.

## Run Tests

```bash
pytest
```

The tests use a fake Groq service, so they do not require a real API key or network access.

## Architecture And Workflow

1. The client sends a request to a FastAPI route.
2. Pydantic validates the request body and rejects blank prompts.
3. Rate limiting middleware checks whether the client has exceeded the request limit.
4. The route delegates generation to `GroqContentService`.
5. `GroqContentService` calls the Groq SDK and returns generated text.
6. The route returns a typed Pydantic response.
7. Timing middleware adds `X-Process-Time-MS` and logs request duration.
8. Central exception handlers convert validation, rate limit, Groq, and unexpected errors into consistent JSON responses.

## Production Notes

- Replace the in-memory rate limiter with Redis or another shared store when running multiple API instances.
- Keep `.env` out of source control and inject secrets through your deployment platform.
- Pin dependency versions more strictly for regulated production environments.
- Add authentication if this endpoint is exposed outside a trusted environment.
