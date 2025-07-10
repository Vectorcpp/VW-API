### **VectorWorks API (VWApi) Documentation**

This document outlines the functionality of the VectorWorks API (VWApi), a web service designed for user authentication, session management, and device security.

### **1. Overview**

The VWApi is a Flask-based web application that provides a secure backend for managing user sessions and device interactions. Key features include:

*   **Device Authentication:** Ensures that only trusted devices can communicate with the API.
*   **User Authentication & Session Management:** A nonce-based authentication system to securely log in users and manage their sessions.
*   **Security:** Implements rate limiting, device banning, and logging to protect against abuse.
*   **Admin & Dashboard:** Provides endpoints for administrators to monitor and manage devices and a dashboard to view system status.
*   **Certain Game Handling:** Currently, I'm still working on this, I just currently have the base API's down, no game separation ALSO The "Dashboard", is just placeholder stuff, thats why its admin only also you can brute force the password if you want

### **2. Base URLs**

*   **API:** `/api`
*   **Authentication:** `/api/auth`
*   **Photon Network:** `/api/photonnetwork/`
*   **Photon Webhook:** `/api/photonwebhook`
*   **Dashboard:** `/dashboard`
*   **Admin:** `/admin`

### **3. Authentication**

The API uses a two-step authentication process involving device authentication and user session authentication.

#### **3.1 Device Authentication**

Most endpoints require device authentication via custom HTTP headers. This ensures that requests originate from a valid client.

**Required Headers:**

*   `X-Device-ID`: A unique identifier for the device (16-128 characters, alphanumeric with hyphens and underscores).
*   `X-Device-Signature`: An HMAC-SHA256 signature of the device ID and timestamp.
*   `X-Timestamp`: A Unix timestamp of the request time.

#### **3.2 User Authentication**

User authentication is managed through a nonce and session system.

**Workflow:**

1.  The client requests a nonce from the `/api/auth/nonce` endpoint.
2.  The client uses the received nonce to log in via the `/api/auth/login` endpoint.
3.  Upon successful login, the server provides a session nonce that is used for subsequent authenticated requests.

### **4. API Endpoints**

#### **4.1 Authentication Endpoints (`/api/auth`)**

**POST** `/nonce`
*Description*: Requests a temporary, single-use nonce for authentication.
*Headers*: Requires device authentication headers.
*Request Body*:
```json
{
  "player_id": "string",
  "cosmetics": {
    "hat": "string",
    "face": "string",
    ...
  }
}
```
*Success Response (200)*:
```json
{
  "success": true,
  "nonce": "string",
  "expires_at": "integer",
  "expires_in": "integer",
  "message": "Nonce generated successfully"
}
```

**POST** `/login`
*Description*: Logs in a user with a valid nonce.
*Headers*: Requires device authentication headers.
*Request Body*:
```json
{
  "player_id": "string",
  "nonce": "string"
}
```
*Success Response (200)*:
```json
{
  "success": true,
  "session_nonce": "string",
  "session_expires_at": "integer",
  "session_expires_in": "integer",
  "message": "Login successful"
}
```

**POST** `/logout`
*Description*: Logs out a user and invalidates their session.
*Headers*: Requires device and session authentication headers.
*Success Response (200)*:
```json
{
  "success": true,
  "message": "Logout successful"
}
```

**GET** `/status`
*Description*: Checks the status of the current user's session.
*Headers*: Requires device and session authentication headers.
*Success Response (200)*:
```json
{
  "success": true,
  "player_id": "string",
  "device_id": "string",
  "session_nonce": "string",
  "session_expires_at": "integer",
  "session_expires_in": "integer",
  "message": "Session active"
}
```

#### **4.2 Game Endpoints (`/api/game`)**

**POST** `/join`
*Description*: Allows a player to join a game room.
*Headers*: Requires device and session authentication headers.
*Request Body*:
```json
{
  "room_id": "string"
}
```
*Success Response (200)*:
```json
{
  "success": true,
  "game_session": {
    "player_id": "string",
    "device_id": "string",
    "room_id": "string",
    "joined_at": "integer"
  },
  "message": "Player {player_id} joined room {room_id}"
}
```

#### **4.3 Cosmetics Endpoints (`/api/cosmetics`)**

**GET** `/from_nonce`
*Description*: Retrieves cosmetic data associated with a nonce.
*Headers*: Requires device and session authentication headers.
*Query Parameters*: `nonce` (string)
*Success Response (200)*:
```json
{
  "status": "okay",
  "cosmetics": {},
  "message": "got cosmetics!"
}
```

