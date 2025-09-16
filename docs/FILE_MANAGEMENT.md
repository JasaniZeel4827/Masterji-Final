# 📂 File Management System

## Overview

Masterji uses AWS S3-compatible storage for secure and scalable file management. This system handles all file uploads, including course materials, user avatars, and other media assets.

## 🏗️ Architecture

### Components

1. **Frontend**
   - File uploader component
   - Drag-and-drop interface
   - File preview and validation

2. **API Layer**
   - File upload endpoint
   - File deletion endpoint
   - Access control middleware

3. **Storage**
   - AWS S3 or S3-compatible storage
   - Organized bucket structure
   - CORS configuration

## 🔄 File Upload Flow

1. **Client-Side**
   - User selects a file through the UI
   - File is validated (size, type, etc.)
   - File is uploaded directly to S3 using pre-signed URLs
   - Upload progress is shown to the user

2. **Server-Side**
   - Generate pre-signed URL for upload
   - Verify user permissions
   - Process file metadata
   - Update database records

## 🛠️ Implementation

### S3 Client Configuration

```typescript
// lib/S3Client.ts
import { S3Client, PutObjectCommand, DeleteObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

const s3Client = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
  endpoint: process.env.AWS_S3_ENDPOINT,
  forcePathStyle: true,
});

export async function getUploadUrl(key: string, contentType: string) {
  const command = new PutObjectCommand({
    Bucket: process.env.AWS_S3_BUCKET_NAME,
    Key: key,
    ContentType: contentType,
  });

  return await getSignedUrl(s3Client, command, { expiresIn: 3600 });
}

export async function deleteFile(key: string) {
  const command = new DeleteObjectCommand({
    Bucket: process.env.AWS_S3_BUCKET_NAME,
    Key: key,
  });

  return await s3Client.send(command);
}
```

### File Upload API

```typescript
// app/api/s3/upload/route.ts
import { NextResponse } from "next/server";
import { getUploadUrl } from "@/lib/S3Client";
import { auth } from "@/lib/auth";

export async function POST(request: Request) {
  try {
    const session = await auth();
    if (!session) {
      return new NextResponse("Unauthorized", { status: 401 });
    }

    const { fileName, fileType } = await request.json();
    
    // Generate a unique key for the file
    const key = `uploads/${session.user.id}/${Date.now()}-${fileName}`;
    
    // Get pre-signed URL
    const url = await getUploadUrl(key, fileType);
    
    return NextResponse.json({ url, key });
  } catch (error) {
    console.error("Error generating upload URL:", error);
    return new NextResponse("Internal Server Error", { status: 500 });
  }
}
```

### File Upload Component

```tsx
// components/FileUploader.tsx
"use client";

import { useState, useCallback } from "react";

type FileWithPreview = File & {
  preview: string;
  uploadProgress: number;
  status: "idle" | "uploading" | "done" | "error";
};

export function FileUploader() {
  const [files, setFiles] = useState<FileWithPreview[]>([]);
  const [isUploading, setIsUploading] = useState(false);

  const onDrop = useCallback((acceptedFiles: File[]) => {
    const newFiles = acceptedFiles.map(file => ({
      ...file,
      preview: URL.createObjectURL(file),
      uploadProgress: 0,
      status: "idle" as const,
    }));
    
    setFiles(prev => [...prev, ...newFiles]);
  }, []);

  const uploadFile = async (file: File) => {
    try {
      // 1. Get pre-signed URL from our API
      const response = await fetch("/api/s3/upload", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          fileName: file.name,
          fileType: file.type,
        }),
      });
      
      const { url, key } = await response.json();
      
      // 2. Upload directly to S3 using the pre-signed URL
      const xhr = new XMLHttpRequest();
      
      xhr.upload.onprogress = (event) => {
        if (event.lengthComputable) {
          const progress = Math.round((event.loaded / event.total) * 100);
          updateFileProgress(file.name, progress);
        }
      };
      
      xhr.onload = () => {
        if (xhr.status === 200) {
          updateFileStatus(file.name, "done");
          // Optionally save the S3 key to your database
          saveFileMetadata(key, file.name);
        } else {
          updateFileStatus(file.name, "error");
        }
      };
      
      xhr.open("PUT", url);
      xhr.setRequestHeader("Content-Type", file.type);
      xhr.send(file);
      
    } catch (error) {
      console.error("Upload failed:", error);
      updateFileStatus(file.name, "error");
    }
  };

  // Helper functions to update file state
  const updateFileProgress = (fileName: string, progress: number) => {
    setFiles(prev => 
      prev.map(file => 
        file.name === fileName 
          ? { ...file, uploadProgress: progress, status: "uploading" as const }
          : file
      )
    );
  };

  const updateFileStatus = (fileName: string, status: FileWithPreview["status"]) => {
    setFiles(prev => 
      prev.map(file => 
        file.name === fileName ? { ...file, status } : file
      )
    );
  };

  return (
    <div className="space-y-4">
      <div 
        onDrop={(e) => {
          e.preventDefault();
          onDrop(Array.from(e.dataTransfer.files));
        }}
        onDragOver={(e) => e.preventDefault()}
        className="border-2 border-dashed rounded-lg p-8 text-center"
      >
        <p>Drag & drop files here, or click to select</p>
        <input
          type="file"
          onChange={(e) => e.target.files && onDrop(Array.from(e.target.files))}
          className="hidden"
          id="file-upload"
        />
        <label 
          htmlFor="file-upload"
          className="mt-2 inline-block px-4 py-2 bg-blue-500 text-white rounded cursor-pointer"
        >
          Select Files
        </label>
      </div>
      
      <div className="space-y-2">
        {files.map((file) => (
          <div key={file.name} className="border rounded p-3">
            <div className="flex justify-between items-center">
              <div className="truncate flex-1">
                <p className="font-medium truncate">{file.name}</p>
                <p className="text-sm text-gray-500">
                  {Math.round(file.size / 1024)} KB • {file.status}
                </p>
              </div>
              {file.status === "uploading" && (
                <div className="w-24 bg-gray-200 rounded-full h-2.5">
                  <div 
                    className="bg-blue-600 h-2.5 rounded-full" 
                    style={{ width: `${file.uploadProgress}%` }}
                  />
                </div>
              )}
              {file.status === "done" && (
                <span className="text-green-500">✓</span>
              )}
              {file.status === "error" && (
                <span className="text-red-500">✕</span>
              )}
            </div>
          </div>
        ))}
      </div>
      
      <button
        onClick={() => {
          setIsUploading(true);
          files.forEach(uploadFile);
        }}
        disabled={isUploading || files.length === 0}
        className="w-full py-2 px-4 bg-blue-600 text-white rounded disabled:opacity-50"
      >
        {isUploading ? "Uploading..." : `Upload ${files.length} Files`}
      </button>
    </div>
  );
}
```

