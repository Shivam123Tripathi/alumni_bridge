# Connection Request Feature - Complete Guide

## Overview
The connection request feature allows users to send connection requests to other users on the network and manage their professional connections.

## Frontend Code (Fully Implemented in `frontend/pages/network.html`)

### 1. **Modal Dialog for Connection Requests**
```html
<!-- Connection Request Modal -->
<div id="connectionModal" class="modal">
  <div class="modal-content">
    <div class="modal-header">
      <h2>Send Connection Request</h2>
      <button class="close-btn" onclick="closeConnectionModal()">&times;</button>
    </div>
    <div class="modal-body">
      <!-- User Profile Section -->
      <div class="user-profile-section">
        <div class="user-profile-avatar" id="modalUserAvatar">👤</div>
        <h3 class="user-profile-name" id="modalUserName">User Name</h3>
        <p class="user-profile-role" id="modalUserRole">Role</p>
      </div>

      <!-- User Details -->
      <div class="user-details">
        <div class="detail-item">
          <span class="detail-label">🎓 Degree:</span>
          <span class="detail-value" id="modalUserDegree">N/A</span>
        </div>
        <!-- ... more details ... -->
      </div>

      <!-- Connection Message -->
      <div class="connection-message">
        <label for="connectionMessage">Send a message (optional)</label>
        <textarea id="connectionMessage" placeholder="Hi! I'd like to connect..."></textarea>
      </div>
    </div>

    <div class="modal-footer">
      <button class="modal-btn modal-btn-secondary" onclick="closeConnectionModal()">Cancel</button>
      <button class="modal-btn modal-btn-primary" onclick="sendConnectionRequestFromModal()">Send Request</button>
    </div>
  </div>
</div>
```

### 2. **JavaScript Functions for Connection Management**

#### Opening the Connection Modal
```javascript
function openConnectionModal(userId) {
  selectedUserId = userId;
  const user = allUsers.find(u => u.id === userId);
  
  if (!user) return;

  // Populate modal with user details
  document.getElementById('modalUserName').textContent = user.name;
  document.getElementById('modalUserRole').textContent = user.role === 'STUDENT' ? '👨‍🎓 Student' : '👨‍💼 Alumni';
  document.getElementById('modalUserDegree').textContent = user.degreeName || 'N/A';
  document.getElementById('modalUserBatch').textContent = user.batchYear || 'N/A';
  document.getElementById('modalUserCollege').textContent = user.institute || 'N/A';
  document.getElementById('modalUserEmail').textContent = user.email || 'N/A';
  document.getElementById('modalUserBio').textContent = user.bio || 'No bio available';
  document.getElementById('modalUserAvatar').textContent = user.name.charAt(0).toUpperCase();
  
  // Clear previous message
  document.getElementById('connectionMessage').value = '';

  // Show modal
  document.getElementById('connectionModal').style.display = 'block';
}
```

#### Closing the Modal
```javascript
function closeConnectionModal() {
  document.getElementById('connectionModal').style.display = 'none';
  selectedUserId = null;
}
```

#### Sending Connection Request
```javascript
async function sendConnectionRequestFromModal() {
  if (!selectedUserId) return;

  try {
    const user = allUsers.find(u => u.id === selectedUserId);
    const message = document.getElementById('connectionMessage').value.trim();
    const btn = document.getElementById('sendConnectBtn');

    btn.disabled = true;
    btn.textContent = '⏳ Sending...';

    // Send connection request with optional message
    const payload = message ? { message } : {};
    await apiCall(`/users/connect/${selectedUserId}`, { 
      method: "POST",
      body: JSON.stringify(payload)
    });

    // Update connection status
    connectionStatuses.set(selectedUserId, "sent");

    // Update button in the grid
    const gridBtn = document.getElementById(`btn-${selectedUserId}`);
    if (gridBtn) {
      gridBtn.className = "btn btn-pending";
      gridBtn.textContent = "⏳ Request Sent";
      gridBtn.disabled = true;
    }

    // Show success and close modal
    showNotification(`Connection request sent to ${user.name}!`, "success");
    closeConnectionModal();

    // Reset button state
    btn.disabled = false;
    btn.textContent = 'Send Request';
  } catch (error) {
    console.error("Failed to send connection request:", error);
    const btn = document.getElementById('sendConnectBtn');
    btn.disabled = false;
    btn.textContent = 'Send Request';
    showNotification("Failed to send connection request. Please try again.", "error");
  }
}
```

