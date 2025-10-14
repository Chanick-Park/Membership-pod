# DMC Payment Flow – App & WCS Integration

## Overview

This document describes the flows for displaying payment-related UI in the app, based on DMC fetch and validation, and handling provisioning and delinking of credit cards via WCS and backend services.

---

## 1. UI Display Logic

- Show link: **‘Add Payment’** or **'Payment Info'**
- **Pilot users:**  
  - If `payments` data is non-empty, show payment-related UI.
  - If `payments` is empty, hide payment-related UI.

---

## 2. DMC Fetch (DMCHelper.java)

**Endpoint:**  
`https://www-vqa6.costco.com/dmc?deviceId=123&cclink=true|false`

- **`cclink=true`**: Returns **mock linked CC data** (for test); shows 'provisioned card' link.
- **`cclink=false`**: Pulls linked CC info from DMC EPR service; shows 'add payment' link.

> `cclink` is a temporary param for mock/test and must be removed when backend services are ready.

---

## 3. Sequence Diagram: DMC Fetch

```mermaid
sequenceDiagram
    participant App
    participant WCS
    participant DMC
    participant EPR

    App->>WCS: GET /dmc?deviceId=123&cclink=true|false
    WCS->>DMC: Fetch DMC data
    alt cclink=true (mock/test)
        DMC-->>WCS: Return mock provisioned CC data
        WCS-->>App: Show 'Provisioned Card' link
    else cclink=false
        DMC->>EPR: Query DMC EPR service for linked CC info
        alt Service Up
            EPR-->>DMC: Return isCCLinked=true/false
            DMC-->>WCS: Return CC info (isCCLinked)
            WCS-->>App: Show 'Add Payment' or 'Provisioned Card' link
        else Service Down
            DMC-->>WCS: Return isCCLinked=false
            WCS-->>App: Show 'Add Payment' link (Backend down)
        end
    end
```

---

## 4. DMC JSON Payload (with payments data)

```json
{
  "digitalCards": [
    {
      "memberCardNumber": "string",
      // ... other card info ...
      "payments": {
        "isCCLinked": true,    // CC provisioned via DMC-EPR service? Added to dmcHash
        "ccProductType": "Visa",
        "ccDigits": "1234"
      }
    }
  ],
  "dmcHash": "string"
}
```

---

## 5. DMC Validation (Refresh) – DMCValidationCmd.java

**Payload Example:**

```json
{
  "status": "valid",
  "rewardEstimates": [
    // ... estimates ...
  ],
  "payments": {
    "memberCardNumber": "string",
    "isCCLinked": true
  }
}
```

---

## 6. Display Rules

- **Show 'Add Payment' link:**  
  - If `isCCLinked` is **false** in JSON
  - If backend services are **down**
- **Show provisioned CC:**  
  - If `isCCLinked` is **true**
  - Linked CC is retrieved from DMC service
- **Backend Down:**  
  - Show ‘Add Payment’ link and/or generic user message

---

## 7. Sequence Diagram: Add Payment Flow

```mermaid
sequenceDiagram
    participant App
    participant WCS
    participant Citi

    App->>WCS: Click 'Add Payment'
    WCS->>Citi: POST /dmcprovision (device/profile info, mockResponse, otpResponse)
    alt mockResponse=GREEN
        Citi-->>WCS: Provision Success (200)
        WCS-->>App: Show provisioned CC info
    else mockResponse=YELLOW/RED/DELINK
        Citi-->>WCS: Provision Failure or Delink (400)
        WCS-->>App: Show Add Payment link or Cancel flow
    else Service Down
        Citi-->>WCS: Service unavailable
        WCS-->>App: Show generic user message
    end
```

---

## 8. Sequence Diagram: Payment Info (Delink) Flow

```mermaid
sequenceDiagram
    participant App
    participant WCS
    participant Citi

    App->>WCS: Click 'Payment Info'
    WCS->>Citi: POST /dmcprovision (device/profile info, mockResponse, otpResponse)
    alt mockResponse=GREEN
        Citi-->>WCS: Return Payment Info
        WCS-->>App: Display Payment Info
    else mockResponse=DELINK
        Citi-->>WCS: Delink Card, isCCLinked=false
        WCS-->>App: Show 'Add Payment' link
    else Service Down
        Citi-->>WCS: Service unavailable
        WCS-->>App: Show generic user message
    end
```

---

## 9. Error Handling

- If backend services are down or provisioning fails:
  - WCS displays a **generic user message**
  - ‘Cancel’ flow: App returns to previous state, DMC remains unchanged.

---

## References

- [DMC Test Data Google Sheet](https://docs.google.com/spreadsheets/d/1TmZkhx5Zhd2lkV_JqlFTIaosvSWB0fFU1PmZzwvq700/edit?usp=sharing)
- [DMC/CITI: Data json](https://docs.google.com/document/d/1ydwSCgp4k91vF_hiNjxxQ5F-YW3MR-jhuFlQi8XvRmU/edit?tab=t.0)
