# StreamRepo Backend API Documentation

This document provides a comprehensive overview of the StreamRepo backend API, including setup instructions, usage examples, and endpoint specifications.  The backend is designed to manage user accounts, authentication (including OAuth2 with GitHub), email verification, password resets, user profile updates, and a waiting list. It utilizes Spring Boot, MongoDB, and other technologies.

## Overview

The StreamRepo backend API provides a RESTful interface for various functionalities related to user management and authentication. Key features include:

* **User Registration and Authentication:** Secure user registration and login using standard credentials or OAuth 2.0 with GitHub.
* **Email Verification:**  A magic link system for verifying user email addresses.
* **Password Management:** Secure password storage using BCrypt and a password reset functionality via email.
* **User Profile Updates:** Allows users to update their profile information (name, bio, picture).
* **Waiting List:** Manages a waiting list for users.
* **Admin Functionality:**  Provides an admin endpoint (requires authentication).

The backend uses MongoDB for persistent data storage.

## Setup

1. **Prerequisites:** Ensure you have Java 17+, Maven, and MongoDB installed.
2. **Clone the repository:**  Clone the project from the Git repository.
3. **Configure Cloudinary:** Obtain Cloudinary credentials and configure them in `application.properties` or `application.yml`.  The `cloudinary` prefix is used for configuration properties.
4. **Configure Email:** Configure your email SMTP server settings in `application.properties` (replace placeholders with your credentials).
5. **Configure Frontend URL:** Set the `frontend.url` property in `application.properties` to point to your frontend application's URL. This is crucial for generating correct redirect URLs for magic links and password resets.
6. **Build and Run:** Run `mvn clean package` to build the project, then run `java -jar target/backend-*.jar` to start the application.

## Usage

The API uses standard HTTP methods (GET, POST, PUT) to interact with different endpoints.  Authentication is handled via JWT for standard login and OAuth2 for GitHub login.

## Endpoints

### Admin Endpoints

| Method | Path             | Summary       | Parameters     | Response          | JS Example                                                                    | Dart (Flutter) Example                                                              |
|--------|-----------------|----------------|-----------------|-------------------|---------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| POST   | `/api/admin/write-docs` | Write Docs    | None            | `{"message": "Welcome, mighty Admin!"}` | `fetch('/api/admin/write-docs', { method: 'POST', headers: { 'Authorization': 'Bearer <your_token>' } })` | `final response = await http.post(Uri.parse('/api/admin/write-docs'), headers: {'Authorization': 'Bearer <your_token>'});` |


### Authentication Endpoints

| Method | Path                | Summary                     | Parameters               | Response                               | JS Example                                                                                                 | Dart (Flutter) Example                                                                                                    |
|--------|---------------------|------------------------------|---------------------------|-------------------------------------------|-------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| POST   | `/api/auth/register` | Register User                | `UserDTO`                  | `ResponseDetails`                           | `fetch('/api/auth/register', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({...}) })` | `final response = await http.post(Uri.parse('/api/auth/register'), headers: {'Content-Type': 'application/json'}, body: jsonEncode({...}));` |
| POST   | `/api/auth/login`    | Login User                   | `UserDTO`                  | `LoginResponse`                            | `fetch('/api/auth/login', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({...}) })` | `final response = await http.post(Uri.parse('/api/auth/login'), headers: {'Content-Type': 'application/json'}, body: jsonEncode({...}));` |
| POST   | `/api/auth/magic-link` | Request Magic Link           | `MagicLinkRequest`         | `ResponseDetails`                           | `fetch('/api/auth/magic-link', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({...}) })` | `final response = await http.post(Uri.parse('/api/auth/magic-link'), headers: {'Content-Type': 'application/json'}, body: jsonEncode({...}));` |
| GET    | `/api/auth/validate-magic-link` | Validate Magic Link         | `link` (query parameter) | `ResponseDetails`                           | `fetch('/api/auth/validate-magic-link?link=<your_link>')`                                                    | `final response = await http.get(Uri.parse('/api/auth/validate-magic-link?link=<your_link>'));`                                    |
| GET    | `/api/auth/oauth2/github` | OAuth2 GitHub Login          | None                       | Redirect to frontend                      |  (handled automatically by OAuth2 library)                                                             | (handled automatically by OAuth2 library)                                                                     |
| POST   | `/api/auth/forgot-password` | Request Password Reset       | `MagicLinkRequest`         | `ResponseDetails`                           | `fetch('/api/auth/forgot-password', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({...}) })` | `final response = await http.post(Uri.parse('/api/auth/forgot-password'), headers: {'Content-Type': 'application/json'}, body: jsonEncode({...}));` |
| POST   | `/api/auth/reset-password` | Reset Password              | `PasswordResetRequest`    | `ResponseDetails`                           | `fetch('/api/auth/reset-password', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({...}) })` | `final response = await http.post(Uri.parse('/api/auth/reset-password'), headers: {'Content-Type': 'application/json'}, body: jsonEncode({...}));` |


### User Endpoints

| Method | Path             | Summary           | Parameters     | Response          | JS Example                                                                    | Dart (Flutter) Example                                                              |
|--------|-----------------|--------------------|-----------------|-------------------|---------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| PUT    | `/api/user/update` | Update User Profile | `UserDTO`        | `ResponseDetails` | `fetch('/api/user/update', { method: 'PUT', headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer <your_token>' }, body: JSON.stringify({...}) })` | `final response = await http.put(Uri.parse('/api/user/update'), headers: {'Content-Type': 'application/json', 'Authorization': 'Bearer <your_token>'}, body: jsonEncode({...}));` |


### Waiting List Endpoints

| Method | Path                | Summary                 | Parameters           | Response          | JS Example                                                                                             | Dart (Flutter) Example                                                                                                |
|--------|---------------------|--------------------------|-----------------------|-------------------|----------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| POST   | `/waiting-list/user` | Add User to Waiting List | `WaitingListDTO`     | `ResponseDetails` | `fetch('/waiting-list/user', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({...}) })` | `final response = await http.post(Uri.parse('/waiting-list/user'), headers: {'Content-Type': 'application/json'}, body: jsonEncode({...}));` |


## Error Handling

The API returns standard HTTP status codes to indicate success or failure.  Error responses include a `ResponseDetails` object with a timestamp, message, status, and path.  See the `GlobalExceptionHandler` for details on specific exception handling.


## UML Sequence Diagram (PlantUML)

```plantuml
@startuml
actor Client
participant API Gateway
participant Auth Service
participant User Service
participant Database

Client -> API Gateway: Request (e.g., /api/auth/login)
activate API Gateway
API Gateway -> Auth Service: Authenticate User
activate Auth Service
Auth Service -> Database: Check Credentials
activate Database
Database --> Auth Service: Credentials Valid/Invalid
deactivate Database
Auth Service --> API Gateway: Authentication Result
deactivate Auth Service
API Gateway -> Client: Response (Success/Failure)
deactivate API Gateway

Client -> API Gateway: Request (e.g., /api/user/update)
activate API Gateway
API Gateway -> Auth Service: Authenticate User
activate Auth Service
Auth Service -> Database: Check Token
activate Database
Database --> Auth Service: Token Valid/Invalid
deactivate Database
Auth Service --> API Gateway: Authentication Result
deactivate Auth Service
API Gateway -> User Service: Update User
activate User Service
User Service -> Database: