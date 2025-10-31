# User Service Schema Documentation

This document contains schema definitions related to the user-service, including PostgreSQL table schemas and corresponding JPA entity classes.

## Enums

### UserRole Enum

```java
public enum UserRole {
    USER,
    ADMIN
}
```

## Database Tables

### users

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    encrypted_password VARCHAR(255) NOT NULL,
    role VARCHAR(10) NOT NULL CHECK (role IN ('USER', 'ADMIN')),
    is_email_confirmed BOOLEAN DEFAULT FALSE,
    last_login_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### user_profiles

```sql
CREATE TABLE user_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
    
    -- Basic info
    full_name VARCHAR(255),
    avatar_url TEXT,
    phone_number VARCHAR(20),
    date_of_birth TIMESTAMP,
    bio TEXT,
    
    -- Academic / Professional Info
    affiliation VARCHAR(255),
    position_title VARCHAR(255),
    research_interests TEXT,
    google_scholar_url TEXT,
    personal_website_url TEXT,
    orcid_id VARCHAR(50),
    linkedin_url TEXT,
    twitter_url TEXT,
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### user_identity_providers

```sql
CREATE TABLE user_identity_providers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    provider VARCHAR(50) NOT NULL,
    provider_user_id VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(provider, provider_user_id)
);
```

## JPA Entity Classes

### User Entity

```java
@Getter
@Setter
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    @Column(name = "id", columnDefinition = "uuid")
    private UUID id;

    @Column(name = "email", nullable = false, unique = true)
    private String email;

    @Column(name = "encrypted_password", nullable = false)
    private String encryptedPassword;

    @Enumerated(EnumType.STRING)
    @Column(name = "role", nullable = false)
    private UserRole role;

    @Column(name = "is_email_confirmed")
    private boolean isEmailConfirmed;

    @Column(name = "last_login_at")
    private Instant lastLoginAt;

    @CreationTimestamp
    @Column(name = "created_at")
    private Instant createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at")
    private Instant updatedAt;

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    private UserProfile profile;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<UserIdentityProvider> identityProviders;
}
```

### UserProfile Entity

```java
@Getter
@Setter
@Entity
@Table(name = "user_profiles")
public class UserProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    @Column(name = "id")
    private UUID id;

    @OneToOne
    @JoinColumn(name = "user_id", nullable = false, unique = true)
    private User user;

    @Column(name = "full_name")
    private String fullName;

    @Column(name = "avatar_url")
    private String avatarUrl;

    @Column(name = "phone_number")
    private String phoneNumber;

    @Column(name = "date_of_birth")
    private Instant dateOfBirth;

    @Column(name = "bio")
    private String bio;

    @Column(name = "affiliation")
    private String affiliation;

    @Column(name = "position_title")
    private String positionTitle;

    @Column(name = "research_interests")
    private String researchInterests;

    @Column(name = "google_scholar_url")
    private String googleScholarUrl;

    @Column(name = "personal_website_url")
    private String personalWebsiteUrl;

    @Column(name = "orcid_id")
    private String orcidId;

    @Column(name = "linkedin_url")
    private String linkedInUrl;

    @Column(name = "twitter_url")
    private String twitterUrl;

    @CreationTimestamp
    @Column(name = "created_at")
    private Instant createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at")
    private Instant updatedAt;
}
```

### UserIdentityProvider Entity

```java
@Getter
@Setter
@Entity
@Table(name = "user_identity_providers")
public class UserIdentityProvider {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    @Column(name = "id")
    private UUID id;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Column(name = "provider")
    private String provider;

    @Column(name = "provider_user_id")
    private String providerUserId;

    @CreationTimestamp
    @Column(name = "created_at")
    private Instant createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at")
    private Instant updatedAt;
} 