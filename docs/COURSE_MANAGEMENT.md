# 🎓 Course Management System

## Overview

The course management system in Masterji allows instructors to create, organize, and manage online courses with a hierarchical structure of chapters and lessons. This document outlines the architecture, components, and implementation details of the course management system.

## 🏗️ Architecture

### Core Components

1. **Course**
   - Top-level container for course content
   - Contains metadata like title, description, and pricing
   - Has many chapters

2. **Chapter**
   - Groups related lessons together
   - Has a title and optional description
   - Contains many lessons

3. **Lesson**
   - Individual learning unit
   - Can contain various content types (text, video, quiz, etc.)
   - Tracks completion status for each user

## 📚 Database Schema

```prisma
// prisma/schema.prisma

model Course {
  id          String    @id @default(cuid())
  title       String
  description String?
  price       Decimal   @default(0)
  imageUrl    String?
  isPublished Boolean   @default(false)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  userId      String
  user        User      @relation(fields: [userId], references: [id])
  chapters    Chapter[]
  enrollments Enrollment[]
  
  @@index([userId])
}

model Chapter {
  id          String    @id @default(cuid())
  title       String
  description String?
  position    Int
  isPublished Boolean   @default(false)
  courseId    String
  course      Course    @relation(fields: [courseId], references: [id], onDelete: Cascade)
  lessons     Lesson[]
  
  @@index([courseId])
}

model Lesson {
  id          String    @id @default(cuid())
  title       String
  description String?
  position    Int
  content     Json?     // Rich text content
  videoUrl    String?
  isFree      Boolean   @default(false)
  isPublished Boolean   @default(false)
  chapterId   String
  chapter     Chapter   @relation(fields: [chapterId], references: [id], onDelete: Cascade)
  userProgress UserProgress[]
  
  @@index([chapterId])
}
```

## 🛠️ Implementation

### Course Creation Flow

1. **Create Course**
   - Basic course information (title, description, price)
   - Upload course image
   - Set course visibility (draft/published)

2. **Add Chapters**
   - Organize content into logical sections
   - Set chapter order
   - Optional chapter descriptions

3. **Create Lessons**
   - Add lesson content using rich text editor
   - Upload videos or embed external content
   - Set lesson order within chapters
   - Mark lessons as free/premium

### Course Editor Component

```tsx
// components/course/CourseEditor.tsx
"use client";

import { useState } from "react";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { FileUpload } from "@/components/file-upload";

const courseFormSchema = z.object({
  title: z.string().min(1, "Title is required"),
  description: z.string().optional(),
  price: z.coerce.number().min(0, "Price must be 0 or greater"),
  imageUrl: z.string().optional(),
});

type CourseFormValues = z.infer<typeof courseFormSchema>;

export function CourseEditor({
  initialData,
  onSubmit,
  isSubmitting,
}: {
  initialData?: Partial<CourseFormValues>;
  onSubmit: (data: CourseFormValues) => Promise<void>;
  isSubmitting: boolean;
}) {
  const [imageUrl, setImageUrl] = useState(initialData?.imageUrl || "");
  
  const form = useForm<CourseFormValues>({
    resolver: zodResolver(courseFormSchema),
    defaultValues: {
      title: initialData?.title || "",
      description: initialData?.description || "",
      price: initialData?.price || 0,
    },
  });

  const handleImageUpload = (url: string) => {
    setImageUrl(url);
    form.setValue("imageUrl", url);
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
      <div className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-1">
            Course Image
          </label>
          <FileUpload
            value={imageUrl}
            onChange={handleImageUpload}
            endpoint="courseImage"
          />
        </div>
        
        <div>
          <label htmlFor="title" className="block text-sm font-medium mb-1">
            Course Title
          </label>
          <Input
            id="title"
            placeholder="e.g., Introduction to Web Development"
            {...form.register("title")}
            disabled={isSubmitting}
          />
          {form.formState.errors.title && (
            <p className="text-sm text-red-500 mt-1">
              {form.formState.errors.title.message}
            </p>
          )}
        </div>
        
        <div>
          <label htmlFor="description" className="block text-sm font-medium mb-1">
            Description
          </label>
          <Textarea
            id="description"
            placeholder="What's this course about?"
            {...form.register("description")}
            disabled={isSubmitting}
            rows={4}
          />
        </div>
        
        <div>
          <label htmlFor="price" className="block text-sm font-medium mb-1">
            Price (in USD)
          </label>
          <Input
            id="price"
            type="number"
            step="0.01"
            placeholder="0.00"
            {...form.register("price")}
            disabled={isSubmitting}
          />
          {form.formState.errors.price && (
            <p className="text-sm text-red-500 mt-1">
              {form.formState.errors.price.message}
            </p>
          )}
        </div>
      </div>
      
      <div className="flex justify-end space-x-2">
        <Button type="button" variant="outline" disabled={isSubmitting}>
          Cancel
        </Button>
        <Button type="submit" disabled={isSubmitting}>
          {isSubmitting ? "Saving..." : "Save"}
        </Button>
      </div>
    </form>
  );
}
```

