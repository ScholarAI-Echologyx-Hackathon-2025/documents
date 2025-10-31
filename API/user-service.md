# ScholarAI User Service API Documentation

## Overview

The ScholarAI User Service provides authentication and user profile management functionality. This API supports traditional email/password authentication as well as social authentication via Google and GitHub OAuth.

**Base URL:** `http://localhost:8081`  
**API Version:** `v1`  
**Content Type:** `application/json`

## Authentication

The API uses JWT (JSON Web Tokens) for authentication. Protected endpoints require a valid JWT token in the Authorization header:

```
Authorization: Bearer <your-access-token>
```

### Token Management

- **Access Token:** Valid for 15 minutes, used for API authentication
- **Refresh Token:** Valid for 7 days, stored in HttpOnly cookie, used to refresh access tokens

## Response Format

All API responses follow a standard format:

```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Success message",
  "data": {
    // Response data here
  }
}
```

## Error Response Format

```json
{
  "timestamp": "2024-01-01T12:00:00",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Error description",
  "details": ["Field validation errors"],
  "suggestion": "How to fix the error"
}
```

---

## Authentication Endpoints

### 1. User Registration

**Endpoint:** `POST /api/v1/auth/register`

**Description:** Register a new user account with email and password.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "role": "USER"
}
```

**Request Validation:**
- `email`: Required, valid email format
- `password`: Required, minimum 8 characters
- `role`: Optional, defaults to "USER" (values: USER, ADMIN)

**Response (201 - Created):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 201,
  "message": "User registered successfully",
  "data": null
}
```

**Response (400 - Bad Request):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 400,
  "message": "Registration failed: Email already exists",
  "data": null
}
```

### 2. User Login

**Endpoint:** `POST /api/v1/auth/login`

**Description:** Authenticate user and get JWT tokens.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "your-password"
}
```

**Request Validation:**
- `email`: Required, valid email format
- `password`: Required, minimum 8 characters

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Login successful",
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "stored-in-httponly-cookie",
    "tokenType": "Bearer",
    "expiresIn": 900
  }
}
```

**Response (401 - Unauthorized):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 401,
  "message": "Invalid email or password",
  "data": null
}
```

### 3. Refresh Access Token

**Endpoint:** `POST /api/v1/auth/refresh`

**Description:** Refresh the access token using the refresh token stored in cookies.

**Headers:** No additional headers required (refresh token is in HttpOnly cookie)

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Token refreshed successfully",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tokenType": "Bearer",
    "expiresIn": 900
  }
}
```

**Response (401 - Unauthorized):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 401,
  "message": "Invalid refresh token",
  "data": null
}
```

### 4. Logout

**Endpoint:** `POST /api/v1/auth/logout`

**Description:** Logout user and invalidate tokens.

**Headers:** 
```
Authorization: Bearer <access-token>
```

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Logged out successfully",
  "data": null
}
```

### 5. Forgot Password

**Endpoint:** `POST /api/v1/auth/forgot-password`

**Description:** Generate a password reset code for the given email.

**Query Parameters:**
- `email`: Email address for password reset

**Example:** `POST /api/v1/auth/forgot-password?email=user@example.com`

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Reset code generated successfully. Code: 123456 (This will be sent via notification service later)",
  "data": "123456"
}
```

**Response (400 - Bad Request):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 400,
  "message": "User not found with email: user@example.com",
  "data": null
}
```

### 6. Reset Password

**Endpoint:** `POST /api/v1/auth/reset-password`

**Description:** Reset user password using the reset code and new password.

**Query Parameters:**
- `email`: Email address
- `code`: Reset code received via email
- `newPassword`: New password

**Example:** `POST /api/v1/auth/reset-password?email=user@example.com&code=123456&newPassword=newSecurePassword123`

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Password reset successfully.",
  "data": null
}
```

**Response (400 - Bad Request):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 400,
  "message": "Invalid reset code or code expired",
  "data": null
}
```

---

## Social Authentication Endpoints

### 1. Google OAuth Login

**Endpoint:** `POST /api/v1/auth/social/google-login`

**Description:** Authenticate user using Google OAuth ID token.

**Request Body:**
```json
{
  "idToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjEyMzQ1Njc4OTAiLCJ0eXAiOiJKV1QifQ..."
}
```

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Google login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tokenType": "Bearer",
    "expiresIn": 900
  }
}
```

**Response (400 - Bad Request):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 400,
  "message": "ID token is missing.",
  "data": null
}
```

**Response (401 - Unauthorized):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 401,
  "message": "Invalid Google ID token: Token validation failed",
  "data": null
}
```

### 2. GitHub OAuth Login

**Endpoint:** `POST /api/v1/auth/social/github-login`