#### **4.4 Photon Webhook Endpoints (`/api/photonwebhook`)**

**POST** `/RoomCreate`
*Description*: A webhook for creating a game room, likely triggered by a Photon server. It validates the user's session before proceeding.
*Request Body*:
```json
{
    "player_id": "string",
    "session_nonce": "string",
    "device_id": "string",
    "room_name": "string"
}
```
*Success Response (200)*:
```json
{
    "success": true,
    "room_id": "string",
    "room_name": "string",
    "created_by": "string",
    "device_id": "string",
    "created_at": "integer",
    "message": "Room '{room_name}' created successfully"
}
```

**POST** `/request_rpc`
*Description*: Handles WebRPC calls from a Photon client, enabling secure server-side actions like equipping cosmetics.
*Headers*: Requires device and session authentication.
*Request Body (Example for `equip_cosmetic`)*:
```json
{
    "Type": "WebRpc",
    "ProcedureCall": {
        "RpcName": "request_rpc",
        "Parameters": {
            "rpc_name": "equip_cosmetic",
            "slot": "hat",
            "cosmetic": "blue_cap"
        }
    }
}
```
*Success Response (200)*:
```json
{
    "ReturnCode": 0,
    "Message": "Cosmetic equipped",
    "ReturnData": {
        "slot": "hat",
        "cosmetic": "blue_cap"
    }
}
```

#### **4.5 Admin Endpoints (`/admin`)**

**GET** `/devices`
*Description*: Lists all registered devices and their status.
*Success Response (200)*:
```json
{
    "success": true,
    "devices": [
        {
            "device_id": "string",
            "player_id": "string",
            "registered_at": "iso_timestamp",
            "last_seen": "iso_timestamp",
            "is_banned": "boolean",
            "ban_expires": "iso_timestamp",
            "ban_reason": "string"
        }
    ],
    "total_devices": "integer",
    "banned_devices": "integer"
}
```

**POST** `/device/<device_id>/ban`
*Description*: Bans a specific device.
*Request Body*:
```json
{
    "duration": "integer",
    "reason": "string"
}
```
*Success Response (200)*:
```json
{
    "success": true,
    "message": "Device {device_id} banned for {duration} seconds",
    "reason": "string"
}
```

**POST** `/device/<device_id>/unban`
*Description*: Unbans a specific device.
*Success Response (200)*:
```json
{
    "success": true,
    "message": "Device {device_id} unbanned successfully"
}
```

**POST** `/cleanup`
*Description*: Manually triggers the cleanup of expired sessions, nonces, and bans.
*Success Response (200)*:
```json
{
  "success": true,
  "message": "Cleanup completed successfully"
}
```

### **5. Other Endpoints**

**GET** `/dashboard`
*Description*: Renders an HTML dashboard displaying the current status of the server, including active sessions, device counts, and recent activity.

**GET** `/health`
*Description*: A simple health check endpoint to confirm the service is running. Returns a `200 OK` with a success status.

**GET** `/api/device/info`
*Description*: Retrieves information about the authenticated device.
*Headers*: Requires device authentication headers.
*Success Response (200)*:
```json
{
    "success": true,
    "device_id": "string",
    "player_id": "string",
    "registered_at": "integer",
    "last_seen": "integer",
    "is_banned": "boolean",
    "ban_expires": "integer",
    "ban_reason": "string"
}
```

### **6. Error Handling**

The API returns standard HTTP status codes for errors and includes a JSON body with details about the error.

*   **400 (Bad Request):** Missing or invalid parameters.
*   **401 (Unauthorized):** Invalid or missing authentication credentials.
*   **403 (Forbidden):** Access is denied (e.g., banned device, invalid client).
*   **404 (Not Found):** The requested endpoint does not exist.
*   **429 (Too Many Requests):** Rate limit has been exceeded.
*   **500 (Internal Server Error):** An unexpected server-side error occurred.

**Example Error Response:**
```json
{
    "error": "INVALID_DEVICE_SIGNATURE",
    "message": "Invalid device signature",
    "timestamp": "iso_timestamp",
    "suggestion": "Please check your device authentication"
}
```

Another thing to note, is i wont publicy release the api url until its fully done, and is secure enough on where i can release it, but you can still read the documentation if needed.
