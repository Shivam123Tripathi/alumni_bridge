# Connection Request Feature - Code Reference

This document contains code snippets and examples for implementing the connection request feature.

## 1. FRONTEND - JavaScript Functions

### Open Connection Modal

```javascript
function openConnectionModal(userId) {
  const selectedUserId = userId;
  const user = allUsers.find(u => u.id === userId);
  
  if (!user) return;

  // Populate modal with user information
  document.getElementById('modalUserName').textContent = user.name;
  document.getElementById('modalUserRole').textContent = 
    user.role === 'STUDENT' ? '👨‍🎓 Student' : '👨‍💼 Alumni';
  document.getElementById('modalUserDegree').textContent = user.degreeName || 'N/A';
  document.getElementById('modalUserBatch').textContent = user.batchYear || 'N/A';
  document.getElementById('modalUserCollege').textContent = user.institute || 'N/A';
  document.getElementById('modalUserEmail').textContent = user.email || 'N/A';
  document.getElementById('modalUserBio').textContent = user.bio || 'No bio available';
  
  // Clear previous message and show modal
  document.getElementById('connectionMessage').value = '';
  document.getElementById('connectionModal').style.display = 'block';
}
```

### Close Connection Modal

```javascript
function closeConnectionModal() {
  document.getElementById('connectionModal').style.display = 'none';
}
```

### Send Connection Request

```javascript
async function sendConnectionRequestFromModal() {
  const selectedUserId = userId;  // Retrieved from openConnectionModal()
  
  if (!selectedUserId) return;

  try {
    const user = allUsers.find(u => u.id === selectedUserId);
    const message = document.getElementById('connectionMessage').value.trim();

    const response = await fetch(`/api/users/connect/${selectedUserId}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify({
        message: message || null
      })
    });

    if (response.ok) {
      closeConnectionModal();
      displayToast('Connection request sent!', 'success');
      
      // Update button state
      const btn = document.getElementById(`btn-${selectedUserId}`);
      btn.textContent = '⏳ Request Sent';
      btn.disabled = true;
      btn.className = 'user-card-btn btn-pending';
      
      // Reload statuses
      loadConnectionStatuses();
    } else {
      displayToast('Failed to send connection request', 'error');
    }
  } catch (error) {
    console.error('Error sending connection request:', error);
    displayToast('Error sending request', 'error');
  }
}
```

### Load Connection Statuses

```javascript
async function loadConnectionStatuses() {
  try {
    const token = localStorage.getItem('token');
    
    // Fetch three endpoints to determine status
    const [sentRequests, receivedRequests, connections] = await Promise.all([
      fetch('/api/users/sent-requests', {
        headers: { 'Authorization': `Bearer ${token}` }
      }).then(r => r.json()),
      fetch('/api/users/connection-requests', {
        headers: { 'Authorization': `Bearer ${token}` }
      }).then(r => r.json()),
      fetch('/api/users/connections', {
        headers: { 'Authorization': `Bearer ${token}` }
      }).then(r => r.json())
    ]);

    // Store for later use
    userSentRequests = sentRequests || [];
    userReceivedRequests = receivedRequests || [];
    userConnections = connections || [];
  } catch (error) {
    console.error('Error loading connection statuses:', error);
  }
}
```

## 2. Connection Status Logic

```javascript
// Determine button state for each user
function getConnectionButtonState(userId) {
  const currentUserId = localStorage.getItem('userId');
  
  // Check if already connected
  if (userConnections.some(c => c.id === userId)) {
    return { text: '✓ Connected', className: 'btn-connected', disabled: true };
  }
  
  // Check if request already sent
  if (userSentRequests.some(r => r.id === userId)) {
    return { text: '⏳ Request Sent', className: 'btn-pending', disabled: true };
  }
  
  // Check if received pending request
  if (userReceivedRequests.some(r => r.id === userId)) {
    return { text: '✓ Pending', className: 'btn-pending', disabled: true };
  }
  
  // Default: no connection
  return { text: '+ Connect', className: 'btn-connect', disabled: false };
}
```

## 3. HTML Structure

### Connection Request Modal

```html
<div id="connectionModal" class="modal">
  <div class="modal-content">
    <div class="modal-header">
      <h2>Send Connection Request</h2>
      <button class="close-btn" onclick="closeConnectionModal()">&times;</button>
    </div>
    <div class="modal-body">
      <!-- User Profile -->
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
        <div class="detail-item">
          <span class="detail-label">📚 Batch:</span>
          <span class="detail-value" id="modalUserBatch">N/A</span>
        </div>
        <div class="detail-item">
          <span class="detail-label">🏫 College:</span>
          <span class="detail-value" id="modalUserCollege">N/A</span>
        </div>
        <div class="detail-item">
          <span class="detail-label">✉️ Email:</span>
          <span class="detail-value" id="modalUserEmail">N/A</span>
        </div>
        <div class="detail-item">
          <span class="detail-label">📝 Bio:</span>
          <span class="detail-value" id="modalUserBio">N/A</span>
        </div>
      </div>

      <!-- Message -->
      <div class="connection-message">
        <label for="connectionMessage">Send a message (optional)</label>
        <textarea id="connectionMessage" 
          placeholder="Hi! I'd like to connect with you..."></textarea>
      </div>
    </div>

    <!-- Footer -->
    <div class="modal-footer">
      <button class="modal-btn modal-btn-secondary" onclick="closeConnectionModal()">
        Cancel
      </button>
      <button class="modal-btn modal-btn-primary" onclick="sendConnectionRequestFromModal()">
        Send Request
      </button>
    </div>
  </div>
