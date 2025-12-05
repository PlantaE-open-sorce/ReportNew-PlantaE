# Routing Guide

This document provides an overview of how routing is handled in the Plantae backend, built with Spring Boot.

## Spring Web MVC Routing

Routing is managed using Spring Web MVC annotations. Here are the key annotations used in this project:

- **`@RestController`**: This annotation is applied to a class to mark it as a request handler. It combines `@Controller` and `@ResponseBody`, which means that the return value of the methods in this class will be automatically serialized into JSON and sent back in the HTTP response body.

- **`@RequestMapping("/api/v1/...")`**: This annotation is used at the class level to map a specific request path prefix to a controller. All the request mappings in the controller will be relative to this path.

- **`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`**: These are method-level annotations used to map HTTP requests to specific handler methods. They are shortcuts for `@RequestMapping(method = RequestMethod.GET)`, etc.

- **`@PathVariable`**: This annotation is used to bind a method parameter to a URI template variable. For example, in `@GetMapping("/{id}")`, you can use `@PathVariable String id` to get the `id` from the URL.

- **`@RequestBody`**: This annotation is used to bind a method parameter to the body of the HTTP request. Spring will deserialize the JSON from the request body into a Java object.

- **`@RequestParam`**: This annotation is used to bind a method parameter to a request parameter from the query string.

## Main Application Routes

The following table lists the base routes for each controller in the application. All routes are prefixed with `/api/v1`.

| HTTP Method | Path | Controller | Description |
|---|---|---|---|
| `POST` | `/iam/login` | `IamController` | Authenticates a user and returns a JWT token. |
| `POST` | `/iam/register` | `IamController` | Registers a new user. |
| `GET` | `/iam/profile` | `IamController` | Retrieves the profile of the authenticated user. |
| `GET` | `/devices` | `DeviceController` | Lists all devices for the current user. |
| `POST` | `/devices` | `DeviceController` | Registers a new device. |
| `GET` | `/devices/{id}` | `DeviceController` | Retrieves a specific device by its ID. |
| `PUT` | `/devices/{id}/note` | `DeviceController` | Updates the note for a device. |
| `GET` | `/alerts` | `AlertController` | Retrieves a list of alerts. |
| `PUT` | `/alerts/{id}/resolve` | `AlertController` | Resolves an alert. |
| `GET` | `/plants` | `PlantController` | Searches for plants. |
| `POST` | `/plants` | `PlantController` | Creates a new plant. |
| `GET` | `/plants/{id}` | `PlantController` | Retrieves a specific plant by its ID. |
| `GET` | `/profiles/{slug}` | `ProfileController` | Retrieves a user profile by its slug. |
| `PUT` | `/profiles` | `ProfileController` | Updates the profile of the current user. |
| `GET` | `/profile-settings/details` | `ProfileSettingsController` | Gets the profile details of the current user. |
| `PUT` | `/profile-settings/details` | `ProfileSettingsController` | Updates the profile details of the current user. |
| `POST` | `/reports/summary/pdf` | `ReportController` | Generates a summary report in PDF format. |
| `GET` | `/sensors` | `SensorController` | Searches for sensors. |
| `POST` | `/sensors/ingest` | `SensorController` | Ingests a new sensor reading. |
| `GET` | `/sensors/{id}` | `SensorController` | Retrieves a specific sensor by its ID. |

For more detailed information on the endpoints, including request and response formats, please refer to the `COMPONENT_DOCUMENTATION.md` file.
