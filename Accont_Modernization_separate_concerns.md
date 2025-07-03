# AccountViewModel: Separated Concerns & Mermaid Diagram

This document summarizes the separated concerns for `AccountViewModel` and visualizes the suggested architecture for the `MyAccount` module with Mermaid syntax.

---

## Separated Concerns

| Concern                                | Extraction Target/Class             | Description |
|-----------------------------------------|-------------------------------------|-------------|
| 1. Combine subscription patterns        | Subscription helper method          | Generic helper to reduce repetitive `.sink` code. |
| 2. Menu logic                          | `MenuBuilder` (utility class)       | Sorting, filtering, and updating menu sections/items. |
| 3. Session/business/logout logic        | `AccountSessionService`             | Handles side effects, logout, and session clearing. |
| 4. QR code & card logic                 | `QRCodeManager`, `QRCodeGenerator`  | QR code generation/caching, card filtering, eligibility. |
| 5. Analytics/notification/widget logic  | `AnalyticsService`, `NotificationDispatcher`, `WidgetUpdater` | Analytics, notification posting, widget updates. |
| 6. Storage/key-value/defaults           | `AccountPreferences`                | Centralizes user/session storage and key management. |
| 7. Tutorial/app tutorial logic          | `AppTutorialCoordinator`            | Handles when/how to show/dismiss tutorials. |

---

## Suggested Architecture Diagram (Mermaid)

```mermaid
classDiagram
    class AccountViewModel {
        - store
        - accountSessionSvc
        - analyticsSvc
        - notificationDisp
        - widgetUpdater
        - preferences
        - qrCodeManager
        - tutorialCoordinator
        + loadAccountMenu()
        + handleUserStateChange()
        + getQRcodeImage()
        + ...
    }

    class MenuBuilder {
        <<static utility>>
    }

    AccountViewModel --> MenuBuilder : uses

    class AccountSessionServicing
    class AccountSessionService
    AccountViewModel --> AccountSessionServicing : depends on
    AccountSessionService ..|> AccountSessionServicing

    class AnalyticsServicing
    class AnalyticsService
    AccountViewModel --> AnalyticsServicing : depends on
    AnalyticsService ..|> AnalyticsServicing

    class NotificationDispatching
    class NotificationDispatcher
    AccountViewModel --> NotificationDispatching : depends on
    NotificationDispatcher ..|> NotificationDispatching

    class WidgetUpdating
    class WidgetUpdater
    AccountViewModel --> WidgetUpdating : depends on
    WidgetUpdater ..|> WidgetUpdating

    class QRCodeManaging
    class QRCodeManager
    AccountViewModel --> QRCodeManaging : depends on
    QRCodeManager ..|> QRCodeManaging

    class AccountPreferring
    class AccountPreferences
    AccountViewModel --> AccountPreferring : depends on
    AccountPreferences ..|> AccountPreferring

    class TutorialCoordinating
    class AppTutorialCoordinator
    AccountViewModel --> TutorialCoordinating : depends on
    AppTutorialCoordinator ..|> TutorialCoordinating
```

---

**How to use:**  
Copy the Mermaid code block above into a Mermaid-compatible viewer (e.g., [Mermaid Live Editor](https://mermaid.live/)) to visualize the diagram.

---
