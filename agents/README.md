# Backend Development Agents - Quick Reference

This directory contains 9 specialized agents for backend development following MAANG-level engineering standards.

## 📋 Available Agents

### 🎨 System Design
1. **system-design-architect** (Opus)
   - Comprehensive system design and architecture
   - Extracts requirements, designs HLD/LLD
   - Infrastructure decisions (SQL vs NoSQL, caching, K8s)
   - SOLID principles and design patterns
   - **Trigger**: "Design a system...", "Architecture for...", "Should we use SQL or NoSQL?"

### 💻 Code Writers (Backend - Production Quality)
2. **backend-python-writer** (Sonnet)
   - FastAPI, Django, Flask
   - Type hints, Pydantic, async/await
   - SQLAlchemy, error handling, observability
   - **Trigger**: "Implement a Python endpoint...", "Write Python backend code..."

3. **backend-java-writer** (Sonnet)
   - Spring Boot, immutability, null safety
   - JPA/Hibernate, Lombok patterns
   - Circuit breakers, proper exception handling
   - **Trigger**: "Implement a Java service...", "Write Spring Boot code..."

4. **backend-go-writer** (Sonnet)
   - Idiomatic Go, goroutines, channels
   - Context management, error handling
   - Chi/Gin routers, httptest patterns
   - **Trigger**: "Implement a Go handler...", "Write Go backend code..."

### 🧪 Test Engineers (Comprehensive Testing)
5. **backend-python-test-engineer** (Sonnet)
   - pytest, testify, fixtures
   - Unit + integration tests, mocking
   - 80%+ coverage, table-driven tests
   - **Trigger**: "Write tests for Python code...", "Test this Python service..."

6. **backend-java-test-engineer** (Sonnet)
   - JUnit 5, Mockito, MockMvc
   - Testcontainers, integration tests
   - AssertJ assertions, parametrized tests
   - **Trigger**: "Write tests for Java code...", "Test this Spring Boot service..."

7. **backend-go-test-engineer** (Sonnet)
   - testing package, testify
   - Table-driven tests, httptest
   - Benchmarks, race detection
   - **Trigger**: "Write tests for Go code...", "Test this Go service..."

### 🔍 Code Review
8. **backend-mr-reviewer** (Sonnet)
   - Security, performance, maintainability review
   - Language-agnostic (Python/Java/Go)
   - OWASP Top 10, code quality, testing coverage
   - **Trigger**: "Review this MR/PR...", "Code review...", "Review my changes..."

### 📝 Planning
9. **backend-implementation-planner** (Sonnet)
   - Breaks features into actionable tasks
   - Identifies files to create/modify
   - Plans technical approach, edge cases
   - Estimates complexity and blockers
   - **Trigger**: "Plan implementation for...", "How do I implement...", "Break down this feature..."

## 🚀 Usage Examples

### Example 1: Full Feature Development Flow
```
User: "I need to implement a user registration API in Python"

1. Planner: "Plan implementation for user registration in Python"
   → Gets structured plan with all tasks, files, edge cases

2. Code Writer: "Implement the user registration endpoint in Python"
   → Gets production-quality FastAPI code

3. Test Engineer: "Write tests for the user registration service"
   → Gets comprehensive pytest tests with >80% coverage

4. Reviewer: "Review the user registration PR"
   → Gets thorough code review with security/performance checks
```

### Example 2: System Design First
```
User: "Design a URL shortener that handles 1000 req/sec"

1. Architect: Automatically invoked
   → Complete system design with HLD, LLD, database choices, caching

2. Code Writer: "Implement the URL shortening service in Go"
   → Production-ready Go code following the design

3. Test Engineer: "Write tests for the URL shortener"
   → Comprehensive test suite
```

### Example 3: Code Review
```
User: "Review my PR at github.com/user/repo/pull/123"

Reviewer: Automatically invoked
→ Detailed review covering:
  - Security vulnerabilities
  - Performance issues
  - Test coverage
  - Code quality
  - Maintainability
```

## 📐 Standards Enforced

All agents enforce MAANG-level standards:

### ✅ Code Quality
- SOLID principles
- DRY (Don't Repeat Yourself)
- Single Responsibility Principle
- Proper error handling
- No commented-out code

### 🔒 Security
- Input validation (all inputs)
- SQL injection prevention (parameterized queries)
- No hardcoded secrets
- Authentication/authorization
- Rate limiting
- OWASP Top 10 compliance

### ⚡ Performance
- Connection pooling
- Caching strategies (Redis)
- N+1 query prevention
- Pagination for large datasets
- Async/non-blocking I/O
- Proper indexing

### 📊 Observability
- Structured logging (request_id, user_id, trace_id)
- Metrics (Prometheus/StatsD)
- Distributed tracing (OpenTelemetry)
- Proper error logging
- Health checks

### 🧪 Testing
- 80%+ test coverage
- Unit + integration tests
- Edge cases covered
- Mocking external dependencies
- Test independence

## 🎯 Language-Specific Features

### Python
- Type hints everywhere (Python 3.10+)
- Pydantic for validation
- FastAPI/Django patterns
- Async/await
- pytest best practices

### Java
- Spring Boot conventions
- Immutability (Lombok @Value)
- Optional<T> instead of null
- Constructor injection
- JUnit 5 + Mockito

### Go
- Idiomatic Go patterns
- Context management
- Goroutines & channels
- Error wrapping with %w
- Table-driven tests

## 💡 Pro Tips

1. **Use the planner first** for complex features to get a roadmap
2. **Let architects design** before coding (prevents rework)
3. **Write code and tests together** - don't leave testing for later
4. **Review your own code** with the MR reviewer before submitting
5. **Agents automatically invoke** when appropriate - just describe what you need

## 📞 Getting Help

Each agent has comprehensive examples and patterns built-in. Just ask:
- "Show me an example of [pattern]"
- "How do I handle [scenario]"
- "What's the best practice for [situation]"

## 🔄 Continuous Improvement

These agents are based on battle-tested patterns from MAANG companies. They enforce:
- Production-quality code from the start
- Security by default
- Performance and scalability
- Comprehensive testing
- Thorough documentation

Your code will be deployment-ready, not just "works on my machine" quality.
