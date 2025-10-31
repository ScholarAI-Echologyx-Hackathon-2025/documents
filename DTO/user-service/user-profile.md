# User Service Profile DTOs

This document contains the Data Transfer Objects (DTOs) used for user profile operations in the user service.

## Overview

The user profile system uses the following DTOs:
- **Request DTOs**: Used for incoming requests (CreateProfileDTO, UpdateProfileDTO)
- **Response DTOs**: Used for outgoing responses (ProfileResponse, ProfileListResponse)

## Request DTOs

## CreateProfileDTO

Used for creating a new user profile.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CreateProfileDTO {
    @NotBlank(message = "Full name is required")
    @Size(max = 255, message = "Full name cannot exceed 255 characters")
    private String fullName;

    @Size(max = 255, message = "Avatar URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for avatar")
    private String avatarUrl;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Please provide a valid phone number")
    private String phoneNumber;

    @Past(message = "Date of birth must be in the past")
    private Instant dateOfBirth;

    @Size(max = 1000, message = "Bio cannot exceed 1000 characters")
    private String bio;

    @Size(max = 255, message = "Affiliation cannot exceed 255 characters")
    private String affiliation;

    @Size(max = 255, message = "Position title cannot exceed 255 characters")
    private String positionTitle;

    @Size(max = 2000, message = "Research interests cannot exceed 2000 characters")
    private String researchInterests;

    @Size(max = 255, message = "Google Scholar URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for Google Scholar")
    private String googleScholarUrl;

    @Size(max = 255, message = "Personal website URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for personal website")
    private String personalWebsiteUrl;

    @Pattern(regexp = "^\\d{4}-\\d{4}-\\d{4}-\\d{4}$", message = "Please provide a valid ORCID ID")
    private String orcidId;

    @Size(max = 255, message = "LinkedIn URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for LinkedIn")
    private String linkedInUrl;

    @Size(max = 255, message = "Twitter URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for Twitter")
    private String twitterUrl;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `fullName` | String | Yes | `@NotBlank`, `@Size(max = 255)` | User's full name |
| `avatarUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's avatar image |
| `phoneNumber` | String | No | `@Pattern(regexp = "^\\+?[1-9]\\d{1,14}$")` | User's phone number in international format |
| `dateOfBirth` | Instant | No | `@Past` | User's date of birth (must be in the past) |
| `bio` | String | No | `@Size(max = 1000)` | User's biographical information |
| `affiliation` | String | No | `@Size(max = 255)` | User's institutional or organizational affiliation |
| `positionTitle` | String | No | `@Size(max = 255)` | User's job title or position |
| `researchInterests` | String | No | `@Size(max = 2000)` | User's research interests and areas of expertise |
| `googleScholarUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's Google Scholar profile |
| `personalWebsiteUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's personal website |
| `orcidId` | String | No | `@Pattern(regexp = "^\\d{4}-\\d{4}-\\d{4}-\\d{4}$")` | User's ORCID identifier |
| `linkedInUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's LinkedIn profile |
| `twitterUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's Twitter profile |

### Validation Rules

- **Full Name**: Must be provided and cannot exceed 255 characters
- **Avatar URL**: Must be a valid URL if provided
- **Phone Number**: Must follow international phone number format if provided
- **Date of Birth**: Must be a date in the past if provided
- **Bio**: Cannot exceed 1000 characters if provided
- **Affiliation**: Cannot exceed 255 characters if provided
- **Position Title**: Cannot exceed 255 characters if provided
- **Research Interests**: Cannot exceed 2000 characters if provided
- **URLs**: Must be valid URLs if provided (Google Scholar, Personal Website, LinkedIn, Twitter)
- **ORCID ID**: Must follow the standard ORCID format (XXXX-XXXX-XXXX-XXXX) if provided

## UpdateProfileDTO

Used for updating an existing user profile.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UpdateProfileDTO {
    @Size(max = 255, message = "Full name cannot exceed 255 characters")
    private String fullName;

    @Size(max = 255, message = "Avatar URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for avatar")
    private String avatarUrl;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Please provide a valid phone number")
    private String phoneNumber;

    @Past(message = "Date of birth must be in the past")
    private Instant dateOfBirth;

    @Size(max = 1000, message = "Bio cannot exceed 1000 characters")
    private String bio;

    @Size(max = 255, message = "Affiliation cannot exceed 255 characters")
    private String affiliation;

    @Size(max = 255, message = "Position title cannot exceed 255 characters")
    private String positionTitle;

    @Size(max = 2000, message = "Research interests cannot exceed 2000 characters")
    private String researchInterests;

    @Size(max = 255, message = "Google Scholar URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for Google Scholar")
    private String googleScholarUrl;

    @Size(max = 255, message = "Personal website URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for personal website")
    private String personalWebsiteUrl;

    @Pattern(regexp = "^\\d{4}-\\d{4}-\\d{4}-\\d{4}$", message = "Please provide a valid ORCID ID")
    private String orcidId;

    @Size(max = 255, message = "LinkedIn URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for LinkedIn")
    private String linkedInUrl;

    @Size(max = 255, message = "Twitter URL cannot exceed 255 characters")
    @URL(message = "Please provide a valid URL for Twitter")
    private String twitterUrl;
}
```

### Fields

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| `fullName` | String | No | `@Size(max = 255)` | User's full name |
| `avatarUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's avatar image |
| `phoneNumber` | String | No | `@Pattern(regexp = "^\\+?[1-9]\\d{1,14}$")` | User's phone number in international format |
| `dateOfBirth` | Instant | No | `@Past` | User's date of birth (must be in the past) |
| `bio` | String | No | `@Size(max = 1000)` | User's biographical information |
| `affiliation` | String | No | `@Size(max = 255)` | User's institutional or organizational affiliation |
| `positionTitle` | String | No | `@Size(max = 255)` | User's job title or position |
| `researchInterests` | String | No | `@Size(max = 2000)` | User's research interests and areas of expertise |
| `googleScholarUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's Google Scholar profile |
| `personalWebsiteUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's personal website |
| `orcidId` | String | No | `@Pattern(regexp = "^\\d{4}-\\d{4}-\\d{4}-\\d{4}$")` | User's ORCID identifier |
| `linkedInUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's LinkedIn profile |
| `twitterUrl` | String | No | `@Size(max = 255)`, `@URL` | URL to user's Twitter profile |

### Validation Rules

Same as CreateProfileDTO, but all fields are optional for updates.

## Response DTOs

## ProfileResponse

Used for returning user profile information.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ProfileResponse {
    private UUID id;
    private UUID userId;
    private String fullName;
    private String avatarUrl;
    private String phoneNumber;
    private Instant dateOfBirth;
    private String bio;
    private String affiliation;
    private String positionTitle;
    private String researchInterests;
    private String googleScholarUrl;
    private String personalWebsiteUrl;
    private String orcidId;
    private String linkedInUrl;
    private String twitterUrl;
    private Instant createdAt;
    private Instant updatedAt;
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier of the profile |
| `userId` | UUID | Unique identifier of the user this profile belongs to |
| `fullName` | String | User's full name |
| `avatarUrl` | String | URL to user's avatar image |
| `phoneNumber` | String | User's phone number |
| `dateOfBirth` | Instant | User's date of birth |
| `bio` | String | User's biographical information |
| `affiliation` | String | User's institutional or organizational affiliation |
| `positionTitle` | String | User's job title or position |
| `researchInterests` | String | User's research interests and areas of expertise |
| `googleScholarUrl` | String | URL to user's Google Scholar profile |
| `personalWebsiteUrl` | String | URL to user's personal website |
| `orcidId` | String | User's ORCID identifier |
| `linkedInUrl` | String | URL to user's LinkedIn profile |
| `twitterUrl` | String | URL to user's Twitter profile |
| `createdAt` | Instant | Timestamp when the profile was created |
| `updatedAt` | Instant | Timestamp when the profile was last updated |

## ProfileListResponse

Used for returning a list of user profiles with pagination.

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ProfileListResponse {
    private List<ProfileResponse> profiles;
    private int totalElements;
    private int totalPages;
    private int currentPage;
    private int pageSize;
    private boolean hasNext;
    private boolean hasPrevious;
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `profiles` | List<ProfileResponse> | List of user profiles |
| `totalElements` | int | Total number of profiles available |
| `totalPages` | int | Total number of pages |
| `currentPage` | int | Current page number (0-based) |
| `pageSize` | int | Number of profiles per page |
| `hasNext` | boolean | Whether there is a next page |
| `hasPrevious` | boolean | Whether there is a previous page |

## Request-Response Mapping

| Endpoint | Request DTO | Response DTO | Description |
|----------|-------------|--------------|-------------|
| `POST /profiles` | `CreateProfileDTO` | `ProfileResponse` | Create a new user profile |
| `GET /profiles/{id}` | - | `ProfileResponse` | Get a specific user profile |
| `GET /profiles/user/{userId}` | - | `ProfileResponse` | Get profile by user ID |
| `PUT /profiles/{id}` | `UpdateProfileDTO` | `ProfileResponse` | Update an existing user profile |
| `PATCH /profiles/{id}` | `UpdateProfileDTO` | `ProfileResponse` | Partially update a user profile |
| `DELETE /profiles/{id}` | - | - | Delete a user profile |
| `GET /profiles` | - | `ProfileListResponse` | Get paginated list of profiles |

## Usage Examples

### Create Profile Flow

**Request** (POST /profiles):
```json
{
  "fullName": "Dr. Jane Smith",
  "avatarUrl": "https://example.com/avatar.jpg",
  "phoneNumber": "+1234567890",
  "dateOfBirth": "1985-03-15T00:00:00Z",
  "bio": "Research scientist specializing in machine learning and artificial intelligence.",
  "affiliation": "Stanford University",
  "positionTitle": "Associate Professor",
  "researchInterests": "Machine Learning, Deep Learning, Computer Vision, Natural Language Processing",
  "googleScholarUrl": "https://scholar.google.com/citations?user=example",
  "personalWebsiteUrl": "https://janesmith.com",
  "orcidId": "0000-0001-2345-6789",
  "linkedInUrl": "https://linkedin.com/in/janesmith",
  "twitterUrl": "https://twitter.com/janesmith"
}
```

**Response** (201 Created):
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "userId": "123e4567-e89b-12d3-a456-426614174001",
  "fullName": "Dr. Jane Smith",
  "avatarUrl": "https://example.com/avatar.jpg",
  "phoneNumber": "+1234567890",
  "dateOfBirth": "1985-03-15T00:00:00Z",
  "bio": "Research scientist specializing in machine learning and artificial intelligence.",
  "affiliation": "Stanford University",
  "positionTitle": "Associate Professor",
  "researchInterests": "Machine Learning, Deep Learning, Computer Vision, Natural Language Processing",
  "googleScholarUrl": "https://scholar.google.com/citations?user=example",
  "personalWebsiteUrl": "https://janesmith.com",
  "orcidId": "0000-0001-2345-6789",
  "linkedInUrl": "https://linkedin.com/in/janesmith",
  "twitterUrl": "https://twitter.com/janesmith",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}
```

### Update Profile Flow

**Request** (PUT /profiles/123e4567-e89b-12d3-a456-426614174000):
```json
{
  "fullName": "Dr. Jane Smith",
  "bio": "Updated bio with new research focus on quantum computing.",
  "researchInterests": "Machine Learning, Deep Learning, Computer Vision, Natural Language Processing, Quantum Computing"
}
```

**Response** (200 OK):
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "userId": "123e4567-e89b-12d3-a456-426614174001",
  "fullName": "Dr. Jane Smith",
  "avatarUrl": "https://example.com/avatar.jpg",
  "phoneNumber": "+1234567890",
  "dateOfBirth": "1985-03-15T00:00:00Z",
  "bio": "Updated bio with new research focus on quantum computing.",
  "affiliation": "Stanford University",
  "positionTitle": "Associate Professor",
  "researchInterests": "Machine Learning, Deep Learning, Computer Vision, Natural Language Processing, Quantum Computing",
  "googleScholarUrl": "https://scholar.google.com/citations?user=example",
  "personalWebsiteUrl": "https://janesmith.com",
  "orcidId": "0000-0001-2345-6789",
  "linkedInUrl": "https://linkedin.com/in/janesmith",
  "twitterUrl": "https://twitter.com/janesmith",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T11:45:00Z"
}
```

### Get Profile List Flow

**Request** (GET /profiles?page=0&size=10):
```json
// No request body
```

**Response** (200 OK):
```json
{
  "profiles": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "userId": "123e4567-e89b-12d3-a456-426614174001",
      "fullName": "Dr. Jane Smith",
      "avatarUrl": "https://example.com/avatar.jpg",
      "phoneNumber": "+1234567890",
      "bio": "Research scientist specializing in machine learning and artificial intelligence.",
      "affiliation": "Stanford University",
      "positionTitle": "Associate Professor",
      "researchInterests": "Machine Learning, Deep Learning, Computer Vision, Natural Language Processing",
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-01-15T10:30:00Z"
    }
  ],
  "totalElements": 1,
  "totalPages": 1,
  "currentPage": 0,
  "pageSize": 10,
  "hasNext": false,
  "hasPrevious": false
}
```

## Dependencies

These DTOs use the following annotations from the Jakarta Validation API:
- `@NotBlank`: Ensures the field is not null, empty, or contains only whitespace
- `@Size`: Validates the size of the field (maximum length for strings)
- `@URL`: Validates that the field contains a valid URL
- `@Pattern`: Validates the field against a regular expression pattern
- `@Past`: Ensures the date is in the past

And Lombok annotations for boilerplate code reduction:
- `@Data`: Generates getters, setters, toString, equals, and hashCode methods
- `@AllArgsConstructor`: Generates a constructor with all fields
- `@NoArgsConstructor`: Generates a no-argument constructor
