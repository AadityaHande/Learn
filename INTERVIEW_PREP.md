# - Hackathon Interview Cheat Sheet

> **Read this once. You'll pass the technical round (if god wills.. xD).**

---

## 1. How Any App Works (The System Flow)

```
┌─────────────────────────────────────────────────────────┐
│                      USER                               │
│              (Mobile/Website/App)                       │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ Clicks button / Submits form
                 ▼
┌─────────────────────────────────────────────────────────┐
│                    FRONTEND                             │
│          (What user sees & clicks on)                   │
│       React/Next.js/Vue or simple HTML                  │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ Sends data (API call)
                 ▼
┌─────────────────────────────────────────────────────────┐
│                    BACKEND                              │
│       (Receives request, does logic, sends response)    │
│        Node.js/Python/Java + Express/Flask              │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ Saves/retrieves data
                 ▼
┌─────────────────────────────────────────────────────────┐
│   DATABASE + STORAGE (Files, Images, User Info)         │
│     Firebase/PostgreSQL/MongoDB + Cloud Storage         │
└─────────────────────────────────────────────────────────┘
                 │
                 │ Response sent back
                 ▼
┌─────────────────────────────────────────────────────────┐
│              FRONTEND (Updated)                         │
│            User sees result on screen                   │
└─────────────────────────────────────────────────────────┘
```

**What happens in 1 second:** User clicks → Frontend packages request → Backend processes → Database updates → Response comes back → UI refreshes.

---

## 2. Database vs Storage – What Goes Where?

### Database (Stores DATA you search/filter)
- User names, emails, phone numbers
- Scores, status, timestamps
- Orders, payments, user preferences
- **Why?** Need to search, sort, filter quickly
- **Example:** Find all "pending" orders from a specific date
- **Tools:** Firebase Firestore, PostgreSQL, MongoDB

### Storage (Stores FILES)
- Profile pictures, documents, videos
- PDFs, images, uploads
- Large files that don't need searching
- **Why?** Just need to store and retrieve, not search inside
- **Example:** User's profile photo, hackathon submission PDF
- **Tools:** AWS S3, Google Cloud Storage, Firebase Storage

**Simple Rule:**
```
DATA (searchable) → Database
FILES (not searchable) → Storage
```

---

## 3. How Image Upload Works (In 5 Steps)

```
Step 1: User selects image from phone/computer
        ↓
Step 2: Frontend sends image to Backend
        (Not as email attachment, as binary data)
        ↓
Step 3: Backend receives image
        - Checks: Is it actually an image? (file type)
        - Checks: Not too large? (size limit)
        - Checks: Not virus? (scan)
        ↓
Step 4: Backend sends image to Storage
        (Google Cloud Storage, AWS S3, Firebase)
        Storage gives back a URL
        ↓
Step 5: Backend saves URL in Database
        (Example: users table, profile_pic_url column)
        ↓
Frontend: Shows image to user from that URL
```

**Interview Answer:**
"User uploads image → Backend validates → Stores in cloud storage → URL saved in DB → Frontend displays from URL"

---

## 4. What is an API? (Simple Definition)

**API = A way for Frontend to talk to Backend**

Think of it like a restaurant:
- **Frontend = Customer** (You)
- **API = Waiter** (Takes your order)
- **Backend = Kitchen** (Processes your order)

Frontend says: "Show me all users" → Backend responds: "Here are 50 users" (as JSON data)

**Example API calls:**
```
GET /users            → "Give me all users"
POST /users           → "Create a new user"
GET /users/123        → "Give me user with ID 123"
PUT /users/123        → "Update user with ID 123"
DELETE /users/123     → "Delete user with ID 123"
```

---

## 5. What is a Backend? (Simple Definition)

**Backend = The brain of the app**

It:
- Receives requests from Frontend
- Checks if user is allowed (authentication)
- Runs business logic (calculations, rules, decisions)
- Talks to Database (save/get data)
- Sends response back to Frontend

