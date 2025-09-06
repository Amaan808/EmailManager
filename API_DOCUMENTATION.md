# AI Communication Assistant - API Documentation

## 🌐 Base URL

```
Development: http://localhost:5001/api
Production: [To be configured]
```

## 📧 Email Management APIs

### GET /api/emails

Retrieve emails with filtering and pagination.

**Query Parameters:**

- `page` (number): Page number (default: 1)
- `limit` (number): Items per page (default: 10)
- `priority` (string): Filter by priority (urgent, normal)
- `sentiment` (string): Filter by sentiment (positive, negative, neutral)
- `status` (string): Filter by status (pending, processing, completed, sent)
- `search` (string): Search in subject or sender

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "sender": "user@example.com",
      "subject": "Support Request: Login Issue",
      "body": "I'm having trouble logging into my account...",
      "received_date": "2025-09-06T10:30:00Z",
      "priority": "normal",
      "sentiment": "negative",
      "status": "completed",
      "created_at": "2025-09-06T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 150,
    "page": 1,
    "limit": 10,
    "hasMore": true
  }
}
```

### GET /api/emails/:id

Get specific email details.

**Response:**

```json
{
  "success": true,
  "data": {
    "id": 1,
    "sender": "user@example.com",
    "subject": "Support Request: Login Issue",
    "body": "Complete email body content...",
    "received_date": "2025-09-06T10:30:00Z",
    "priority": "normal",
    "sentiment": "negative",
    "status": "completed"
  }
}
```

### POST /api/emails/process

Trigger manual email processing from email server.

**Response:**

```json
{
  "success": true,
  "message": "Email processing initiated",
  "data": {
    "processedEmails": 5,
    "newEmails": 3
  }
}
```

### PUT /api/emails/:id/status

Update email processing status.

**Request Body:**

```json
{
  "status": "completed"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Email status updated successfully"
}
```

## 🤖 AI Processing APIs

### POST /api/ai/process/:emailId

Process specific email with AI analysis.

**Response:**

```json
{
  "success": true,
  "data": {
    "emailId": 1,
    "sentiment": "negative",
    "priority": "normal",
    "extractedInfo": {
      "contactDetails": { "email": "user@example.com" },
      "requirements": "Login access restoration",
      "urgentKeywords": [],
      "issueCategory": "account"
    },
    "responseGenerated": true
  }
}
```

### POST /api/ai/generate-response/:emailId

Generate AI response for specific email.

**Response:**

```json
{
  "success": true,
  "data": {
    "emailId": 1,
    "responseId": 25,
    "originalText": "I'm having trouble logging into...",
    "sentiment": "negative",
    "priority": "normal",
    "generatedResponse": "Thank you for contacting us. I understand you're experiencing login difficulties..."
  }
}
```

### GET /api/ai/analysis/:emailId

Get AI analysis results for email.

**Response:**

```json
{
  "success": true,
  "data": {
    "sentiment": "negative",
    "priority": "normal",
    "extractedInfo": {
      "contactDetails": { "email": "user@example.com" },
      "requirements": "Password reset assistance"
    },
    "processingTime": "2.3s"
  }
}
```

### POST /api/ai/test

Test AI functionality with sample data.

**Request Body:**

```json
{
  "emailContent": "I need help with my account",
  "senderEmail": "test@example.com"
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "sentiment": "neutral",
    "priority": "normal",
    "testPassed": true
  }
}
```

## 💬 Response Management APIs

### GET /api/responses

Get all responses with pagination and filtering.

**Query Parameters:**

- `limit` (number): Items per page (default: 10)
- `offset` (number): Offset for pagination (default: 0)
- `status` (string): Filter by status (draft, sent, failed)
- `search` (string): Search in response content

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "email_id": 5,
      "email_subject": "Support Request: Login Issue",
      "email_from": "user@example.com",
      "content": "Thank you for contacting us...",
      "status": "draft",
      "created_at": "2025-09-06T11:00:00Z"
    }
  ],
  "pagination": {
    "total": 50,
    "limit": 10,
    "offset": 0,
    "hasMore": true
  }
}
```

### GET /api/responses/:emailId

