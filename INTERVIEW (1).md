# INTERVIEW.md – Full-Stack Interview Guide (React + FastAPI + MySQL)

> **Master the complete stack. Interview-ready in one read.**

---

## 1. Why This Stack? (The Stack Choice)

### React + Vite + TailwindCSS (Frontend)
- **React** – Component reusability, Virtual DOM, state management, ecosystem (Router, Query, Hook Form)
- **Vite** – Lightning-fast dev server (10x faster than webpack), instant HMR, optimized builds, zero-config
- **TailwindCSS** – Utility-first CSS, responsive-first, no naming headaches, tiny final bundle

### FastAPI + SQLAlchemy + MySQL (Backend)
- **FastAPI** – Async-first, auto-generates OpenAPI docs, type hints enable validation, blazing fast
- **SQLAlchemy** – ORM prevents SQL injection, relationships auto-handle joins, migrations with Alembic
- **MySQL** – ACID-compliant, relational integrity via foreign keys, proven at scale (Facebook, Netflix, Airbnb)

### JWT Authentication (Stateless)
- **No session storage** – Scales horizontally across servers
- **Self-contained tokens** – User ID embedded, verified on each request
- **Security** – Bcrypt hashes passwords, JWT signed with SECRET_KEY

**Interview line:**  
*"We chose React for component reusability, Vite for speed, FastAPI for async performance, SQLAlchemy for ORM safety, and MySQL for data integrity. JWT tokens scale horizontally without session storage."*

---

## 2. How Frontend & Backend Communicate

### The Request-Response Cycle

```
React Component (user clicks "Create Task")
    ↓
Custom hook (useTasks) calls api.tasks.create(title, description)
    ↓
HTTP POST /api/v1/tasks with Authorization: Bearer {token}
    ↓
FastAPI route handler receives request
    ↓
Dependency injection: get_current_user verifies JWT
    ↓
Pydantic validates request body (title, description)
    ↓
Business logic: Create Task in database via SQLAlchemy
    ↓
Return Task JSON {id, title, description, status, created_at}
    ↓
React state updates, component re-renders with new task
    ↓
Optimistic update makes UI instant
```

### Example: User Login Flow

1. **Frontend:** User enters email/password, clicks "Login"
2. **Frontend:** `api.auth.login(email, password)` calls `POST /api/v1/auth/login`
3. **Backend:** Queries database for user by email
4. **Backend:** Verifies password with bcrypt (hashed comparison)
5. **Backend:** Creates JWT token with `{user_id, email}` and signs with SECRET_KEY
6. **Backend:** Returns `{access_token, token_type: "bearer", user: {id, email}}`
7. **Frontend:** Stores token in localStorage
8. **Frontend:** Subsequent requests include `Authorization: Bearer {token}` header
9. **Backend:** Middleware extracts token, verifies it, injects user into route handler
10. **Backend:** If token invalid/expired, returns 401 Unauthorized
11. **Frontend:** Catches 401, clears localStorage, redirects to login

**Interview line:**  
*"Frontend and backend are separate concerns. Frontend handles UI, backend handles data. They communicate via REST API. JWT tokens are stateless and scale horizontally."*

---

## 3. Project Structure (Full-Stack Organization)

