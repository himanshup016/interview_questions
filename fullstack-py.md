# Deloitte — Full Stack Engineer (Python) — Interview Prep

## 2. JavaScript / Frontend (React, HTML, CSS)

**Q1. Explain the difference between `var`, `let`, and `const`, including scoping.**
A: `var` is function-scoped and hoisted with `undefined` initialization. `let`/`const` are block-scoped and live in the "temporal dead zone" until declaration — accessing them earlier throws a `ReferenceError`. `const` prevents reassignment of the binding (not deep immutability of objects/arrays).

**Q2. What is the Virtual DOM and how does React's reconciliation work?**
A: The Virtual DOM is an in-memory tree representation of the UI. On state change, React builds a new VDOM tree, diffs it against the previous one (using keys for list items to match elements efficiently), and computes the minimal set of real DOM mutations needed, then batches and applies them — avoiding expensive full re-renders.

**Q3. Explain React hooks you've used and a pitfall with `useEffect`.**
A: I commonly use `useState`, `useEffect`, `useMemo`, `useCallback`, and custom hooks for data fetching with React Query/TanStack. A common pitfall is missing/incorrect dependency arrays in `useEffect`, causing stale closures or infinite re-render loops; I also clean up subscriptions/timers in the return function to avoid memory leaks.

**Q4. How do you manage state in a large React app?**
A: Local component state via `useState`/`useReducer` for UI-only concerns, Redux Toolkit for shared/global app state, and React Query for server state (caching, revalidation, background refetching) — keeping server state separate from client state avoids a lot of boilerplate and sync bugs.

**Q5. CSS: explain the box model and flexbox vs grid.**
A: Box model = content, padding, border, margin (and `box-sizing: border-box` includes padding/border in width). Flexbox is one-dimensional (row or column) and great for component-level alignment; Grid is two-dimensional and better for overall page/layout structures with rows and columns simultaneously.

**Q6. What's the difference between `==` and `===` in JS?**
A: `==` performs type coercion before comparing; `===` checks both type and value without coercion. Best practice is to always use `===`/`!==` to avoid subtle bugs.

**Q7. Explain closures with an example.**
A: A closure is a function that retains access to its lexical scope even after the outer function has returned. Example: a counter factory function returning an inner function that increments and remembers a private variable — used for things like memoization or private state in custom hooks.

**Q8. Explain event delegation.**
A: Attaching a single event listener to a parent element instead of each child, leveraging event bubbling to handle events from dynamically added children efficiently — reduces memory overhead and handles dynamic DOM content.

**Q9. How do you optimize frontend performance? (tie to your resume numbers)**
A: Code-splitting and lazy loading routes/components, memoizing expensive computations (`useMemo`/`useCallback`), virtualizing long lists, optimizing bundle size, image lazy-loading, and caching API responses. At Harbinger this combination contributed to a 30% load performance improvement and consistently 90+ Lighthouse scores.

**Q10. Angular/Vue familiarity (JD mentions React, Angular, or Vue) — how would you answer if you've mainly used React?**
A: Be honest: "My production experience is primarily React/Next.js, but I understand Angular's component/module/DI architecture and Vue's reactivity system conceptually, and I'd ramp up quickly given my component-architecture background — the underlying concepts of state, props/inputs, and lifecycle are transferable."

---

## 3. Python / FastAPI / Django / Flask

**Q11. Why FastAPI over Flask/Django for an API-heavy service?**
A: FastAPI is built on Starlette + Pydantic, supports native `async`/`await`, gives automatic OpenAPI/Swagger docs, and has built-in request/response validation via type hints — which is why I used it for our LMS/e-commerce services needing high-throughput async I/O (50,000+ daily calls, sub-200ms targets). Django is more opinionated/batteries-included (ORM, admin panel, auth) — better for content-heavy monoliths; Flask is minimal/unopinionated, good for small services or when you want full control over architecture.

**Q12. Explain Pydantic and why validation at the schema level matters.**
A: Pydantic models define expected request/response shapes using Python type hints; FastAPI uses them to automatically validate, serialize/deserialize, and document payloads. It catches malformed data early (422 errors) before it reaches business logic, and removes a lot of manual validation boilerplate.

