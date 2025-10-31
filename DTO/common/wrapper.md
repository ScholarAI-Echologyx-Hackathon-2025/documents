# Common API Response Wrappers

This document contains the standard response wrapper classes used across all services in the ScholarAI platform to ensure consistent API response formatting.

## Overview

The platform uses two main wrapper classes:
- **APIResponse<T>**: For successful responses with generic data
- **APIErrorResponse**: For error responses with detailed error information

## APIResponse<T>

Generic wrapper for successful API responses that can contain any type of data.

```java
@Data
@Builder
public class APIResponse<T> {
    @Builder.Default
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy HH:mm:ss")
    private LocalDateTime timestamp = LocalDateTime.now();

    private int status;
    private String message;
    private T data;
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | LocalDateTime | Response timestamp (auto-generated, format: dd-MM-yyyy HH:mm:ss) |
| `status` | int | HTTP status code (e.g., 200, 201, 400, 404, 500) |
| `message` | String | Human-readable response message |
| `data` | T | Generic data payload (can be any type) |



### Usage Examples

#### Successful Login Response
```java
AuthResponse authData = new AuthResponse(/* ... */);
APIResponse<AuthResponse> response = APIResponse.builder()
    .status(200)
    .message("Login successful")
    .data(authData)
    .build();
```

**JSON Output:**
```json
{
  "timestamp": "15-12-2024 14:30:25",
  "status": 200,
  "message": "Login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "email": "user@example.com",
    "userId": "123e4567-e89b-12d3-a456-426614174000",
    "role": "USER"
  }
}
```

#### User Profile Response
```java
UserProfile profile = userProfileService.getProfile(userId);
APIResponse<UserProfile> response = APIResponse.builder()
    .status(200)
    .message("User profile retrieved successfully")
    .data(profile)
    .build();
```

**JSON Output:**
```json
{
  "timestamp": "15-12-2024 14:30:25",
  "status": 200,
  "message": "User profile retrieved successfully",
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "fullName": "John Doe",
    "email": "john.doe@example.com",
    "affiliation": "University of Technology",
    "researchInterests": "Machine Learning, AI"
  }
}
```

## APIErrorResponse

Standardized error response structure for consistent error handling across the platform.

```java
/**
 * Standard error response structure for the API.
 */
@Data
@Builder
public class APIErrorResponse {
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ss")
    private LocalDateTime timestamp;

    private int status;
    private String code;
    private String message;
    private List<String> details;
    private String suggestion;
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | LocalDateTime | Error timestamp (format: yyyy-MM-dd'T'HH:mm:ss) |
| `status` | int | HTTP status code (e.g., 400, 401, 404, 500) |
| `code` | String | Error code for programmatic handling |
| `message` | String | Human-readable error message |
| `details` | List<String> | List of detailed error messages |
| `suggestion` | String | Suggested action to resolve the error |

### Usage Examples

#### Validation Error
```java
APIErrorResponse error = APIErrorResponse.builder()
    .timestamp(LocalDateTime.now())
    .status(400)
    .code("VALIDATION_ERROR")
    .message("Invalid input data")
    .details(Arrays.asList(
        "Email must be a valid email address",
        "Password must be at least 8 characters long"
    ))
    .suggestion("Please check your input and try again")
    .build();
```

**JSON Output:**
```json
{
  "timestamp": "2024-12-15T14:30:25",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Invalid input data",
  "details": [
    "Email must be a valid email address",
    "Password must be at least 8 characters long"
  ],
  "suggestion": "Please check your input and try again"
}
```

#### Authentication Error
```java
APIErrorResponse error = APIErrorResponse.builder()
    .timestamp(LocalDateTime.now())
    .status(401)
    .code("UNAUTHORIZED")
    .message("Invalid credentials")
    .details(Arrays.asList("Email or password is incorrect"))
    .suggestion("Please verify your credentials and try again")
    .build();
```

**JSON Output:**
```json
{
  "timestamp": "2024-12-15T14:30:25",
  "status": 401,
  "code": "UNAUTHORIZED",
  "message": "Invalid credentials",
  "details": [
    "Email or password is incorrect"
  ],
  "suggestion": "Please verify your credentials and try again"
}
```

#### Resource Not Found
```java
APIErrorResponse error = APIErrorResponse.builder()
    .timestamp(LocalDateTime.now())
    .status(404)
    .code("RESOURCE_NOT_FOUND")
    .message("User not found")
    .details(Arrays.asList("User with ID 123e4567-e89b-12d3-a456-426614174000 does not exist"))
    .suggestion("Please check the user ID and try again")
    .build();
```

## Best Practices

### For APIResponse<T>

1. **Use appropriate HTTP status codes**:
   - 200: OK (GET, PUT, PATCH)
   - 201: Created (POST)
   - 204: No Content (DELETE)

2. **Provide meaningful messages**:
   - Be specific about what operation succeeded
   - Use consistent language across similar operations

3. **Handle null data gracefully**:
   - For empty results, use empty collections rather than null
   - Consider using Optional<T> for nullable data

### For APIErrorResponse

1. **Use standard error codes**:
   - `VALIDATION_ERROR`: Input validation failures
   - `UNAUTHORIZED`: Authentication required
   - `FORBIDDEN`: Insufficient permissions
   - `RESOURCE_NOT_FOUND`: Requested resource doesn't exist
   - `INTERNAL_SERVER_ERROR`: Unexpected server errors

2. **Provide actionable suggestions**:
   - Give users clear guidance on how to fix the issue
   - Include relevant documentation links when appropriate

3. **Include relevant details**:
   - List specific validation errors
   - Include field names for validation issues
   - Provide context when helpful

## Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Input validation failed |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `RESOURCE_NOT_FOUND` | 404 | Requested resource not found |
| `CONFLICT` | 409 | Resource conflict (e.g., duplicate email) |
| `INTERNAL_SERVER_ERROR` | 500 | Unexpected server error |
| `SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable |

## Dependencies

These wrapper classes use:
- **Lombok**: `@Data`, `@Builder` for boilerplate code reduction
- **Jackson**: `@JsonFormat` for custom timestamp formatting
- **Java Time**: `LocalDateTime` for timestamp handling
