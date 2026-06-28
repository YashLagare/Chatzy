# PROJECT_DOCUMENTATION

## Project Name
**Chatzy - Realtime Chat App**

### Project Description
A full-stack 1-1 real-time chat application where users can sign up/login, update their profile picture (stored in Cloudinary), browse other users, and exchange text/image messages with real-time delivery using Socket.io.

### Business Problem Solved
Provide a secure, practical real-time messaging experience with:
- persistent authentication (JWT stored in an HTTP-only cookie)
- private 1-1 conversations
- real-time message delivery and online presence indicators
- avatar/profile image upload and message image upload

### Target Users
- Individual users who want private direct messaging
- Developers/interviewers needing a complete full-stack real-time chat codebase

### Key Benefits
- Real-time messaging via **Socket.io**
- Online presence indicators tracked by active socket connections
- JWT cookie-based authentication with server-side verification
- Profile and message images stored in **Cloudinary**
- Responsive UI with **Tailwind CSS + DaisyUI themes**

<img width="1918" height="910" alt="image" src="https://github.com/user-attachments/assets/355e7cc5-d59b-4904-8d5e-423c099652d5" />


---

# 1. EXECUTIVE SUMMARY

### Project Purpose
Chatzy enables authenticated users to exchange private messages (text + images) in real time.

### Core Features
- User signup/login/logout with cookie-based JWT
- Protected profile update (avatar upload to Cloudinary)
- Protected message APIs for 1-1 chat history and messaging
- Real-time message delivery over Socket.io
- Online presence via a socket user map
- Theme customization using DaisyUI themes (saved in localStorage)

### Technology Summary
- **Frontend:** React 19 + Vite, Axios (cookies), React Router DOM, Zustand, Socket.io-client, Tailwind CSS + DaisyUI
- **Backend:** Node.js (ESM) + Express, Socket.io, Mongoose + MongoDB, JWT, bcryptjs, Cloudinary

### Architecture Overview
```text
User
 │
 ▼
Frontend (React + Zustand + Socket.io-client)
 │
 ▼
API Layer (Axios + Express routes)
 │
 ▼
Backend (Express controllers + Socket.io server)
 │
 ▼
Database (MongoDB via Mongoose)

Cloudinary
```

### Business Value
- Reduces friction for building a production-like chat feature set (auth, profile media, messaging, presence)
- Demonstrates a complete end-to-end real-time architecture suitable for portfolio/interview discussions

---

# 2. PROJECT OVERVIEW

### Project Name
Chatzy - Realtime Chat App

### Objective
Deliver secure, private 1-1 real-time messaging with:
- JWT cookie authentication
- image upload (avatars and message attachments)
- online status indicators

### Scope
Implemented in the repository:
- Frontend SPA UI, stores, and socket connection
- Backend REST APIs for auth and messaging
- Socket.io real-time delivery for messages + presence list
- MongoDB persistence for users and messages

Not implemented (explicitly not present):
- Group chats (no group model/routes)
- Refresh tokens / token rotation (JWT expiry handling exists but refresh is not implemented)
- Rate limiting / advanced input sanitization libraries (not present in code)

---

# 3. TECHNOLOGY STACK

## Frontend
- **Framework:** React 19 (with Vite)
- **UI:** Tailwind CSS + DaisyUI
- **Routing:** React Router DOM v7
- **State Management:** Zustand
- **HTTP Client:** Axios (`withCredentials: true`)
- **Real-time:** Socket.io-client
- **UI Icons / UX:** lucide-react, react-hot-toast
- **Build Tools:** Vite + PostCSS + ESLint

## Backend
- **Runtime:** Node.js using ES Modules
- **Framework:** Express
- **Real-time:** Socket.io (server + `io.to(socketId).emit(...)`)
- **Database:** MongoDB using Mongoose
- **Auth:** JWT signed and verified server-side
- **Password Hashing:** bcryptjs
- **Media Storage:** Cloudinary
- **Middleware:** express.json, express.urlencoded, cookie-parser, cors

