# Notification Service Schema Documentation

This document contains schema definitions related to the notification-service, including PostgreSQL table schemas, corresponding JPA entity classes, and notification type enumerations.

## Database Tables

### notifications

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    type VARCHAR(100) NOT NULL,
    title VARCHAR(255) NOT NULL,
    message TEXT,
    action_url TEXT,
    is_read BOOLEAN DEFAULT FALSE,
    is_archived BOOLEAN DEFAULT FALSE,
    payload TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    read_at TIMESTAMP
);
```

## JPA Entity Classes

### Notification Entity

```java
import jakarta.persistence.*;
import java.time.Instant;
import java.util.UUID;

@Entity
@Table(name = "notifications")
public class Notification {
    @Id
    @GeneratedValue
    private UUID id;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private NotificationType type; // Enum-based type

    @Column(nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT")
    private String message;

    private String actionUrl;

    private boolean isRead = false;

    private boolean isArchived = false;

    @Column(columnDefinition = "TEXT") // Use jsonb if PostgreSQL
    private String payload; // JSON string for variable notification data

    private Instant createdAt;

    private Instant readAt;

    // Getters and setters
}
```

### NotificationType Enum

```java
public enum NotificationType {
    ACCOUNT_CREATED,
    ACCOUNT_DELETED,
    PAPER_RETRIEVAL_COMPLETED,
    GAP_ANALYSIS_COMPLETED,
    COMMENT_MENTION,
    SYSTEM_ALERT
}
```
