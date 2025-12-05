# API Usage Guides

This document provides guides for using the Plantae API to interact with each integrated functionality.

## Device Management

The Device Management API allows you to register, configure, and manage your IoT devices.

### Registering a New Device

To register a new device, send a `POST` request to `/api/v1/devices` with the device details.

**Request:**
```http
POST /api/v1/devices
Content-Type: application/json

{
  "name": "My New Sensor",
  "type": "SOIL_MOISTURE"
}
```

**Response:**
```json
{
  "id": "device-xyz-123",
  "name": "My New Sensor",
  "type": "SOIL_MOISTURE",
  "status": "ACTIVE",
  "note": null
}
```

### Listing Your Devices

To get a list of all your registered devices, send a `GET` request to `/api/v1/devices`.

**Request:**
```http
GET /api/v1/devices
```

**Response:**
```json
[
  {
    "id": "device-xyz-123",
    "name": "My New Sensor",
    "type": "SOIL_MOISTURE",
    "status": "ACTIVE",
    "note": null
  },
  {
    "id": "device-abc-456",
    "name": "Living Room Climate Sensor",
    "type": "TEMPERATURE",
    "status": "ACTIVE",
    "note": "Monitors the temperature and humidity."
  }
]
```

## Plant Management

The Plant Management API allows you to create, monitor, and manage your plants.

### Creating a New Plant

To create a new plant, send a `POST` request to `/api/v1/plants` with the plant details.

**Request:**
```http
POST /api/v1/plants
Content-Type: application/json

{
  "name": "My Fiddle Leaf Fig",
  "species": "Ficus lyrata",
  "birthDate": "2023-01-15T10:00:00Z"
}
```

**Response:**
```json
{
  "id": "plant-abc-789",
  "name": "My Fiddle Leaf Fig",
  "species": "Ficus lyrata",
  "birthDate": "2023-01-15T10:00:00Z",
  "status": "HEALTHY"
}
```

### Getting Plant Details

To get the details of a specific plant, send a `GET` request to `/api/v1/plants/{id}`.

**Request:**
```http
GET /api/v1/plants/plant-abc-789
```

**Response:**
```json
{
  "id": "plant-abc-789",
  "name": "My Fiddle Leaf Fig",
  "species": "Ficus lyrata",
  "birthDate": "2023-01-15T10:00:00Z",
  "status": "HEALTHY"
}
```

## Sensor Data Ingestion

The Sensor Data Ingestion API allows you to send sensor readings to the platform.

### Ingesting a Sensor Reading

To ingest a sensor reading, send a `POST` request to `/api/v1/sensors/ingest`.

**Request:**
```http
POST /api/v1/sensors/ingest
Content-Type: application/json

{
  "sensorId": "sensor-xyz-123",
  "metric": "TEMPERATURE",
  "value": 22.5
}
```

**Response:**
```json
{
  "message": "Sensor reading ingested successfully."
}
```

## User and Profile Management

The User and Profile Management API allows you to manage user accounts and profiles.

### User Login

To log in a user, send a `POST` request to `/api/v1/iam/login`.

**Request:**
```http
POST /api/v1/iam/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "your-password"
}
```

**Response:**
```json
{
  "token": "your-jwt-token",
  "profile": {
    "id": "profile-abc-123",
    "username": "user",
    "slug": "user"
  }
}
```

### Getting the Current User's Profile

To get the profile of the currently authenticated user, send a `GET` request to `/api/v1/iam/profile`.

**Request:**
```http
GET /api/v1/iam/profile
Authorization: Bearer your-jwt-token
```

**Response:**
```json
{
  "id": "profile-abc-123",
  "username": "user",
  "slug": "user"
}
```