## Database
- **Type:** MongoDB
- **ORM/ODM:** Mongoose
- **Storage Strategy:**
  - `User` documents store profile metadata and Cloudinary URL
  - `Message` documents store sender/receiver references and optional text/image URL

## Authentication
Implemented authentication:
- **JWT** stored in an **HTTP-only cookie** named `jwt`
- Server verifies token via `protectRoute` middleware (`jsonwebtoken.verify`)

No other auth providers (OAuth/Firebase/etc.) exist in the codebase.

## DevOps & Deployment
- Backend production mode can serve the built frontend from `frontend/dist` (configured in `backend/src/index.js`)
- No CI/CD configuration is present in the repository code we analyzed
- No Docker/Kubernetes manifests are present

---

# 4. FEATURES LIST

| Feature Name | Purpose | User Benefit | Related Components |
|---|---|---|---|
| Signup (POST `/api/auth/signup`) | Create user with hashed password | Register and access Chatzy | `backend/src/controllers/auth.controller.js`, `backend/src/routes/auth.route.js` |
| Login (POST `/api/auth/login`) | Verify credentials and set JWT cookie | Start a session securely | `auth.controller.js`, `utils.generateToken`, `axiosInstance` |
| Logout (POST `/api/auth/logout`) | Clear JWT cookie | End session | `auth.controller.js` |
| Auth Check (GET `/api/auth/check`) | Validate JWT and return user | Persist authentication on refresh | `protectRoute`, `checkAuth`, `useAuthStore.checkAuth` |
| Protected Route Middleware | JWT verification + attach `req.user` | Prevent unauthorized access | `backend/src/middleware/auth.middleware.js` |
| Profile Update (PUT `/api/auth/update-profile`) | Upload avatar to Cloudinary and update user | Personalized profile | `updateProfile`, `cloudinary`, `ProfilePage` |
| User Discovery (GET `/api/messages/users`) | List all users excluding self | Find contacts | `getUserForSidebar`, `Sidebar.jsx`, `useChatStore.getUsers` |
| Chat History (GET `/api/messages/:id`) | Query messages between two users | View private conversation history | `getMessages`, `ChatContainer`, `useChatStore.getMessages` |
| Send Message (POST `/api/messages/send/:id`) | Save message + optional image + realtime emit | Real-time 1-1 messaging | `sendMessage`, `Message` model, `useChatStore.sendMessage` |
| Real-time Delivery (Socket.io `newMessage`) | Emit message to receiver socket if online | Immediate updates | `backend/src/lib/socket.js`, `message.controller.js`, `useChatStore.subscribeToMessages` |
| Online Presence (`getOnlineUsers`) | Maintain `userSocketMap` and broadcast online user IDs | Show Online/Offline in UI | `backend/src/lib/socket.js`, `useAuthStore.connectSocket`, `Sidebar`, `ChatHeader` |
| Theme Selection | Store DaisyUI theme name in localStorage | UI personalization | `frontend/src/pages/SettingPage.jsx`, `useThemesStore` |

Feature Screenshot Placeholder:

[INSERT_FEATURE_SCREENSHOT_HERE]

---

# 5. FOLDER STRUCTURE

