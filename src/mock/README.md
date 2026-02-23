# mock

This project is a mock API for the system, designed to simulate backend functionality for frontend testing. It provides endpoints for user authentication, application management, AI audits, reviews, and more, adhering to the specified API documentation.

当前mock可登录账号存储位置在src\services\mockDb.js
（账号有student1、student2、teacher、admin，密码都是“password”）

## Project Structure

```
mock
├── src
│   ├── app.js                # Entry point of the application
│   ├── routes                # Contains all route definitions
│   │   ├── auth.js           # Authentication routes
│   │   ├── users.js          # User-related routes
│   │   ├── applications.js    # Application management routes
│   │   ├── aiAudits.js       # AI audit routes
│   │   ├── reviews.js        # Review management routes
│   │   ├── teacher.js        # Teacher-related routes
│   │   ├── archives.js       # Archive management routes
│   │   ├── files.js          # File management routes
│   │   ├── notifications.js   # Notification management routes
│   │   ├── system.js         # System configuration routes
│   │   └── announcements.js   # Announcement management routes
│   ├── middleware             # Middleware for authentication and error handling
│   │   ├── auth.js           # Authentication middleware
│   │   └── errorHandler.js    # Error handling middleware
│   ├── services               # Services for token management and mock database
│   │   ├── tokenService.js    # Token management functions
│   │   └── mockDb.js         # Simulated database
│   └── utils                 # Utility functions and constants
│       ├── constants.js      # Constants used throughout the application
│       └── response.js       # Utility functions for API responses
├── package.json              # NPM configuration file
└── README.md                 # Project documentation
```

## Setup Instructions

1. **Install dependencies**:
   ```
   npm install
   ```

2. **Run the application**:
   ```
   npm start
   ```

3. **Access the API**:
   The API will be available at `http://localhost:8080/api/v1`.

## API Usage

The mock API provides various endpoints for testing. Below are some examples:

- **Login**: `POST /auth/login`
- **Get Current User**: `GET /users/me`
- **Create Application**: `POST /applications`
- **Fetch AI Audit Report**: `GET /ai-audits/{application_id}/report`
- **Submit Review Decision**: `POST /reviews/{application_id}/decision`

Refer to the individual route files for detailed endpoint specifications and expected request/response formats.
