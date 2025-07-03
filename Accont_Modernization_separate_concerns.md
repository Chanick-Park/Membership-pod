# AccountViewModel: Separated Concerns, Steps & Mermaid Diagram

This document summarizes the separated concerns for `AccountViewModel`, shows the suggested architectural diagram (Mermaid), and provides clear, step-by-step suggestions for each concern.

---

## Separated Concerns

| # | Concern                                | Extraction Target/Class             |
|:-:|----------------------------------------|-------------------------------------|
| 1 | Combine subscription patterns          | Subscription helper method          |
| 2 | Menu logic                             | `MenuBuilder` (utility class)       |
| 3 | Session/business/logout logic          | `AccountSessionService`             |
| 4 | QR code & card logic                   | `QRCodeManager`, `QRCodeGenerator`  |
| 5 | Analytics/notification/widget logic    | `AnalyticsService`, `NotificationDispatcher`, `WidgetUpdater` |
| 6 | Storage/key-value/defaults             | `AccountPreferences`                |
| 7 | Tutorial/app tutorial logic            | `AppTutorialCoordinator`            |

---

## Steps for Each Concern

### 1. Combine Subscription Patterns

**Step-by-step:**
1. Create a generic `subscribe` helper that takes a publisher and a closure.
2. Replace all repetitive `.sink { ... }.store(in: &subscriptions)` code in your ViewModel with calls to this helper.

---

### 2. Menu Logic

**Step-by-step:**
1. Identify all menu transformation code (`updateRewardsMenuItemIfNeeded`, `sortMenuItems`, etc.).
2. Move these into a static utility class/struct named `MenuBuilder`.
3. In your ViewModel, replace logic with calls to `MenuBuilder` methods.

---

### 3. Session/Business/Logout Logic

**Step-by-step:**
1. Extract all business logic or side-effects (logout, session clearing, data deletion) into a new service class, e.g., `AccountSessionService`.
2. Define a protocol (e.g., `AccountSessionServicing`) for this service.
3. Inject this service into the ViewModel.
4. Replace direct logic in the ViewModel with service method calls.

---

### 4. QR Code & Card Logic

**Step-by-step:**
1. Move QR generation/caching logic into `QRCodeManager`.
2. Extract QR code image creation to a static `QRCodeGenerator`.
3. If card filtering/eligibility logic is complex, create a `CardManager`.
4. Inject these into the ViewModel and delegate logic accordingly.

---

### 5. Analytics/Notification/Widget Logic

**Step-by-step:**
1. Create separate service classes: `AnalyticsService`, `NotificationDispatcher`, `WidgetUpdater`.
2. Define protocols for each service.
3. Move all analytics, notification, and widget update code to these services.
4. Inject them into the ViewModel and replace direct calls with service calls.

---

### 6. Storage/Key-Value/Defaults

**Step-by-step:**
1. Create an `AccountPreferences` abstraction to wrap all key-value storage logic.
2. Move scattered `userDefaults`, `secureDefaults`, etc. code into this class.
3. Define a protocol for testability (`AccountPreferring`).
4. Use only this abstraction in your ViewModel.

---

### 7. Tutorial/App Tutorial Logic

**Step-by-step:**
1. Extract all tutorial-related logic (when to show, dismiss, orientation handling) into a coordinator/service, e.g., `AppTutorialCoordinator`.
2. Define a protocol (`TutorialCoordinating`).
3. Inject this coordinator into your ViewModel and use it for all tutorial actions.

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
Copy the Mermaid code block above into a Mermaid-compatible viewer (e.g., [Mermaid Live Editor](https://mermaid.live/)) to see the diagram.

---
