WhatsApp API Server - API Documentation
API Overview
This document outlines the RESTful API endpoints and WebSocket events available in the WhatsApp API Server. All API routes are prefixed with /api and require authentication unless otherwise specified.
Authentication
JWT Authentication
The API uses JWT (JSON Web Tokens) for authentication. After successful login, a JWT token is stored in an HTTP-only cookie.
API Authentication Endpoints
Register a New User
POST /api/register

Request Body:

{
"username": "example_user",
"password": "securePassword123"
}

Response:

{
"success": true,
"message": "User registered successfully",
"user": {
"id": "user123",
"username": "example_user"
}
}
User Login
POST /api/login

Request Body:

{
"username": "example_user",
"password": "securePassword123"
}

Response:

{
"success": true,
"message": "Login successful",
"user": {
"id": "user123",
"username": "example_user"
}
}

Note: The JWT token is automatically set as an HTTP-only cookie.
User Logout
POST /api/logout

Response:

{
"success": true,
"message": "Logged out successfully"
}
Get Current User
GET /api/user

Response:

{
"id": "user123",
"username": "example_user"
}
WhatsApp API Endpoints
Check WhatsApp Connection Status
GET /api/status

Response:

{
"status": "CONNECTED",
"info": {
"pushname": "John Doe",
"wid": "1234567890@c.us"
}
}

Possible status values: DISCONNECTED, CONNECTING, CONNECTED
Send WhatsApp Message
POST /api/send

Request Body:

{
"to": "1234567890@c.us",
"message": "Hello world!"
}

Response:

{
"success": true,
"messageId": "msg_123456789",
"timestamp": 1619712345
}
Get All Chats
GET /api/chats

Response:

{
"chats": [
{
"id": "1234567890@c.us",
"name": "John Doe",
"timestamp": 1619712345,
"lastMessage": "Hello world!",
"unreadCount": 2
},
{
"id": "9876543210@c.us",
"name": "Jane Smith",
"timestamp": 1619712300,
"lastMessage": "Hi there!",
"unreadCount": 0
}
]
}
Get Chat Messages
GET /api/chats/:chatId/messages

Request Parameters:

limit: Maximum number of messages (default: 50)
before: Fetch messages before this message ID (for pagination)

Response:

{
"chatId": "1234567890@c.us",
"messages": [
{
"id": "msg_123456789",
"body": "Hello world!",
"timestamp": 1619712345,
"fromMe": true
},
{
"id": "msg_987654321",
"body": "Hi there!",
"timestamp": 1619712300,
"fromMe": false
}
],
"hasMore": true
}
Logout from WhatsApp
POST /api/logout-whatsapp

Response:

{
"success": true,
"message": "WhatsApp session logged out successfully"
}
WebSocket API
The API server also provides real-time communication via Socket.IO.
Connection Setup
To connect to the WebSocket server:

const socket = io("/");

// Authenticate the socket connection
socket.emit("authenticate", { token: "your-jwt-token" });

// Handle authentication errors
socket.on("auth_error", (error) => {
console.error("Socket authentication failed:", error.message);
});
WhatsApp Events
After successful authentication, the socket will receive the following events:
QR Code Event
socket.on("qr", (data) => {
// data.qr contains the QR code data to be displayed
// Use a QR code library to display this for WhatsApp Web scanning
console.log("QR Code received:", data.qr);
});
Ready Event
socket.on("ready", () => {
console.log("WhatsApp is ready");
});
Message Event
socket.on("message", (message) => {
console.log("New message:", message);
/_
message = {
id: "msg_id",
from: "1234567890@c.us",
body: "Hello!",
timestamp: 1619712345,
hasMedia: false
}
_/
});
Disconnected Event
socket.on("disconnected", (reason) => {
console.log("WhatsApp disconnected:", reason);
});
Error Handling
All API endpoints follow a consistent error format:

{
"success": false,
"error": "Error message",
"code": "ERROR_CODE"
}
Common Error Codes
Code
Description
AUTHENTICATION_FAILED
Invalid credentials or expired token
WHATSAPP_NOT_CONNECTED
WhatsApp session is not active
INVALID_PHONE_NUMBER
Phone number format is invalid
MESSAGE_FAILED
Failed to send message
RATE_LIMIT_EXCEEDED
Too many requests in a short time

Rate Limiting
API endpoints are rate-limited to prevent abuse. The current limits are:

Authentication endpoints: 10 requests per minute
Message sending: 30 messages per minute
Chat listing: 60 requests per minute

When rate limited, the API will respond with HTTP status 429 and include a Retry-After header.
Webhook Integration
For server-to-server integrations, you can configure webhooks to receive notifications:

POST /api/webhooks/configure

Request Body:

{
"url": "https://your-server.com/webhook",
"events": ["message", "message.ack", "connection.update"],
"secret": "your-webhook-secret"
}

All webhook requests will include an X-Webhook-Signature header for verification.