### Chapter and Lesson Management

```tsx
// components/course/ChapterList.tsx
"use client";

import { useState } from "react";
import { Chapter, Lesson } from "@prisma/client";
import { Button } from "@/components/ui/button";
import { Plus, Pencil, Trash2, ChevronUp, ChevronDown, GripVertical } from "lucide-react";
import { 
  DragDropContext, 
  Droppable, 
  Draggable,
  DropResult
} from "@hello-pangea/dnd";

interface ChapterListProps {
  chapters: (Chapter & { lessons: Lesson[] })[];
  courseId: string;
  onReorder: (updateData: { id: string; position: number }[]) => void;
  onEdit: (id: string) => void;
  onDelete: (id: string) => void;
  onCreate: () => void;
}

export function ChapterList({
  chapters = [],
  courseId,
  onReorder,
  onEdit,
  onDelete,
  onCreate,
}: ChapterListProps) {
  const [isReordering, setIsReordering] = useState(false);

  const onDragEnd = (result: DropResult) => {
    if (!result.destination) return;
    
    const items = Array.from(chapters);
    const [reorderedItem] = items.splice(result.source.index, 1);
    items.splice(result.destination.index, 0, reorderedItem);
    
    const updatedChapters = items.map((item, index) => ({
      id: item.id,
      position: index,
    }));
    
    onReorder(updatedChapters);
    setIsReordering(false);
  };

  return (
    <div className="border rounded-md">
      <div className="flex items-center justify-between p-4 border-b">
        <h3 className="font-medium">Course Chapters</h3>
        <div className="flex space-x-2">
          <Button
            onClick={() => setIsReordering(!isReordering)}
            variant="outline"
            size="sm"
          >
            {isReordering ? "Done" : "Reorder"}
          </Button>
          <Button onClick={onCreate} size="sm">
            <Plus className="h-4 w-4 mr-2" />
            Add Chapter
          </Button>
        </div>
      </div>
      
      <DragDropContext onDragEnd={onDragEnd}>
        <Droppable droppableId="chapters">
          {(provided) => (
            <div {...provided.droppableProps} ref={provided.innerRef} className="divide-y">
              {chapters.map((chapter, index) => (
                <Draggable 
                  key={chapter.id} 
                  draggableId={chapter.id} 
                  index={index}
                  isDragDisabled={!isReordering}
                >
                  {(provided) => (
                    <div
                      ref={provided.innerRef}
                      {...provided.draggableProps}
                      className="p-4 bg-white hover:bg-gray-50"
                    >
                      <div className="flex items-center justify-between">
                        <div className="flex items-center space-x-2">
                          {isReordering ? (
                            <div {...provided.dragHandleProps} className="cursor-move">
                              <GripVertical className="h-5 w-5 text-gray-400" />
                            </div>
                          ) : null}
                          <h4 className="font-medium">{chapter.title}</h4>
                        </div>
                        <div className="flex space-x-2">
                          <Button
                            variant="ghost"
                            size="icon"
                            onClick={() => onEdit(chapter.id)}
                          >
                            <Pencil className="h-4 w-4" />
                          </Button>
                          <Button
                            variant="ghost"
                            size="icon"
                            onClick={() => onDelete(chapter.id)}
                          >
                            <Trash2 className="h-4 w-4 text-red-500" />
                          </Button>
                        </div>
                      </div>
                      
                      {/* Lessons list */}
                      <div className="mt-2 pl-8 space-y-2">
                        {chapter.lessons.map((lesson) => (
                          <div key={lesson.id} className="flex items-center justify-between p-2 hover:bg-gray-50 rounded">
                            <span className="text-sm">{lesson.title}</span>
                            <div className="flex space-x-2">
                              <Button
                                variant="ghost"
                                size="icon"
                                onClick={() => {}}
                              >
                                <Pencil className="h-3 w-3" />
                              </Button>
                              <Button
                                variant="ghost"
                                size="icon"
                                onClick={() => {}}
                              >
                                <Trash2 className="h-3 w-3 text-red-500" />
                              </Button>
                            </div>
                          </div>
                        ))}
                        <Button variant="ghost" size="sm" className="text-xs">
                          <Plus className="h-3 w-3 mr-1" />
                          Add Lesson
                        </Button>
                      </div>
                    </div>
                  )}
                </Draggable>
              ))}
              {provided.placeholder}
            </div>
          )}
        </Droppable>
      </DragDropContext>
    </div>
  );
}
```

