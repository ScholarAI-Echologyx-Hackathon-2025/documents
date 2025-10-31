# User Service Authentication DTOs

This document contains the Data Transfer Objects (DTOs) used for authentication operations in the user service.

## Overview

The authentication system uses the following DTOs:
- **Request DTOs**: Used for incoming requests (LoginDTO, SignupDTO)
- **Response DTOs**: Used for outgoing responses (AuthResponse)

## Request DTOs

## LoginDTO

Used for user login requests.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class LoginDTO {
    @NotBlank(message = "Email is required")
    @Email(message = "Please provide a valid email address")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters long")
    private String password;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `email` | String | Yes | `@NotBlank`, `@Email` | User's email address for authentication |
| `password` | String | Yes | `@NotBlank`, `@Size(min = 8)` | User's password (minimum 8 characters) |

### Validation Rules

- **Email**: Must be a valid email format and cannot be blank
- **Password**: Must be at least 8 characters long and cannot be blank

## SignupDTO

Used for user registration requests.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class SignupDTO {
    @NotBlank
    @Email
    private String email;

    @NotBlank
    @Size(min = 8)
    private String password;

    @NotNull
    private UserRole role;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `email` | String | Yes | `@NotBlank`, `@Email` | User's email address for registration |
| `password` | String | Yes | `@NotBlank`, `@Size(min = 8)` | User's password (minimum 8 characters) |
| `role` | UserRole | Yes | `@NotNull` | User's role (USER or ADMIN) |

### Validation Rules

- **Email**: Must be a valid email format and cannot be blank
- **Password**: Must be at least 8 characters long and cannot be blank
- **Role**: Must be a valid UserRole enum value and cannot be null

### UserRole Values

- `USER`: Standard user role
- `ADMIN`: Administrator role with elevated privileges

## Response DTOs

## AuthResponse

Used for successful authentication responses (login and signup).

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class AuthResponse {
    private String accessToken;
    private String refreshToken;
    private String email;
    private UUID userId;
    private UserRole role; 
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `accessToken` | String | JWT access token for API authentication |
| `refreshToken` | String | JWT refresh token for obtaining new access tokens |
| `email` | String | User's email address |
| `userId` | UUID | Unique identifier of the authenticated user |
| `role` | UserRole | User's role (USER or ADMIN) |

### Usage

This response is returned for:
- **Successful login** (POST /auth/login)
- **Successful signup** (POST /auth/signup)

## Request-Response Mapping

| Endpoint | Request DTO | Response DTO | Description |
|----------|-------------|--------------|-------------|
| `POST /auth/login` | `LoginDTO` | `AuthResponse` | Authenticate existing user |
| `POST /auth/signup` | `SignupDTO` | `AuthResponse` | Register new user |

## Usage Examples

### Login Flow

**Request** (POST /auth/login):
```json
{
  "email": "user@example.com",
  "password": "securepassword123"
}
```

**Response** (200 OK):
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "email": "user@example.com",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "role": "USER"
}
```

### Signup Flow

**Request** (POST /auth/signup):
```json
{
  "email": "newuser@example.com",
  "password": "securepassword123",
  "role": "USER"
}
```

**Response** (201 Created):
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "email": "newuser@example.com",
  "userId": "123e4567-e89b-12d3-a456-426614174001",
  "role": "USER"
}
```

## Dependencies

These DTOs use the following annotations from the Jakarta Validation API:
- `@NotBlank`: Ensures the field is not null, empty, or contains only whitespace
- `@Email`: Validates that the field contains a valid email address
- `@Size`: Validates the size of the field (minimum length for strings)
- `@NotNull`: Ensures the field is not null

And Lombok annotations for boilerplate code reduction:
- `@Data`: Generates getters, setters, toString, equals, and hashCode methods
- `@AllArgsConstructor`: Generates a constructor with all fields
- `@NoArgsConstructor`: Generates a no-argument constructor
