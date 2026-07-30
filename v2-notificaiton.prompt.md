# Notification Feature Specification

## Project

- **Project:** agentswipe
- **Module:** Notification

## User Story

Users can receive, view, read, and delete notifications for system and application events.

## Flow

- Global Header → Notification Center
- **Prerequisite:** Authenticated user

---

# Features

## Notification List

- Display notifications sorted by **newest first**.
- Show unread indicator (`isRead`).
- Clicking a notification:

  - Marks it as read.
  - Redirects using `type` + `data`.

- Support:

  - Mark All as Read
  - Delete Notification
  - Delete All Notifications

- Empty state:

  - **"You have no new notifications."**

---

# GraphQL Schema

## Types

```graphql
type Notification {
  id: ID!
  userId: ID!
  type: NotificationType!
  title: String!
  message: String!
  isRead: Boolean!
  data: JSON
  createdAt: DateTime!
  updatedAt: DateTime!
}

enum NotificationType {
  WELCOME
  PASSWORD_CHANGED
  PASSWORD_RESET
  PROFILE_UPDATED
  NEW_FEATURE
  NEW_DEVICE_LOGIN
  MESSAGE_RECEIVED
  COMMENT_REPLY
  MENTION
  LIKE
  USER_REPORT
  NEW_USER_SIGNUP
  TASK_REMINDER
  PUSH_OPT_IN_REMINDER
  CONNECTION_REQUEST_RECEIVED
  CONNECTION_ACCEPTED
  MUTUAL_MATCH
  ADDED_TO_FAVOURITE
  LICENSE_VERIFIED
  LICENSE_REJECTED
  SUBSCRIPTION_ACTIVATED
  SUBSCRIPTION_EXPIRY_REMINDER
  SUBSCRIPTION_EXPIRED
  USAGE_LIMIT_REACHED
  SYSTEM_NOTIFICATION
}
```

## Inputs

```graphql
input MarkNotificationReadInput {
  id: ID!
}

input DeleteNotificationInput {
  id: ID!
}
```

## Queries

```graphql
type Query {
  getNotifications(limit: Int = 20, offset: Int = 0): [Notification!]!
  getUnreadNotificationCount: Int!
}
```

## Mutations

```graphql
type Mutation {
  markNotificationAsRead(input: MarkNotificationReadInput!): Notification!
  markAllNotificationsAsRead: Boolean!
  deleteNotification(input: DeleteNotificationInput!): Boolean!
  deleteAllNotifications: Boolean!
}
```

---

# Validation

- `id` is required for read/delete operations.
- `id` must be a valid MongoDB ObjectId.

---

# Business Rules

## Authorization

- Authenticated users only.
- Users can access and modify only their own notifications.

## Notification Behavior

- Always sort by `createdAt DESC`.
- Read state is tracked via `isRead`.
- Delete operations are **hard deletes**.
- Client determines redirect destination using `type` and `data`.

## Processing

- Notification creation is asynchronous using **AWS SQS**.
- After processing:

  - Deliver real-time events via **Server-Sent Events (SSE)**.
  - Trigger **Firebase Cloud Messaging (FCM)** push notifications (initialization pending).

## Localization

- Reuse existing localization keys from `common.json` or `user.json` for success and error messages whenever available.
