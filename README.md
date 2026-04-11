# REALTIME CHAT APP - COMPLETE TECHNICAL DOCUMENTATION

## 1. Project Overview
**Realtime_Chat_App** is a full-stack real-time messaging application enabling users to create accounts, update profiles with avatar uploads, discover other users, and exchange text/image messages with real-time delivery via WebSocket.

**Business Problem Solved**: Secure, scalable 1-1 chat platform with rich UI (themes, skeletons), profile customization, and production-ready authentication (JWT cookies).

**System Summary**: 
- SPA Frontend → REST APIs + Socket.io → MongoDB Backend
- 30+ DaisyUI themes, responsive chat UI, online status indicators
- Cloudinary for image storage, bcrypt for passwords

## 2. Technology Stack

### Frontend
| Category | Technology |
|----------|------------|
| Framework | React 19 + Vite |
| Styling | Tailwind CSS + DaisyUI (30+ themes) |
| State Mgmt | Zustand (auth/chat/themes stores) |
| Routing | React Router DOM 7 |
| HTTP Client | Axios (withCredentials for cookies) |
| Real-time | Socket.io-client |
| UI Components | Lucide React icons, React Hot Toast |
| Dev | ESLint 9, PostCSS |

### Backend
| Category | Technology |
|----------|------------|
| Runtime | Node.js 20+ (ES Modules) |
| Framework | Express.js |
| Database | MongoDB + Mongoose |
| Auth | JWT (cookies) + bcryptjs |
| Real-time | Socket.io 4 |
| File Storage | Cloudinary |
| Middleware | cors, cookie-parser |
| Dev | Nodemon |

## 3. Features List (COMPLETE)
✓ User Registration (fullName, email, password≥6)
✓ User Login/Logout (JWT cookie)
✓ Profile Update (avatar upload to Cloudinary, 300x300 crop)
✓ User Discovery (sidebar with all users except self)
✓ 1-1 Private Chat (text + images)
✓ **Real-time Messaging** (Socket.io delivery)
✓ **Online Status Indicators** (socket map)
✓ 30+ Chat Themes (DaisyUI)
✓ Responsive Chat UI (sidebar + container)
✓ Loading Skeletons (sidebar/messages)
✓ Auth Persistence (auto-check on load)
✓ Error Handling + Toast Notifications
✓ Protected Routes (frontend/backend)

## 4. Folder Structure Explanation

```
Realtime_Chat_App/
├── backend/                    # Express + Socket.io API
│   ├── src/
│   │   ├── controllers/        # Business logic
│   │   ├── lib/                # Config (db, socket, utils)
│   │   ├── middleware/         # JWT auth
│   │   ├── models/             # Mongoose schemas
│   │   ├── routes/             # Express routers
│   │   └── seeds/              # Initial data
│   └── package.json            # Node deps
├── frontend/                   # React SPA
│   ├── public/                 # Static assets
│   ├── src/
│   │   ├── components/         # Chat UI components
│   │   ├── constants/          # Themes list
│   │   ├── lib/                # Axios client, utils
│   │   ├── pages/              # Auth + Home
│   │   ├── store/              # Zustand hooks
│   │   └── App.jsx             # Root layout
│   ├── tailwind.config.js      # DaisyUI themes
│   └── vite.config.js          # React 19 config
└── README.md                   # High-level guide
```

## 5. System Architecture

```
CLIENT (React 19 + Zustand + Socket.io-client)
        ↓ withCredentials=true (JWT cookie)
REST API (Express 4.18)
        ↓ protectRoute middleware → req.user
Controllers → Mongoose Models → MongoDB
        ↓ Real-time
Socket.io Server (userSocketMap tracking)
        ↓ HTTP long-polling + WS upgrade
MongoDB (users + messages collections)
Cloudinary (profile pics, chat images)
```

**Request Lifecycle**:
1. Frontend Axios POST → Backend Route
2. protectRoute verifies JWT cookie → attaches req.user
3. Controller → Model.find/create → DB
4. Socket.io emit to receiver socketId
5. Frontend store updates → UI re-renders

## 6. DFD (Data Flow Diagram)

### Level 0
```
Actors: Registered User

User ──► Login/Signup ──► Backend ──► MongoDB
             ▲                    │
             └──── JWT Cookie ─────┘

User ──► Send Message ──► Backend ──► MongoDB
                    │             ▲
                    └─ Socket.io ──┘
                          │
                    Other User (Real-time)
```