**Description:** Authenticate user using GitHub OAuth authorization code.

**Request Body:**
```json
{
  "code": "abc123def456ghi789"
}
```

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "GitHub login successful",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tokenType": "Bearer",
    "expiresIn": 900
  }
}
```

**Response (400 - Bad Request):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 400,
  "message": "GitHub authorization code is missing",
  "data": null
}
```

**Response (401 - Unauthorized):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 401,
  "message": "GitHub login failed: Invalid authorization code",
  "data": null
}
```

---

## User Profile Endpoints

### 1. Get User Profile

**Endpoint:** `GET /api/v1/profile`

**Description:** Retrieve the current user's profile information.

**Headers:** 
```
Authorization: Bearer <access-token>
```

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Profile fetched successfully",
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "userId": "123e4567-e89b-12d3-a456-426614174001",
    "fullName": "John Doe",
    "bio": "Software Engineer passionate about AI",
    "avatarUrl": "https://example.com/avatar.jpg",
    "dateOfBirth": "1990-01-01T00:00:00Z",
    "affiliation": "Tech Company",
    "positionTitle": "Senior Engineer",
    "researchInterests": "AI, Machine Learning",
    "phoneNumber": "+1-555-123-4567",
    "googleScholarUrl": "https://scholar.google.com/citations?user=example",
    "personalWebsiteUrl": "https://johndoe.com",
    "orcidId": "0000-0000-0000-0000",
    "linkedInUrl": "https://linkedin.com/in/johndoe",
    "twitterUrl": "https://twitter.com/johndoe",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

**Response (401 - Unauthorized):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 401,
  "message": "Unauthorized",
  "data": null
}
```

**Response (404 - Not Found):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 404,
  "message": "Profile not found for user",
  "data": null
}
```

### 2. Update User Profile

**Endpoint:** `PATCH /api/v1/profile`

**Description:** Update the current user's profile information.

**Headers:** 
```
Authorization: Bearer <access-token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "fullName": "John Smith",
  "bio": "Updated bio information",
  "dateOfBirth": "1990-01-01T00:00:00Z",
  "affiliation": "New Company",
  "positionTitle": "Lead Engineer",
  "researchInterests": "AI, ML, Deep Learning",
  "phoneNumber": "+1-555-987-6543",
  "googleScholarUrl": "https://scholar.google.com/citations?user=newuser",
  "personalWebsiteUrl": "https://johnsmith.com",
  "orcidId": "0000-0000-0000-0001",
  "linkedInUrl": "https://linkedin.com/in/johnsmith",
  "twitterUrl": "https://twitter.com/johnsmith"
}
```

**Field Validation Rules:**
- `fullName`: Max 255 characters
- `avatarUrl`: Max 500 characters, valid HTTP/HTTPS URL
- `phoneNumber`: Max 20 characters, digits, spaces, hyphens, parentheses, optional plus sign
- `bio`: Max 1000 characters
- `affiliation`: Max 255 characters
- `positionTitle`: Max 255 characters
- `researchInterests`: Max 1000 characters
- `googleScholarUrl`: Max 500 characters, valid HTTP/HTTPS URL
- `personalWebsiteUrl`: Max 500 characters, valid HTTP/HTTPS URL
- `orcidId`: Max 50 characters, format XXXX-XXXX-XXXX-XXXX
- `linkedInUrl`: Max 500 characters, valid HTTP/HTTPS URL
- `twitterUrl`: Max 500 characters, valid HTTP/HTTPS URL

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Profile updated successfully",
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "userId": "123e4567-e89b-12d3-a456-426614174001",
    "fullName": "John Smith",
    "bio": "Updated bio information",
    "avatarUrl": "https://example.com/avatar.jpg",
    "dateOfBirth": "1990-01-01T00:00:00Z",
    "affiliation": "New Company",
    "positionTitle": "Lead Engineer",
    "researchInterests": "AI, ML, Deep Learning",
    "phoneNumber": "+1-555-987-6543",
    "googleScholarUrl": "https://scholar.google.com/citations?user=newuser",
    "personalWebsiteUrl": "https://johnsmith.com",
    "orcidId": "0000-0000-0000-0001",
    "linkedInUrl": "https://linkedin.com/in/johnsmith",
    "twitterUrl": "https://twitter.com/johnsmith",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-02T00:00:00Z"
  }
}
```

**Response (400 - Bad Request):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 400,
  "message": "Invalid date format for dateOfBirth",
  "data": null
}
```

### 3. Upload Avatar

**Endpoint:** `POST /api/v1/profile/avatar`

**Description:** Upload a new avatar image for the current user.

**Headers:** 
```
Authorization: Bearer <access-token>
Content-Type: multipart/form-data
```

**Request Body:**
- `avatar`: Image file (JPG, PNG, GIF, max 5MB)

**File Requirements:**
- Supported formats: JPG, PNG, GIF
- Maximum size: 5MB
- Recommended dimensions: 200x200 pixels or larger

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Avatar uploaded successfully",
  "data": {
    "avatarUrl": "https://placeholder.com/avatar/123e4567-e89b-12d3-a456-426614174000.jpg"
  }
}
```

