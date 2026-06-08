# FastAPI Interview Questions

## Questions

1. Why is FastAPI a good fit for AI backends?
Answer: It combines async support, request validation, OpenAPI generation, and clean dependency patterns, which fits I/O-heavy AI services well.

2. What is dependency injection in FastAPI?
Answer: It is the mechanism for declaring reusable dependencies such as DB sessions, auth checks, config, and clients via `Depends()`.

3. Middleware vs dependencies?
Answer: Middleware wraps the full HTTP lifecycle globally, while dependencies inject resources or checks at route scope.

4. Why use `yield` in a dependency?
Answer: It supports setup and teardown, such as opening a DB session and ensuring cleanup after the request.

5. How would you stream AI responses in FastAPI?
Answer: Use streaming responses backed by generators or async generators, and choose SSE or WebSockets based on client interaction needs.

## Related Topics

- [fastapi-advanced.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/02-FastAPI/fastapi-advanced.md)
- [ai-streaming-chat-api.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/12-Projects/ai-streaming-chat-api.md)