```
fullstack-app/
│
├── frontend/                    (React + Vite)
│   ├── src/
│   │   ├── components/          (Reusable UI components)
│   │   │   ├── TaskCard.jsx
│   │   │   ├── Button.jsx
│   │   │   └── Header.jsx
│   │   │
│   │   ├── pages/               (Full-page components, tied to routes)
│   │   │   ├── TaskList.jsx     (Fetch and display tasks)
│   │   │   ├── TaskDetail.jsx   (Single task page)
│   │   │   ├── Login.jsx        (Auth page)
│   │   │   └── NotFound.jsx     (404 page)
│   │   │
│   │   ├── hooks/               (Custom React hooks)
│   │   │   ├── useFetch.js      (Data fetching logic)
│   │   │   ├── useTasks.js      (Task CRUD operations)
│   │   │   └── useAuth.js       (Auth state + methods)
│   │   │
│   │   ├── services/            (API calls, external integrations)
│   │   │   └── api.js           (All fetch calls to backend)
│   │   │
│   │   ├── context/             (Global state)
│   │   │   └── AuthContext.jsx  (Auth state shared across app)
│   │   │
│   │   ├── utils/               (Helper functions)
│   │   │   ├── formatDate.js
│   │   │   └── validation.js
│   │   │
│   │   ├── App.jsx              (Root component, routes defined)
│   │   └── main.jsx             (Entry point)
│   │
│   ├── package.json
│   ├── vite.config.js
│   └── .env.local               (VITE_API_URL=http://localhost:8000/api/v1)
│
├── backend/                     (FastAPI + MySQL)
│   ├── app/
│   │   ├── core/
│   │   │   ├── config.py        (Settings from environment)
│   │   │   ├── security.py      (JWT, password hashing)
│   │   │   └── constants.py     (Error messages, status codes)
│   │   │
│   │   ├── db/
│   │   │   ├── base.py          (SQLAlchemy Base class)
│   │   │   ├── session.py       (Engine, SessionLocal, get_db)
│   │   │   └── models.py        (User, Task ORM models)
│   │   │
│   │   ├── schemas/             (Pydantic request/response validation)
│   │   │   ├── user.py          (UserCreate, UserLogin, TokenResponse)
│   │   │   └── task.py          (TaskCreate, TaskResponse, TaskUpdate)
│   │   │
│   │   ├── api/
│   │   │   ├── v1/
│   │   │   │   ├── routes/
│   │   │   │   │   ├── auth.py  (POST /login, /register)
│   │   │   │   │   └── tasks.py (GET/POST/PUT/DELETE /tasks)
│   │   │   │   └── __init__.py
│   │   │   └── deps.py          (Dependency injection: get_current_user, get_db)
│   │   │
│   │   └── services/            (Business logic, domain services)
│   │       ├── task_service.py
│   │       └── user_service.py
│   │
│   ├── main.py                  (FastAPI app, CORS, routers)
│   ├── requirements.txt          (Dependencies: FastAPI, SQLAlchemy, bcrypt, etc.)
│   ├── .env                      (DATABASE_URL, SECRET_KEY – git-ignored)
│   └── .env.example              (Template for .env)
│
├── docker-compose.yml           (MySQL, backend, frontend orchestration)
├── .gitignore
└── README.md
```

**Interview line:**  
*"Frontend separates components (UI), pages (routes), hooks (logic), services (API calls), and context (global state). Backend separates core (config), db (ORM), schemas (validation), api (routes), and services (business logic). Clear separation of concerns."*

---

## 4. Frontend Architecture Deep Dive

### Component Hierarchy

**Smart Components (Pages)** – Fetch data, manage state
- TaskList (fetches all tasks, manages UI state)
- TaskDetail (fetches single task)
- Login (handles auth)

**Dumb Components (Reusable)** – Accept props, render UI
- TaskCard (displays one task)
- Button (reusable button)
- Header (navigation bar)

### Custom Hooks Pattern

**useFetch** – Generic data fetching
```javascript
// Handles loading, error, retry
const { data: tasks, loading, error } = useFetch('/api/v1/tasks');
```

**useTasks** – Task-specific CRUD
```javascript
// Wraps useFetch, adds create/update/delete methods
const { tasks, createTask, updateTask, deleteTask } = useTasks();
await createTask("New Task", "Description");
```

**useAuth** – Authentication state
```javascript
// Manages user, token, login/logout
const { user, isLoading, login, logout } = useAuth();
```

### Global State Pattern

**AuthContext** – Share auth state across app
```javascript
// Wrap App with <AuthProvider>
const { user } = useAuthContext();
// Available in any component without prop drilling
```

### Data Flow (Unidirectional)

