# AI-Powered Communication Assistant

An intelligent email management system that automatically filters, categorizes, prioritizes, and generates responses for support emails.

## 🚀 Features

- **Email Filtering**: Automatically filters support-related emails
- **AI Analysis**: Sentiment analysis and priority classification
- **Auto-Response**: Context-aware response generation using Groq API
- **Priority Queue**: Urgent emails processed first
- **Dashboard**: Real-time analytics and email management
- **Information Extraction**: Extracts key details from emails

## 🛠️ Tech Stack

- **Backend**: Node.js, Express.js, PostgreSQL
- **Frontend**: React, Material-UI, React Router
- **AI**: Groq API (openai/gpt-oss-20b)
- **Email**: IMAP/Gmail API integration
- **Database**: PostgreSQL with migrations

## 📦 Installation

### Prerequisites

- Node.js 16+
- PostgreSQL
- Groq API key
- Email service credentials

### Backend Setup

```bash
cd backend
npm install
cp .env.example .env
# Configure your environment variables in .env
npm run migrate
npm run dev
```

### Frontend Setup

```bash
cd frontend
npm install
cp .env.example .env
# Configure your environment variables in .env
npm start
```

## 🔧 Configuration

### Environment Variables

#### Backend (.env)

- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`: Database configuration
- `GROQ_API_KEY`: Your Groq API key
- `EMAIL_USER`, `EMAIL_PASSWORD`: Email service credentials
- `JWT_SECRET`: Secret for JWT tokens

#### Frontend (.env)

- `REACT_APP_API_URL`: Backend API URL

## 📊 Database Schema

- **emails**: Stores filtered emails with analysis results
- **responses**: Generated AI responses
- **extracted_info**: Key information extracted from emails
- **analytics**: Daily statistics and metrics

## 🎯 API Endpoints

### Emails

- `GET /api/emails` - Fetch filtered emails
- `GET /api/emails/:id` - Get specific email
- `POST /api/emails/process` - Process new emails
- `PUT /api/emails/:id/status` - Update email status

### Responses

- `GET /api/responses/:emailId` - Get responses for email
- `PUT /api/responses/:id` - Edit response
- `POST /api/responses/:id/send` - Send response

### Analytics

- `GET /api/analytics/dashboard` - Dashboard statistics
- `GET /api/analytics/trends` - Trend data
- `GET /api/analytics/sentiment` - Sentiment distribution

## 🧪 Testing

```bash
# Backend tests
cd backend
npm test

# Frontend tests
cd frontend
npm test
```

## 🚀 Deployment

1. Set up production environment variables
2. Build the frontend: `npm run build`
3. Set up PostgreSQL database
4. Run migrations: `npm run migrate`
5. Start the backend: `npm start`

## 📝 Development Status

This project was developed as part of the Unstop Hackathon (Sep 3-6, 2025).

## 👥 Team

Hackathon Team - Unstop AI Engineer Challenge

## 📄 License

MIT License
