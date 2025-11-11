#  FreelancerHub API Contract Summary

The base URL for all routes is `/api/v1` unless specified otherwise. All protected routes require JWT tokens stored in `httpOnly` cookies for **Authentication** (via `verifyToken` middleware).

---

## 1. Authentication & User Management

| Route | Method | Purpose | Authentication | Authorization | Request Body/Params | Success Response (200 OK) | Errors |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/:userDetails/login` | POST | User login for clients and admins | Not required | N/A | `userDetails` (path: `Client` or `Manager`); `email`, `password` (XOR-encrypted); `secretCode` (Manager only) | `{ "message": "Login successful", ... }` | 400, 401, 403, 404 |
| `/logout` | GET | User logout and clear authentication token | Required | N/A | N/A | `{ "message": "Logout successful" }` | N/A |
| `/verify-chatting-id` | POST | Verify and retrieve current user's chat ID | Required | N/A | N/A | `{ "chat_id": "USER_ID" }` | N/A |
| `/send-otp` | POST | Send OTP to email for account verification | Not required | N/A | `{ "email": "STRING" }` | `{ "message": "OTP sent successfully" }` | 400, 500 |
| `/security/checkAuth/permission/client` | POST | Verify client authorization | Required | `client` only | N/A | `{ "message": true }` | N/A |
| `/security/checkAuth/permission/freelancer` | POST | Verify freelancer authorization | Required | `freelancer` only | N/A | `{ "message": true }` | N/A |

---

## 2. File Uploads & Retrieval (AWS S3)

| Route | Method | Purpose | Authentication | Authorization | Request Body/Params | Success Response (200 OK) | Errors |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/upload-profile` | POST | Upload user profile picture | Required | `client` or `freelancer` | `multipart/form-data` (`file`: max 5MB, JPEG/PNG/GIF/WebP) | `{ "profile_Picture_Update": "Successfully" }` | 400, 404 |
| `/upload-resume` | POST | Upload freelancer resume | Required | `freelancer` only | `multipart/form-data` (`file`: max 10MB, PDF) | `{ "resume_Update": "Successfully" }` | 400, 404 |
| `/profile-picture` | GET | Get user's profile picture (with signed URL) | Required | N/A | N/A | `{ "profilePictureUrl": "SIGNED_S3_URL (expires in 1 hour)" }` | 404 |
| `/profile-picture/:userId` | GET | Get user's profile picture (public) | Not required | N/A | Path: `userId` | Image buffer | 404, 500 |
| `/resume/view/:userId` | GET | View user's resume as PDF | Not required | N/A | Path: `userId` | PDF document stream | 404, 500 |
| `/resume/download/:userId` | GET | Download user's resume as PDF | Not required | N/A | Path: `userId` | PDF file download | 404, 500 |
| `/worksubmission/upload-file` | POST | Upload project file | Required | N/A | `multipart/form-data` (`file`, `projectId`, `fileName`) | `{ "message": "File uploaded successfully", "fileUrl": "S3_URL" }` | N/A |
| `/worksubmission/projects/:projectId/files/:fileId` | DELETE | Delete project file | Optional | N/A | Path: `projectId`, `fileId` | `{ "message": "File deleted successfully" }` | N/A |

---

## 3. Chat & Real-time Communication

| Route | Method | Purpose | Authentication | Authorization | Request Body/Params | Success Response (200 OK) | Errors |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/chat/users` | GET | Get list of all chat users with last message and unread count | Required | N/A | N/A | `{ "users": [...] }` | N/A |
| `/chat/send` | POST | Send encrypted message to another user | Required | N/A | `{ "sender", "receiver", "message" }` | `{ "message": "Message sent successfully" }` | 400, 403, 500 |
| `/chat/messages` | GET | Retrieve conversation history (max 50 messages) | Required | N/A | Query: `sender`, `receiver` | `[{ ... message object ... }]` | N/A |
| **WebSocket** | N/A | Real-time messaging | N/A | N/A | Endpoint: `ws://SERVER:9000/USER_ID` | N/A | N/A |

---

## 4. Freelancer Specific Routes