**Q13. How does dependency injection work in FastAPI?**
A: Using `Depends()`, you declare reusable dependencies (DB sessions, auth checks, pagination params) as functions; FastAPI resolves and injects them per-request. I used this for shared concerns like JWT validation and DB session lifecycle management across endpoints.

**Q14. Explain `async`/`await` in Python and when async actually helps.**
A: `async def` defines a coroutine; `await` suspends execution until an awaited I/O operation completes, freeing the event loop to handle other requests. It helps for I/O-bound work (DB calls, external API calls, file I/O) but doesn't help CPU-bound work — for that, you'd offload to a thread/process pool (e.g., `run_in_executor` or Celery workers).

**Q15. GIL — what is it and how does it affect concurrency in Python?**
A: The Global Interpreter Lock allows only one thread to execute Python bytecode at a time in CPython, so threads don't give true parallelism for CPU-bound tasks. Async I/O sidesteps this for I/O-bound concurrency; for CPU-bound parallelism, use multiprocessing.

**Q16. How do you structure a FastAPI project for maintainability?**
A: Layered structure: `routers/` (endpoints), `schemas/` (Pydantic models), `services/` (business logic), `models/` (ORM/DB models), `core/` (config, security), `db/` (session/engine setup) — keeping routers thin and logic testable in isolation. This is the structure I used to keep our LMS services modular.

**Q17. Django ORM vs raw SQL vs SQLAlchemy — your take?**
A: Django ORM is convenient and integrates tightly with the framework's migrations/admin; SQLAlchemy (often paired with FastAPI) gives more explicit control over queries and is framework-agnostic. I lean toward ORM for standard CRUD, but drop to raw SQL/aggregation pipelines when query performance is critical — that's how I hit sub-100ms queries on the e-commerce SKU search.

**Q18. How do you handle background/long-running tasks in FastAPI?**
A: FastAPI's built-in `BackgroundTasks` for lightweight post-response work, or Celery + Redis/RabbitMQ as a broker for heavier, retryable async job processing (e.g., sending emails, processing uploads).

**Q19. Explain Python decorators and give a practical use case.**
A: A decorator wraps a function to extend its behavior without modifying its code — e.g., `@app.get("/users")` in FastAPI registers a route, or a custom `@require_role("admin")` decorator could enforce RBAC checks before an endpoint executes.

**Q20. How do you handle exceptions/error responses consistently across a FastAPI app?**
A: Custom exception classes + FastAPI's `@app.exception_handler()` to map them to consistent JSON error responses with proper status codes, plus global middleware for catching unhandled exceptions and logging them — this is similar to how I enforced consistent API contracts at Harbinger.

---

## 4. REST APIs & WebSockets

**Q21. What makes an API "RESTful"? Key constraints?**
A: Statelessness, resource-based URIs, standard HTTP verbs (GET/POST/PUT/PATCH/DELETE) mapped to CRUD, representation via JSON, and HATEOAS (less commonly fully implemented in practice). I documented these contracts with OpenAPI/Swagger for the team.

**Q22. PUT vs PATCH vs POST?**
A: POST creates a new resource (or triggers an action), PUT replaces a resource entirely (idempotent), PATCH partially updates a resource (may or may not be idempotent depending on implementation).

**Q23. How do you design pagination for a large dataset (you mention 100,000+ course records)?**
A: I used server-side pagination combined with infinite scrolling on the frontend — cursor-based pagination (using an indexed column like `created_at`/`id`) is preferable over offset-based for large, frequently-changing datasets, since offset pagination degrades in performance and can skip/duplicate records under concurrent writes.

**Q24. How do you secure REST APIs?**
A: JWT-based authentication, RBAC for authorization, HTTPS everywhere, input validation (Pydantic), rate limiting, CORS configuration, and avoiding sensitive data in URLs/logs. At Harbinger this reduced security vulnerabilities by 60%.

**Q25. Explain WebSockets vs REST — when would you choose one over the other?**
A: REST is request/response and stateless — good for typical CRUD. WebSockets maintain a persistent, full-duplex connection — needed for real-time use cases like live notifications, chat, or live dashboards (e.g., real-time inventory updates in the e-commerce platform). I'd use REST for standard resource operations and layer WebSockets only where true bidirectional real-time push is needed, to avoid unnecessary connection overhead.

