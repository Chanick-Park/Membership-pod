````swift
import Foundation
import UIKit

enum NotificationStatus {
    case idle
    case refreshInboxRequested
    case inboxRefreshed
    case event(NotificationEvent)
    case error(Error)
}

struct NotificationEvent {
    enum Kind {
        case push(userInfo: [AnyHashable: Any], completion: (UIBackgroundFetchResult) -> Void)
        case tap(options: [AnyHashable: Any])
        case deeplink(url: URL)
        case registration(deviceToken: Data)
        case registrationFailed(error: Error)
        case badgeReset
        case storePending
        case cleanupExpired(onlyExpired: Bool)
        // Add more as needed
    }
    let kind: Kind
}
````
# NotificationStatus Replacement Table

This table outlines the mapping from original notification service function calls to the unified `costcoAppState.notificationStatus` approach using the updated `NotificationEvent.Kind` enum.

| **Original Function Call**                                                      | **New NotificationStatus Replacement**                                                                                                                         |
|--------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `reloadNewInboxMessages()`<br>or<br>`Inbox.InboxNotificationService.shared.reloadInboxMessages()` | `costcoAppState.notificationStatus = .refreshInboxRequested`                                                                                                   |
| `callReceivedInboxNotificationService(userInfo, completionHandler)`             | `costcoAppState.notificationStatus = .event(NotificationEvent(kind: .push(userInfo: userInfo, completion: completionHandler)))`                                |
| `handleNotificationTap(options)`                                               | `costcoAppState.notificationStatus = .event(NotificationEvent(kind: .tap(options: options)))`                                                                 |
| `callRegisterInboxNotificationService(deviceToken)`                             | `costcoAppState.notificationStatus = .event(NotificationEvent(kind: .registration(deviceToken: deviceToken)))`                                                |
| `callRegisterFailInboxNotificationService(error)`                              | `costcoAppState.notificationStatus = .event(NotificationEvent(kind: .registrationFailed(error: error)))`                                                      |
| `storePendingInboxMessagesFromServiceExtension()`                              | `costcoAppState.notificationStatus = .event(NotificationEvent(kind: .storePending))`                                                                         |
| `inboxNotificationService.updateCount(count)`<br>`resetBadgeCount()`           | `costcoAppState.notificationStatus = .event(NotificationEvent(kind: .badge(count: count)))` <br/>(Use `count = 0` for reset)                                 |
| `cleanUpExpiredMessages(onlyExpired: true)`                                    | `costcoAppState.notificationStatus = .event(NotificationEvent(kind: .cleanupExpired(onlyExpired: true)))`                                                    |

---
