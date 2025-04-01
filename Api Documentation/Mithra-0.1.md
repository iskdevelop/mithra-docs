# Overview
Mithra v0.1 is a platform that facilitates live academic meetings and presentations. The backend API should enable managing presentations and meetings, along with handling user roles and interactions within live sessions. Below are the key requirements for the MVP API, which are to be used by the backend team for implementation.
## Key Concepts:
- **Presentation**: A markdown-based document representing the content of a presentation. It contains raw markdown content and will be rendered on the front-end.
- **Canvas**: The actual file representing the markdown content of a presentation.
- **Meeting**: A live, real-time session where a presentation is shared with participants.
- **Roles in Meeting**:
    - **Presenter**: Has full control over the presentation and can modify content.
    - **Guest**: View-only participant.
- **UDP for Live Updates**: While the REST API handles the session and content management, live updates of the content (like modifying the markdown) will be done via UDP.

## API Requirements:
1. **Presentation Management**:
    - Create, update, retrieve, and delete presentations.
    - Presentations should be stored as markdown files with a unique ID for referencing in live sessions.
2. **Meeting Management**:
    - Ability to create a new meeting linked to an existing presentation.
    - Support for joining the meeting as either a Presenter or a Guest.
    - No scheduling of meetings—only ad-hoc live meetings.

## MVP API Features:
- **Creating a Presentation**: The backend should allow creating a new presentation (markdown file) via the API.
- **Joining a Meeting**: Guests should be able to join live meetings, with the API managing the session state (e.g., Presenter role and meeting ID).
- **Updating the Presentation**: The Presenter role should be able to modify the content of the presentation in real-time, and these updates should be reflected to all participants in the meeting via the UDP-based update system.

The frontend will interact with this API to fetch the initial content of the presentations and handle joining meetings in real time.

# API Endpoints
## Base URL:
```
https://api.mithra.io/v0.1
```
## Authentication Flow

```
POST /auth/login
```

- Authenticate user and create a session
- Body: 
```json
	{ 
		"email": string,
		 "password": string
	}
```
- Response: 
```json
	{ 
		"userSessionKey": string,
		"expiresAt": timestamp
	}
```

---

```
POST /auth/logout
```

- Invalidate the current session
- Headers: `Authorization: Bearer {userSessionKey}`
- Response: 
```json
	{ 
		"success": boolean 
	}	
```

## Canvas Management

All endpoints require authentication via the `userSessionKey` in the Authorization header:

```
Authorization: Bearer {userSessionKey}
```

```
POST /canvases
```

- Create a new canvas (presentation)
- Headers: `Authorization: Bearer {userSessionKey}`
- Body: 

```json
  {
	  title: "Title of Canvas",
	  slides: [],
  }
```

- Response: 

```json
	{
		canvasId: "asdjasFAJGWOVEJVO3rn1ONOASD",
		title: "Title of Canvas",
		createdAt: timestamp
	}
```

---

```
GET /canvases
```

- Get list of user's canvases
- Headers: `Authorization: Bearer {userSessionKey}`
- Response:  

```json
	{ 
		"canvases": [{
			"canvasId": string,
			"title": string, 
			"createdAt": timestamp 
			 }]
	}
```

---

```
GET /canvases/{canvasId}
```

- Get specific canvas details
- Headers: `Authorization: Bearer {userSessionKey}`
- Response: 

```json
	{ 
		"canvasId": string, 
		"title": string, 
		"slides": [{
			"id": string,
			"content": string
		}], 
		"createdAt": timestamp, 
		"updatedAt": timestamp 
	}
```

---

```
PUT /canvases/{canvasId}
```

- Update canvas content
- Headers: `Authorization: Bearer {userSessionKey}`
- Body:

```json
	{ 
		"title": string,
		"slideId": string,
		"content": string
	}
```

- Response: 

```json
	{ 
		"canvasId": string, 
		"title": string, 
		"updatedAt": timestamp,
		change: { 
			"title": string,
			"slideId": string,
			"content": string
			}
	}
```

---

```
DELETE /canvases/{canvasId}
```

- Delete a canvas
- Headers: `Authorization: Bearer {userSessionKey}`
- Response: 

```json
	{ 
		"success": boolean 
	}
```

---
## Meeting Management

```
POST /meetings
```

- Create a new meeting
- Headers: `Authorization: Bearer {userSessionKey}`
- Body: 

```json
	{ 
		"canvasId": string
	}
```

- Response: 

```json
	{ 
		"meetingId": string, 
		"canvasId": string, 
		"presenterToken": string, 
		"guestLink": string, 
		"createdAt": timestamp
	}
```


```
GET /meetings/{meetingId}
```

- Get meeting details
- Headers: `Authorization: Bearer {userSessionKey}`
- Response: 

```json
	{ 
		"meetingId": string, 
		"canvasId": string, 
		"status": string, 
		"createdAt": timestamp, 
		"updatedAt": timestamp, 
		"participants": number
	}
```

---

```
POST /meetings/{meetingId}/join
```

- Join an existing meeting
- Headers: `Authorization: Bearer {userSessionKey}` (for presenter) or no auth header (for guests)
- Body:  `presenterToken` required only for presenter role

```json
	{ 
		"role": "presenter"|"guest",
		"presenterToken": string
	}
```

- Response: 

```json
	{ 
		"meetingId": string, 
		"role": string, 
		"meetingSessionId": string, 
		"rtcDetails": { 
			"serverIp": string, 
			"port": number } 
	}
```

---

```
POST /meetings/{meetingId}/end
```

- End a meeting (presenter only)
- Headers: `Authorization: Bearer {userSessionKey}`
- Response: 

```json
	{ 
		"success": boolean 
	}
```

# Data Models

## UserSession

- userSessionKey (string) - unique session identifier
- userId (string) - reference to the user
- createdAt (timestamp)
- expiresAt (timestamp)
- isActive (boolean)

## MeetingSession

- meetingSessionId (string) - unique meeting participant session identifier
- meetingId (string) - reference to the meeting
- userSessionKey (string) - reference to the user session (null for guests)
- role (string) - "presenter" or "guest"
- joinedAt (timestamp)
- isActive (boolean)

# Implementation Notes

1. **Authentication Workflow**:
    - The backend should validate the userSessionKey for all authenticated endpoints
    - For each request, check if the session exists and is not expired
    - For canvas operations, verify the authenticated user is the owner of the canvas
    - For meeting creation, verify the authenticated user is the owner of the associated canvas
2. **Guest Access**:
    - Guests can join meetings via the guest link without authentication
    - The backend should generate a meetingSessionId for guests to track their participation
3. **UDP Authentication**:
    - The UDP protocol should validate the meetingSessionId received from the REST API
    - For canvas updates, verify the session has presenter role

