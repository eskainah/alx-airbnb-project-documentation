### **1. User Authentication**

**Feature Overview:**
Handles user registration, login, logout, and session/token management for renters and hosts.

#### API Endpoints:

**POST /api/auth/register**

* **Description**: Registers a new user.
* **Input (JSON)**:

  ```json
  {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "SecureP@ss123"
  }
  ```
* **Output (Success - 201)**:

  ```json
  {
    "message": "User registered successfully.",
    "userId": "u123",
    "token": "jwt_token_here"
  }
  ```
* **Validation Rules**:

  * `email`: valid format, unique
  * `password`: min 8 chars, at least one uppercase, one number
  * `name`: non-empty

---

**POST /api/auth/login**

* **Description**: Authenticates a user and returns a JWT token.
* **Input (JSON)**:

  ```json
  {
    "email": "john@example.com",
    "password": "SecureP@ss123"
  }
  ```
* **Output (Success - 200)**:

  ```json
  {
    "message": "Login successful.",
    "token": "jwt_token_here"
  }
  ```
* **Validation Rules**:

  * Check if user exists
  * Password matches hashed password

---

**POST /api/auth/logout**

* **Description**: Logs the user out (optional server-side token invalidation).
* **Input**: Auth header with token
* **Output (Success - 200)**:

  ```json
  {
    "message": "Logged out successfully."
  }
  ```

---

**Performance Criteria**:

* Token generation within 100ms
* Support up to 10,000 concurrent logins
* Login response time < 300ms under load

---

### **2. Property Management**

**Feature Overview:**
Allows hosts to create, update, retrieve, and delete property listings.

#### API Endpoints:

**POST /api/properties**

* **Description**: Create a new property listing.
* **Input (JSON)**:

  ```json
  {
    "title": "Modern Apartment in City Center",
    "description": "2 bed, 1 bath with city view",
    "location": "New York, NY",
    "pricePerNight": 120.00,
    "amenities": ["WiFi", "AC"],
    "images": ["url1", "url2"]
  }
  ```
* **Output (201)**:

  ```json
  {
    "propertyId": "prop123",
    "message": "Property created successfully."
  }
  ```
* **Validation Rules**:

  * `title`: 10–100 chars
  * `pricePerNight`: > 0
  * `location`: non-empty
  * Auth token required for host

---

**GET /api/properties/{id}**

* **Description**: Get property details by ID.
* **Output (200)**:

  ```json
  {
    "propertyId": "prop123",
    "title": "Modern Apartment in City Center",
    "location": "New York, NY",
    ...
  }
  ```

---

**PUT /api/properties/{id}**

* **Description**: Update an existing property.
* **Input**: Same as POST
* **Auth**: Only the owner can update
* **Output (200)**:

  ```json
  {
    "message": "Property updated."
  }
  ```

---

**DELETE /api/properties/{id}**

* **Description**: Remove a property listing.
* **Output (200)**:

  ```json
  {
    "message": "Property deleted."
  }
  ```

---

**Performance Criteria**:

* CRUD operations response time < 400ms
* Scalable to 100,000+ listings
* Image upload via CDN or async background job

---

### **3. Booking System**

**Feature Overview:**
Allows renters to book properties, check availability, and view bookings.

#### API Endpoints:

**POST /api/bookings**

* **Description**: Create a booking.
* **Input (JSON)**:

  ```json
  {
    "propertyId": "prop123",
    "startDate": "2025-06-10",
    "endDate": "2025-06-15"
  }
  ```
* **Output (201)**:

  ```json
  {
    "bookingId": "book789",
    "totalPrice": 600,
    "message": "Booking confirmed."
  }
  ```
* **Validation Rules**:

  * Auth token required (renter)
  * `startDate` < `endDate`, dates not in past
  * Check availability (no overlaps)

---

**GET /api/bookings/{id}**

* **Description**: Retrieve a specific booking.
* **Output (200)**:

  ```json
  {
    "bookingId": "book789",
    "status": "confirmed",
    "propertyId": "prop123",
    "userId": "user456",
    ...
  }
  ```

---

**GET /api/users/{id}/bookings**

* **Description**: Retrieve all bookings for a user.
* **Output**: Array of bookings

---

**DELETE /api/bookings/{id}**

* **Description**: Cancel a booking (before start date).
* **Output**:

  ```json
  {
    "message": "Booking cancelled."
  }
  ```

---

**Performance Criteria**:

* Booking creation under 500ms
* Prevent double bookings (race condition handling with locking/transactions)
* Ensure idempotency for duplicate booking requests