**Q26. How do you scale WebSocket connections across multiple server instances?**
A: Use a shared pub/sub layer (Redis Pub/Sub or a message broker) so that messages broadcast from one instance reach clients connected to other instances, combined with sticky sessions or a connection-aware load balancer.

**Q27. How do you version an API?**
A: URI versioning (`/api/v1/...`) is simplest and most explicit for clients; alternatives are header-based versioning. I'd default to URI versioning for clarity in a consulting/multi-client context like Deloitte's.

---

## 5. Databases (PostgreSQL / MySQL / MongoDB)

**Q28. When would you choose PostgreSQL/MySQL vs MongoDB?**
A: Relational DBs (Postgres/MySQL) for structured data with strong consistency needs and complex joins/transactions (e.g., orders, payments). MongoDB for flexible/nested schemas, high write throughput, or rapidly evolving data shapes (e.g., course content, logs). In my LMS and e-commerce work I used both together — relational for transactional integrity, MongoDB for flexible content/catalog data.

**Q29. How do indexes work and how did you use them to improve performance?**
A: Indexes (B-tree typically) create a sorted structure on a column to avoid full table scans. I added indexes on frequently filtered/sorted columns and used MongoDB aggregation pipelines for analytics-style queries, which together cut average API response time by 45%.

**Q30. Explain MongoDB aggregation pipeline stages you've used.**
A: Common stages: `$match` (filter early to reduce dataset), `$group` (aggregate/group by), `$project` (shape output fields), `$lookup` (join-like operation), `$sort`, `$limit`. Ordering `$match` before `$group`/`$lookup` is key for performance.

**Q31. What is database normalization, and when might you denormalize?**
A: Normalization organizes data to reduce redundancy (1NF–3NF). Denormalization trades some redundancy for read performance — useful in read-heavy systems like our course catalog where joins were getting expensive at scale.