## 🗑️ File Deletion

### Delete API Endpoint

```typescript
// app/api/s3/delete/route.ts
import { NextResponse } from "next/server";
import { deleteFile } from "@/lib/S3Client";
import { auth } from "@/lib/auth";

export async function DELETE(request: Request) {
  try {
    const session = await auth();
    if (!session) {
      return new NextResponse("Unauthorized", { status: 401 });
    }

    const { key } = await request.json();
    
    // Verify the user has permission to delete this file
    // (e.g., check if they own the resource)
    
    await deleteFile(key);
    
    return new NextResponse(null, { status: 204 });
  } catch (error) {
    console.error("Error deleting file:", error);
    return new NextResponse("Internal Server Error", { status: 500 });
  }
}
```

## 🔒 Security Considerations

1. **Access Control**
   - Verify user permissions before allowing uploads/deletions
   - Use pre-signed URLs with short expiration times
   - Implement CORS policies to restrict origins

2. **File Validation**
   - Validate file types on both client and server
   - Set maximum file size limits
   - Scan files for malware if possible

3. **Storage Organization**
   - Organize files by user ID and date
   - Use unique filenames to prevent overwrites
   - Implement lifecycle policies for old files

4. **Performance**
   - Use client-side compression for large files
   - Implement chunked uploads for very large files
   - Use CDN for file delivery

## 📊 Database Integration

### File Metadata Model

```prisma
// prisma/schema.prisma
model File {
  id        String   @id @default(cuid())
  key       String   @unique
  name      String
  size      Int
  type      String
  url       String
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### Saving File Metadata

```typescript
// app/actions/file.ts
"use server";

import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function saveFileMetadata(key: string, name: string, size: number, type: string) {
  const session = await auth();
  if (!session) {
    throw new Error("Unauthorized");
  }

  const file = await prisma.file.create({
    data: {
      key,
      name,
      size,
      type,
      url: `${process.env.S3_PUBLIC_URL}/${key}`,
      userId: session.user.id,
    },
  });

  return file;
}
```

## 🎯 Best Practices

1. **Client-Side**
   - Show file size and type before upload
   - Provide clear error messages
   - Show upload progress
   - Allow cancellation of in-progress uploads

2. **Server-Side**
   - Validate all uploads
   - Set appropriate Content-Type headers
   - Implement rate limiting
   - Log all file operations

3. **Storage**
   - Enable versioning for important files
   - Set up logging for S3 bucket
   - Configure lifecycle policies
   - Enable server-side encryption

## 🚀 Advanced Features

### 1. Image Processing

```typescript
// Example using Sharp for image processing
import sharp from "sharp";

export async function processImage(buffer: Buffer) {
  return await sharp(buffer)
    .resize(1200, 800, { fit: "inside", withoutEnlargement: true })
    .webp({ quality: 80 })
    .toBuffer();
}
```

### 2. Chunked Uploads

For large files, implement chunked uploads:

1. Split file into chunks on the client
2. Upload chunks in parallel
3. Notify server to combine chunks
4. Verify file integrity

### 3. Direct Uploads

For better performance, implement direct-to-S3 uploads:

1. Generate pre-signed POST URLs on the server
2. Upload directly from client to S3
3. Handle CORS appropriately
4. Verify upload completion

## 🔗 Related Documentation

- [AWS SDK for JavaScript v3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/)
- [Next.js API Routes](https://nextjs.org/docs/pages/building-your-application/routing/api-routes)
- [File Uploads with Next.js and AWS S3](https://dev.to/aws-builders/serverless-file-uploads-with-nextjs-aws-sdk-s3-presignedurl-1m6f)
