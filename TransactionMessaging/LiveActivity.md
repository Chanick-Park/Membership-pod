#### Live Activity Sequence Diagram
sequenceDiagram
    participant User as User
    participant App as iOS App
    participant ActivityKit as ActivityKit (Live Activity)
    participant Backend as Delivery Backend
    participant Push as Push Notification Service (APNs/Adobe)

    User->>App: Places order
    App->>Backend: Create order API (returns orderID)
    App->>ActivityKit: Start Live Activity (initial status)
    loop User opens app or wants to check status
        App->>Backend: GET /order/{orderID}/status
        Backend-->>App: Respond with delivery status
        App->>ActivityKit: Update Live Activity (optional, if status changed)
    end

    alt Real-time push updates
        Backend->>Push: Send status update push for orderID
        Push->>App: Push notification (status update)
        App->>ActivityKit: Update Live Activity
    end

    alt Delivery complete
        Backend->>Push: Send "delivered" status push
        Push->>App: Push notification ("delivered")
        App->>ActivityKit: End Live Activity
    end