```text
Realtime_Chat_App/
├── backend/
│   ├── package.json
│   └── src/
│       ├── controllers/
│       │   ├── auth.controller.js
│       │   └── message.controller.js
│       ├── lib/
│       │   ├── cloudinary.js
│       │   ├── db.js
│       │   ├── socket.js
│       │   └── utils.js
│       ├── middleware/
│       │   └── auth.middleware.js
│       ├── models/
│       │   ├── message.model.js
│       │   └── user.model.js
│       ├── routes/
│       │   ├── auth.route.js
│       │   └── message.route.js
│       └── seeds/
│           └── user.seed.js
├── frontend/
│   ├── package.json
│   ├── index.html
│   ├── public/
│   │   ├── avatar.png
│   │   ├── favicon-32x32.png
│   │   ├── favicon.ico
│   │   └── vite.svg
│   ├── src/
│   │   ├── App.jsx
│   │   ├── components/
│   │   │   ├── ChatContainer.jsx
│   │   │   ├── ChatHeader.jsx
│   │   │   ├── MessageInput.jsx
│   │   │   ├── Navbar.jsx
│   │   │   ├── NoChatSelected.jsx
│   │   │   ├── Sidebar.jsx
│   │   │   └── Skeleton/
│   │   │       ├── MessageSkeleton.jsx
│   │   │       └── SidebarSkeleton.jsx
│   │   ├── constants/
│   │   │   └── index.js
│   │   ├── lib/
│   │   │   ├── axios.js
│   │   │   └── utils.js
│   │   ├── pages/
│   │   │   ├── HomePage.jsx
│   │   │   ├── LoginPage.jsx
│   │   │   ├── ProfilePage.jsx
│   │   │   ├── SettingPage.jsx
│   │   │   └── SignupPage.jsx
│   │   └── store/
│   │       ├── useAuthStore.js
│   │       ├── useChatStore.js
│   │       └── useThemesStore.js
│   ├── tailwind.config.js
│   └── vite.config.js
└── PROJECT_DOCUMENTATION.md
```

Important notes:
- Backend is an Express app with separate routers for `auth` and `messages`.
- Socket.io runs on the same HTTP server as Express and is initialized in `backend/src/lib/socket.js`.
- Frontend uses Zustand stores to manage auth state, online users, and chat state.

---

# 6. SYSTEM ARCHITECTURE

### Architecture Pattern
- Monolithic backend (Express + Socket.io in a single Node process)
- MVC-ish separation on backend: `routes/` → `controllers/` → `models/`
- Frontend: component-based UI + store-managed state

### Request Lifecycle
```text
Client (Browser)
   │
   ▼
Frontend (Axios)
   │
   ▼
Express Route Handler
   │
   ▼
Middleware: protectRoute (JWT verify)
   │
   ▼
Controller
   │
   ▼
Mongoose Model / MongoDB
   │
   ▼
JSON Response
```

### Data Flow / Communication Model
- **REST:**
  - Signup/Login/Logout/Check
  - Fetch user list
  - Fetch messages between two users
  - Send message (saves to DB)
- **WebSocket (Socket.io):**
  - Connection includes `userId` query parameter
  - Receiver socket id is looked up from an in-memory `userSocketMap`
  - Backend emits `newMessage` to receiver socket
  - Backend broadcasts online user ids via `getOnlineUsers`

---

# 7. DATABASE DESIGN

Implemented Mongoose schemas/models:

## USER Entity (`backend/src/models/user.model.js`)
Purpose: Stores application users.

Fields:
| Field | Type | Required | Constraints / Notes |
|---|---|---:|---|
| `email` | String | Yes | `unique: true` |
| `fullName` | String | Yes | Required |
| `password` | String | Yes | `minlength: 6` (hash is stored) |
| `profilePic` | String | No | Defaults to `""` |
| `timestamps` | Mongoose timestamps | - | `createdAt`, `updatedAt` |

Relationships:
- `Message.senderId` and `Message.receiverId` reference `User` documents.

## MESSAGE Entity (`backend/src/models/message.model.js`)
Purpose: Stores 1-1 chat messages (text and/or image).

Fields:
| Field | Type | Required | Constraints / Notes |
|---|---|---:|---|
| `senderId` | ObjectId (ref: `User`) | Yes | References User |
| `receiverId` | ObjectId (ref: `User`) | Yes | References User |
| `text` | String | No | Optional |
| `image` | String | No | Optional Cloudinary secure URL |
| `timestamps` | Mongoose timestamps | - | `createdAt`, `updatedAt` |

