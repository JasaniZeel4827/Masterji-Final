# Masterji - Modern Learning Management System

[![Next.js](https://img.shields.io/badge/Next.js-15.4.6-black?style=flat-square&logo=next.js)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19.1.0-blue?style=flat-square&logo=react)](https://reactjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![Prisma](https://img.shields.io/badge/Prisma-6.13.0-blue?style=flat-square&logo=prisma)](https://www.prisma.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-blue?style=flat-square&logo=postgresql)](https://www.postgresql.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Overview

**Masterji** is a modern, full-stack Learning Management System (LMS) built with cutting-edge web technologies. It provides a comprehensive platform for creating, managing, and delivering online courses with advanced features like drag-and-drop course structuring, rich text editing, and secure file management.

## Key Features

- **User Authentication**: Secure login with multiple providers (GitHub, email OTP)
- **Course Management**: Create and organize courses with chapters and lessons
- **Rich Content Editor**: Advanced text editing with TipTap integration
- **File Management**: Secure file uploads with S3-compatible storage
- **Responsive Design**: Mobile-first approach with modern UI components
- **Admin Dashboard**: Comprehensive course administration tools
- **Payment Processing**: Integrated Stripe payment system

## Quick Start

1. Clone the repository
2. Install dependencies: `npm install`
3. Set up environment variables (see [.env.example](.env.example))
4. Run database migrations: `npx prisma migrate dev`
5. Start the development server: `npm run dev`

For detailed setup instructions, see the [Development Guide](/docs/DEVELOPMENT.md).

## Tech Stack

### Frontend
- **Framework**: Next.js 15.4.6 (App Router)
- **Language**: TypeScript 5.0
- **Styling**: Tailwind CSS 4.0
- **UI Components**: Radix UI + shadcn/ui
- **State Management**: React Hook Form + Zod validation
- **Icons**: Lucide React, Tabler Icons

### Backend
- **Runtime**: Node.js
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: Better Auth with multiple providers
- **API**: Next.js API Routes
- **Validation**: Zod schemas

### Infrastructure & Services
- **File Storage**: AWS S3-compatible storage
- **Email Service**: Resend for transactional emails
- **Security**: Arcjet for bot detection and rate limiting
- **Deployment**: Vercel-ready configuration

## Folder Structure

```
masterji/
├── app/                          # Next.js App Router
│   ├── (auth)/                  # Authentication routes
│   │   ├── _components/         # Auth-specific components
│   │   ├── login/               # Login page
│   │   └── verify-request/      # Email verification
│   ├── (public)/                # Public routes
│   │   ├── _components/         # Public components (Navbar, UserDropdown)
│   │   └── page.tsx             # Landing page
│   ├── admin/                   # Admin dashboard
│   │   ├── courses/             # Course management
│   │   │   ├── _components/     # Admin course components
│   │   │   ├── [courseId]/      # Dynamic course routes
│   │   │   │   └── edit/        # Course editing interface
│   │   │   └── create/          # Course creation
│   │   └── layout.tsx           # Admin layout
│   ├── api/                     # API routes
│   │   ├── auth/                # Authentication endpoints
│   │   └── s3/                  # File upload/delete endpoints
│   ├── data/                    # Server-side data functions
│   │   └── admin/               # Admin-specific data operations
│   ├── globals.css              # Global styles
│   └── layout.tsx               # Root layout
├── components/                   # Reusable UI components
│   ├── file-uploader/           # File upload components
│   ├── rich-text-editor/        # TipTap-based text editor
│   ├── sidebar/                 # Navigation components
│   └── ui/                      # shadcn/ui components
├── hooks/                       # Custom React hooks
├── lib/                         # Utility libraries
│   ├── generated/               # Prisma generated types
│   ├── auth.ts                  # Authentication configuration
│   ├── db.ts                    # Database connection
│   ├── S3Client.ts              # S3 client configuration
│   └── zodSchemas.ts            # Data validation schemas
├── prisma/                      # Database schema and migrations
│   └── schema.prisma            # Prisma schema definition
├── public/                      # Static assets
└── middleware.ts                # Next.js middleware
```

## Contributing

### Prerequisites
- Node.js 18+ 
- PostgreSQL database
- S3-compatible storage service
- GitHub OAuth application (for social login)

### 1. Clone Repository
```bash
git clone <repository-url>
cd masterji
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Environment Configuration
Create a `.env.local` file in the root directory:

```env
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/masterji"

# Authentication
BETTER_AUTH_SECRET="your-secret-key"
BETTER_AUTH_URL="http://localhost:3000"
AUTH_GITHUB_CLIENT_ID="your-github-client-id"
AUTH_GITHUB_SECRET="your-github-client-secret"

# Email Service
RESEND_API_KEY="your-resend-api-key"

# Security
ARCJET_KEY="your-arcjet-key"

# AWS S3 Storage
AWS_ACCESS_KEY_ID="your-access-key"
AWS_SECRET_ACCESS_KEY="your-secret-key"
AWS_ENDPOINT_URL_S3="your-s3-endpoint"
AWS_ENDPOINT_URL_IAM="your-iam-endpoint"
AWS_REGION="your-region"

# Client-side
NEXT_PUBLIC_S3_BUCKET_NAME_IMAGES="your-bucket-name"
```

### 4. Database Setup
```bash
# Generate Prisma client
npx prisma generate

# Run database migrations
npx prisma db push

# (Optional) Seed database
npx prisma db seed
```

### 5. Development Server
```bash
npm run dev
```

The application will be available at `http://localhost:3000`

## 💻 Usage

### For Learners
1. **Browse Courses**: Visit the public landing page to explore available courses
2. **User Registration**: Sign up using email or GitHub OAuth
3. **Course Access**: Enroll in courses and track progress
4. **Learning Interface**: Access course content through organized chapters and lessons

### For Instructors/Admins
1. **Admin Access**: Login with admin credentials
2. **Course Creation**: Use the admin dashboard to create new courses
3. **Content Management**: Organize courses into chapters and lessons
4. **Media Upload**: Upload images and videos using the integrated file manager
5. **Course Publishing**: Manage course status (draft, published, archived)

### Key Functionalities
- **Authentication**: Secure login with multiple providers
- **Course CRUD**: Full course lifecycle management
- **Content Organization**: Hierarchical chapter-lesson structure
- **File Management**: Secure upload and storage of media files
- **Rich Text Editing**: Advanced content creation with TipTap
- **Responsive Design**: Mobile-optimized interfaces

## 🌟 Features

### Core LMS Features
- **Course Management**: Create, edit, and organize online courses
- **Content Hierarchy**: Structured chapters and lessons system
- **User Roles**: Admin and learner role management
- **Progress Tracking**: Monitor learning progress and achievements

### Advanced Features
- **Drag & Drop**: Intuitive course structure management
- **Rich Text Editor**: Professional content creation tools
- **File Upload**: Secure media management with progress tracking
- **Responsive UI**: Modern, mobile-first design
- **Theme Support**: Light/dark mode with system preference detection

### Security Features
- **Bot Protection**: Advanced bot detection and blocking
- **Rate Limiting**: API protection against abuse
- **Secure Authentication**: Multi-provider auth with session management
- **File Security**: Presigned URLs for secure file access

## 🔐 Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string | ✅ |
| `BETTER_AUTH_SECRET` | Authentication secret key | ✅ |
| `BETTER_AUTH_URL` | Application base URL | ✅ |
| `AUTH_GITHUB_CLIENT_ID` | GitHub OAuth client ID | ✅ |
| `AUTH_GITHUB_SECRET` | GitHub OAuth client secret | ✅ |
| `RESEND_API_KEY` | Resend email service API key | ✅ |
| `ARCJET_KEY` | Arcjet security service key | ✅ |
| `AWS_ACCESS_KEY_ID` | S3 storage access key | ✅ |
| `AWS_SECRET_ACCESS_KEY` | S3 storage secret key | ✅ |
| `AWS_ENDPOINT_URL_S3` | S3 service endpoint | ✅ |
| `AWS_ENDPOINT_URL_IAM` | IAM service endpoint | ✅ |
| `AWS_REGION` | AWS region | ✅ |
| `NEXT_PUBLIC_S3_BUCKET_NAME_IMAGES` | S3 bucket name for images | ✅ |

## 🚀 Deployment

### Local Development
```bash
npm run dev          # Development server
npm run build        # Production build
npm run start        # Production server
npm run lint         # Code linting
```

### Production Deployment

#### Vercel (Recommended)
1. Connect your GitHub repository to Vercel
2. Configure environment variables in Vercel dashboard
3. Deploy automatically on push to main branch

#### Docker Deployment
```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

#### Manual Deployment
1. Build the application: `npm run build`
2. Start the production server: `npm run start`
3. Configure reverse proxy (nginx/Apache) if needed
4. Set up SSL certificates for HTTPS

## 📱 Screenshots & Demo

### Screenshots
- Landing page with hero section
- Admin dashboard showing course management
- Course creation interface
- Rich text editor in action
- File upload progress
- Mobile responsive design

### Demo Links
- Live demo: [Add your demo URL]
- Video walkthrough: [Add video link]

## 🤝 Contributing

We welcome contributions to Masterji! Please follow these guidelines:

### Development Setup
1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Make your changes following the existing code style
4. Add tests for new functionality
5. Commit your changes: `git commit -m 'Add amazing feature'`
6. Push to the branch: `git push origin feature/amazing-feature`
7. Open a Pull Request

### Code Style
- Follow TypeScript best practices
- Use Prettier for code formatting
- Follow ESLint rules
- Write meaningful commit messages
- Add JSDoc comments for complex functions

### Testing
- Ensure all tests pass before submitting PR
- Add tests for new features
- Update tests when modifying existing functionality

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

## 🔗 Integrations & Services

### Authentication & Security
- **Better Auth**: Modern authentication library with multiple providers
- **Arcjet**: Bot detection and rate limiting protection
- **GitHub OAuth**: Social login integration

### Storage & Media
- **AWS S3**: Scalable file storage for images and videos
- **Presigned URLs**: Secure, time-limited file access

### Database & ORM
- **Prisma**: Type-safe database client and migrations
- **PostgreSQL**: Robust relational database

### Email & Communication
- **Resend**: Transactional email service for verification

### UI & Components
- **shadcn/ui**: High-quality, accessible UI components
- **Radix UI**: Unstyled, accessible component primitives
- **Tailwind CSS**: Utility-first CSS framework

## 🚀 Performance & Best Practices

### Optimizations Implemented
- **Next.js 15**: Latest framework with App Router
- **Turbopack**: Fast bundler for development
- **Image Optimization**: Next.js built-in image optimization
- **Code Splitting**: Automatic route-based code splitting
- **TypeScript**: Type safety and better developer experience

### Security Best Practices
- **Input Validation**: Zod schema validation
- **Rate Limiting**: API protection against abuse
- **Secure File Uploads**: Presigned URLs and file type validation
- **Authentication**: Secure session management
- **Environment Variables**: Proper secret management

### Accessibility
- **Radix UI**: Built-in accessibility features
- **Semantic HTML**: Proper HTML structure
- **Keyboard Navigation**: Full keyboard support
- **Screen Reader**: ARIA labels and descriptions