```
User Action (click button)
    ↓
Event Handler (onClick)
    ↓
Call Hook Method (createTask)
    ↓
Hook calls API service (api.tasks.create)
    ↓
API sends HTTP request
    ↓
Hook receives response
    ↓
Hook updates state (setTasks)
    ↓
Component re-renders with new state
    ↓
UI reflects change
```

**Interview line:**  
*"Smart components fetch data via custom hooks. Dumb components receive data as props. State flows down, events flow up. Custom hooks replace Redux for small to medium apps."*

---

## 5. Backend Architecture Deep Dive

### Layered Architecture

**Routes (API layer)** – Handle HTTP requests
- Dependency injection: `get_current_user`, `get_db`
- Request validation: Pydantic schemas
- Response serialization: Pydantic models
- Status codes: 200, 201, 400, 401, 403, 404

**Services (Business logic layer)** – Complex operations
- Task filtering, sorting, analytics
- User validation, password reset
- Email notifications

**Models (Data access layer)** – SQLAlchemy ORM
- User, Task models with relationships
- Foreign keys enforce integrity
- Indexes speed up queries

**Security (Cross-cutting concern)**
- Bcrypt password hashing (one-way, irreversible)
- JWT token creation and verification
- Dependency injection for auth checks

### Dependency Injection Pattern

```python
@router.get("/tasks")
def get_tasks(
    db: Session = Depends(get_db),              # Database session
    user: User = Depends(get_current_user)      # Authenticated user
):
    # Both injected automatically
    return db.query(Task).filter(Task.owner_id == user.id).all()
```

**Benefits:**
- Automatic injection of dependencies
- Easy to test (mock dependencies)
- Clean separation of concerns
- Type hints enable validation

### Request-Response Validation

**Request Validation (Pydantic schemas)**
- UserCreate: validates email format, password strength
- TaskCreate: validates title length, description
- Invalid data rejected before reaching database

**Response Serialization (Pydantic models)**
- UserResponse: returns {id, email, username, created_at}
- TaskResponse: returns {id, title, description, status, created_at}
- Never expose hashed_password or sensitive fields

### Database Relationships

```python
User(id=1) ──one-to-many──> Task(owner_id=1)
                             Task(owner_id=1)
                             Task(owner_id=1)
```

**Foreign keys enforce:**
- User owns all their tasks (owner_id always points to existing user)
- Deleting user cascades delete to all tasks
- Task.owner always available via relationship

**Interview line:**  
*"Dependency injection provides database sessions and authenticated users automatically. Pydantic validates input before reaching database. Foreign keys enforce referential integrity. Services handle business logic separate from routes."*

---

## 6. Authentication & Authorization

### Bcrypt Password Hashing

```python
# Registration
hashed = hash_password("password123")  # Returns "$2b$12$..." (random, one-way)
db.add(User(email=email, hashed_password=hashed))

# Login
user = db.query(User).filter(User.email == email).first()
if verify_password("password123", user.hashed_password):  # True
    # Create JWT token
```