Relationships:
- Implicit many-to-many relationship between users via messages.

---

# 8. ENTITY RELATIONSHIP DIAGRAM (ERD)

```text
Users
├── _id
├── email (unique)
├── fullName
├── password (hashed)
└── profilePic

        1:N (as sender)
        ───────────────

Messages
├── _id
├── senderId (→ Users._id)
├── receiverId (→ Users._id)
├── text
└── image

        N:1 (as receiver)
```

---

# 9. SECURITY ARCHITECTURE

### Authentication
- JWT stored in an HTTP-only cookie named `jwt`
- Verified via `protectRoute` middleware

### Authorization
- Authorization in practice is route protection by `protectRoute`
- `req.user` is attached from the JWT and used as the authenticated user context

### Session / Token Strategy (as implemented)
- Token is generated in `backend/src/lib/utils.js` using `jwt.sign({ userid: userId }, JWT_SECRET, { expiresIn: '7d' })`
- Cookie options:
  - `httpOnly: true`
  - `secure: NODE_ENV === 'production'`
  - `sameSite: 'lax'`
  - `maxAge: 7 days`

### Protected Routes
- `PUT /api/auth/update-profile`
- `GET /api/auth/check`
- `GET /api/messages/users`
- `GET /api/messages/:id`
- `POST /api/messages/send/:id`

### Validation / Input Handling (existing)
- Signup validates required fields and password length ≥ 6
- Login requires email/password match
- updateProfile requires `profilePic`
- sendMessage accepts `text` and optional `image`

### Encryption
- Passwords are hashed using bcryptjs before storage.
- JWT signing uses `JWT_SECRET`.

### Not implemented (explicit)
- CSRF tokens (not present)
- Rate limiting middleware (not present)
- Role-based authorization (not present)

---

# 10. AUTHENTICATION FLOW

```text
User submits credentials
    │
    ▼
Frontend (Axios)
    │
    ▼
POST /api/auth/login or POST /api/auth/signup
    │
    ▼
Backend Controller
  - bcrypt.compare (login)
  - bcrypt.hash (signup)
    │
    ▼
Token Creation (generateToken)
  jwt.sign({ userid }, JWT_SECRET, { expiresIn: '7d' })
    │
    ▼
Set HTTP-only Cookie: jwt
    │
    ▼
Response with user details (no password)

For protected requests:

Browser sends cookie automatically (withCredentials)
    │
    ▼
protectRoute middleware
    │
    ▼
jwt.verify(token, JWT_SECRET)
    │
    ▼
User.findById(decoded.userid).select('-password')
    │
    ▼
Attach req.user and continue
    │
    ▼
Protected controller response
```

---

# 11. APPLICATION FLOW

```text
Application Start
    │
    ▼
Frontend main.jsx mounts App
    │
    ▼
App.jsx loads Zustand stores
    │
    ▼
useAuthStore.checkAuth()
    │
    ▼
GET /api/auth/check (JWT cookie verification)
    │
    ▼
Connect Socket.io with query userId
    │
    ▼
Render Home routes
    │
    ▼
User selects contact
    │
    ▼
GET /api/messages/:id (chat history)
    │
    ▼
MessageInput sends:
POST /api/messages/send/:id
    │
    ▼
Socket emits newMessage to receiver socket
    │
    ▼
Receiver UI updates via subscribeToMessages()
```

---

# 12. BACKEND INTERNAL FLOW

```text
HTTP Request
   │
   ▼
Express Router
   │
   ▼
Middleware: protectRoute (JWT verify)
   │
   ▼
Controller
   │
   ▼
Mongoose Model operations
   │
   ▼
Database (MongoDB)
   │
   ▼
JSON Response

Real-time event for sending message:

Controller: sendMessage
   │
   ▼
If receiver is online: io.to(receiverSocketId).emit('newMessage', message)
   │
   ▼
Client receives and updates store
```