#### Loading Connection Statuses
```javascript
async function loadConnectionStatuses() {
  try {
    const pendingRequests = await apiCall("/users/connection-requests", {
      method: "GET",
    });
    const sentRequests = await apiCall("/users/sent-requests", {
      method: "GET",
    });
    const activeConnections = await apiCall("/users/connections", {
      method: "GET",
    });

    // Map pending requests
    if (pendingRequests && Array.isArray(pendingRequests)) {
      pendingRequests.forEach((req) => {
        connectionStatuses.set(req.sender.id, "pending");
      });
    }

    // Map sent requests
    if (sentRequests && Array.isArray(sentRequests)) {
      sentRequests.forEach((req) => {
        connectionStatuses.set(req.receiver.id, "sent");
      });
    }

    // Map active connections
    if (activeConnections && Array.isArray(activeConnections)) {
      activeConnections.forEach((conn) => {
        connectionStatuses.set(conn.id, "connected");
      });
    }
  } catch (error) {
    console.error("Failed to load connection statuses:", error);
  }
}
```

### 3. **User Card with Connect Button**
```javascript
<button class="${btnClass}" onclick="openConnectionModal(${user.id})" 
        id="btn-${user.id}" ${btnDisabled ? "disabled" : ""}>
  ${btnText}
</button>
```

Button states:
- **+ Connect** (Blue): No connection yet
- **⏳ Request Sent** (Yellow): Request already sent
- **✓ Connected** (Green): Already connected
- **✓ Pending** (Yellow): User sent you a request

## Backend Code (Already Implemented)

### 1. **UserController.java**
```java
@PostMapping("/connect/{receiverId}")
public ResponseEntity<?> sendConnection(
        @AuthenticationPrincipal UserDetails userDetails,
        @PathVariable Long receiverId) {

    if (userDetails == null) {
        return ResponseEntity.status(401).body("Unauthorized");
    }

    try {
        Long senderId = userService.getUserIdByEmail(userDetails.getUsername());
        userService.sendConnectionRequest(senderId, receiverId);
        return ResponseEntity.ok("Connection request sent");
    } catch (Exception e) {
        e.printStackTrace();
        return ResponseEntity.status(500).body("Error sending connection request: " + e.getMessage());
    }
}
```

### 2. **UserService.java**
```java
public void sendConnectionRequest(Long senderId, Long receiverId) {

    User sender = userRepository.findById(senderId)
            .orElseThrow(() -> new ResourceNotFoundException("Sender not found"));

    User receiver = userRepository.findById(receiverId)
            .orElseThrow(() -> new ResourceNotFoundException("Receiver not found"));

    connectionRepository.findBySenderAndReceiver(sender, receiver)
            .ifPresent(cr -> {
                throw new IllegalArgumentException("Connection request already sent");
            });

    ConnectionRequest cr = new ConnectionRequest();
    cr.setSender(sender);
    cr.setReceiver(receiver);
    cr.setStatus(ConnectionRequest.Status.PENDING);

    connectionRepository.save(cr);
}
```

## API Endpoints

### Send Connection Request
```
POST /api/users/connect/{receiverId}
Authorization: Bearer {jwt_token}
Content-Type: application/json

Body (Optional):
{
  "message": "Hi! I'd like to connect..."
}

Response:
200 OK
{
  "message": "Connection request sent"
}
```

