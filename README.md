# 🎓 Campus Food Waste & Smart Dining Management System

A beginner-friendly, high-performance, and modular RESTful backend built with **Node.js, Express.js, MongoDB (Mongoose), and JWT Authentication**.

The system connects **students** with **campus cafeterias** in real time to:
- 📊 **Track Cafeteria Crowd Levels & Wait Times:** Helps students avoid peak rushes.
- 📋 **Check Menu Availability:** Live inventory tracking with auto-toggle of item availability.
- 🎟️ **Reserve Meals & Pickup Slots:** Prevents overselling, automatically locks inventory, and restores stock on cancellation.
- 🔥 **Flash Surplus Food Alerts:** Notifies students of surplus meals at discounted prices before closing to prevent campus food waste.
- 🌿 **Track Sustainability & Waste Metrics:** Calculates rescued meals, waste diversion rate (%), student financial savings, and CO2 emissions prevented.

---

## 🛠️ Tech Stack

- **Runtime:** Node.js (v18+)
- **Framework:** Express.js (v5)
- **Database:** MongoDB with Mongoose (with seamless zero-config in-memory fallback)
- **Authentication:** JSON Web Tokens (JWT) + bcryptjs password hashing
- **Environment Management:** `dotenv`
- **Testing:** Automated integration test suite (`npm test`) & Postman Collection v2.1
- **Client Explorer:** Built-in interactive Web Dashboard at `http://localhost:5000`

---

## 👥 User Roles & Permissions

| Role | Permissions |
| :--- | :--- |
| **Student** | • View cafeterias, crowd levels, and estimated wait times<br>• View cafeteria menus and live availability<br>• Reserve meals and pickup slots<br>• View own reservation history and cancel reservations<br>• Browse and claim surplus food flash offers |
| **Cafeteria Staff** | • Add and update cafeteria menu items and pricing<br>• Update live inventory stock (auto updates `isAvailable`)<br>• Update real-time crowd levels (`LOW`, `MEDIUM`, `HIGH`) and wait times<br>• Broadcast surplus food flash offers near closing time<br>• View student reservations queue and mark meals as completed<br>• Track food waste reduction metrics |

---

## 🗄️ Database Models

### 1. `User`
```json
{
  "id": "usr_student_01",
  "name": "Rahul Sharma",
  "email": "rahul@example.com",
  "passwordHash": "$2a$10$...",
  "role": "student", // "student" | "staff"
  "campusId": "CAMPUS_01",
  "createdAt": "2026-10-08T12:00:00.000Z"
}
```

### 2. `Cafeteria`
```json
{
  "id": "caf001",
  "name": "Main Cafeteria",
  "location": "Block A, Ground Floor",
  "description": "Central campus dining hall offering multicuisine breakfast, lunch, and dinner.",
  "openingTime": "08:00",
  "closingTime": "21:00",
  "currentCrowdLevel": "HIGH", // "LOW" | "MEDIUM" | "HIGH"
  "estimatedWaitTime": 15,
  "isOpen": true
}
```

### 3. `MenuItem`
> **Automatic Rule:** When `quantityAvailable` becomes 0, `isAvailable` automatically becomes `false`. When inventory is replenished, `isAvailable` automatically becomes `true`.
```json
{
  "id": "food001",
  "cafeteriaId": "caf001",
  "name": "Veg Biryani",
  "description": "Fragrant basmati rice cooked with fresh seasonal vegetables.",
  "price": 80,
  "category": "Main Course",
  "quantityAvailable": 25,
  "isAvailable": true
}
```

### 4. `Reservation`
> **Inventory Protection:** Decreases item inventory upon confirmation. Rejects reservations when requested quantity exceeds available stock. Restores inventory if cancelled.
```json
{
  "id": "res001",
  "studentId": "usr_student_01",
  "cafeteriaId": "caf001",
  "menuItemId": "food001",
  "quantity": 2,
  "pickupDate": "2026-10-09",
  "pickupTime": "13:00",
  "status": "CONFIRMED" // "PENDING" | "CONFIRMED" | "COMPLETED" | "CANCELLED"
}
```

### 5. `SurplusFood`
```json
{
  "id": "surplus001",
  "cafeteriaId": "caf001",
  "menuItemId": "food001",
  "quantityAvailable": 10,
  "originalPrice": 80,
  "discountedPrice": 30,
  "pickupStartTime": "20:00",
  "pickupEndTime": "21:00",
  "status": "AVAILABLE", // "AVAILABLE" | "CLAIMED" | "EXPIRED"
  "claimedCount": 4
}
```

---

## 🚀 Quick Start Guide