---

# 13. FRONTEND INTERNAL FLOW

```text
Entry Point (index.html)
    │
    ▼
main.jsx
    │
    ▼
App.jsx (Router)
    │
    ▼
Page Component (Home/Login/Signup/Profile/Settings)
    │
    ▼
ChatContainer / Sidebar
    │
    ▼
useChatStore fetches data via axiosInstance
    │
    ▼
subscribeToMessages attaches socket listener
    │
    ▼
State update triggers re-render
```

---

# 14. API DOCUMENTATION

Base paths:
- REST base: `/api` (frontend axios config)

Auth is required on protected message routes and check/update-profile.

| Method | Endpoint | Description | Auth Required |
|---|---|---|---|
| POST | `/api/auth/signup` | Create new user | No |
| POST | `/api/auth/login` | Login and set JWT cookie | No |
| POST | `/api/auth/logout` | Logout and clear JWT cookie | No |
| PUT | `/api/auth/update-profile` | Upload avatar to Cloudinary and update user | Yes |
| GET | `/api/auth/check` | Verify JWT and return user | Yes |
| GET | `/api/messages/users` | Get users excluding the logged-in user | Yes |
| GET | `/api/messages/:id` | Get chat history between logged-in user and `:id` | Yes |
| POST | `/api/messages/send/:id` | Save a message and emit it via Socket.io | Yes |

## Detailed Endpoints

### POST `/api/auth/signup`
Purpose: Register new user.

Auth Requirements: No

Request Body:
```json
{
  "fullName": "string",
  "email": "string",
  "password": "string"
}
```

Success Response:
```json
{
  "success": true,
  "_id": "ObjectId",
  "fullName": "string",
  "email": "string",
  "profilePic": "string"
}
```

Error Responses:
- `400` `{ "message": "All fields are required" }`
- `400` `{ "message": "Password must be at least 6 characters" }`
- `400` `{ "message": "Email already exists" }`
- `500` `{ "message": "Internal Server Error" }`

Internal Flow:
```text
Route
  │
  ▼
Controller: signup
  │
  ▼
bcrypt.genSalt + bcrypt.hash
  │
  ▼
User.save
  │
  ▼
generateToken (JWT cookie)
  │
  ▼
Response
```

### POST `/api/auth/login`
Purpose: Authenticate and set JWT cookie.

Auth Requirements: No

Request Body:
```json
{
  "email": "string",
  "password": "string"
}
```

Success Response:
```json
{
  "_id": "ObjectId",
  "fullName": "string",
  "email": "string",
  "profilePic": "string"
}
```

Error Responses:
- `400` `{ "message": "Invalid credentials" }`
- `500` `{ "message": "Internal Server Error" }`

Internal Flow:
```text
Route
  │
  ▼
Controller: login
  │
  ▼
User.findOne({ email })
  │
  ▼
bcrypt.compare
  │
  ▼
generateToken (JWT cookie)
  │
  ▼
Response
```

### POST `/api/auth/logout`
Purpose: Clear JWT cookie.

Auth Requirements: No

Request Body: none

Success Response (as implemented):
```json
{
  "message": "Logged out successfully"
}
```

Internal Flow:
```text
Route
  │
  ▼
Controller: logout
  │
  ▼
res.cookie('jwt','',{maxAge:0})
  │
  ▼
Response
```

### PUT `/api/auth/update-profile`
Purpose: Upload profile avatar to Cloudinary and update user document.

Auth Requirements: Yes (`protectRoute`)

Request Body:
```json
{
  "profilePic": "base64-data-url-or-string"
}
```

Success Response:
- `200` returns updated `User` document (as returned by `findByIdAndUpdate` with `{ new: true }`)

Error Responses:
- `400` `{ "message": "Profile pic is required" }`
- `500` `{ "message": "Internal Server Error" }`