**Response (400 - Bad Request):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 400,
  "message": "File is empty or invalid format",
  "data": null
}
```

### 4. Delete Avatar

**Endpoint:** `DELETE /api/v1/profile/avatar`

**Description:** Remove the current user's avatar image.

**Headers:** 
```
Authorization: Bearer <access-token>
```

**Response (200 - OK):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 200,
  "message": "Avatar deleted successfully",
  "data": null
}
```

**Response (400 - Bad Request):**
```json
{
  "timestamp": "01-01-2024 12:00:00",
  "status": 400,
  "message": "Failed to delete avatar",
  "data": null
}
```

---

## Data Models

### UserRole Enum
```json
{
  "USER": "Regular user",
  "ADMIN": "Administrator"
}
```

### AuthResponse
```json
{
  "accessToken": "string",
  "refreshToken": "string",
  "email": "string",
  "userId": "uuid",
  "role": "UserRole"
}
```

### UserProfileDTO (Update Request)
```json
{
  "fullName": "string (max 255 chars)",
  "avatarUrl": "string (max 500 chars, valid URL)",
  "phoneNumber": "string (max 20 chars)",
  "dateOfBirth": "ISO 8601 timestamp",
  "bio": "string (max 1000 chars)",
  "affiliation": "string (max 255 chars)",
  "positionTitle": "string (max 255 chars)",
  "researchInterests": "string (max 1000 chars)",
  "googleScholarUrl": "string (max 500 chars, valid URL)",
  "personalWebsiteUrl": "string (max 500 chars, valid URL)",
  "orcidId": "string (format: XXXX-XXXX-XXXX-XXXX)",
  "linkedInUrl": "string (max 500 chars, valid URL)",
  "twitterUrl": "string (max 500 chars, valid URL)"
}
```

### UserProfileResponseDTO (Response)
```json
{
  "id": "uuid",
  "userId": "uuid",
  "fullName": "string",
  "avatarUrl": "string",
  "phoneNumber": "string",
  "dateOfBirth": "ISO 8601 timestamp",
  "bio": "string",
  "affiliation": "string",
  "positionTitle": "string",
  "researchInterests": "string",
  "googleScholarUrl": "string",
  "personalWebsiteUrl": "string",
  "orcidId": "string",
  "linkedInUrl": "string",
  "twitterUrl": "string",
  "createdAt": "ISO 8601 timestamp",
  "updatedAt": "ISO 8601 timestamp"
}
```

---

## Frontend Integration Examples

### JavaScript/TypeScript Examples

#### 1. User Registration
```javascript
const registerUser = async (userData) => {
  try {
    const response = await fetch('http://localhost:8081/api/v1/auth/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email: userData.email,
        password: userData.password,
        role: 'USER'
      })
    });
    
    const result = await response.json();
    return result;
  } catch (error) {
    console.error('Registration failed:', error);
    throw error;
  }
};
```

#### 2. User Login
```javascript
const loginUser = async (credentials) => {
  try {
    const response = await fetch('http://localhost:8081/api/v1/auth/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email: credentials.email,
        password: credentials.password
      }),
      credentials: 'include' // Important for cookies
    });
    
    const result = await response.json();
    
    if (result.success) {
      // Store access token
      localStorage.setItem('accessToken', result.data.accessToken);
    }
    
    return result;
  } catch (error) {
    console.error('Login failed:', error);
    throw error;
  }
};
```

#### 3. Get User Profile
```javascript
const getUserProfile = async () => {
  try {
    const accessToken = localStorage.getItem('accessToken');
    const response = await fetch('http://localhost:8081/api/v1/profile', {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json',
      }
    });
    
    const result = await response.json();
    return result;
  } catch (error) {
    console.error('Failed to get profile:', error);
    throw error;
  }
};
```

#### 4. Update User Profile
```javascript
const updateUserProfile = async (profileData) => {
  try {
    const accessToken = localStorage.getItem('accessToken');
    const response = await fetch('http://localhost:8081/api/v1/profile', {
      method: 'PATCH',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(profileData)
    });
    
    const result = await response.json();
    return result;
  } catch (error) {
    console.error('Failed to update profile:', error);
    throw error;
  }
};
```

