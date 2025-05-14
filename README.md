# Alx Airbnb Backend Documentation

This document outlines the technical and functional requirements for key backend features of the Airbnb Clone project, including User Authentication, Property Management (Create Listing), and the Booking System (Make a Booking).

## 1. User Authentication

**Description:** This feature enables users (guests and hosts) to create accounts, and log in securely.

**Functional Requirements:**

* **User Registration:** The system shall allow new users to register with their email address, first name, last name, and a password.
* **User Login:** Registered users shall be able to log in using their email address and password.
* **Secure Password Storage:** The system shall securely store user passwords using a strong hashing algorithm. Plain text passwords shall never be stored.
* **Authentication Token Generation:** Upon successful login, the system shall generate a secure authentication token (e.g., JWT - JSON Web Tokens) for the user.
* **Token-Based Authentication:** Subsequent requests from authenticated users shall include the authentication token in the request headers (e.g., `Authorization: Bearer <token>`).
* **Token Validation:** The backend shall validate the authenticity and expiration of the provided authentication token on protected routes.
* **User Logout:** Authenticated users shall be able to log out, which may invalidate their current authentication token (depending on the implementation strategy).

**Technical Requirements:**

* **API Endpoints:**
  * `POST /api/auth/register`
  * `POST /api/auth/login`
  * `POST /api/auth/logout`

* **Input Specifications:**
  * **`POST /api/auth/register`:**

    ```json
        {
          "first_name": "string (required, minLength: 2, maxLength: 50)",
          "last_name": "string (required, minLength: 2, maxLength: 50)",
          "email": "string (required, format: email, maxLength: 100, unique)",
          "password": "string (required, minLength: 8, maxLength: 100)"
        }
    ```

  * **`POST /api/auth/login`:**

    ```json
        {
          "email": "string (required, format: email, maxLength: 100)",
          "password": "string (required, minLength: 8, maxLength: 100)"
        }
    ```

  * **`POST /api/auth/logout` (or `DELETE`):**
    * Requires authentication token in the `Authorization` header. No request body typically needed.
* **Output Specifications:**
  * **`POST /api/auth/register`:**
    * **Success (HTTP 201 Created):**
            ```json
            {
              "message": "User registered successfully"
            }
            ```
    * **Error (HTTP 400 Bad Request):** Validation errors (e.g., invalid email, password too short).
    * **Error (HTTP 409 Conflict):** Email address already exists.
  * **`POST /api/auth/login`:**
    * **Success (HTTP 200 OK):**
            ```json
            {
              "token": "string (JWT authentication token)"
            }
            ```
    * **Error (HTTP 401 Unauthorized):** Invalid credentials (incorrect email or password).
  * **`POST /api/auth/logout` (or `DELETE`):**
    * **Success (HTTP 204 No Content):** Logout successful.
    * **Error (HTTP 401 Unauthorized):** Invalid or missing authentication token.
* **Validation Rules:**
  * Email format must be valid.
  * Password must meet minimum length requirements.
  * First and last names must have a minimum length.
  * Email address must be unique during registration.
* **Performance Criteria:**
  * User registration should complete within 1 second under normal load.
  * User login should complete within 500 milliseconds under normal load.
  * Token validation should add minimal overhead (less than 50 milliseconds) to protected API requests.

## 2. Property Management - Create a Property Listing

**Description:** This feature allows authenticated hosts to create new property listings on the platform.

**Functional Requirements:**

* **Host Authentication:** Only authenticated users with the 'host' role shall be authorized to create property listings.
* **Property Details Input:** Hosts shall be able to provide detailed information about their property, including name, description, location (address, city, country, coordinates), price per night, amenities (e.g., Wi-Fi, kitchen, parking), number of guests allowed, number of bedrooms, number of bathrooms, and property type (e.g., apartment, house, villa).
* **Image Upload:** Hosts shall be able to upload multiple images of their property.
* **Data Validation:** The system shall validate all input data to ensure it meets required formats and constraints.
* **Property Listing Creation:** Upon successful validation, the system shall create a new property listing in the database associated with the host's user ID.

**Technical Requirements:**

* **API Endpoint:**
  * `POST /api/properties` (Requires authentication token in the `Authorization` header with 'host' role)
* **Input Specifications:**
  * **`POST /api/properties`:**
    
    ```json
        {
          "name": "string (required, minLength: 5, maxLength: 200)",
          "description": "string (required, minLength: 20, maxLength: 1000)",
          "address": "string (required, maxLength: 255)",
          "city": "string (required, maxLength: 100)",
          "country": "string (required, maxLength: 100)",
          "latitude": "number (required, format: float)",
          "longitude": "number (required, format: float)",
          "price_per_night": "number (required, format: float, min: 1)",
          "amenities": "array (string, optional)",
          "guests_allowed": "integer (required, min: 1)",
          "bedrooms": "integer (required, min: 0)",
          "bathrooms": "number (required, format: float, min: 0.5)",
          "property_type": "string (required, enum: ['apartment', 'house', 'villa', ...])",
          "images": "array (string, format: URL or base64 encoded, optional, maxItems: 10)"
        }
    ```

