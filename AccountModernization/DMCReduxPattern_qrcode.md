```mermaid
flowchart TD
    subgraph TCA
        QRCodeView["QRCodeView (SwiftUI View)"]
        QRCodeViewModel["QRCodeViewModel (Store<ViewState, ViewAction>)"]
        QRCodeState["QRCodeState (State)"]
        QRCodeReducer["QRCodeReducer (Reducer)"]
        QRCodeAction["QRCodeAction (Action)"]
        QRCodeTimerScheduler["QRCodeTimerScheduler (Effect/Dependency)"]
    end

    QRCodeView -- bind/observe --> QRCodeViewModel
    QRCodeViewModel -- observes --> QRCodeState
    QRCodeView -- sends --> QRCodeViewModel
    QRCodeViewModel -- sends --> QRCodeAction
    QRCodeAction -- handled by --> QRCodeReducer
    QRCodeReducer -- updates --> QRCodeState
    QRCodeReducer -- triggers --> QRCodeTimerScheduler
    QRCodeTimerScheduler -- dispatches --> QRCodeAction

    classDef view fill:#f1f9ff,stroke:#14a4fa,color:#333;
    classDef state fill:#f3f6fb,stroke:#3c66b0,color:#333;
    classDef reducer fill:#f7e0b7,stroke:#b37316,color:#333;
    classDef action fill:#fbeee6,stroke:#d48f36,color:#333;
    classDef effect fill:#e7f7e7,stroke:#1e8449,color:#333;

    class QRCodeView view;
    class QRCodeViewModel state;
    class QRCodeState state;
    class QRCodeReducer reducer;
    class QRCodeAction action;
    class QRCodeTimerScheduler effect;
```

Explanation:

QRCodeView: SwiftUI view, observes the Store/ViewModel and sends user actions.
QRCodeViewModel (Store): Owns QRCodeState, receives actions from view, exposes state to the view.
QRCodeState: Holds QR code value, expiration, validity, etc.
QRCodeAction: Enum of user/system actions (refresh, tick, expire).
QRCodeReducer: Handles actions, updates state, triggers effects/timers.
QRCodeTimerScheduler: Timer effect/dependency; dispatches actions (e.g., tick, expire) back to the store.