Internal Flow:
```text
Route (protectRoute)
  │
  ▼
Controller: updateProfile
  │
  ▼
cloudinary.uploader.upload(profilePic, { folder, transformation })
  │
  ▼
User.findByIdAndUpdate(userId, { profilePic: secure_url })
  │
  ▼
Response
```

### GET `/api/auth/check`
Purpose: Verify JWT and return logged-in user.

Auth Requirements: Yes (`protectRoute`)

Request Body: none

Success Response:
- `200` JSON = `req.user` (password excluded via `select('-password')`)

Internal Flow:
```text
Route (protectRoute)
  │
  ▼
Controller: checkAuth
  │
  ▼
res.status(200).json(req.user)
```

### GET `/api/messages/users`
Purpose: Get all users except the authenticated user.

Auth Requirements: Yes (`protectRoute`)

Request Body: none

Success Response:
- `200` array of users (password excluded)

Error Responses:
- `500` `{ "message": "Internal Server Error" }`

Internal Flow:
```text
Route (protectRoute)
  │
  ▼
Controller: getUserForSidebar
  │
  ▼
User.find({ _id: { $ne: loggedInUserId } }).select('-password')
  │
  ▼
Response
```

### GET `/api/messages/:id`
Purpose: Get chat history between authenticated user and `:id`.

Auth Requirements: Yes (`protectRoute`)

URL Parameters:
- `:id` = other user id

Success Response:
- `200` array of `Message` documents

Error Responses:
- `500` `{ "message": "Internal Server Error" }`

Internal Flow:
```text
Route (protectRoute)
  │
  ▼
Controller: getMessages
  │
  ▼
Message.find({
  $or: [
    { senderId: myId, receiverId: userToChatId },
    { senderId: userToChatId, receiverId: myId }
  ]
})
  │
  ▼
Response
```

### POST `/api/messages/send/:id`
Purpose: Send a message (optional image) to `:id`.

Auth Requirements: Yes (`protectRoute`)

URL Parameters:
- `:id` = receiver user id

Request Body:
```json
{
  "text": "string", 
  "image": "base64-image-string (optional)"
}
```

Success Response:
- `201` returns created `Message` document

Error Responses:
- `500` `{ "message": "Internal Server Error" }`

Internal Flow:
```text
Route (protectRoute)
  │
  ▼
Controller: sendMessage
  │
  ▼
If image: cloudinary.uploader.upload(image)
  │
  ▼
Message.create({ senderId, receiverId, text, image })
  │
  ▼
Look up receiver socket id: getReceiverSocketId(receiverId)
  │
  ▼
If receiver is online: io.to(socketId).emit('newMessage', newMessage)
  │
  ▼
Response
```

---

# 15. THIRD-PARTY INTEGRATIONS

## Cloudinary
Used for:
- Profile picture uploads (with transformation to 300x300 crop)
- Message image uploads

Integration flow:
```text
Client uploads base64 image string
    │
    ▼
Backend controller uploads to Cloudinary
    │
    ▼
Cloudinary returns secure_url
    │
    ▼
Backend saves secure_url in MongoDB (User.profilePic or Message.image)
    │
    ▼
Client renders image from secure_url
```

## Socket.io
Used for:
- Real-time message delivery (`newMessage`)
- Online presence updates (`getOnlineUsers`)

Integration flow:
```text
Frontend connects to Socket.io
    │
    ▼
Backend stores socket.id in userSocketMap[userId]
    │
    ▼
Backend emits getOnlineUsers with list of online user IDs
    │
    ▼
When a message is sent, backend emits newMessage to receiver socket
```

No payment gateways, email services, analytics, or external REST APIs are implemented.

---

# 16. ENVIRONMENT VARIABLES

Detected from code:

