# ScholarAI General Use Cases

## UC-G01: User Authentication & Authorization

### Actors
- Researcher (end-user)
- System (Spring Boot Auth Service)
- Social Providers (Google, GitHub)

### Goal
Researcher securely signs up or logs in and gains access to ScholarAI features based on their role.

### Preconditions
- For Login: Researcher has already registered and confirmed email (if using email/password).
- For Social Auth: Researcher has an account with Google or GitHub.
- JWT keys are configured in backend.
- OAuth 2.0 credentials for Google and GitHub are configured in backend.

### Main Flow (Login - Email/Password)
1. Researcher navigates to Login page.
2. Researcher enters email & password and submits.
3. Frontend (Next.js) sends credentials to Spring Boot Auth endpoint.
4. Auth service validates credentials against user store.
5. On success, system issues a signed JWT + refresh token.
6. Frontend stores JWT (e.g. HttpOnly cookie) and updates UI to "logged in".
7. Researcher gains access to authorized UI routes.

### Main Flow (Signup - Email/Password)
1. Researcher navigates to Signup page.
2. Researcher enters email & password and submits.
3. Frontend (Next.js) sends credentials to Spring Boot Auth endpoint for registration.
4. Auth service creates a new user record (status: pending confirmation).
5. System sends a confirmation email to the researcher.
6. Researcher clicks the confirmation link in the email.
7. Auth service activates the user account.
8. Researcher can now log in using the new credentials.

### Main Flow (Login/Signup - Social Auth)
1. Researcher navigates to Login or Signup page.
2. Researcher clicks "Login with Google" or "Login with GitHub".
3. Frontend redirects to the respective Social Provider's authentication page.
4. Researcher authenticates with the Social Provider.
5. Social Provider redirects back to ScholarAI (callback URL) with an authorization code.
6. Frontend sends the authorization code to Spring Boot Auth endpoint.
7. Auth service exchanges the code for an access token from the Social Provider.
8. Auth service retrieves user profile information (email, name) from the Social Provider.
9. If user with this email exists, system links social account. If not, system creates a new user record.
10. System issues a signed JWT + refresh token.
11. Frontend stores JWT and updates UI to "logged in".
12. Researcher gains access to authorized UI routes.

### Alternative Flows
- **Invalid Credentials (Email/Password Login)**: System returns 401 → frontend shows "Invalid email or password".
- **Email Already Exists (Email/Password Signup)**: System returns 409 → frontend shows "Email already registered. Try logging in or use a different email."
- **Social Auth Failure**: Social Provider returns an error → frontend shows a generic error message.
- **Expired Refresh Token**: On token refresh attempt, system returns 403 → user is redirected to login.
- **Unconfirmed Email (Login Attempt)**: System returns 403 → frontend prompts to check email for confirmation link.

---

## UC-G02: User Profile Management

### Actors
- Authenticated User
- System (User Service)

### Goal
Manage personal information and settings

### Main Flow
1. User navigates to “Profile & Settings”
2. System displays:
   - Display name
   - Profile image
   - Email, password, linked accounts
   - Bio, theme (dark/light)
   - Notification preferences
3. User edits any field → system validates & updates backend
4. Changes are reflected immediately

### Postconditions
- User info is updated and stored
- UI reflects changes immediately

---

## UC-G03: Notification System

### Actors
- User
- Notification Service

### Goal
Deliver real-time and email alerts for key events

### Main Flow
1. System triggers event (e.g., paper scored, reminder due)
2. Notification Service:
   - Sends in-site alert (UI badge, toast)
   - Sends email (if enabled)
3. User clicks notification → redirected to relevant page

### Alternative Flows
- Email server failure → retry or fallback to in-app only

---

## UC-G04: Global Reading Progress Tracker

### Actors
- Authenticated User

### Goal
Track reading status of papers across all projects

### Main Flow
1. User opens “Global Dashboard”
2. System fetches all papers across all projects
3. Papers are shown with their status:
   - To Read
   - Reading
   - Done
4. User can update status or click to open paper

### Postconditions
- Global state reflects user's reading habits
- Status updates affect project-level dashboards too