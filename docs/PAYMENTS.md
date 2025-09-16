# 💳 Payment Integration with Stripe

## Overview

Masterji integrates with Stripe to handle all payment processing, including one-time payments for course purchases and subscription management. This document outlines the payment flow, integration points, and implementation details.

## 🏗️ Architecture

### Components

1. **Frontend**
   - Checkout forms
   - Payment method management
   - Purchase history

2. **Backend**
   - Stripe webhook handlers
   - Payment processing
   - Subscription management

3. **Stripe**
   - Payment processing
   - Customer management
   - Subscription billing

## 🔄 Payment Flow

1. **Course Selection**
   - User selects a course to purchase
   - System checks if user is already enrolled
   - If not enrolled, shows pricing and payment options

2. **Checkout Process**
   - Collects payment method (card, etc.)
   - Validates payment details
   - Creates a payment intent

3. **Payment Processing**
   - Processes payment through Stripe
   - Handles 3D Secure authentication if required
   - Confirms successful payment

4. **Enrollment**
   - Creates enrollment record
   - Grants access to course content
   - Sends confirmation email

## 🛠️ Implementation

### 1. Stripe Setup

```typescript
// lib/stripe.ts
import Stripe from "stripe";

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2023-10-16",
  typescript: true,
});
```

### 2. Create Checkout Session

```typescript
// app/api/checkout/route.ts
import { NextResponse } from "next/server";
import { auth } from "@/lib/auth";
import { stripe } from "@/lib/stripe";

export async function POST(req: Request) {
  try {
    const session = await auth();
    if (!session?.user?.id) {
      return new NextResponse("Unauthorized", { status: 401 });
    }

    const { courseId } = await req.json();

    // Get course details
    const course = await prisma.course.findUnique({
      where: { id: courseId },
    });

    if (!course) {
      return new NextResponse("Course not found", { status: 404 });
    }

    // Check if already enrolled
    const existingEnrollment = await prisma.enrollment.findFirst({
      where: {
        userId: session.user.id,
        courseId,
      },
    });

    if (existingEnrollment) {
      return new NextResponse("Already enrolled", { status: 400 });
    }

    // Create checkout session
    const stripeSession = await stripe.checkout.sessions.create({
      payment_method_types: ["card"],
      mode: "payment",
      line_items: [
        {
          price_data: {
            currency: "usd",
            product_data: {
              name: course.title,
              description: course.description || undefined,
            },
            unit_amount: Math.round(course.price * 100), // Convert to cents
          },
          quantity: 1,
        },
      ],
      metadata: {
        userId: session.user.id,
        courseId,
      },
      success_url: `${process.env.NEXT_PUBLIC_APP_URL}/courses/${course.id}?success=1`,
      cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/courses/${course.id}?canceled=1`,
    });

    return NextResponse.json({ url: stripeSession.url });
  } catch (error) {
    console.error("[CHECKOUT_ERROR]", error);
    return new NextResponse("Internal Error", { status: 500 });
  }
}
```

### 3. Handle Webhook Events

```typescript
// app/api/webhook/stripe/route.ts
import { headers } from "next/headers";
import { NextResponse } from "next/server";
import { stripe } from "@/lib/stripe";
import { prisma } from "@/lib/prisma";

const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(req: Request) {
  try {
    const body = await req.text();
    const signature = headers().get("Stripe-Signature") as string;

    let event: Stripe.Event;

    try {
      event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
    } catch (error: any) {
      console.error(`Webhook Error: ${error.message}`);
      return new NextResponse(`Webhook Error: ${error.message}`, { status: 400 });
    }

    // Handle the event
    switch (event.type) {
      case "checkout.session.completed":
        await handleCheckoutSessionCompleted(event.data.object as Stripe.Checkout.Session);
        break;
      case "payment_intent.succeeded":
        // Handle successful payment
        break;
      case "payment_intent.payment_failed":
        // Handle failed payment
        break;
      // ... handle other event types
      default:
        console.log(`Unhandled event type ${event.type}`);
    }

    return new NextResponse(null, { status: 200 });
  } catch (error) {
    console.error("[WEBHOOK_ERROR]", error);
    return new NextResponse("Webhook error", { status: 400 });
  }
}

async function handleCheckoutSessionCompleted(session: Stripe.Checkout.Session) {
  const userId = session.metadata?.userId;
  const courseId = session.metadata?.courseId;

  if (!userId || !courseId) {
    throw new Error("Missing metadata in session");
  }

  // Create enrollment
  await prisma.enrollment.create({
    data: {
      userId,
      courseId,
      paymentId: session.payment_intent as string,
      amount: session.amount_total ? session.amount_total / 100 : 0,
    },
  });

  // TODO: Send confirmation email
}
```

## 🔒 Security Considerations

1. **Webhook Security**
   - Verify webhook signatures
   - Use webhook secrets
   - Handle events idempotently

2. **Data Protection**
   - Never store full payment details
   - Use Stripe Elements for secure card input
   - Comply with PCI DSS requirements

3. **Rate Limiting**
   - Implement rate limiting on API endpoints
   - Monitor for suspicious activity
   - Set up alerts for failed payment attempts

## 💰 Pricing and Plans

### Course Pricing
- One-time payment per course
- Discounts for bundled courses
- Promo codes support

### Subscription Model (Future)
- Monthly/Annual subscriptions
- Access to all courses
- Cancel anytime

## 📊 Reporting

### Sales Reports
- Daily/Monthly/Yearly sales
- Most popular courses
- Revenue by course/category

### User Analytics
- Conversion rates
- Average order value
- Refund rates

## 🚀 Best Practices

1. **Error Handling**
   - Handle all possible error cases
   - Provide clear error messages
   - Log all payment-related errors

2. **Testing**
   - Use Stripe test mode
   - Test all payment scenarios
   - Test webhook delivery

3. **Compliance**
   - Display terms and conditions
   - Handle refunds and disputes
   - Maintain proper records

## 🔗 Related Documentation

- [Stripe Documentation](https://stripe.com/docs)
- [Next.js API Routes](https://nextjs.org/docs/pages/building-your-application/routing/api-routes)
- [Stripe Webhooks](https://stripe.com/docs/webhooks)