**Why bcrypt?**
- One-way hashing (can't decrypt)
- Salted (same password produces different hashes)
- Slow (prevents brute-force)
- Industry standard (used by Django, Laravel, etc.)

### JWT Token Flow

```
User logs in with email/password
    ↓
Backend verifies password
    ↓
Backend creates JWT: {header}.{payload}.{signature}
    Payload: {user_id: 123, email: "user@example.com", exp: 1234567890}
    Signed with SECRET_KEY
    ↓
Backend returns token to frontend
    ↓
Frontend stores in localStorage (or secure HTTP-only cookie)
    ↓
Frontend includes in Authorization header: Bearer {token}
    ↓
Backend middleware extracts token
    ↓
Backend verifies signature with SECRET_KEY
    ↓
Backend checks expiration (exp field)
    ↓
If valid: injects user into route handler
If invalid/expired: returns 401 Unauthorized
    ↓
Frontend catches 401, clears localStorage, redirects to login
```

**Why JWT?**
- Stateless (no session database needed)
- Scales horizontally (any server can verify)
- Self-contained (user data embedded in token)
- Secure (signed, can't be tampered with)

### Authorization (Resource Ownership)

```python
@router.get("/tasks/{task_id}")
def get_task(task_id: int, user: User = Depends(get_current_user), db: Session = Depends(get_db)):
    task = db.query(Task).filter(Task.id == task_id).first()
    
    # Authorization check: only owner can view
    if task.owner_id != user.id:
        raise HTTPException(status_code=403, detail="Permission denied")
    
    return task
```

**Interview line:**  
*"Bcrypt hashes passwords irreversibly. JWT tokens are stateless—no session storage needed. Every request verifies token and injects user. Routes check user owns the resource before granting access."*

---

## 7. Error Handling Across Stack

### Frontend Error Handling

```javascript
// API call
try {
    await api.tasks.create(title, description);
} catch (err) {
    // err.message comes from backend response.detail
    setError(err.message);  // "Title is required" or "Task already exists"
}

// Component displays error
{error && <div className="text-red-500">{error}</div>}
```

### Backend Error Handling

```python
# Route handler
@router.post("/tasks")
def create_task(task_data: TaskCreate, ...):
    # Pydantic validation errors caught automatically
    # Status 422 if validation fails
    
    if db.query(Task).filter(Task.title == task_data.title).first():
        raise HTTPException(status_code=400, detail="Title already exists")
    
    # If reaches database error
    try:
        db.commit()
    except IntegrityError:
        raise HTTPException(status_code=400, detail="Database constraint violated")
```

### HTTP Status Codes Convention

| Code | Meaning | When to Use |
|------|---------|------------|
| 200 | OK | Successful GET, PUT, DELETE |
| 201 | Created | Successful POST (new resource) |
| 400 | Bad Request | Validation error, user input error |
| 401 | Unauthorized | Missing/invalid auth token |
| 403 | Forbidden | User lacks permission (owns different resource) |
| 404 | Not Found | Resource doesn't exist |
| 500 | Server Error | Unhandled exception in backend |

**Interview line:**  
*"Frontend catches errors, displays user-friendly messages. Backend returns HTTP status codes and detail text. We distinguish user errors (400) from auth errors (401) from permission errors (403)."*

---

## 8. Testing Strategy

### Backend Testing (pytest)

```
Test database fixtures
    ↓
Create test user, test task
    ↓
Test auth routes (login, register, invalid credentials)
    ↓
Test task routes (CRUD, authorization, 404s)
    ↓
Mock external dependencies
    ↓
Assert HTTP status codes
    ↓
Assert response data structure
```

### Frontend Testing (React Testing Library)

```
Mock API responses
    ↓
Render component
    ↓
Simulate user interactions (click, type)
    ↓
Assert DOM changes
    ↓
Assert API calls made
    ↓
Test loading/error states
```

### Integration Testing

```
Start both services (backend + frontend)
    ↓
Sign up new user
    ↓
Login with credentials
    ↓
Create task
    ↓
Update task
    ↓
Delete task
    ↓
Logout
    ↓
Verify redirect to login
```

**Interview line:**  
*"We test backend and frontend independently with mocked dependencies. Integration tests verify the full flow. Tests catch regressions early and enable refactoring with confidence."*

---

## 9. Deployment & DevOps

### Local Development (Docker Compose)

```bash
docker-compose up
# Starts MySQL, FastAPI (with hot-reload), React (with HMR)
# http://localhost:5173 (frontend)
# http://localhost:8000 (backend)
# http://localhost:8000/docs (API docs)
```

### Production Deployment

**Frontend → Vercel/Netlify**
- Auto-deploys from GitHub
- CDN for fast global delivery
- Environment variables (VITE_API_URL=https://api.yourdomain.com/api/v1)

**Backend → Heroku/Railway/DigitalOcean**
- Docker container
- Environment variables (DATABASE_URL, SECRET_KEY)
- Automatic HTTPS

**Database → AWS RDS / PlanetScale**
- Managed MySQL service
- Automatic backups
- High availability

### Environment Configuration

**Development (.env.local)**
```
VITE_API_URL=http://localhost:8000/api/v1
```

**Production (.env.production)**
```
VITE_API_URL=https://api.yourdomain.com/api/v1
```

**Interview line:**  
*"Docker Compose runs entire stack locally with one command. Frontend deploys to CDN, backend to container service, database to managed service. Environment variables switch between dev/prod."*

---

## 10. Database Design Best Practices

### Relationships

```
User (one-to-many) Task
User.id ←→ Task.owner_id (foreign key)
```

**Cascade delete:**
- Deleting user → deletes all user's tasks
- Prevents orphaned data

### Indexes

```python
email = Column(String(255), unique=True, index=True)
# Speeds up login queries (WHERE email = "user@example.com")

owner_id = Column(Integer, ForeignKey(...), index=True)
# Speeds up "Get all tasks for user_id = 123"

status = Column(Enum(TaskStatus), index=True)
# Speeds up filtering by status
```

### Queries to Avoid

**N+1 Problem (BAD):**
```python
users = db.query(User).all()  # Query 1
for user in users:
    print(len(user.tasks))     # Query 2, 3, 4, ... (N more queries)
```

**Eager Loading (GOOD):**
```python
users = db.query(User).joinedload(User.tasks).all()  # Single query with JOIN
for user in users:
    print(len(user.tasks))  # No additional queries
```

**Interview line:**  
*"Foreign keys enforce referential integrity. Indexes speed up frequently-queried columns. Eager loading prevents N+1 queries. Cascade delete prevents orphaned data."*

---

## 11. 10 Essential Interview Questions

### Q1: "Walk me through a login request from React to FastAPI."
**A:** "User enters email/password in React form. Frontend calls `api.auth.login(email, password)`. Backend receives POST request, queries database for user, verifies password with bcrypt. Backend creates JWT token with user_id inside, signs it with SECRET_KEY, returns token. Frontend stores token in localStorage. Subsequent requests include Authorization header with token. Backend middleware verifies token signature and expiration."

---

### Q2: "Why use JWT instead of sessions?"
**A:** "JWT is stateless—no database queries on every request. Scales horizontally across servers. Sessions require storing session data server-side (Redis, database). JWT token is self-contained—user data embedded and verified via signature. Trade-off: JWT can't be invalidated immediately (token remains valid until expiration), sessions can be destroyed instantly."

---

### Q3: "What's the difference between authentication and authorization?"
**A:** "Authentication: verifying user is who they claim (login with email/password). Authorization: verifying user has permission to access resource (user owns the task). Backend checks authentication via JWT. Backend checks authorization via resource ownership (task.owner_id == user.id)."

---

### Q4: "How do you prevent SQL injection?"
**A:** "SQLAlchemy ORM uses parameterized queries, not string concatenation. Pydantic validates input types and formats before reaching database. Never construct SQL strings directly. Always use ORM query builders or parameterized queries."

---

### Q5: "What's an N+1 query problem and how do you fix it?"
**A:** "N+1 occurs when fetching parent records causes N additional queries for child records. Example: Get all users → loop through users → query each user's tasks (N extra queries). Fix: Use eager loading with `joinedload()` or `selectinload()` to fetch parents and children in single query."

---

### Q6: "Why React + Vite instead of Create React App?"
**A:** "Vite is 10x faster. CRA uses webpack (slow build), Vite uses ES modules (instant HMR). Vite has instant cold start. Same development experience but faster feedback loop. No magic—just standard React."

---

### Q7: "How do you handle a protected route in React?"
**A:** "Create ProtectedRoute component that checks if user is logged in. If yes, render page. If no, redirect to login. Check user state from localStorage or Context. Example: `user ? <TaskList /> : <Navigate to="/login" />`"

---

### Q8: "What's the difference between PUT and PATCH?"
**A:** "PUT replaces entire resource (send all fields). PATCH updates partial resource (send only changed fields). PUT is idempotent (same result every time). PATCH is also idempotent. Practical difference: PUT easier to implement, PATCH saves bandwidth."

---

### Q9: "How do you optimize a React component that renders 1000 items?"
**A:** "Use virtualization library (react-window) to render only visible items. Memoize items with React.memo. Use useCallback for event handlers. Or paginate instead (fetch 20 items, load next 20 on scroll). Lazy load images. Avoid re-renders with proper dependency arrays."

---

### Q10: "How do you deploy this full-stack app?"
**A:** "Frontend: Build with `npm run build`, deploy to Vercel/Netlify (auto-deploys from GitHub). Backend: Build Docker image, push to Docker Hub or cloud registry, deploy to Heroku/Railway/DigitalOcean. Database: Use managed MySQL (AWS RDS, PlanetScale). Set environment variables for prod domain and API URL."

---

## 12. 30-Second Full-Stack Pitch

```
Frontend is React with Vite for speed and TailwindCSS for styling.
Custom hooks fetch data from backend, manage component state.
Components are small and reusable. Global state via Context.

Backend is FastAPI for async performance and auto-generated docs.
SQLAlchemy ORM prevents SQL injection, handles relationships.
MySQL ensures ACID compliance and data integrity.

JWT tokens are stateless—no session storage needed. Scales horizontally.
Every route protected with authentication and authorization checks.
User ownership enforced in database queries.

Frontend and backend communicate via REST API. CORS allows requests.
Docker Compose runs entire stack locally with one command.
Tests on both sides catch bugs early.

Ready to scale. Production-ready security hardened.
```

---

## 13. 5 Killer Full-Stack Lines

1. **"FastAPI async handles 1000s of concurrent requests without threading."**
2. **"SQLAlchemy ORM prevents SQL injection and makes queries readable."**
3. **"JWT tokens scale horizontally—no session storage needed."**
4. **"React hooks encapsulate logic. Components focus on rendering."**
5. **"Docker Compose runs MySQL, backend, frontend locally with one command."**

---

## Quick Reference: Key Concepts

| Concept | Frontend | Backend |
|---------|----------|---------|
| State Management | useState, Context | Database (MySQL) |
| Data Fetching | Custom hooks (useFetch) | SQLAlchemy queries |
| Validation | React Hook Form | Pydantic schemas |
| Authentication | Store JWT in localStorage | Create/verify JWT tokens |
| Authorization | Check user owns resource (UI) | Check user owns resource (DB) |
| Error Handling | try-catch, display messages | HTTPException, status codes |
| Testing | React Testing Library | pytest, mocked DB |

---

## Full-Stack Development Checklist

**Frontend:**
- [ ] Components are small and focused
- [ ] State placed at right level (local vs global)
- [ ] API calls in service layer or custom hooks
- [ ] Forms validate on both client and server
- [ ] Loading and error states displayed
- [ ] Auth token sent with requests
- [ ] Mobile-responsive design
- [ ] Accessibility basics (labels, alt text)

**Backend:**
- [ ] Settings loaded from environment (no hardcoded secrets)
- [ ] Database configured for connection pooling
- [ ] Models have proper indexes and foreign keys
- [ ] All routes protected with authentication
- [ ] Input validated via Pydantic
- [ ] Error handling with meaningful messages
- [ ] Logging configured for debugging
- [ ] Tests passing with good coverage

**Both:**
- [ ] API contract documented (requests/responses)
- [ ] CORS configured for both environments
- [ ] Deployment tested on staging
- [ ] Security scan completed
- [ ] Monitoring and alerting set up

---

## The Golden Full-Stack Rule

> **"Frontend sends requests. Backend validates and queries database. Database is source of truth. HTTP is the bridge. JWT tokens scale horizontally."**

---

*Interview Ready | Full-Stack | React + FastAPI + MySQL*
