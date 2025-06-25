````swift
import Foundation
import UIKit

public enum NotificationStatus {
    case idle
    case refreshInboxRequested
    case event(NotificationEvent)
    case error(Error)
}

public struct NotificationEvent {
    public enum Kind {
        case push(userInfo: [AnyHashable: Any], completion: (UIBackgroundFetchResult) -> Void)
        case tap(options: [AnyHashable: Any]) // handle deeplink
        case registration(deviceToken: Data)
        case registrationFailed(error: Error)
        case storePending
        case cleanupMessages(onlyExpired: Bool)
        case showAllMessages(includeExpired: Bool)
        case badge(count: Int)
    }
    public let kind: Kind
    public init(kind: Kind) {
        self.kind = kind
    }
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
 - Subscribe `notificationStatus` in `InboxNotificationService.swift`
````swift
private func subscribeToNotificationStatus() {
    store.state.$notificationStatus
        .receive(on: DispatchQueue.main)
        .sink { [weak self] status in
            guard let self = self else { return }
            switch status {
            case .refreshInboxRequested:
                // Handle inbox refresh
                self.reloadInboxMessages()
            case .event(let event):
                switch event.kind {
                case .push(let userInfo, let completion):
                    // Handle remote push notification received
                    self.didReceiveRemoteNotification(userInfo: userInfo, fetchCompletionHandler: completion)
                case .tap(let options):
                    // Handle notification tap (Handle deeplink navigation)
                    self.handleNotificationTap(response: options)
                case .registration(let deviceToken):
                    // Handle device registration
                    self.didRegisterForRemoteNotifications(deviceToken: deviceToken)
                case .registrationFailed(let error):
                    // Handle registration failure
                    self.didFailToRegisterForRemoteNotifications(error: error)
                case .storePending:
                    // Handle storing pending inbox messages
                    self.storePendingInboxMessagesFromServiceExtension()
                case .cleanupMessages(let onlyExpired):
                    // Handle cleanup of expired messages
                    let migration = Inbox.InboxMessageDataMigration(container: self.container)
                    migration.cleanUpExpiredInboxMessages(onlyExpired: onlyExpired)
                case .showAllMessages(let includeExpired):
                    let migration = Inbox.InboxMessageDataMigration(container: self.container)
                    migration.showALLMessages(includeExpired)
                case .badge(let count):
                    // Handle badge update/reset
                    self.updateCount(count)
                }
            case .idle:
                break
            case .error(let error):
                logger.errorWithDefaultValues(error, "Unexpected Error: \(error.localizedDescription, privacy: .publicLog)")
            }
        }
        .store(in: &cancellables)
}
````