* **Output Specifications:**
  * **Success (HTTP 201 Created):**

    ```json
        {
          "property_id": "UUID",
          "message": "Property listing created successfully"
        }
    ```

  * **Error (HTTP 400 Bad Request):** Validation errors (e.g., invalid input format, missing required fields).
  * **Error (HTTP 401 Unauthorized):** User is not authenticated or does not have the 'host' role.
  * **Error (HTTP 500 Internal Server Error):** Database error during creation.
* **Validation Rules:**
  * All required fields must be present.
  * String lengths must be within the specified limits.
  * `email` format must be valid.
  * `latitude` and `longitude` must be valid coordinates.
  * `price_per_night`, `guests_allowed`, `bedrooms`, and `bathrooms` must be positive numbers within reasonable limits.
  * `property_type` must be one of the allowed values.
  * Image URLs or base64 strings should be in a valid format.
* **Performance Criteria:**
  * Creating a property listing (excluding image processing) should complete within 2 seconds under normal load.
  * Image uploads should complete within a reasonable time depending on the file size and network conditions (e.g., under 5 seconds for each image up to 2MB).

## 3. Booking System - Make a Booking

**Description:** This feature allows authenticated guests to book available properties for a specified date range.

**Functional Requirements:**

* **Guest Authentication:** Only authenticated users (guests) shall be authorized to make bookings.
* **Property Availability Check:** The system shall allow guests to check the availability of a property for a specific start and end date.
* **Booking Request:** Guests shall be able to submit a booking request for a property, specifying the check-in date, check-out date, and the number of guests.
* **Price Calculation:** The system shall calculate the total booking price based on the property's price per night, the duration of the stay, and potentially other factors (e.g., service fees, discounts).
* **Validation:** The system shall validate the requested dates (check-in must be before check-out, dates must be in the future), the number of guests (must not exceed the property's limit), and the property's availability.
* **Confirmation:** Upon successful validation and payment (handled by the Payments feature), the system shall create a booking record in the database, associating the guest, the property, and the booking dates.
* **Booking Status:** The booking shall have a status (e.g., 'pending', 'confirmed').

**Technical Requirements:**

* **API Endpoints:**
  * `GET /api/properties/{property_id}/availability?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD` (Public endpoint)
  * `POST /api/bookings` (Requires authentication token in the `Authorization` header with 'guest' role)
* **Input Specifications:**
  * **`GET /api/properties/{property_id}/availability`:**
    * Path Parameter: `property_id` (UUID, required)
    * Query Parameters:
      * `start_date`: `string (required, format: YYYY-MM-DD)`
      * `end_date`: `string (required, format: YYYY-MM-DD)`
  * **`POST /api/bookings`:**

    ```json
        {
          "property_id": "UUID (required)",
          "start_date": "string (required, format: YYYY-MM-DD)",
          "end_date": "string (required, format: YYYY-MM-DD)",
          "guests": "integer (required, min: 1)"
        }
    ```

* **Output Specifications:**
  * **`GET /api/properties/{property_id}/availability`:**
    * **Success (HTTP 200 OK):**

    ```json
            {
              "available": "boolean"
            }
    ```

    * **Error (HTTP 400 Bad Request):** Invalid date format or missing parameters.
    * **Error (HTTP 404 Not Found):** Property not found.
  * **`POST /api/bookings`:**
    * **Success (HTTP 201 Created):**

        ```json
            {
              "booking_id": "UUID",
              "total_price": "number (format: float)",
              "message": "Booking request submitted successfully (awaiting payment)"
            }
        ```

    * **Error (HTTP 400 Bad Request):** Validation errors (e.g., invalid dates, guest limit exceeded, property not available).
    * **Error (HTTP 401 Unauthorized):** User is not authenticated or does not have the 'guest' role.
    * **Error (HTTP 404 Not Found):** Property not found.
    * **Error (HTTP 500 Internal Server Error):** Database error during booking creation.
* **Validation Rules:**
  * `property_id` must be a valid UUID.
  * `start_date` and `end_date` must be valid dates in the format YYYY-MM-DD.
  * `start_date` must be before `end_date`.
  * Requested dates must be in the future (or have a configurable allowed past window for specific scenarios).
  * The number of `guests` must not exceed the `guests_allowed` for the property.
  * The requested booking dates must not overlap with existing confirmed bookings for the property.
* **Performance Criteria:**
  * Checking property availability should complete within 1 second under normal load.
  * Submitting a booking request (excluding payment processing) should complete within 2 seconds under normal load.