Get responses for specific email.

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "generated_response": "Thank you for contacting us...",
      "edited_response": null,
      "is_sent": false,
      "created_at": "2025-09-06T11:00:00Z"
    }
  ]
}
```

### PUT /api/responses/:id

Edit a generated response.

**Request Body:**

```json
{
  "editedResponse": "Updated response content..."
}
```

**Response:**

```json
{
  "success": true,
  "message": "Response updated successfully"
}
```

### POST /api/responses/:id/send

Send an approved response.

**Response:**

```json
{
  "success": true,
  "message": "Response sent successfully",
  "data": {
    "id": 1,
    "is_sent": true,
    "sent_at": "2025-09-06T12:00:00Z"
  }
}
```

## 📊 Analytics APIs

### GET /api/analytics/dashboard

Get dashboard statistics.

**Response:**

```json
{
  "success": true,
  "data": {
    "totalEmails": 150,
    "todayEmails": 12,
    "resolvedEmails": 140,
    "pendingEmails": 10,
    "urgentEmails": 5,
    "positiveEmails": 80,
    "negativeEmails": 30,
    "neutralEmails": 40,
    "totalResponses": 135,
    "sentResponses": 120
  }
}
```

### GET /api/analytics/ai-performance

Get AI processing performance metrics.

**Response:**

```json
{
  "success": true,
  "data": {
    "totalProcessed": 150,
    "sentimentAnalyzed": 150,
    "priorityClassified": 150,
    "infoExtracted": 145,
    "responsesGenerated": 135,
    "positiveSentiment": 80,
    "negativeSentiment": 30,
    "neutralSentiment": 40,
    "urgentPriority": 15,
    "normalPriority": 135
  }
}
```

### GET /api/analytics/trends

Get trend data for analytics charts.

**Query Parameters:**

- `period` (string): Time period (7d, 30d, 90d)

**Response:**

```json
{
  "success": true,
  "data": {
    "dates": ["2025-09-01", "2025-09-02", "2025-09-03"],
    "emailCounts": [10, 15, 12],
    "responseCounts": [8, 12, 10],
    "avgResponseTime": [2.5, 3.1, 2.8]
  }
}
```

## 🚦 Queue Management APIs

### GET /api/queue/status

Get current queue processing status.

**Response:**

```json
{
  "success": true,
  "data": {
    "isProcessing": true,
    "pendingEmails": 5,
    "processingEmails": 2,
    "failedEmails": 1,
    "lastProcessedAt": "2025-09-06T12:00:00Z",
    "averageProcessingTime": "3.2s"
  }
}
```

### POST /api/queue/process-all

Process all pending emails in queue.

**Response:**

```json
{
  "success": true,
  "message": "Queue processing initiated",
  "data": {
    "pendingCount": 10,
    "estimatedTime": "30s"
  }
}
```

### POST /api/queue/queue/:emailId

Add specific email to processing queue.

**Response:**

```json
{
  "success": true,
  "message": "Email queued for processing"
}
```

### POST /api/queue/pause

Pause queue processing.

**Response:**

```json
{
  "success": true,
  "message": "Queue processing paused"
}
```

### POST /api/queue/resume

Resume queue processing.

**Response:**

```json
{
  "success": true,
  "message": "Queue processing resumed"
}
```

## 🔧 System APIs

### GET /api/health

Health check endpoint.

**Response:**

```json
{
  "success": true,
  "status": "healthy",
  "timestamp": "2025-09-06T12:00:00Z",
  "uptime": "2h 30m",
  "database": "connected",
  "aiService": "operational"
}
```

## ⚠️ Error Responses

All endpoints follow consistent error response format:

```json
{
  "success": false,
  "error": "Error message description",
  "code": "ERROR_CODE",
  "timestamp": "2025-09-06T12:00:00Z"
}
```

### Common HTTP Status Codes

- `200` - Success
- `400` - Bad Request
- `401` - Unauthorized
- `404` - Not Found
- `429` - Rate Limit Exceeded
- `500` - Internal Server Error

### Common Error Codes

- `INVALID_EMAIL_ID` - Invalid email ID provided
- `EMAIL_NOT_FOUND` - Email not found in database
- `AI_SERVICE_UNAVAILABLE` - GROQ API service unavailable
- `RATE_LIMIT_EXCEEDED` - API rate limit exceeded
- `DATABASE_ERROR` - Database operation failed

## 🔐 Authentication

Currently, the API uses environment-based configuration. For production deployment, implement:

- JWT token authentication
- API key validation
- Role-based access control

## 📝 Rate Limiting

- GROQ API: 8000 tokens per minute
- Database operations: Connection pooled
- API endpoints: No explicit limits (implement as needed)

## 🧪 Testing

Use the provided test endpoints:

- `POST /api/ai/test` - Test AI functionality
- `GET /api/health` - System health check
- `POST /api/emails/process` - Test email processing

---

**API Version:** 1.0  
**Documentation Generated:** September 6, 2025  
**Base URL:** http://localhost:5001/api  
**Status:** Production Ready