### 1. Clone & Install
```bash
git clone https://github.com/rakshitha1132/demo.git
cd demo
npm install
```

### 2. Configure Environment (`.env`)
A default `.env` is already provided. You can customize port or database connection:
```env
PORT=5000
NODE_ENV=development
MONGODB_URI=mongodb://localhost:27017/campus_smart_dining
JWT_SECRET=super_secret_campus_smart_dining_jwt_key_2026
JWT_EXPIRES_IN=7d
```

> **Zero-Setup Guarantee:** If you don't have MongoDB installed locally, the server starts up automatically using a high-performance in-memory fallback store with pre-seeded data so you can test all APIs immediately!

### 3. Start the Server
```bash
# Start backend server
npm start

# Or with live file reload
npm run dev
```
Visit the interactive dashboard at: **`http://localhost:5000`**

### 4. Run Automated Test Suite
```bash
npm test
```
Runs 25 automated end-to-end integration tests verifying every role, token, constraint, inventory check, and surplus claim!

---

## 🔑 Pre-seeded Test Accounts

| Role | Email | Password | Access |
| :--- | :--- | :--- | :--- |
| **Student** | `rahul@example.com` | `password123` | Reserve meals, view history, claim surplus |
| **Student** | `priya@example.com` | `password123` | Reserve meals, view history, claim surplus |
| **Staff/Admin** | `staff@campusdining.com` | `admin123` | Update crowd, wait times, menu, surplus offers |

---

## 📡 REST API Documentation

### 1. Authentication

#### Register User
```http
POST /api/auth/register
Content-Type: application/json

{
  "name": "Rahul Sharma",
  "email": "rahul@example.com",
  "password": "password123",
  "role": "student",
  "campusId": "CAMPUS_01"
}
```

#### Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "rahul@example.com",
  "password": "password123"
}
```
**Response:**
```json
{
  "success": true,
  "message": "Login successful.",
  "token": "eyJhbGciOi...",
  "user": {
    "id": "usr_student_01",
    "name": "Rahul Sharma",
    "email": "rahul@example.com",
    "role": "student",
    "campusId": "CAMPUS_01"
  }
}
```

---

### 2. Cafeterias & Crowd Tracking

#### Get All Cafeterias
```http
GET /api/cafeterias
```
**Response:**
```json
[
  {
    "id": "caf001",
    "name": "Main Cafeteria",
    "location": "Block A, Ground Floor",
    "crowdLevel": "HIGH",
    "estimatedWaitTime": 15,
    "isOpen": true
  }
]
```

#### Get Cafeteria Details
```http
GET /api/cafeterias/:id
```

#### Get Current Crowd Status
```http
GET /api/cafeterias/:id/crowd
```
**Response:**
```json
{
  "crowdLevel": "MEDIUM",
  "estimatedWaitTime": 10,
  "lastUpdated": "2026-10-08T12:30:00Z"
}
```

#### Update Crowd Status (Staff Only)
```http
PUT /api/cafeterias/:id/crowd
Authorization: Bearer <STAFF_JWT_TOKEN>
Content-Type: application/json

{
  "crowdLevel": "HIGH",
  "estimatedWaitTime": 20
}
```

---

### 3. Menu & Inventory

#### Get Cafeteria Menu
```http
GET /api/cafeterias/:id/menu
```

#### Add Menu Item (Staff Only)
```http
POST /api/cafeterias/:id/menu
Authorization: Bearer <STAFF_JWT_TOKEN>
Content-Type: application/json

{
  "name": "Veg Biryani",
  "price": 80,
  "category": "Main Course",
  "quantityAvailable": 25
}
```

#### Update Inventory (Staff Only)
```http
PUT /api/menu/:menuItemId/inventory
Authorization: Bearer <STAFF_JWT_TOKEN>
Content-Type: application/json

{
  "quantityAvailable": 30
}
```
*Note: Automatically updates `isAvailable = false` if `quantityAvailable = 0`.*

---

### 4. Reservations

#### Create Reservation (Student)
```http
POST /api/reservations
Authorization: Bearer <STUDENT_JWT_TOKEN>
Content-Type: application/json