| Variable | Purpose | Required |
|---|---|---:|
| `MONGODB_URI` | MongoDB connection string | Yes |
| `JWT_SECRET` | Secret used to sign/verify JWT | Yes |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name | Yes |
| `CLOUDINARY_API_KEY` | Cloudinary API key | Yes |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret | Yes |
| `PORT` | Backend server port | Optional (default 5001) |
| `NODE_ENV` | Affects cookie secure flag and static serve logic | Optional |

Environment variable examples:
```env
MONGODB_URI=mongodb://localhost:27017/chatapp
JWT_SECRET=your-super-secret-key
CLOUDINARY_CLOUD_NAME=xxxx
CLOUDINARY_API_KEY=xxxx
CLOUDINARY_API_SECRET=xxxx
PORT=5001
NODE_ENV=development
```

---

# 17. DEPENDENCIES

## Frontend Dependencies (major)
- `react`, `react-dom`: UI rendering
- `vite`: build/dev server
- `react-router-dom`: routing
- `axios`: REST API calls
- `socket.io-client`: real-time socket connection
- `zustand`: client state management
- `tailwindcss`, `daisyui`: styling and theme system
- `react-hot-toast`: toast notifications
- `lucide-react`: icons

## Backend Dependencies (major)
- `express`: REST server
- `socket.io`: WebSocket server
- `mongoose`: MongoDB ODM
- `jsonwebtoken`: JWT creation/verification
- `bcryptjs`: password hashing
- `cloudinary`: image upload
- `cors`: CORS configuration
- `cookie-parser`: JWT cookie parsing
- `dotenv`: environment variable loading
- `nodemon`: dev server reload

---

# 18. INSTALLATION GUIDE

## Prerequisites
- Node.js installed
- MongoDB running or reachable
- Cloudinary account with credentials

## Setup (Backend)
```bash
cd backend
npm install
```

Create environment variables (example):
```env
MONGODB_URI=mongodb://localhost:27017/chatapp
JWT_SECRET=your-super-secret-key
CLOUDINARY_CLOUD_NAME=xxxx
CLOUDINARY_API_KEY=xxxx
CLOUDINARY_API_SECRET=xxxx
PORT=5001
NODE_ENV=development
```

Run backend in development:
```bash
npm run dev
```

## Setup (Frontend)
```bash
cd frontend
npm install
```

Run frontend in development:
```bash
npm run dev
```

---

# 19. DEPLOYMENT GUIDE

## Deployment Architecture
```text
Browser
   │
   ▼
Frontend Hosting (Vite build or dev server)
   │
   ▼
Backend Hosting (Express + Socket.io)
   │
   ▼
MongoDB

Cloudinary (media storage)
```

## Production Notes (as implemented)
- Backend supports production serving of frontend static files from `frontend/dist` when `NODE_ENV === 'production'`.

Production run considerations (based on existing code):
- Set `NODE_ENV=production` to enable static serving and cookie `secure` flag.
- Ensure CORS and Socket.io `cors.origin` align with deployed frontend origin (currently hardcoded in code to `http://localhost:5173`).

---

# 20. RUNTIME FLOW

```text
Startup
   │
   ▼
Backend initializes Express + Socket.io
   │
   ▼
connectDB() opens MongoDB connection
   │
   ▼
Frontend loads, calls /api/auth/check
   │
   ▼
JWT verified by protectRoute
   │
   ▼
Frontend connects Socket.io with userId
   │
   ▼
Backend updates userSocketMap and emits getOnlineUsers

At message send:

Client POST /api/messages/send/:id
   │
   ▼
Backend saves message in MongoDB
   │
   ▼
Backend emits newMessage to receiver if online
   │
   ▼
Receiver frontend socket listener updates store and UI
```

---

# 21. CHALLENGES & LEARNINGS

## Technical Challenges (detected from implementation)
- Online presence requires mapping socket connections to user ids
- Real-time message handling requires careful listener attachment/removal
- Image uploads require base64 input and cloud storage coordination

