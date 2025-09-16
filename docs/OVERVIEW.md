# Masterji - System Overview

## 🏗️ Architecture

Masterji follows a modern full-stack architecture built with Next.js 13+ using the App Router. The application is structured as follows:

### Frontend
- **Framework**: Next.js 15.4.6 with TypeScript
- **UI Components**: Built with Radix UI primitives and shadcn/ui
- **State Management**: React Context + React Query for server state
- **Form Handling**: React Hook Form with Zod validation
- **Rich Text Editing**: TipTap for rich content editing
- **Styling**: Tailwind CSS with CSS Modules

### Backend
- **API Routes**: Next.js API routes for backend functionality
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: Custom auth implementation with NextAuth.js
- **File Storage**: AWS S3-compatible storage for media files
- **Payments**: Stripe integration for payment processing

## 📦 Core Modules

1. **Authentication Module**
   - Multi-provider authentication (GitHub, Email OTP)
   - Role-based access control (Admin, Instructor, Student)
   - Session management

2. **Course Management**
   - Course creation and organization
   - Chapter and lesson hierarchy
   - Rich content editing
   - File attachments and media embedding

3. **User Management**
   - User profiles and preferences
   - Enrollment management
   - Progress tracking

4. **Payment Processing**
   - Course purchase flow
   - Subscription management
   - Payment history

5. **Content Delivery**
   - Video streaming
   - File downloads
   - Progress tracking

## 🧩 Key Components

### App Router Structure
```
app/
├── (auth)/               # Authentication routes
├── (public)/             # Publicly accessible routes
├── admin/                # Admin dashboard
│   └── courses/          # Course management
├── api/                  # API routes
│   ├── auth/             # Authentication endpoints
│   └── s3/               # File upload endpoints
├── dashboard/            # User dashboard
└── payment/              # Payment processing
```

### Database Schema

The database schema is defined using Prisma and includes the following main models:

- **User**: User accounts and authentication
- **Course**: Course information and metadata
- **Chapter**: Course chapters
- **Lesson**: Individual lessons within chapters
- **Enrollment**: Tracks user enrollments in courses
- **Payment**: Payment records and transactions

## 🚀 Getting Started

1. **Prerequisites**
   - Node.js 18+
   - PostgreSQL database
   - AWS S3-compatible storage
   - Stripe account (for payments)

2. **Development Setup**
   ```bash
   # Install dependencies
   npm install
   
   # Set up environment variables
   cp .env.example .env.local
   # Edit .env.local with your configuration
   
   # Run database migrations
   npx prisma migrate dev
   
   # Start development server
   npm run dev
   ```

3. **Building for Production**
   ```bash
   # Build the application
   npm run build
   
   # Start production server
   npm start
   ```

## 📚 Next Steps

- [Authentication Flow](/docs/AUTHENTICATION.md)
- [File Management](/docs/FILE_MANAGEMENT.md)
- [Course Management](/docs/COURSE_MANAGEMENT.md)
- [Payment Integration](/docs/PAYMENTS.md)
- [Security Measures](/docs/SECURITY.md)