## 🎥 Video Content Management

### Video Upload and Processing

```typescript
// app/actions/lesson.ts
"use server";

import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";

export async function uploadLessonVideo(
  lessonId: string,
  file: File
): Promise<{ success: boolean; url?: string; error?: string }> {
  try {
    const session = await auth();
    if (!session) {
      return { success: false, error: "Unauthorized" };
    }

    // 1. Get pre-signed URL for upload
    const response = await fetch("/api/s3/upload", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        fileName: file.name,
        fileType: file.type,
        folder: `courses/lessons/${lessonId}`,
      }),
    });

    if (!response.ok) {
      throw new Error("Failed to get upload URL");
    }

    const { url, key } = await response.json();

    // 2. Upload the file to S3
    const uploadResponse = await fetch(url, {
      method: "PUT",
      body: file,
      headers: {
        "Content-Type": file.type,
      },
    });

    if (!uploadResponse.ok) {
      throw new Error("Upload failed");
    }

    // 3. Update lesson with video URL
    const videoUrl = `${process.env.NEXT_PUBLIC_S3_BASE_URL}/${key}`;
    
    await prisma.lesson.update({
      where: { id: lessonId },
      data: { videoUrl },
    });

    // 4. Trigger video processing (optional)
    await fetch(`/api/videos/process`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ videoKey: key, lessonId }),
    });

    return { success: true, url: videoUrl };
  } catch (error) {
    console.error("Error uploading video:", error);
    return { success: false, error: "Failed to upload video" };
  }
}
```

## 📊 Progress Tracking

### Track Lesson Completion

```typescript
// app/actions/progress.ts
"use server";

import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";

export async function markLessonAsCompleted(lessonId: string) {
  try {
    const session = await auth();
    if (!session?.user?.id) {
      throw new Error("Unauthorized");
    }

    // Check if already completed
    const existingProgress = await prisma.userProgress.findFirst({
      where: {
        userId: session.user.id,
        lessonId,
      },
    });

    if (existingProgress) {
      return { success: true };
    }

    // Mark as completed
    await prisma.userProgress.create({
      data: {
        userId: session.user.id,
        lessonId,
        isCompleted: true,
      },
    });

    return { success: true };
  } catch (error) {
    console.error("Error marking lesson as completed:", error);
    return { success: false, error: "Failed to update progress" };
  }
}

// Get course progress

export async function getCourseProgress(courseId: string) {
  const session = await auth();
  if (!session?.user?.id) {
    return { progress: 0, completedLessons: 0, totalLessons: 0 };
  }

  const [totalLessons, completedLessons] = await Promise.all([
    // Count total lessons in course
    prisma.lesson.count({
      where: {
        chapter: {
          courseId,
        },
      },
    }),
    
    // Count completed lessons for user
    prisma.userProgress.count({
      where: {
        userId: session.user.id,
        isCompleted: true,
        lesson: {
          chapter: {
            courseId,
          },
        },
      },
    }),
  ]);

  const progress = totalLessons > 0 
    ? Math.round((completedLessons / totalLessons) * 100) 
    : 0;

  return {
    progress,
    completedLessons,
    totalLessons,
  };
}
```

## 🚀 Best Practices

1. **Performance**
   - Implement pagination for courses with many lessons
   - Use server components for better performance
   - Optimize images and videos

2. **User Experience**
   - Show progress indicators for long-running operations
   - Provide clear error messages
   - Implement undo/redo for content editing

3. **Data Integrity**
   - Use database transactions for related operations
   - Implement proper error handling and rollbacks
   - Validate all user inputs

4. **Security**
   - Verify user permissions for all operations
   - Sanitize user-generated content
   - Protect against XSS and other web vulnerabilities

## 🔗 Related Documentation

- [Prisma Documentation](https://www.prisma.io/docs)
- [React DnD](https://react-dnd.github.io/react-dnd/about)
- [Next.js Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions)
