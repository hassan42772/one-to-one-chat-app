# One-to-One Chat App

A full-stack real-time chat application for private conversations, account authentication, two-factor authentication, and peer-to-peer audio/video calls.

## Features

- User registration and login
- Email OTP verification
- Forgot password and password reset flows
- TOTP-based two-factor authentication with QR setup
- reCAPTCHA verification during registration
- JWT-protected REST APIs
- Real-time one-to-one messaging with Socket.IO
- Online user presence
- Sent, delivered, and read message status
- Paginated conversation history
- WebRTC audio and video calls
- Incoming, accepted, rejected, ended, offline, and failed call states
- Responsive React dashboard
- Daily cleanup of unverified accounts older than 24 hours

## Tech Stack

### Frontend

- React 19
- Vite 8
- React Router 7
- Axios
- Socket.IO Client
- WebRTC
- reCAPTCHA
- Lucide and React Icons

### Backend

- Node.js
- Express 5
- MySQL
- Sequelize
- Socket.IO
- JSON Web Tokens
- bcrypt
- Speakeasy
- Nodemailer
- node-cron

## Project Structure

```text
1to1Chat-app/
├── backend/
│   ├── src/
│   │   ├── config/          # MySQL and Sequelize configuration
│   │   ├── controllers/     # REST request handlers
│   │   ├── middlewares/     # Authentication, logging, and errors
│   │   ├── models/          # User and Message Sequelize models
│   │   ├── routes/          # Auth, user, and message routes
│   │   ├── services/        # Business logic
│   │   ├── sockets/         # Socket.IO messaging and call events
│   │   └── utils/           # JWT, email, response, and cron utilities
│   └── package.json
└── frontend/
    ├── src/
    │   ├── api/             # Axios configuration
    │   ├── context/         # Chat and Socket.IO context
    │   ├── features/        # Auth, chat, users, calls, and dashboard
    │   ├── routes/          # Public and protected routes
    │   └── utils/           # Shared frontend utilities
    └── package.json
```

## Prerequisites

- Node.js 18 or newer
- npm
- MySQL Server
- An SMTP email account for OTP and password reset emails
- A Google reCAPTCHA v2/v3 secret key
- A modern browser with microphone and camera support for calls

## Installation

Clone the repository and enter the project directory:

```bash
git clone <your-repository-url>
cd 1to1Chat-app
```

### Backend setup

```bash
cd backend
npm install
```

Create `backend/.env` with the variables described below. Then create the MySQL database:

```sql
CREATE DATABASE chat_app;
```

Sequelize creates and synchronizes the `users` and `messages` tables automatically when the backend starts.

### Frontend setup

Open a second terminal:

```bash
cd frontend
npm install
```

Create `frontend/.env`:

```env
VITE_API_URL=http://localhost:5000/api
VITE_SOCKET_URL=http://localhost:5000
VITE_RECAPTCHA_SITE_KEY=your_recaptcha_site_key
VITE_GOOGLE_CLIENT_ID=your_google_oauth_client_id
```

Although a Google OAuth client ID is included in the frontend configuration, the current Google authentication route is disabled in the backend.

## Environment Variables

### Backend

```env
PORT=5000

DB_HOST=localhost
DB_PORT=3306
DB_NAME=chat_app
DB_USER=root
DB_PASSWORD=your_mysql_password

JWT_SECRET=replace_with_a_long_random_secret

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@example.com
SMTP_PASS=your_smtp_password_or_app_password

RECAPTCHA_SECRET_KEY=your_recaptcha_secret_key
GOOGLE_CLIENT_ID=your_google_oauth_client_id
```

Never commit real credentials or secret keys. Keep `backend/.env` out of source control.

### Frontend

| Variable | Purpose |
|---|---|
| `VITE_API_URL` | Base URL for REST API requests |
| `VITE_SOCKET_URL` | Socket.IO server URL |
| `VITE_RECAPTCHA_SITE_KEY` | Public reCAPTCHA site key |
| `VITE_GOOGLE_CLIENT_ID` | Google OAuth client ID for future/current Google authentication |

