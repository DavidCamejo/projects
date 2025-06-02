# WhatsApp API Manager

WhatsApp API Manager is an unofficial API for WhatsApp that allows you to integrate WhatsApp messaging capabilities into your applications. This project provides a robust backend API and a user-friendly frontend for managing WhatsApp sessions and interactions.

## Features

- **Session Management:** Create, manage, and delete WhatsApp sessions
- **Multiple Concurrent Sessions:** Support for multiple WhatsApp accounts simultaneously
- **Secure Authentication:** JWT-based authentication with role-based access control
- **Real-time Updates:** WebSocket integration for real-time status updates
- **Message Management:** Send and receive WhatsApp messages programmatically
- **QR Code Authentication:** Easy session authentication with QR code scanning
- **Persistent Sessions:** Automatic restoration of sessions after server restarts
- **WordPress Integration:** Optional authentication via WordPress JWT tokens

## Prerequisites

- Node.js 16.x or higher
- MongoDB 4.x or higher
- NPM or PNPM package manager

## Installation

1. Clone the repository:
   

2. Install dependencies:
   

3. Configure environment variables:
   - Copy `.env.example` to `.env` and update the values.
   

4. Run the application:
   

## Project Structure

## API Documentation

The WhatsApp API Manager provides a comprehensive set of endpoints for authentication, session management, and messaging. All API endpoints are prefixed with `/api`.

### Authentication Endpoints

| Method | Endpoint | Description | Request Body | Response |
|--------|----------|-------------|-------------|----------|
| `POST` | `/api/auth/login` | Authenticate with username/password | `{ "username": "string", "password": "string" }` | `{ "token": "JWT_TOKEN", "user": { "id": "string", "username": "string", "roles": ["string"] } }` |
| `POST` | `/api/auth/wordpress` | Authenticate with WordPress JWT token | `{ "wpToken": "string" }` | `{ "token": "JWT_TOKEN", "user": { "id": "string", "username": "string", "roles": ["string"] } }` |
| `GET`  | `/api/auth/verify` | Verify JWT token | _N/A_ (requires Authorization header) | `{ "id": "string", "username": "string", "roles": ["string"] }` |

### Session Management Endpoints

| Method | Endpoint | Description | Request Body/Params | Response |
|--------|----------|-------------|-------------|----------|
| `GET`  | `/api/sessions` | List all available sessions | _N/A_ | Array of session objects |
| `POST` | `/api/sessions` | Create a new WhatsApp session | `{ "clientId": "string", "config": {} }` | `{ "clientId": "string", "status": "string", "createdAt": "date", "updatedAt": "date" }` |
| `GET`  | `/api/sessions/:clientId/status` | Get session status | `clientId` in URL | `{ "clientId": "string", "status": "string", "updatedAt": "date" }` |
| `GET`  | `/api/sessions/:clientId/qr` | Get QR code for session authentication | `clientId` in URL | `{ "clientId": "string", "qrCode": "string", "status": "string" }` |
| `DELETE` | `/api/sessions/:clientId` | Delete a session | `clientId` in URL | `{ "success": true }` |

### Messaging Endpoints

| Method | Endpoint | Description | Request Body/Params | Response |
|--------|----------|-------------|-------------|----------|
| `POST` | `/api/messages/:clientId/send` | Send a message | `clientId` in URL, `{ "chatId": "string", "message": "string", "options": {} }` | `{ "messageId": "string" }` |
| `GET`  | `/api/messages/:clientId/:chatId/history` | Get message history | `clientId` and `chatId` in URL, `limit` and `before` in query params | Array of message objects |

### Authentication

Most endpoints require authentication using a JWT token. Include the token in the Authorization header as follows:

```
Authorization: Bearer YOUR_JWT_TOKEN
```

### Role-Based Access Control

The API implements role-based access control for all protected endpoints:

- `admin`: Full access to all features
- `user`: Limited access based on assigned permissions
- `vendor`: Access only to their own sessions and messages

### Error Handling

All API endpoints follow a consistent error response format:

```json
{
  "error": "Error message description"
}
```

Common HTTP status codes:

- `200`: Success
- `201`: Resource created
- `400`: Bad request / Invalid input
- `401`: Unauthorized / Invalid credentials
- `403`: Forbidden / Insufficient permissions
- `404`: Resource not found
- `500`: Server error