</div>
```

### User Card Button

```html
<button onclick="openConnectionModal(${user.id})" 
        id="btn-${user.id}" 
        class="${btnClass}" 
        ${btnDisabled ? 'disabled' : ''}>
  ${btnText}
</button>
```

**Button States:**
- `+ Connect` - Blue, enabled (no connection)
- `⏳ Request Sent` - Yellow, disabled (request sent by current user)
- `✓ Connected` - Green, disabled (already connected)
- `✓ Pending` - Yellow, disabled (they sent request to current user)

## 4. CSS Styling

```css
.modal {
  display: none;
  position: fixed;
  z-index: 10000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background-color: white;
  margin: 2% auto;
  padding: 0;
  border-radius: 12px;
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}

.modal-header {
  background: linear-gradient(135deg, #4285f4 0%, #1967d2 100%);
  color: white;
  padding: 20px 25px;
  border-radius: 12px 12px 0 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  z-index: 100;
}

.modal-body {
  padding: 25px;
  max-height: calc(90vh - 140px);
  overflow-y: auto;
}

.modal-footer {
  padding: 20px 25px;
  border-top: 1px solid #e0e0e0;
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  background: white;
  position: sticky;
  bottom: 0;
}

.connection-message textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 14px;
  min-height: 80px;
  resize: vertical;
}
```

## 5. Page Load Initialization

```javascript
document.addEventListener("DOMContentLoaded", function () {
  checkAuth();
  const currentUserId = localStorage.getItem("userId");
  loadBatches();
  loadDegrees();
  loadUsers();
  loadConnectionStatuses();  // Load connection statuses
});
```

## 6. Usage Flow

```
1. User views Network page
   ↓
2. Users are loaded from /api/users/search
   ↓
3. Connection statuses loaded from 3 API endpoints
   ↓
4. Each user card shows appropriate button:
   - "+ Connect" if no connection
   - "⏳ Request Sent" if already sent
   - "✓ Connected" if connected
   - "✓ Pending" if they sent request
   ↓
5. User clicks connect button
   ↓
6. openConnectionModal(userId) opens modal
   ↓
7. Modal displays user details
   ↓
8. User can add optional message
   ↓
9. Click "Send Request"
   ↓
10. sendConnectionRequestFromModal() calls:
    POST /api/users/connect/{userId}
   ↓
11. Backend creates ConnectionRequest in DB
   ↓
12. Button updates to "⏳ Request Sent"
   ↓
13. Toast notification shows success
   ↓
14. Other user sees "✓ Pending" button
   ↓
15. Other user can accept/reject request
   ↓
16. Both users see "✓ Connected" status
```

---

**Note:** This is a reference document for the connection request feature implementation. See `CONNECTION_REQUEST_GUIDE.md` for complete API documentation and error handling details.