## Running the Application

Start MySQL, then run the backend:

```bash
cd backend
npm run dev
```

The API and Socket.IO server run on `http://localhost:5000` by default.

Start the frontend in a separate terminal:

```bash
cd frontend
npm run dev
```

Open the Vite URL shown in the terminal, normally `http://localhost:5173`.

## REST API

All protected endpoints require:

```http
Authorization: Bearer <jwt_token>
```

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth/register` | Register an account and send an email OTP |
| POST | `/api/auth/login` | Login with email and password |
| POST | `/api/auth/verify-otp` | Verify signup or two-factor OTP |
| POST | `/api/auth/2fa/generate` | Generate a TOTP secret and QR code |
| POST | `/api/auth/2fa/verify-setup` | Verify and enable TOTP two-factor authentication |
| PUT | `/api/auth/2fa/disable` | Disable TOTP two-factor authentication |
| POST | `/api/auth/forgot-password` | Send a password reset OTP |
| POST | `/api/auth/reset-password` | Reset a password using an email OTP |

### Users

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/users` | Get the authenticated user's available user list |

### Messages

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/messages` | Send a message |
| GET | `/api/messages/:userId?page=1` | Get a paginated 20-message conversation |

## Real-Time Events

### Socket.IO messaging and presence

| Event | Direction | Description |
|---|---|---|
| `addNewUser` | Client to server | Register a user's socket as online |
| `getOnlineUsers` | Server to client | Broadcast online users |
| `openChat` | Client to server | Mark a chat as active |
| `closeChat` | Client to server | Mark a chat as closed |
| `sendMessage` | Client to server | Send a message |
| `receiveMessage` | Server to client | Receive a message |
| `messageSent` | Server to client | Confirm a sent message |
| `messagesRead` | Server to client | Notify that messages were read |

### WebRTC signaling

| Event | Direction | Description |
|---|---|---|
| `callUser` | Caller to server | Start an audio or video call |
| `incomingCall` | Server to receiver | Deliver the call offer |
| `acceptCall` | Receiver to server | Send the WebRTC answer |
| `callAccepted` | Server to caller | Deliver the answer |
| `rejectCall` | Receiver to server | Reject an incoming call |
| `callRejected` | Server to caller | Notify the caller |
| `endCall` | Client to server | End the call |
| `callEnded` | Server to peer | Notify the peer |
| `callFailed` | Server to caller | Notify that the receiver is offline |

Socket signaling exchanges offer and answer descriptions. Media itself uses WebRTC peer-to-peer connections.

## Authentication Flow

1. The user registers and receives an email OTP.
2. The OTP is verified within 10 minutes.
3. The login endpoint validates credentials and returns a JWT.
4. If TOTP is enabled, login requires the authenticator code before a JWT is issued.
5. Protected REST endpoints validate the JWT with `Authorization: Bearer <token>`.
6. The frontend stores the token in local storage and attaches it through an Axios interceptor.

## Database Models

### User

- Integer primary key
- Name
- Unique email
- Password hash
- Email verification status
- OTP and OTP expiration
- TOTP enabled status
- TOTP secret
- Created and updated timestamps

### Message

- Integer primary key
- Sender ID
- Receiver ID
- Message text
- Status: `sent`, `delivered`, or `read`
- Created and updated timestamps

## Scripts

### Backend

```bash
npm run dev   # Start with nodemon
npm start     # Start the server
npm test      # Currently no test suite configured
```

### Frontend

```bash
npm run dev     # Start Vite development server
npm run build   # Create production build
npm run lint    # Run ESLint
npm run preview # Preview a production build
```

## Production Notes

- Replace localhost URLs and ports with deployed frontend/backend URLs.
- Configure CORS and Socket.IO origins for the production domain.
- Use HTTPS, secure JWT secrets, and appropriate database credentials.
- Configure SMTP with an account or provider that permits automated email.
- Serve WebRTC calls behind a deployment with suitable STUN/TURN support; NATs and restrictive networks may require TURN.
- Add test coverage and CI validation before production release.