| Route | Method | Purpose | Authentication | Authorization | Request Body/Params | Success Response (200 OK) | Errors |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/freelancer/skills` | GET | Get freelancer's skills | Required | `freelancer` only | N/A | `{ "skills": [...] }` | N/A |
| `/freelancer/wallet/details` | GET | Get freelancer's wallet, escrow balances, and transactions | Required | `freelancer` only | N/A | `{ "transactions": [...], "projects": [...] }` | N/A |
| `/freelancer/oldProjects` | GET | Get completed/old projects for freelancer | Required | `freelancer` only | N/A | `{ "projects": [...] }` | N/A |
| `/freelancer/projects/:projectId/:status` | PATCH | Update milestone/project status | Required | `freelancer` only | Path: `projectId`, `status` | `{ "message": "Milestone status updated" }` | N/A |
| `/freelancer/projects/:projectId/bid` | POST | Submit bid for a project | Required | `freelancer` only | Path: `projectId`; Body: `{ "amount", "message", "resumePermission" }` | `{ "message": "Bid submitted successfully" }` | N/A |
| `/freelancer/projects/bid/finalized` | GET | Get all bids submitted by freelancer | Required | `freelancer` only | N/A | `{ "bids": [...] }` | N/A |
| `/freelancer/projects/approved/work` | GET | Get approved projects for freelancer | Required | `freelancer` only | N/A | `{ "projects": [...] }` | N/A |
| `/freelancer/projects/:projectId/change-status/:status` | POST | Update project status by freelancer | Required | `freelancer` only | Path: `projectId`, `status` | `{ "message": "Status changed successfully" }` | N/A |
| `/freelancer/client-rating/:projectId` | POST | Submit rating/review for project | Required | `freelancer` or `client` | Path: `projectId`; Body: `{ "rating", "comments" }` | `{ "message": "Rating submitted successfully" }` | N/A |
| `/freelancer/freelancer/reviews` | GET | Get all reviews for freelancer | Required | `freelancer` only | N/A | `{ "reviews": [...] }` | N/A |

---

## 5. Client Specific Routes

| Route | Method | Purpose | Authentication | Authorization | Request Body/Params | Success Response (200 OK) | Errors |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/client/:freelancerId` | POST | Hire freelancer for a project | Required | `client` only | Path: `freelancerId`; Query: `projectId` | `{ "message": "Freelancer hired successfully" }` | N/A |
| `/client/ongoing/projects` | GET | Get all ongoing projects for client | Required | N/A | N/A | `[{ ... project object ... }]` | N/A |
| `/client/clients/projects` | GET | Get all projects created by client | Required | `client` only | N/A | `{ "projects": [...] }` | N/A |
| `/client/clients/projects/bids` | GET | Get open projects with pending bids | Required | `client` only | N/A | `{ "projects": [...] }` | N/A |
| `/client/clients/add-project` | POST | Create new project | Required | `client` only | `{ "title", "description", "budget", "deadline", "skills" (JSON string array), "Form_id" }` | `{ "message": "Project added successfully", "email", "username", "projectId" }` | N/A |
| `/client/get/wallet/:clientId` | GET | Get client's wallet, escrow, and transaction details | Required | `client` only | Path: `clientId` | `{ "total_balance", "refunded_balance", "total_deposited", "total_withdrawn", "transaction_history": [...] }` | N/A |

---

## 6. Payment (Razorpay & Escrow)

| Route | Method | Purpose | Authentication | Authorization | Request Body/Params | Success Response (200 OK) | Errors |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/payments/create-order` | POST | Create Razorpay payment order | Required | `client` only | `{ "amount", "currency", "project_id", "client_id" }` | `{ "id": "razorpay_order_id", "status": "created", ... }` | N/A |
| `/payments/verify-payment` | POST | Verify Razorpay payment and release to escrow | Required | `client` only | `{ "razorpay_order_id", "razorpay_payment_id", "razorpay_signature", "project_id", "client_id" }` | `{ "message": "Payment verified successfully", "escrowId": "ESCROW_ID" }` | 400, 500 |
| `/payments/release-payment` | POST | Release escrowed payment to freelancer | Required | `client` only | `{ "project_id", "freelancer_id" }` | `{ "message": "Funds released to freelancer" }` | N/A |
| `/payments/reject-project/:project_id` | POST | Reject project and refund payment | Required | `client` only | Path: `project_id`; Body: `{ "clientFeedback" }` | `{ "message": "Project rejected and freelancer notified" }` | N/A |
| `/payments/freelancer/withdraw/balance` | POST | Request withdrawal of freelancer balance | Required | `freelancer` only | `{ "accountNumber", "accountName", "ifscCode", "amount" (min 500) }` | `{ "message": "Withdrawal of ₹{amount} initiated." }` | 400, 500 |

---

## 7. Work Submission & Task Management

| Route | Method | Purpose | Authentication | Authorization | Request Body/Params | Success Response (200 OK) | Errors |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/worksubmission/tasks` | POST | Add task to ongoing project | Required | N/A | `{ "projectId", "title" }` | `{ "message": "Task added successfully", "task": { ... } }` | N/A |
| `/worksubmission/tasks/:taskId` | PATCH | Update task completion status | Required | N/A | Path: `taskId`; Body: `{ "projectId", "completed" }` | `{ "message": "Task updated successfully", "task": { "task_object" } }` | N/A |
| `/worksubmission/tasks/:taskId/:projectId` | DELETE | Delete task from project | Required | N/A | Path: `taskId`, `projectId` | `{ "message": "Task deleted successfully" }` | N/A |
| `/worksubmission/ongoing/projects/V1` | GET | Get all ongoing projects for freelancer | Required | N/A | N/A | `[{ ... project object ... }]` | N/A |
| `/worksubmission/projects/:id` | GET | Get specific project details | Required | N/A | Path: `id` (PROJECT_ID) | `[{ ... project details ... }]` | N/A |

---

## Global Configuration & Security Notes

* **CORS Origins (Allowed)**: `http://localhost:8080`, `http://localhost:8081`, `http://localhost:4000`, `https://freelancerhub-five.vercel.app`, `https://freelancerhub-loadbalancer.vercel.app`, `https://freelancer-admin.vercel.app`
* **Authentication**: JWT tokens stored in `httpOnly` cookies
* **Security**: Helmet security enabled, X-Powered-By header removed, Server header removed
* **Request/Response Limit**: 50MB
* **WebSocket**: Port 9000, Encryption using AES-256-GCM

### Common Error Structure

All errors follow this JSON structure:
```json
{
  "message": "ERROR_MESSAGE",
  "error": "OPTIONAL_ERROR_DETAILS"
}

```

| Status Code | Description |
| :--- | :--- |
| 200 | Success |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden/Banned |
| 404 | Not Found |
| 500 | Internal Server Error |
| 503 | Service Unavailable |