# Design Patterns

This document describes the key design patterns used in the Plantae backend.

## Layered Architecture (Hexagonal/Onion)

The project follows a layered architecture, separating concerns into four main layers:

- **`domain`**: The core of the application, containing business entities, value objects, and repository interfaces. It has no dependencies on other layers.
- **`application`**: Orchestrates the domain logic. It contains application services, command/query handlers, and ports for external services. It depends only on the `domain` layer.
- **`infrastructure`**: Provides concrete implementations for interfaces defined in the `application` and `domain` layers. This includes database access, external API integrations, etc.
- **`interfaces`**: The outermost layer, exposing the application to the outside world. In this project, it's primarily the RESTful API controllers.

This separation of concerns makes the application more modular, testable, and easier to maintain.

## Dependency Injection (DI) and Inversion of Control (IoC)

We use Spring's IoC container to manage the lifecycle of our components (beans) and inject dependencies automatically. This is evident in our `@Configuration` classes, such as `SensorBeansConfig`:

```java
@Configuration
public class SensorBeansConfig {

    @Bean
    public RegisterSensorHandler registerSensorHandler(SensorRepository sensorRepository, PlantRepository plantRepository) {
        return new RegisterSensorHandler(sensorRepository, plantRepository);
    }

    // ... other bean definitions
}
```

Here, Spring injects the `SensorRepository` and `PlantRepository` dependencies into the `registerSensorHandler` when creating the bean. This decouples our components and makes them easier to test in isolation.

## Repository Pattern

The Repository pattern is used to abstract the data persistence logic from the business logic. We define repository interfaces in the `domain` layer (e.g., `SensorRepository`, `PlantRepository`) and provide concrete implementations in the `infrastructure` layer (e.g., `SensorRepositoryJpa`).

This allows us to change the underlying data storage mechanism without affecting the application's core logic.

## Command and Handler Pattern (CQS)

We separate write operations (Commands) from read operations (Queries), following the Command Query Separation (CQS) principle.

- **Commands** (`RegisterSensorCommand`, `CreatePlantCommand`) represent an intent to change the system's state. Each command has a dedicated **Handler** (`RegisterSensorHandler`, `CreatePlantHandler`) that processes it.
- **Queries** (`GetSensorByIdQuery`, `SearchPlantsQuery`) represent a request for data. They also have dedicated handlers (`GetSensorByIdHandler`, `SearchPlantsHandler`).

This pattern helps to create a more organized and predictable architecture.

## Data Transfer Object (DTO) Pattern

DTOs are used to transfer data between layers, especially from the application layer to the presentation (`interfaces`) layer. Examples include `DeviceDto`, `PlantDto`, and `AlertDto`.

This prevents leaking our internal domain entities to the client and allows us to shape the data specifically for the needs of the API consumers.
