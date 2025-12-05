# Component Documentation

This document provides a detailed specification of the RESTful endpoints for each component in the Plantae backend.

## Device Component

**Controller:** `DeviceController`

| Method | URI Path | Description | Parameters | Responses |
|---|---|---|---|---|
| GET | /api/v1/devices | List all devices for the current user. | None | 200 OK: A list of `DeviceDto` |
| GET | /api/v1/devices/{id} | Get a device by its ID. | Path: `id` (string) | 200 OK: `DeviceDto` <br> 404 Not Found |
| POST | /api/v1/devices | Register a new device. | Body: `RegisterDeviceCommand` | 201 Created: `DeviceDto` |
| PUT | /api/v1/devices/{id}/note | Update the note of a device. | Path: `id` (string), Body: `UpdateDeviceNoteCommand` | 200 OK: `DeviceDto` <br> 404 Not Found |
| PUT | /api/v1/devices/{id}/link-plant | Link a device to a plant. | Path: `id` (string), Body: `LinkDeviceToPlantCommand` | 200 OK: `DeviceDto` <br> 404 Not Found |
| DELETE | /api/v1/devices/{id} | Deactivate a device. | Path: `id` (string) | 200 OK: MessageResponse <br> 404 Not Found |

## Alert Component

**Controller:** `AlertController`

| Method | URI Path | Description | Parameters | Responses |
|---|---|---|---|---|
| GET | /api/v1/alerts | Get a list of alerts based on search criteria. | Query: `AlertSearchCriteria` | 200 OK: A list of `AlertDto` |
| GET | /api/v1/alerts/recent | Get a list of recent alerts for the current user. | None | 200 OK: A list of `AlertDto` |
| PUT | /api/v1/alerts/{id}/resolve | Resolve an alert. | Path: `id` (string) | 200 OK: `AlertDto` <br> 404 Not Found |

## IAM Component

**Controller:** `IamController`

| Method | URI Path | Description | Parameters | Responses |
|---|---|---|---|---|
| POST | /api/v1/iam/login | Login a user. | Body: `LoginUserCommand` | 200 OK: `AuthResponse` <br> 401 Unauthorized |
| POST | /api/v1/iam/register | Register a new user. | Body: `RegisterUserCommand` | 201 Created: `AuthResponse` |
| GET | /api/v1/iam/profile | Get the profile of the current user. | None | 200 OK: `ProfileResponse` |
| PUT | /api/v1/iam/profile/change-password | Change the password of the current user. | Body: `ChangePasswordCommand` | 200 OK: MessageResponse |

## Plant Component

**Controller:** `PlantController`

| Method | URI Path | Description | Parameters | Responses |
|---|---|---|---|---|
| POST | /api/v1/plants | Create a new plant. | Body: `CreatePlantCommand` | 201 Created: `PlantDto` |
| GET | /api/v1/plants/{id} | Get a plant by its ID. | Path: `id` (string) | 200 OK: `PlantDto` <br> 404 Not Found |
| GET | /api/v1/plants | Search for plants based on some criteria. | Query: `PlantSearchCriteria` | 200 OK: `PagedResult<PlantDto>` |
| PUT | /api/v1/plants/{id} | Update a plant. | Path: `id` (string), Body: `UpdatePlantCommand` | 200 OK: `PlantDto` <br> 404 Not Found |
| DELETE | /api/v1/plants/{id} | Delete a plant. | Path: `id` (string) | 200 OK: MessageResponse <br> 404 Not Found |

## Profile Component

**Controller:** `ProfileController`

| Method | URI Path | Description | Parameters | Responses |
|---|---|---|---|---|
| GET | /api/v1/profiles/{slug} | Get a profile by its slug. | Path: `slug` (string) | 200 OK: `ProfileDto` <br> 404 Not Found |
| PUT | /api/v1/profiles | Update the profile of the current user. | Body: `UpdateProfileCommand` | 200 OK: `ProfileDto` |

## Profile Settings Component

**Controller:** `ProfileSettingsController`

| Method | URI Path | Description | Parameters | Responses |
|---|---|---|---|---|
| GET | /api/v1/profile-settings/details | Get the profile details of the current user. | None | 200 OK: `ProfileDetailsDto` |
| PUT | /api/v1/profile-settings/details | Update the profile details of the current user. | Body: `UpdateProfileDetailsCommand` | 200 OK: `ProfileDetailsDto` |
| GET | /api/v1/profile-settings/notifications | Get the notification preferences of the current user. | None | 200 OK: `NotificationSettingsDto` |
| PUT | /api/v1/profile-settings/notifications | Update the notification preferences of the current user. | Body: `UpdateNotificationPreferencesCommand` | 200 OK: `NotificationSettingsDto` |

## Report Component

**Controller:** `ReportController`

| Method | URI Path | Description | Parameters | Responses |
|---|---|---|---|---|
| POST | /api/v1/reports/summary/pdf | Generate a summary PDF report. | Body: `SummaryReportRequestDto` | 200 OK: PDF file |
| POST | /api/v1/reports/plant/pdf | Generate a plant PDF report. | Body: `PlantReportRequestDto` | 200 OK: PDF file |
| POST | /api/v1/reports/plant/csv | Generate a plant CSV report. | Body: `PlantReportRequestDto` | 200 OK: CSV file |

## Sensor Component

**Controller:** `SensorController`

| Method | URI Path | Description | Parameters | Responses |
|---|---|---|---|---|
| GET | /api/v1/sensors | Search for sensors. | Query: `SensorSearchCriteria` | 200 OK: `PagedResult<SensorDto>` |
| POST | /api/v1/sensors/ingest | Ingest a sensor reading. | Body: `IngestSensorReadingCommand` | 200 OK: MessageResponse |
| GET | /api/v1/sensors/{id} | Get a sensor by its ID. | Path: `id` (string) | 200 OK: `SensorDto` <br> 404 Not Found |
| GET | /api/v1/sensors/{id}/readings | Get the readings of a sensor. | Query: `SensorReadingSearchCriteria` | 200 OK: `PagedResult<SensorReadingDto>` |
| GET | /api/v1/sensors/{id}/activity | Get the activity of a sensor. | None | 200 OK: `SensorActivityDto` |
| POST | /api/v1/sensors/{id}/link | Link a sensor to a device. | Path: `id` (string) | 200 OK: `SensorDto` <br> 404 Not Found |
| DELETE | /api/v1/sensors/{id} | Deactivate a sensor. | Path: `id` (string) | 200 OK: MessageResponse <br> 404 Not Found |