{
  "cafeteriaId": "caf001",
  "menuItemId": "food001",
  "quantity": 2,
  "pickupDate": "2026-10-09",
  "pickupTime": "13:00"
}
```
*If insufficient inventory, returns `400 Bad Request`:*
```json
{
  "success": false,
  "message": "Only 1 item is currently available."
}
```

#### Get Student's Reservation History
```http
GET /api/reservations/my
Authorization: Bearer <STUDENT_JWT_TOKEN>
```

#### Cancel Reservation
```http
PUT /api/reservations/:id/cancel
Authorization: Bearer <STUDENT_JWT_TOKEN>
```
*Restores reserved quantity back into cafeteria inventory.*

#### Staff View Cafeteria Reservations
```http
GET /api/reservations/cafeteria/:cafeteriaId
Authorization: Bearer <STAFF_JWT_TOKEN>
```

#### Staff Mark Reservation as Completed
```http
PUT /api/reservations/:id/complete
Authorization: Bearer <STAFF_JWT_TOKEN>
```

---

### 5. Surplus Food Alerts & Rescue

#### Get Available Surplus Offers
```http
GET /api/surplus
```
**Response:**
```json
[
  {
    "id": "surplus001",
    "cafeteria": "Main Cafeteria",
    "food": "Veg Biryani",
    "quantityAvailable": 10,
    "originalPrice": 80,
    "discountedPrice": 30,
    "pickupStartTime": "20:00",
    "pickupEndTime": "21:00",
    "status": "AVAILABLE"
  }
]
```

#### Create Surplus Offer (Staff Only)
```http
POST /api/surplus
Authorization: Bearer <STAFF_JWT_TOKEN>
Content-Type: application/json

{
  "cafeteriaId": "caf001",
  "menuItemId": "food001",
  "quantityAvailable": 15,
  "originalPrice": 80,
  "discountedPrice": 30,
  "pickupStartTime": "20:00",
  "pickupEndTime": "21:00"
}
```

#### Claim Surplus Meal (Student)
```http
POST /api/surplus/:id/claim
Authorization: Bearer <STUDENT_JWT_TOKEN>
Content-Type: application/json

{
  "quantity": 1
}
```

---

### 6. Food Waste & Sustainability Statistics

#### Get Campus Food Waste Stats
```http
GET /api/stats/food-waste
```
**Response:**
```json
{
  "success": true,
  "summary": {
    "surplusMealsRescued": 12,
    "activeSurplusOffers": 2,
    "totalSurplusMealsListed": 20,
    "wasteDiversionRate": "60%",
    "moneySavedByStudents": "₹600",
    "environmentalImpact": {
      "foodSavedKg": "5.4 kg",
      "co2EmissionsPreventedKg": "13.5 kg CO2e"
    }
  }
}
```

---

## 📬 Postman Testing

A complete Postman collection is included in the project:
`postman_collection.json`

1. Open Postman.
2. Click **Import** and choose `postman_collection.json`.
3. Set the `baseUrl` variable to `http://localhost:5000/api`.
4. Run requests in folder `1. Authentication` to automatically populate tokens for student and staff.
5. Execute the remaining requests in any order!

---

## 📁 Project Structure

```
campus-food-smart-dining/
├── .env                          # Environment variables
├── .env.example                  # Template configuration
├── .gitignore                    # Git ignore file
├── package.json                  # NPM dependencies and scripts
├── postman_collection.json       # Postman API Collection v2.1
├── README.md                     # Comprehensive documentation
├── src/
│   ├── app.js                    # Express app initialization
│   ├── server.js                 # Server entry point
│   ├── config/
│   │   └── db.js                 # Database connection & fallback
│   ├── controllers/
│   │   ├── authController.js     # User registration & JWT login
│   │   ├── cafeteriaController.js# Cafeterias & crowd tracking
│   │   ├── menuController.js     # Menus & inventory management
│   │   ├── reservationController.js# Meal booking & queue management
│   │   ├── surplusController.js  # Surplus food offers & claims
│   │   └── statsController.js    # Food waste & sustainability metrics
│   ├── middleware/
│   │   ├── auth.js               # JWT verification & role authorization
│   │   └── errorHandler.js       # Centralized error handler
│   ├── models/
│   │   ├── Cafeteria.js          # Mongoose Cafeteria schema
│   │   ├── MenuItem.js           # Mongoose MenuItem schema (auto-avail)
│   │   ├── Reservation.js        # Mongoose Reservation schema
│   │   ├── SurplusFood.js        # Mongoose SurplusFood schema
│   │   ├── User.js               # Mongoose User schema (bcrypt hash)
│   │   └── store.js              # Resilient dual MongoDB / memory store
│   ├── public/
│   │   ├── app.js                # Interactive dashboard client script
│   │   └── index.html            # Interactive dashboard UI
│   ├── test/
│   │   └── api-test.js           # 25 automated integration tests
│   └── utils/
│       └── seeder.js             # Database seeding script
```

---

## 📄 License
ISC License. Built for Campus Smart Dining & Food Waste Elimination.