### Level 1
```
Auth Flow:    Login → bcrypt.compare → jwt.sign → cookie.set → store.authUser
Chat Flow:    Sidebar(users) → Select → getMessages(both directions) → MessageInput
RT Flow:      sendMessage → cloudinary?(image) → save → socket.emit(receiverSocketId)
```

## 7. ERD (Entity Relationship Diagram)

**Entities**:

```
USER
├── _id: ObjectId [PK]
├── fullName: String [REQ]
├── email: String [REQ, UNIQUE]
├── password: String [REQ, bcrypt]
├── profilePic: String [Cloudinary URL]
└── timestamps

MESSAGE
├── _id: ObjectId [PK]
├── senderId: ObjectId → USER [REQ]
├── receiverId: ObjectId → USER [REQ]
├── text: String [OPT]
├── image: String [OPT, Cloudinary]
└── timestamps
```

**Relationships**: Many-to-Many (users ↔ messages via sender/receiver)

## 8. SECURITY ARCHITECTURE

| Security Layer | Implementation |
|----------------|----------------|
| **Password** | bcryptjs (salt=10) |
| **Auth** | JWT (cookie, server-side verify) |
| **Sessions** | Cookie-only (no localStorage) |
| **CSRF** | Axios withCredentials + CORS origin |
| **Rate Limit** | Express json/urlencoded limit=10mb |
| **Input Val** | Controller checks (password≥6, required fields) |
| **SQL Inj** | Mongoose ODM |
| **XSS** | MongoDB strings (frontend sanitization recommended) |