## Solutions Implemented
- `userSocketMap` in memory + `getOnlineUsers` emission
- Frontend `subscribeToMessages()` uses `socket.off('newMessage')` before re-registering
- Cloudinary upload in `updateProfile` and `sendMessage`

## Key Learnings
- JWT cookie auth is effective for keeping tokens out of JavaScript-accessible storage
- Socket.io can be integrated with REST persistence cleanly by emitting after MongoDB writes

---

# 22. COMMON ERRORS & TROUBLESHOOTING

| Error | Likely Cause | Fix |
|---|---|---|
| CORS/socket connection issues | Origin mismatch (`cors.origin` hardcoded to localhost) | Update backend CORS and Socket.io CORS origin to match frontend domain |
| `Unauthorized – No token provided` | Requests missing cookie or browser not sending cookies | Ensure `withCredentials: true` and cookie is set via login |
| `Token expired` / `Invalid token` | JWT expiry or wrong `JWT_SECRET` | Re-login or verify `JWT_SECRET` |
| MongoDB connection error | `MONGODB_URI` invalid/unreachable | Verify MongoDB URI and running instance |
| Cloudinary upload failures | Incorrect Cloudinary env vars or invalid base64 | Verify Cloudinary keys and image input |

---

# 23. PERFORMANCE ANALYSIS

## Current Optimizations
- Real-time delivery avoids polling for new messages
- UI uses loading skeletons for improved perceived performance

## Potential Bottlenecks
- Message history fetch loads all messages for the two users (`Message.find` without pagination)
- In-memory `userSocketMap` does not persist across backend restarts
- Socket listener updates rely on client-side equality checks (message type contains senderId/receiverId)

## Scalability Considerations
- For multiple backend instances, `userSocketMap` would need shared state (not implemented)
- For message history, add pagination/limits (not implemented)

## Performance Recommendations
- Implement pagination for `getMessages`
- Add database indexes for `senderId` and `receiverId` (not implemented in code)
- Externalize presence tracking for multi-instance deployments (not implemented)

---

# 24. SECURITY REVIEW

## Existing Security Measures
- Password hashing with bcryptjs
- JWT verification for protected routes
- JWT stored in HTTP-only cookie (reduces XSS token theft risk)
- Backend filters password out of `req.user` via `select('-password')`

## Security Risks (based on code)
- Token expiration behavior is handled but no refresh mechanism exists
- CORS origin and Socket.io CORS origin are hardcoded to localhost in code
- No explicit rate limiting or request throttling
- In-memory socket presence is not secure across distributed environments

## Recommended Improvements (realistic)
- Add rate limiting middleware on auth/message endpoints
- Add CSRF protection if cookies are used cross-site
- Add input validation/sanitization server-side for `text` and image payloads
- Implement pagination in `getMessages`

---

# 25. FUTURE ENHANCEMENTS

Based on current architecture (and only what is realistically adjacent):
- Message pagination (limit/skip) for chat history
- Typing indicators / read receipts (would require new events and DB fields—currently not implemented)
- Multi-instance Socket.io scaling (requires shared pub/sub)
- Server-side input validation using a validation library (currently not present)

---

# 26. DEVELOPER NOTES

## Architecture Decisions (from code)
- Cookie-based JWT auth
- MongoDB persistence for users/messages
- Socket.io for online presence and realtime message delivery
- Cloudinary for media storage
- Zustand for frontend state

## Maintainability Notes
- Backend controllers are separated from routes, and middleware isolates JWT verification.
- Frontend stores separate auth and chat state.

## Refactoring Opportunities
- Add pagination and ordering to message queries
- Remove unused/duplicate constants and unused imports if present
- Consider moving hardcoded CORS origins to environment variables

## Technical Debt Observations (explicitly from code)
- `backend/src/seeds/user.seed.js` exists but is not analyzed; if seeds are incomplete, document accordingly.
- Repository contains an inline comment and duplicated documentation content in the root README (not affecting runtime).

---

Written by Yash Lagare

