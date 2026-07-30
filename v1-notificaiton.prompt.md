# Notification Feature Specification

- **Project:** agentswipe
- **Module:** `Notification`

## User Story
As a **User**, I want to **receive, view, and manage my notifications**, so that **I can stay informed about important events, connections, and system updates**.

## Context
The Notification system provides real-time updates and alerts for users across the platform, including system-wide events and core application events like matches and connections.
- Fits into step/flow: `Global Application Header / Notification Center Screen`
- Pre-requisites: `User must be authenticated for most notifications (except some global pushes)`

## Acceptance Criteria
1. **Notification List Screen/Dropdown:**
   - Display a list of notifications sorted by newest first.
   - Display an `isRead` flag or visual indicator for unread notifications.
   - Clicking on a notification marks it as read and redirects the user to the appropriate screen based on the notification type.
   - Provide a "Mark all as read" button to instantly mark all unread notifications as read.
   - Provide a "Delete" button for each individual notification (hard delete).
   - Provide a "Delete all" button to clear all notifications (hard delete).
   - If the list is empty, show an empty state illustration with text "You have no new notifications."

## GraphQL API Specification

### Response Payload (Object Type)
Define the output schema that the client will receive.
```graphql
type Notification {
  id: ID!
  userId: ID!
  type: NotificationType!
  title: String!
  message: String!
  isRead: Boolean!
  data: JSON # Additional metadata for redirect links, like entity IDs
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

### Input Payloads (Input Types)
Notifications are generally system-generated. The client only needs inputs to modify their state.
```graphql
input MarkNotificationReadInput {
  id: ID!
}

input DeleteNotificationInput {
  id: ID!
}
```

### Queries
List the queries to fetch data related to this feature.
```graphql
type Query {
  # Get all notifications for the authenticated user (sorted newest first)
  getNotifications(limit: Int = 20, offset: Int = 0): [Notification!]!
  
  # Get unread notification count
  getUnreadNotificationCount: Int!
}
```

### Mutations
List the mutations for updating and deleting data.
```graphql
type Mutation {
  # Mark a specific notification as read
  markNotificationAsRead(input: MarkNotificationReadInput!): Notification!
  
  # Mark all notifications as read for the user
  markAllNotificationsAsRead: Boolean!
  
  # Delete a specific notification (Hard Delete)
  deleteNotification(input: DeleteNotificationInput!): Boolean!
  
  # Delete all notifications for the user (Hard Delete)
  deleteAllNotifications: Boolean!
}
```

## Business & Validation Rules

1. **Validation Rules for Inputs:**
   - **`id`:** Required for `markNotificationAsRead` and `deleteNotification`. Must be a valid MongoDB ObjectId.

2. **Business Rules:**
   - **Permissions/State Check:** Only authenticated users can fetch, update, or delete their own notifications.
   - **Ownership/Relationships:** A notification must belong to the user attempting to modify or fetch it.
   - **Sorting:** `getNotifications` MUST always sort by `createdAt` in descending order (newest first).
   - **Deletions:** Deletion must be a **hard delete** (record removed from the database), as specified in the requirements.
   - **Redirection Logic:** The frontend will use the `type` and `data` (metadata payload) to determine where to redirect the user when a notification is clicked.
   - **Asynchronous Processing (AWS SQS):** Notification creation and processing should be handled asynchronously using AWS SQS.
   - **Real-Time Delivery (SSE):** After processing the notification from SQS, use Server-Sent Events (SSE) to deliver real-time notifications to active clients.
   - **Push Notifications (FCM):** Alongside SSE, trigger Firebase Cloud Messaging (FCM) push notifications. *(Note: FCM initialization is pending and will be completed later, but the triggers must be implemented in the logic).*
   - **Return Messages:** Reuse existing localization keys from `user.json` or `common.json` (e.g., "Notification deleted successfully", "All notifications marked as read") where applicable.