**Backend does NOT:**
- ❌ Show anything on screen (that's Frontend's job)
- ❌ Store files directly (that's Storage's job)
- ❌ Keep data forever in memory (that's Database's job)

**Simple answer:** "Backend processes requests, enforces rules, manages data"

---

## 6. Role-Based Access Control (Who Can Do What)

**Idea:** Different users have different permissions

### Example: Hospital App

```
ADMIN:
├─ View all users
├─ Delete any user
├─ View system stats
└─ Manage staff

DOCTOR:
├─ View their own patients
├─ Update patient records
└─ Cannot delete users

PATIENT:
├─ View their own data
├─ Book appointments
└─ Cannot view other patients' data

GUEST:
└─ Can only browse public pages
```

### How Backend checks this:
```
User clicks "Delete User" → Backend asks:
  "Are you an ADMIN?"
  If YES → Allow deletion
  If NO → Deny (show error: "You don't have permission")
```

### In code (simple):
```javascript
if (user.role === "admin") {
  deleteUser();  // Allowed
} else {
  sendError("You can't delete users");  // Denied
}
```

**Interview answer:** "We check user role before allowing actions. Admin can do everything. Patient can only see their own data."

---

## 7. Status Tracking (How Things Progress)

### Example: Order in E-commerce

```
Order created by user
        ↓
    PENDING (Waiting for admin to review)
        ↓
    ASSIGNED (Admin assigned to vendor)
        ↓
    PROCESSING (Vendor packing the item)
        ↓
    SHIPPED (Item left warehouse)
        ↓
    DELIVERED (Item reached user)
        ↓
    COMPLETED (User confirmed receipt)
```

### How we store this:

**Database table:**
```
order_id | user_id | status      | timestamp
---------|---------|-------------|-------------------
101      | 5       | PENDING     | 2026-01-15 10:00
102      | 7       | SHIPPED     | 2026-01-15 11:30
103      | 9       | COMPLETED   | 2026-01-15 14:45
```

### When status changes:
```
Backend updates: UPDATE orders SET status = 'SHIPPED' WHERE order_id = 101
Timestamp auto-updates to current time
Frontend shows updated status to user
```

**Interview answer:** "Each order has a status column. Backend updates it based on business rules. Frontend shows current status."

---

## 8. How Fake Data is Prevented

### Problem: Bad actor sends fake data

```
❌ Frontend validation (can be bypassed):
   User opens DevTools, modifies form

✅ Backend validation (MUST do):
   Even if fake data reaches backend, we check it
```

### Example: Age verification

**BAD (Frontend only):**
```javascript
if (age < 18) {
  alert("Must be 18+");  // User bypasses this in DevTools
}
```

**GOOD (Backend validation):**
```javascript
if (user.age < 18) {
  return error("Must be 18+");  // Backend checks even if fake
}
```

### Checklist Backend validates:

```
☐ Is age actually a number? (not "hello")
☐ Is age between 0-150? (not -50 or 999)
☐ Is email format correct? (has @ and domain)
☐ Is password strong enough? (8+ chars, mix of letters/numbers)
☐ Does user exist before updating? (not updating ghost user)
☐ Is file actually an image? (not a virus disguised as JPG)
☐ Is amount positive? (not negative to hack refunds)
```

**Interview answer:** "Frontend shows nice errors, but Backend MUST validate everything. Never trust Frontend data."

---

## 9. What Happens When User Clicks "Submit"

### Example: Sign up form

