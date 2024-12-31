# Wasp Integration Patterns

## Authentication Patterns

### 1. Username/Password Authentication
```wasp
auth {
  userEntity: User,
  methods: {
    usernameAndPassword: {}
  },
  onAuthFailedRedirectTo: "/login"
}

entity User {=psl
  id          Int       @id @default(autoincrement())
  email       String    @unique
  password    String
  profile     Profile?
psl=}
```

### 2. Social Authentication
```wasp
auth {
  userEntity: User,
  methods: {
    google: {
      clientID: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET"
    },
    github: {
      clientID: "YOUR_GITHUB_CLIENT_ID",
      clientSecret: "YOUR_GITHUB_CLIENT_SECRET"
    }
  },
  onAuthSuccessRedirectTo: "/dashboard"
}
```

## Database Integration

### 1. Entity Relationships
```wasp
entity User {=psl
  id        Int       @id @default(autoincrement())
  profile   Profile?
  posts     Post[]
  comments  Comment[]
psl=}

entity Profile {=psl
  id        Int      @id @default(autoincrement())
  bio       String?
  user      User     @relation(fields: [userId], references: [id])
  userId    Int      @unique
psl=}

entity Post {=psl
  id        Int       @id @default(autoincrement())
  title     String
  content   String
  author    User      @relation(fields: [authorId], references: [id])
  authorId  Int
  comments  Comment[]
psl=}
```

### 2. Database Queries
```javascript
// src/server/queries.js
export const getUserProfile = async ({ userId }, context) => {
  return context.entities.User.findUnique({
    where: { id: userId },
    include: {
      profile: true,
      posts: {
        include: {
          comments: true
        }
      }
    }
  })
}
```

## External API Integration

### 1. Payment Processing (Stripe)
```wasp
app MyApp {
  dependencies: [
    ("stripe", "^11.0.0")
  ]
}

action createPayment {
  fn: import { createPayment } from "@server/payments.js",
  entities: [User, Payment]
}
```

```javascript
// src/server/payments.js
import Stripe from 'stripe'

export const createPayment = async ({ amount }, context) => {
  if (!context.user) throw new HttpError(401)
  
  const stripe = new Stripe(process.env.STRIPE_SECRET_KEY)
  
  const paymentIntent = await stripe.paymentIntents.create({
    amount,
    currency: 'usd',
    customer: context.user.stripeCustomerId
  })
  
  return context.entities.Payment.create({
    data: {
      userId: context.user.id,
      amount,
      stripePaymentId: paymentIntent.id
    }
  })
}
```

### 2. Email Service (SendGrid)
```wasp
app MyApp {
  dependencies: [
    ("@sendgrid/mail", "^7.7.0")
  ]
}

action sendEmail {
  fn: import { sendEmail } from "@server/email.js"
}
```

```javascript
// src/server/email.js
import sgMail from '@sendgrid/mail'

export const sendEmail = async ({ to, subject, text }, context) => {
  if (!context.user) throw new HttpError(401)
  
  sgMail.setApiKey(process.env.SENDGRID_API_KEY)
  
  return sgMail.send({
    to,
    from: 'your-verified-sender@example.com',
    subject,
    text
  })
}
```

## File Upload

### 1. S3 Integration
```wasp
app MyApp {
  dependencies: [
    ("@aws-sdk/client-s3", "^3.0.0"),
    ("multer", "^1.4.5-lts.1"),
    ("multer-s3", "^3.0.0")
  ]
}

action uploadFile {
  fn: import { uploadFile } from "@server/upload.js",
  entities: [File]
}
```

```javascript
// src/server/upload.js
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3'
import multer from 'multer'
import multerS3 from 'multer-s3'

const s3 = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  }
})

export const upload = multer({
  storage: multerS3({
    s3,
    bucket: process.env.S3_BUCKET,
    key: function (req, file, cb) {
      cb(null, Date.now().toString() + '-' + file.originalname)
    }
  })
})
```

## Real-time Features

### 1. WebSocket Integration
```wasp
app MyApp {
  dependencies: [
    ("socket.io", "^4.6.0"),
    ("socket.io-client", "^4.6.0")
  ]
}
```

```javascript
// src/server/websocket.js
import { Server } from 'socket.io'

export const setupWebSocket = (server) => {
  const io = new Server(server)
  
  io.on('connection', (socket) => {
    socket.on('join-room', (roomId) => {
      socket.join(roomId)
    })
    
    socket.on('message', (data) => {
      io.to(data.roomId).emit('message', data)
    })
  })
}
```

### 2. Chat Implementation
```wasp
entity Message {=psl
  id        Int      @id @default(autoincrement())
  content   String
  sender    User     @relation(fields: [senderId], references: [id])
  senderId  Int
  room      Room     @relation(fields: [roomId], references: [id])
  roomId    Int
  createdAt DateTime @default(now())
psl=}

entity Room {=psl
  id        Int       @id @default(autoincrement())
  name      String
  messages  Message[]
  users     User[]
psl=}
```

## Background Jobs

### 1. Task Queue
```wasp
job emailDigest {
  executor: EmailDigest,
  perform: {
    fn: import { sendEmailDigest } from "@server/jobs/emailDigest.js"
  },
  schedule: {
    cron: "0 8 * * *"  // Daily at 8 AM
  }
}
```

```javascript
// src/server/jobs/emailDigest.js
export const sendEmailDigest = async (args, context) => {
  const users = await context.entities.User.findMany({
    where: {
      emailDigestEnabled: true
    }
  })
  
  for (const user of users) {
    await sendDigestEmail(user)
  }
}
```

## Error Handling

### 1. Global Error Handler
```javascript
// src/server/middleware/errorHandler.js
export const errorHandler = (err, req, res, next) => {
  console.error(err.stack)
  
  if (err instanceof HttpError) {
    return res.status(err.statusCode).json({
      error: err.message
    })
  }
  
  res.status(500).json({
    error: 'Internal server error'
  })
}
```

### 2. Client Error Boundary
```jsx
// src/client/components/ErrorBoundary.jsx
import React from 'react'

export class ErrorBoundary extends React.Component {
  state = { hasError: false }

  static getDerivedStateFromError(error) {
    return { hasError: true }
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>
    }

    return this.props.children
  }
}
```

## Monitoring Integration

### 1. Sentry Setup
```wasp
app MyApp {
  dependencies: [
    ("@sentry/node", "^7.0.0"),
    ("@sentry/react", "^7.0.0")
  ]
}
```

```javascript
// src/server/monitoring.js
import * as Sentry from "@sentry/node"

export const setupMonitoring = () => {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 1.0
  })
}
```