**Q32. ACID properties — explain briefly.**
A: Atomicity (all-or-nothing), Consistency (valid state transitions), Isolation (concurrent transactions don't interfere), Durability (committed data survives failures) — critical for the payment/checkout flow I built, which needed PCI DSS-compliant, zero-breach transaction handling.

**Q33. How do you handle database migrations?**
A: Alembic for SQLAlchemy/FastAPI projects, or Django's built-in migration system — version-controlled, reviewed in PRs, and run as part of the CI/CD deploy pipeline.

---

## 6. Git / Version Control

**Q34. Describe your typical Git workflow.**
A: Feature-branch workflow off `main`/`develop`, small focused commits, PRs with mandatory code review, squash-merge to keep history clean, and CI checks (lint, tests, SonarQube) gating merges.

**Q35. How do you resolve a merge conflict?**
A: Identify conflicting hunks, communicate with the other author if needed, manually reconcile changes preserving intent of both, run tests locally before pushing the resolution, and never blindly accept "ours/theirs" without verifying logic.

**Q36. `git rebase` vs `git merge`?**
A: Merge preserves full history with a merge commit; rebase rewrites commit history onto a new base for a linear, cleaner history. I rebase feature branches before opening PRs to keep history readable, but avoid rebasing shared/public branches.

---

## 7. Cloud (AWS/Azure/GCP)

**Q37. Which AWS services have you used and how?**
A: EC2/Lambda for compute, S3 for static assets/file storage, CloudFront for CDN delivery to improve load times globally. Combined with Docker for containerized deployments. I also have working exposure to Azure.

**Q38. How would you design a scalable, cost-efficient cloud architecture for a REST API service?**
A: Containerized API behind a load balancer with auto-scaling groups (or serverless via Lambda for spiky/event-driven workloads), managed RDS/Postgres or DynamoDB/MongoDB Atlas for data, CloudFront + S3 for static content, Redis (ElastiCache) for caching hot data, and CloudWatch for monitoring/alerts.

**Q39. Serverless vs containers — trade-offs?**
A: Serverless (Lambda) is great for unpredictable/spiky traffic and zero infra management but has cold-start latency and execution time limits. Containers (ECS/EKS/Docker) give more control, consistent performance, and are better for long-running or stateful services — I used Docker-based CI/CD for our consistently-loaded LMS backend.

---

## 8. CI/CD, DevOps, Docker, Kubernetes (Good-to-have)

**Q40. Walk through the CI/CD pipeline you built.**
A: GitHub Actions triggered on PR/merge: lint → unit tests (Pytest/Jest) → SonarQube quality gate → build Docker image → push to registry → deploy. This cut deployment time by 50% and eliminated manual release errors at Harbinger.

**Q41. Docker vs Kubernetes — what problem does each solve?**
A: Docker packages an app + dependencies into a portable, consistent container image. Kubernetes orchestrates many containers across machines — handling scaling, self-healing, service discovery, and rolling deployments. I've used Docker extensively for builds/deployments; Kubernetes exposure is more conceptual, so I'd be upfront about depth here if asked directly.

**Q42. How do you ensure zero-downtime deployments?**
A: Rolling deployments / blue-green deployments, health checks before routing traffic to new instances, and feature flags to decouple deploy from release.

---

## 9. Testing

**Q43. How do you approach testing in a full-stack app?**
A: Unit tests for isolated logic (Pytest for backend, Jest for frontend), integration tests for API endpoints with a test DB, React Testing Library for component behavior (not implementation details), and E2E tests for critical user flows. I maintained 85%+ coverage and used SonarQube to enforce quality gates with zero critical bugs.

**Q44. What's the difference between mocking and stubbing, and when do you use each?**
A: Stubs return canned responses to isolate the unit under test; mocks additionally verify *how* a dependency was called (call count/args). I use stubs for simple external dependency isolation and mocks when behavior/interaction verification matters (e.g., verifying a payment service was called exactly once).

---

## 10. Agile / Behavioral

**Q45. Tell me about a time you led or mentored others.**
A: At Harbinger, I led technical initiatives for 3 junior developers through code reviews, pair programming, and system design workshops, which improved team code quality by ~35%. I focused on explaining *why* behind feedback rather than just prescribing fixes, so it built their judgment over time.

**Q46. Describe a challenging technical problem you solved.**
A: The LMS needed to handle 100,000+ course records with responsive infinite scrolling. I combined server-side cursor pagination, Redis caching for hot queries, and frontend virtualization/lazy loading — reducing page load time by 40% and supporting 3x user growth without re-architecting the data layer.

**Q47. How do you handle disagreement with a teammate or reviewer on a technical approach?**
A: I bring data/trade-offs to the table rather than opinions — benchmark both approaches if feasible, reference standards or prior incidents, and default to the team's documented architectural principles if it's a close call. I'd rather lose the argument with good reasoning than win it with ego.

**Q48. Why are you looking to move, and why Deloitte specifically?**
A: (Personalize — example framing) "I've grown a lot in a focused product environment at Harbinger and want exposure to varied client domains and larger-scale systems, which a consulting environment like Deloitte offers. My full-stack background — frontend through to API and database performance — maps directly to the breadth this role needs."

**Q49. Describe your sprint/Agile experience.**
A: Standard Scrum — sprint planning, daily standups, refinement, retro — consistently delivering 95%+ sprint completion rate, with story estimation refined over time based on velocity tracking.

**Q50. Where do you see gaps in your experience relative to this JD, and how would you close them?**
A: My Django and Kubernetes exposure is lighter than FastAPI/Docker — I'd close that gap quickly given my existing async Python and containerization background, since the underlying concepts (ORM patterns, container orchestration) build directly on what I already use daily.

---

## Quick-Reference: Your Strongest Numbers to Drop Naturally

- 50,000+ daily API calls, sub-200ms response targets
- 45% reduction in average API response time (indexing + aggregation pipelines)
- 40% reduction in UI code duplication (reusable TS/MUI component library)
- 30% app load performance improvement
- 50% reduction in deployment time (CI/CD via GitHub Actions + Docker)
- 60% reduction in security vulnerabilities (JWT + RBAC)
- 85%+ test coverage (Jest, RTL, Pytest)
- 95%+ sprint completion rate
- 15,000+ active LMS users, 99.8% uptime
- 5× consecutive Quarterly Project Excellence Awards

---

## Notes for You
- The JD lists Angular/Vue and Django/Flask alongside your React/FastAPI background — be ready to honestly frame these as "conceptually strong, less production depth" rather than overclaiming.
- Kubernetes is "good to have" — your Docker/CI-CD story is solid; don't over-promise K8s depth if pressed for specifics.
- This is a Deloitte consulting role — expect at least one question about working with ambiguous/client-driven requirements or multiple stakeholders; your sprint/Agile and cross-functional collaboration points cover this well.