```
STEP 1: Frontend collects data
        ├─ Name: "John"
        ├─ Email: "john@example.com"
        ├─ Password: "SecurePass123"
        └─ Date of birth: "2006-01-15"

STEP 2: Frontend validates (basic check)
        ├─ Is email not empty? ✓
        ├─ Is password 8+ chars? ✓
        └─ If any fail → show error, stop here

STEP 3: Frontend sends to Backend (API call)
        POST /signup
        {
          name: "John",
          email: "john@example.com",
          password: "SecurePass123",
          dob: "2006-01-15"
        }

STEP 4: Backend receives request
        ├─ Check: Is email already registered? No ✓
        ├─ Check: Is age >= 18? (Verify from DOB) Yes ✓
        ├─ Check: Is password strong? Yes ✓
        ├─ Encrypt password (don't store plain text)
        ├─ Save to Database:
        │  INSERT users (name, email, password_hash, dob)
        │  VALUES ("John", "john@example.com", "xyz789", "2006-01-15")
        └─ Send back: { success: true, user_id: 123 }

STEP 5: Frontend receives response
        ├─ If success → Redirect to login page
        └─ If error → Show error message

User sees: "Account created! Please login."
```

---

## 10. What We'll Build in 24–48 Hours (MVP Scope)

### ❌ Too ambitious:
- Mobile app + Web app + Admin dashboard + AI + Chat
- Database with 50+ tables
- Beautiful UI with animations

### ✅ Realistic MVP:
- **1 main feature** that works end-to-end
- **3-4 supporting features** that make it useful
- **Simple, clean UI** (not beautiful, just clear)
- **Database with 4-6 tables** (not more)
- **Real data validation** (not skipped)

### Example MVP for a "Task Management" app:

```
CORE FEATURE:
├─ Users can create tasks
├─ Users can mark tasks as done
├─ Users see their task list

SUPPORTING FEATURES:
├─ Users can edit task title
├─ Users can delete tasks
├─ Users can filter by status (Done/Pending)

DATABASE TABLES:
├─ users (id, name, email, password)
├─ tasks (id, user_id, title, status, created_at)

PAGES:
├─ Sign up / Login
├─ Task list page
├─ Create task page

WHAT WE DON'T BUILD:
❌ Mobile app (too much work)
❌ Chat between users (too complex)
❌ AI recommendations (too risky, might not work)
❌ Beautiful animations (nice-to-have, cut it)
❌ Admin dashboard (not in MVP)
```

**Interview answer:** "We build ONE working end-to-end feature, not 10 half-done features. Quality > Quantity."

---

## 11. 10 Common Interview Questions & Answers

### Q1: "Walk us through your system architecture."
**A:** "User → Frontend → Backend → Database. Frontend shows UI, Backend processes logic, Database stores data. All three talk via APIs."

### Q2: "Where do you store images?"
**A:** "Not in Database (too slow, too big). In Cloud Storage (S3, GCP, Firebase). Database stores only the URL."

### Q3: "What is an API?"
**A:** "A bridge between Frontend and Backend. Frontend sends request (GET /users), Backend responds with data (JSON)."

### Q4: "How do you prevent fake data?"
**A:** "Frontend validates for UX. Backend MUST validate before saving. Never trust Frontend."

### Q5: "How do you handle user roles?"
**A:** "Check user role before allowing action. Admin can delete. User can't. If not allowed, send error."

### Q6: "What happens when user submits a form?"
**A:** "Frontend sends data to Backend. Backend validates. If valid, saves to Database. Sends success response. Frontend shows result."

### Q7: "How do you track order status?"
**A:** "Database has status column. When status changes, we update that column with timestamp. Frontend reads and displays."

### Q8: "Why can't we build an iOS app in a hackathon?"
**A:** "iOS needs Mac, special tools, testing on devices, more time. Web is faster: Just HTML+CSS+JS, deploy in minutes."

### Q9: "What do you do if Backend crashes?"
**A:** "Users can still use Frontend (offline). When Backend is back, data syncs. We store critical data in Database, not just memory."

### Q10: "How do you know if your app is scalable?"
**A:** "If 1 user works, will 1000 users work? We check: Database indexes, API response time, Storage size, Cache strategy."

---

## 12. 30-Second Team Pitch (Any Teammate Can Say This)

```
"We're building [APP NAME] to solve [PROBLEM].

Users can [MAIN ACTION] in 3 clicks.
They save [TIME/MONEY/EFFORT].

How it works: Frontend → Backend → Database.

We validate data everywhere.
We use [Tech: Firebase, Next.js, etc.].

We'll have a working MVP built.
We're ready to execute."
```