#### 5. Upload Avatar
```javascript
const uploadAvatar = async (file) => {
  try {
    const accessToken = localStorage.getItem('accessToken');
    const formData = new FormData();
    formData.append('avatar', file);
    
    const response = await fetch('http://localhost:8081/api/v1/profile/avatar', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
      },
      body: formData
    });
    
    const result = await response.json();
    return result;
  } catch (error) {
    console.error('Failed to upload avatar:', error);
    throw error;
  }
};
```

#### 6. Refresh Token
```javascript
const refreshToken = async () => {
  try {
    const response = await fetch('http://localhost:8081/api/v1/auth/refresh', {
      method: 'POST',
      credentials: 'include' // Important for cookies
    });
    
    const result = await response.json();
    
    if (result.success) {
      // Update stored access token
      localStorage.setItem('accessToken', result.data.accessToken);
    }
    
    return result;
  } catch (error) {
    console.error('Token refresh failed:', error);
    throw error;
  }
};
```

#### 7. Logout
```javascript
const logout = async () => {
  try {
    const accessToken = localStorage.getItem('accessToken');
    const response = await fetch('http://localhost:8081/api/v1/auth/logout', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json',
      },
      credentials: 'include'
    });
    
    // Clear local storage
    localStorage.removeItem('accessToken');
    
    const result = await response.json();
    return result;
  } catch (error) {
    console.error('Logout failed:', error);
    throw error;
  }
};
```

### React Hook Example

```javascript
import { useState, useEffect } from 'react';

const useAuth = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const login = async (credentials) => {
    try {
      setLoading(true);
      const result = await loginUser(credentials);
      if (result.success) {
        setUser(result.data);
      }
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  const logout = async () => {
    try {
      await logout();
      setUser(null);
    } catch (err) {
      setError(err.message);
    }
  };

  const refreshToken = async () => {
    try {
      const result = await refreshToken();
      if (result.success) {
        // Token refreshed successfully
        return true;
      }
      return false;
    } catch (err) {
      // Token refresh failed, redirect to login
      setUser(null);
      return false;
    }
  };

  return {
    user,
    loading,
    error,
    login,
    logout,
    refreshToken
  };
};
```

---

## Error Handling

### Common HTTP Status Codes

- **200 OK**: Request successful
- **201 Created**: Resource created successfully
- **400 Bad Request**: Invalid request data or validation failed
- **401 Unauthorized**: Invalid or missing authentication token
- **404 Not Found**: Resource not found
- **500 Internal Server Error**: Server error

### Error Handling Best Practices

1. **Token Expiration**: Implement automatic token refresh when receiving 401 errors
2. **Validation Errors**: Display field-specific error messages to users
3. **Network Errors**: Implement retry logic for network failures
4. **Rate Limiting**: Handle 429 status codes with exponential backoff

### Example Error Handler

```javascript
const handleApiError = (error, response) => {
  if (response?.status === 401) {
    // Try to refresh token
    const refreshResult = await refreshToken();
    if (!refreshResult) {
      // Redirect to login
      window.location.href = '/login';
    }
  } else if (response?.status === 400) {
    // Handle validation errors
    const errorData = await response.json();
    return errorData.details || errorData.message;
  } else {
    // Handle other errors
    console.error('API Error:', error);
    return 'An unexpected error occurred';
  }
};
```

---

## Security Considerations

1. **HTTPS**: Always use HTTPS in production
2. **Token Storage**: Store access tokens in memory or secure storage, never in localStorage for production
3. **Token Expiration**: Implement automatic token refresh
4. **CSRF Protection**: The API uses HttpOnly cookies for refresh tokens
5. **Input Validation**: Always validate user input on both client and server
6. **Rate Limiting**: Implement rate limiting for authentication endpoints

---

## Testing

### Swagger UI

Access the interactive API documentation at:
`http://localhost:8081/swagger-ui.html`

### Postman Collection

You can import the API endpoints into Postman for testing:

1. Base URL: `http://localhost:8081`
2. Authorization: Bearer Token
3. Headers: `Content-Type: application/json`

### Environment Variables

For local development, ensure these environment variables are set:

```bash
# Database
USER_DB_PORT=5432
USER_DB_USER=your_db_user
USER_DB_PASSWORD=your_db_password

# JWT
JWT_SECRET=your_jwt_secret_here

# OAuth (Optional)
SPRING_GOOGLE_CLIENT_ID=your_google_client_id
SPRING_GOOGLE_CLIENT_SECRET=your_google_client_secret
SPRING_GITHUB_CLIENT_ID=your_github_client_id
SPRING_GITHUB_CLIENT_SECRET=your_github_client_secret
```

---

## Support

For API support and questions, please refer to:
- API Documentation: `http://localhost:8081/swagger-ui.html`
- Health Check: `http://localhost:8081/actuator/health`
- Application Info: `http://localhost:8081/actuator/info` 