**Protected Routes**: All /api/messages/* require `protectRoute` middleware.

## 9. JWT AUTH FLOW

```
1. POST /api/auth/login {email,password}
   ↓
2. Controller: User.findOne(email) → bcrypt.compare
   ↓ SUCCESS
3. lib/utils.generateToken(user._id, res) → jwt.sign → res.cookie('jwt', ...)
4. Frontend store.authUser = response.data
   ↓
5. PROTECTED: middleware/auth.middleware.js
   ↓ req.cookies.jwt → jwt.verify(JWT_SECRET) → User.findById(decoded.userid)
   ↓ req.user = user (-password) → next()
6. Response carries req.user data
```

**Token Storage**: HTTP-only cookie (secure against XSS)
**Expiration**: JWT_SECRET expiry (not specified, default)
**Refresh**: None (re-login on expiry)

## 10. APPLICATION FLOW (STEPWISE)

```
App Startup:
1. main.jsx → BrowserRouter → App.jsx
2. useAuthStore.checkAuth() → /api/auth/check → store.authUser
3. socket.connect(userId) → onlineUsers update

Chat Flow:
1. Sidebar → /api/messages/users → filtered users list
2. Click user → /api/messages/:id → bidirectional messages
3. MessageInput → /api/messages/send/:id → socket.emit → store update
4. UI: ChatContainer re-renders (MessageSkeleton → real)
```

## 11. BACKEND INTERNAL FLOW

```
Route → Middleware(protectRoute) → Controller → Model → DB → Response

Example: POST /api/messages/send/:id
1. express.Router() matches "/send/:id"
2. protectRoute → req.user = loggedInUser
3. message.controller.sendMessage → cloudinary?.upload(image)
4. new Message(senderId=req.user._id, receiverId) → .save()
5. getReceiverSocketId(receiverId) → io.to().emit("newMessage")
6. res.json(newMessage)
```

## 12. FRONTEND INTERNAL FLOW

```
Entry: index.html → main.jsx → App.jsx (Router)

Routing: Login/Signup/Profile/Setting/HomePage

State: useAuthStore (singleton) → API calls + socket lifecycle

Component Tree:
├── Navbar
├── Sidebar (users list + online)
├── ChatContainer
│   ├── ChatHeader
│   ├── Messages (bidirectional)
│   └── MessageInput
└── Skeletons (loading states)

API Pattern: axiosInstance (withCredentials) → store methods → toast + UI update
```

## 13. API DOCUMENTATION

| Method | Endpoint              | Description                    | Auth | Request Body                  | Response                              |
|--------|-----------------------|--------------------------------|------|--------------------------------|---------------------------------------|
| POST   | `/auth/signup`        | Create new user                | No   | `{fullName, email, password}`  | `{success, _id, fullName, email}`     |
| POST   | `/auth/login`         | User login                     | No   | `{email, password}`            | `{ _id, fullName, email, profilePic}` |
| POST   | `/auth/logout`        | Clear JWT cookie               | No   | `{}`                           | `{message: "Logged out"}`             |
| PUT    | `/auth/update-profile`| Upload profile pic             | YES  | `{profilePic: base64}`         | `updated User (-password)`            |
| GET    | `/auth/check`         | Verify JWT + get user          | YES  | `{}`                           | `User (-password)`                    |
| GET    | `/messages/users`     | Get all users (no self)        | YES  | `{}`                           | `[User[] (-password)]`                |
| GET    | `/messages/:userId`   | Get chat history               | YES  | `{}`                           | `[Message[]]`                         |
| POST   | `/messages/send/:id`  | Send message (text/image)      | YES  | `{text?, image?}`              | `new Message`                         |

### Detailed Endpoints

#### POST /api/auth/signup
**Purpose**: Register new user
**Flow**: Validation → bcrypt.hash → User.save → JWT cookie → response
**Errors**: 400 (missing fields/password<6/email-exists)

#### POST /api/auth/login  
**Purpose**: Authenticate + set JWT cookie
**Flow**: User.findOne → bcrypt.compare → JWT cookie → response
**Errors**: 400 (invalid credentials)

#### PUT /api/auth/update-profile
**Purpose**: Upload avatar (base64 → Cloudinary → 300x300 crop)
**Flow**: protectRoute → cloudinary.upload → User.findByIdAndUpdate
**Headers**: Cookie: jwt=...

## 14. Dependencies Installation

### Backend
```bash
cd backend
npm install
# Creates node_modules + package-lock.json
```

**Key Deps**: express, mongoose, socket.io, jsonwebtoken, bcryptjs, cloudinary

### Frontend  
```bash
cd frontend  
npm install
# Tailwind/DaisyUI auto-setup via postcss.config.js
```

**Key Deps**: react@19, vite@6, zustand@5, socket.io-client, axios, daisyui@3.9

## 15. Environment Variables

**Backend (.env)**:
```
MONGODB_URI=mongodb://localhost:27017/chatapp
JWT_SECRET=your-super-secret-key-here-minimum-32-chars
CLOUDINARY_CLOUD_NAME=xxx
CLOUDINARY_API_KEY=xxx  
CLOUDINARY_API_SECRET=xxx
PORT=5001
NODE_ENV=development
```

## 16. How To Run Project

### Prerequisites
- Node.js 20+
- MongoDB (local or MongoDB Atlas)
- Cloudinary account (free tier OK)

### 1. Backend
```bash
cd backend
cp .env.example .env  # Fill env vars
npm run dev           # nodemon → http://localhost:5001
```

### 2. Frontend (new terminal)  
```bash
cd frontend
npm run dev           # vite → http://localhost:5173
```

### 3. Test Flow
1. Signup → Login → Profile (upload avatar)
2. Home → Sidebar users → Click chat → Send text/image
3. Real-time delivery + online indicators

**Production**: `NODE_ENV=production` → backend serves frontend/dist

## 17. Runtime Flow (After Startup)

```
Backend:5001 ←→ Frontend:5173 (CORS)
  ↓
1. App load → authStore.checkAuth → /auth/check → JWT verify
2. Socket connect(userId) → userSocketMap → emit onlineUsers
3. Sidebar → GET /messages/users → populate contacts
4. Chat select → GET /messages/:id → load history  
5. Message → POST /messages/send/:id → DB + socket emit
6. Receiver store receives → ChatContainer updates
```

## 18. Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `MONGODB_URI invalid` | Missing env | Setup Atlas/local MongoDB |
| `JWT_SECRET undefined` | No .env | Add to backend/.env |
| `CORS error` | Origin mismatch | Update cors.origin in socket.js |
| `Cookie not sent` | withCredentials=false | ✅ Already set in axios.js |
| `User not found` | Expired JWT | Re-login |
| `Cloudinary 401` | Wrong API keys | Verify dashboard |
| `Socket not connecting` | Backend not running | npm run dev (backend) |

## 19. Developer Notes

**Scalability**:
- Redis for socket scaling (userSocketMap → pub/sub)
- MongoDB indexes: {senderId:1, receiverId:1, createdAt:-1}
- Rate limiting (express-rate-limit)

**Performance**:
- Message pagination (current: all history)
- Image thumbnails (Cloudinary already crops)
- Debounce message input

**Security**:
- Input sanitization (DOMPurify frontend)
- HTTPS production
- JWT expiry (add exp claim)
- Socket auth (handshake verify)

**Production Checklist**:
```
[ ] MongoDB Atlas + connection pooling
[ ] Cloudinary signed uploads  
[ ] HTTPS + secure cookies
[ ] PM2/Docker for backend
[ ] Vite build → backend static serve
[ ] Error tracking (Sentry)
```

---
**written by Yash Lagare**