### Get Pending Connection Requests (Received)
```
GET /api/users/connection-requests
Authorization: Bearer {jwt_token}

Response:
200 OK
[
  {
    "id": 1,
    "sender": {
      "id": 2,
      "name": "John Doe",
      "email": "john@example.com",
      ...
    },
    "receiver": {
      "id": 1,
      ...
    },
    "status": "PENDING"
  }
]
```

### Get Sent Connection Requests
```
GET /api/users/sent-requests
Authorization: Bearer {jwt_token}

Response:
200 OK
[
  {
    "id": 2,
    "sender": {
      "id": 1,
      ...
    },
    "receiver": {
      "id": 3,
      "name": "Jane Smith",
      "email": "jane@example.com",
      ...
    },
    "status": "PENDING"
  }
]
```

### Get Active Connections
```
GET /api/users/connections
Authorization: Bearer {jwt_token}

Response:
200 OK
[
  {
    "id": 5,
    "name": "Bob Wilson",
    "email": "bob@example.com",
    "role": "ALUMNI",
    ...
  }
]
```

### Respond to Connection Request
```
POST /api/users/connection/{requestId}/respond
Authorization: Bearer {jwt_token}
Content-Type: application/json

Body:
{
  "accept": true  // or false to reject
}

Response:
200 OK
{
  "message": "Connection Accepted"
}
```

## How to Use

### Step 1: Login to Network Page
1. Login with your account
2. Navigate to the Network page
3. You'll see a list of available users

### Step 2: Send Connection Request
1. Find the user you want to connect with
2. Click the **"+ Connect"** button on their card
3. A modal dialog will open showing:
   - User's profile information
   - Degree, Batch, College
   - Email and Bio
   - Statistics (Connections, Pending Requests)

### Step 3: Add Optional Message
1. In the modal, you can write an optional message
2. Click **"Send Request"** to send the connection request
3. You'll see a success notification

### Step 4: Track Connection Status
- **After sending**: Button changes to **"⏳ Request Sent"** (yellow)
- **When accepted**: Button changes to **"✓ Connected"** (green)
- **Incoming request**: Button shows **"✓ Pending"** (yellow)

## Database Schema

### ConnectionRequest Table
```sql
CREATE TABLE connection_request (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sender_id BIGINT NOT NULL,
    receiver_id BIGINT NOT NULL,
    status ENUM('PENDING', 'ACCEPTED', 'REJECTED') DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (sender_id) REFERENCES users(id),
    FOREIGN KEY (receiver_id) REFERENCES users(id),
    UNIQUE KEY unique_connection_request (sender_id, receiver_id)
);
```

## Error Handling

### Common Errors

1. **User doesn't exist**
   - Status: 404
   - Message: "Receiver not found"

2. **Request already sent**
   - Status: 400
   - Message: "Connection request already sent"

3. **Unauthorized**
   - Status: 401
   - Message: "Unauthorized"

4. **Server error**
   - Status: 500
   - Message: "Error sending connection request: {details}"

## Testing the Feature

### Test Scenario:
1. Create two accounts (or use existing accounts)
2. Login as Account 1
3. Navigate to Network page
4. Find Account 2 in the user list
5. Click "Connect" button
6. Add an optional message
7. Click "Send Request"
8. Verify button changes to "⏳ Request Sent"
9. Login as Account 2
10. You should see "✓ Pending" button for Account 1
11. You can accept or reject the request

## Features Implemented
✅ Send connection requests between users
✅ Optional message with connection request
✅ Real-time status tracking
✅ Connection request modal with user details
✅ Toast notifications for feedback
✅ Responsive design for mobile/tablet
✅ Connection status management
✅ View pending and active connections
✅ Accept/Reject connection requests

## File Locations
- Frontend: `frontend/pages/network.html`
- Backend Controller: `backend/src/main/java/com/alumnibridge/controller/UserController.java`
- Backend Service: `backend/src/main/java/com/alumnibridge/service/UserServiceImpl.java`
- API Client: `frontend/js/api.js`
