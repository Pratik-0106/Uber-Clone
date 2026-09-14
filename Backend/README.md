# User Registration Endpoint Documentation

## Endpoint: `/users/register`

### Description

This endpoint allows users to register a new account in the application. It validates the input data, hashes the password using bcrypt, creates a new user in the database, and returns an authentication token.

---

## HTTP Method

**POST**

---

## Request Body

The request must be sent as JSON with the following structure:

```json
{
  "fullname": {
    "firstname": "string (required, min 3 characters)",
    "lastname": "string (optional, min 3 characters if provided)"
  },
  "email": "string (required, valid email format)",
  "password": "string (required, min 6 characters)"
}
```

### Request Parameters

| Field                | Type   | Required | Validation Rules                        |
| -------------------- | ------ | -------- | --------------------------------------- |
| `fullname.firstname` | String | Yes      | Minimum 3 characters long               |
| `fullname.lastname`  | String | No       | Minimum 3 characters long (if provided) |
| `email`              | String | Yes      | Must be a valid email format            |
| `password`           | String | Yes      | Minimum 6 characters long               |

---

## Example Request

```bash
curl -X POST http://localhost:5000/users/register \
  -H "Content-Type: application/json" \
  -d '{
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "password": "password123"
  }'
```

---

## Response Status Codes

| Status Code | Description                                                                 |
| ----------- | --------------------------------------------------------------------------- |
| **201**     | User successfully registered. Returns user object and authentication token. |
| **400**     | Bad Request. Validation error(s) in the request body.                       |

---

## Response Examples

### Success Response (Status 201)

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439011",
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "password": "$2b$10$hashedpasswordstring...",
    "socketId": null
  }
}
```

### Error Response (Status 400)

```json
{
  "errors": [
    {
      "type": "field",
      "value": "invalid-email",
      "msg": "Invalid Email",
      "path": "email",
      "location": "body"
    },
    {
      "type": "field",
      "value": "ab",
      "msg": "First name must be at least 3 characters long",
      "path": "fullname.firstname",
      "location": "body"
    }
  ]
}
```

---

## Input Validation Rules

The endpoint performs the following validations:

1. **Email Validation**: Must be a valid email format
   - Example valid: `user@example.com`
   - Example invalid: `invalid-email`

2. **First Name Validation**: Must be at least 3 characters long
   - Example valid: `John`
   - Example invalid: `Jo`

3. **Password Validation**: Must be at least 6 characters long
   - Example valid: `securePass123`
   - Example invalid: `pass`

If any validation fails, the endpoint returns a **400 Bad Request** status with an array of error objects detailing which fields failed validation.

---

## Notes

- Passwords are hashed using bcrypt with a salt round of 10 before being stored in the database.
- The authentication token is generated using JWT and includes the user's MongoDB `_id`.
- The `socketId` field is optional and used for WebSocket connections.

---

# User Login Endpoint Documentation

## Endpoint: `/users/login`

### Description

This endpoint allows users to log in to their account using their email and password. It validates the input data, verifies the user's credentials against the database, and returns an authentication token upon successful authentication.

---

## HTTP Method

**POST**

---

## Request Body

The request must be sent as JSON with the following structure:

```json
{
  "email": "string (required, valid email format)",
  "password": "string (required, min 6 characters)"
}
```

### Request Parameters

| Field      | Type   | Required | Validation Rules             |
| ---------- | ------ | -------- | ---------------------------- |
| `email`    | String | Yes      | Must be a valid email format |
| `password` | String | Yes      | Minimum 6 characters long    |

---

## Example Request

```bash
curl -X POST http://localhost:5000/users/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "password123"
  }'
```

---

## Response Status Codes

| Status Code | Description                                                                    |
| ----------- | ------------------------------------------------------------------------------ |
| **200**     | User successfully authenticated. Returns user object and authentication token. |
| **400**     | Bad Request. Validation error(s) in the request body.                          |
| **401**     | Unauthorized. Invalid email or password.                                       |

---

## Response Examples

### Success Response (Status 200)

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439011",
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "socketId": null
  }
}
```

### Validation Error Response (Status 400)

```json
{
  "errors": [
    {
      "type": "field",
      "value": "invalid-email",
      "msg": "Invalid Email",
      "path": "email",
      "location": "body"
    },
    {
      "type": "field",
      "value": "pass",
      "msg": "Password must be at least 6 character",
      "path": "password",
      "location": "body"
    }
  ]
}
```

### Authentication Error Response (Status 401)

```json
{
  "message": "Invalid email or password"
}
```

---

## Input Validation Rules

The endpoint performs the following validations:

1. **Email Validation**: Must be a valid email format
   - Example valid: `user@example.com`
   - Example invalid: `invalid-email`

2. **Password Validation**: Must be at least 6 characters long
   - Example valid: `securePass123`
   - Example invalid: `pass`

If validation fails, the endpoint returns a **400 Bad Request** status with an array of error objects detailing which fields failed validation.

---

## Authentication Process

1. The endpoint receives the email and password from the request body.
2. It queries the database to find a user with the provided email.
3. If no user is found, it returns a **401 Unauthorized** status with the message "Invalid email or password".
4. If a user is found, it compares the provided password with the stored hashed password using bcrypt.
5. If the passwords match, an authentication token is generated and returned along with the user object.
6. If the passwords do not match, it returns a **401 Unauthorized** status with the message "Invalid email or password".

---

## Notes

- The password field in the user object is excluded from the response for security reasons.
- The authentication token is generated using JWT and includes the user's MongoDB `_id`.
- Failed login attempts return a generic "Invalid email or password" message to prevent user enumeration attacks.