### Real Example:
```
"We're building a Task Manager to help students organize
assignments without using WhatsApp.

Users can create, edit, and mark tasks as done in 3 clicks.
They save 10 minutes per day on admin.

How it works: Frontend (React) → Backend (Node.js) → Database (Firebase).

We validate all input in Backend.

We'll have sign-up, task creation, and filtering working.
We're ready to execute."
```

---

## 13. 5 Killer Lines to Use If You're Stuck

### Line 1: "Let me break it down into layers."
```
Frontend (what user sees) → Backend (logic) → Database (data)
Works for ANY system question.
```

### Line 2: "Frontend can lie. Backend checks everything."
```
Use this for questions about validation, security, fake data.
Shows you know the real difference.
```

### Line 3: "We prioritize: 2-3 working feature > 10 half-done features."
```
Use this if asked "Why didn't you build X?"
Shows smart prioritization.
```

### Line 4: "That's in our Phase 2 roadmap."
```
Use this for nice-to-have features you didn't build.
Shows you thought ahead.
```

### Line 5: "We store in Database if we need to search it. Storage if it's just a file."
```
Use for any data storage question.
Always correct answer.
```

---

## Quick Reference: Role Cheat Sheet

| Role | Frontend | Backend | Database | Storage |
|------|----------|---------|----------|---------|
| **Frontend Dev** | ✅ Build UI | ❌ | ❌ | ❌ |
| **Backend Dev** | ❌ | ✅ Build APIs | ✅ Read/Write | ✅ Upload/Download |
| **DevOps** | ❌ | ❌ Deploy | ❌ Maintain | ❌ Maintain |
| **Full-stack** | ✅ | ✅ | ✅ | ✅ |

**Your team:** Decide who does what. Frontend dev handles UI. Backend dev handles Backend + Database logic. DevOps handles deployment.

---

## Quick Reference: Status Codes to Mention

When Frontend talks to Backend, Backend responds with a code:

```
200 = Success (✓ worked)
400 = Bad request (❌ Frontend sent wrong data)
401 = Not authorized (❌ User not logged in)
403 = Forbidden (❌ User doesn't have permission)
404 = Not found (❌ Resource doesn't exist)
500 = Server error (❌ Backend crashed)
```

**Interview mention:** "Our APIs return proper status codes so Frontend knows what happened."

---

## Quick Reference: What NOT to Say

```
❌ "We'll use Kubernetes" (overkill, wastes time)
❌ "We built it with React Native" (too complex for hackathon)
❌ "Database stores images" (wrong, slow, expensive)
❌ "Frontend validation is enough" (insecure)
❌ "We built 15 features" (no, you built 3 features and 12 broken ones)
❌ "The frontend will handle everything" (impossible, Backend is critical)
```

---

## Interview Prep Checklist

```
☐ Understand your system flow (User → Frontend → Backend → Database)
☐ Know where images are stored (Cloud Storage, not Database)
☐ Explain API in 1 sentence
☐ Know what Backend does (logic, validation, data management)
☐ Understand role-based access (check permission before action)
☐ Explain status tracking (column update + timestamp)
☐ Know how to prevent fake data (Backend validation)
☐ Describe form submission flow (step by step)
☐ Remember MVP scope (2-3 feature, done. Not 10 features, half-done)
☐ Memorize 30-second pitch
☐ Practice 5 killer lines
☐ Answer all 10 common questions
☐ Show diagram to interviewer (system flow)
```

---

## The Golden Rule

> **"Frontend can be beautiful but wrong. Backend must be correct but can be ugly."**

If you remember nothing else, remember this.

---

## How to Use This File

1. **Read it once** – Understand the flow
2. **Memorize the 5 killer lines** – Use when stuck
3. **Practice the 30-second pitch** – Your team delivers it
4. **Review the 10 Q&As** – Know your answers
5. **Before interview** – Quick mental run-through

---
