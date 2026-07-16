**Vanteo Connect**

**Debit Card Onboarding**

Product Requirements Document

| **Document Status** | Draft                                            |
| ------------------- | ------------------------------------------------ |
| **Version**         | 1.0                                              |
| **Date**            | June 3, 2026                                     |
| **Author**          | Christina Zikeli, Vanteo Product                 |
| **Reviewers**       | Engineering, Design, Compliance, Banking Partner |

# **1\. Overview**

Vanteo Connect is a mobile app that services foreign nationals in the United States on employment or student visas. Many of these users arrive without a U.S. credit history or an established banking relationship. Vanteo Banking will provide a seamless, fast path to a working debit card which is critical to their day-to-day financial independence.

This document defines the product requirements for the Debit Card onboarding flow within the Vanteo Connect mobile application. The flow covers three sequential user actions after a bank account has been opened:

- View virtual debit card and card number, expiration date, and CVV
- Submit a request to mail a physical debit card
- Update Home Address
- Set a debit card PIN for the physical card

**TBD:**

- Freeze a card
- International Travel
- Card Setting
- Dispute a Transaction

Assumptions

# **2\. Goals & Success Metrics**

## **2.1 Goals**

- Provide immediate spending power via a virtual card within seconds of request
- Enable users to securely set a PIN without leaving the app

| US-06 | Compliance officer | Ensure PIN is set securely and never stored in plaintext | The product meets PCI DSS requirements |
| ----- | ------------------ | -------------------------------------------------------- | -------------------------------------- |

# **5\. User Flow**

The Debit Card onboarding flow is triggered from the home page

The flow is linear with a clear back-navigation model.

## **5.1 Entry Point - Bank Card page**

1 - the home screen / Account Balance card / Icon. The card displays a Card icon which navigates to the card setting page.

2 - or from the View Debit Card tile - when tapped shows the Card Settings pages

## **5.2 View Instant Virtual Card Number**

- User taps "View card" icon on the Bank card page
- App calls the banking partner API to issue the virtual card
- On success: card art animates in with full card number, expiry, and CVV revealed
- Card number is available for copy to clipboard
- On failure: inline error with retry option and support contact link

## **5.2 Step 1 Request a Physical Card**

- Screen displays Order a Card page
- Form displays with Order a new Card tile with name and address.
- User's address on file is pre-populated from account data
- User selects current address or enters a new address.
- New mailing address displays inline form to enter US address information.
  - Address fields: Street 1, Street 2 (optional), City, State, ZIP
  - Address must be a US address

## **5.3 Step 2 - Set Pin**

- After card is ordered, the flow auto-advances to PIN setup with a "Next" prompt
- Screen prompts: "Create a 4-digit PIN"
  - Numeric keypad (custom, no system keyboard) is displayed
  - PIN input is masked with dots as user types
- User enters 4 digits; screen advances to "Confirm your PIN"
- User re-enters the same 4 digits
- If PINs match: success toast, flow advances
- If PINs do not match: inline error, fields clear, user retries
- PIN is transmitted to banking partner via encrypted API call; it is never stored by Vanteo

## **5.4 Step 3- Review Order and Confirm**

- Display a Review your Order page to confirm address and name
- Display an estimated delivery date hwith business day
- Success screen: "Your card is on its way" with estimated delivery date range and order reference

# **6\. Screen Specifications**

| **Screen**                   | **Key Elements**                                              | **Primary CTA**        | **Error State**              |
| ---------------------------- | ------------------------------------------------------------- | ---------------------- | ---------------------------- |
| Card Intro                   | Card art, account summary, network logo                       | Freeze, View, Settings | API failure banner + Retry   |
| Debit Card Reveal            | Masked card number/expiry/CVV, copy icons, card art animation | Back to Bank Card Home | N/A                          |
| Order a Physical Card Banner | Card art, address pre-filled, edit fields                     | Order a card           | Address validation error     |
| PIN Entry                    | Custom numpad, 4 masked dots, progress indicator              | Confirm →              | Mismatch error, fields clear |
| PIN Confirm                  | Custom numpad, 4 masked dots                                  | Confirm PIN            | Mismatch error, fields clear |
|                              |                                                               |                        |                              |
| Order your Debit Card        | User name, Address, Add a new address                         | Next →                 | N/A                          |
| Review your Order            | User name, Addresshh                                          | Next →                 | N/A                          |
| Order Confirmation           | Order ref, estimated delivery, card status                    | Go to my account       | N/A                          |

# **7\. Functional Requirements**

## **7.1 Virtual Card Issuance**

- **System must call the banking partner's card issuance API upon user confirmation**

## **7.2 PIN Setup**

- PIN must be exactly 4 numeric digits
- PIN must be confirmed with a second entry; mismatch clears both fields and surfaces an error
- PIN transmission to the banking partner must use end-to-end encryption; Vanteo must not log or store the PIN

## **7.3 Physical Card Request**

- Mailing address must be pre-populated from account data;
- user may add a new address if needed and address stored to profile.
- Submission must call the banking partner API to enqueue physical card production and mailing
- System must display an order reference number and estimated delivery window upon success
- Card status (Processing, Mailed, Delivered) must be surfaced in the card section and updated via webhook from the banking partner

# **8\. Non-Functional Requirements**

| **Category**           | **Requirement**                                                                                |
| ---------------------- | ---------------------------------------------------------------------------------------------- |
| Performance            | Virtual card issuance API response ≤ 5s P99; PIN API response ≤ 3s P99                         |
| Security               | PCI DSS compliance for PIN handling; card details rendered in a secure view, not cached        |
| Accessibility          | WCAG 2.1 AA; custom numpad must support VoiceOver / TalkBack                                   |
| Localization           | English (en-US) at launch; architecture must support additional locales                        |
| Offline / Connectivity | Graceful degradation: show appropriate error if network is unavailable; no partial state saved |
| Session Security       | Card detail reveal requires biometric or passcode re-auth if session is older than 5 minutes   |
| Analytics              | All step completions and drop-offs must be instrumented with Segment events                    |

# **9\. API & Integrations**

| **Integration**                       | **Purpose**                                                           | **Owner**                     |
| ------------------------------------- | --------------------------------------------------------------------- | ----------------------------- |
| Banking Partner - Card Issuance API   | Issue virtual card, retrieve card details                             | Engineering / Banking Partner |
| Banking Partner - PIN Set API         | Transmit encrypted PIN to card processor                              | Engineering / Banking Partner |
| Banking Partner - Physical Card API   | Enqueue physical card production and mailing                          | Engineering / Banking Partner |
| Banking Partner - Card Status Webhook | Receive real-time card status updates (Processing, Mailed, Delivered) | Engineering                   |
| Email Notification Service            | Notify user when deposit clears and card is ready to activate         | Engineering                   |

# **10\. Edge Cases & Error States**

| **Scenario**                                 | **Expected Behavior**                                                                                |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Banking partner API timeout on card issuance | Show error banner; allow retry up to 3 times; offer support contact link after 3rd failure           |
| User loses connectivity mid-flow             | Show offline error; resume from last completed step when connectivity restores                       |
| PIN confirmation mismatch (3rd attempt)      | Invalidate session; prompt user to restart flow from Step 2 with a clear message                     |
|                                              |                                                                                                      |
| Physical card API failure after PIN success  | Show error; allow retry without requiring PIN re-entry; do not block use of virtual card             |
| User navigates back during PIN entry         | Warn user that PIN setup is incomplete; data cleared on back navigation                              |
| Duplicate card request (user submits twice)  | Banking partner API must be idempotent; UI disables CTA after first tap to prevent double submission |

# **11\. Compliance & Security**

- PIN is collected on device and transmitted directly to the banking partner using the partner's tokenization SDK; Vanteo servers never receive or log the PIN value PCI DSS Level 1:
- Full card number and CVV are shown only in an in-memory secure view; they must not be written to logs, analytics events, or local storage Card detail display:
- Unauthenticated re-auth is required to reveal card details after 5 minutes of inactivity Session timeout:
- Physical card issuance and mailing are subject to the banking partner's Reg E and CARD Act obligations; partner is responsible for compliant card delivery Regulatory:
- All card-related API calls must route through Vanteo's U.S.-based infrastructure in accordance with banking partner data residency requirements Data residency:

# **12\. Open Questions**

| **#** | **Question**                                                                                                              | **Owner**                                | **Due**     |
| ----- | ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- | ----------- |
| Q1    | Confirm new addresses stored and dated in backend                                                                         | Engineering                              | TBD         |
| Q2    | Can the address be validated checked against standard mailing addresses                                                   | Engineering                              | TBD         |
| Q3    | Will the PIN set API require the banking partner's mobile SDK, or is it a standard REST call with client-side encryption? | Engineering                              | TBD         |
| Q4    | Can Push Notifications be enabled?                                                                                        | Engineering                              | TBD         |
| Q5    | USPS Address Validation API                                                                                               | Validate and standardize mailing address | Engineering |

# **14\. Appendix**

## **14.1 Glossary**

| **Term**        | **Definition**                                                                                                          |
| --------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Virtual card    | A digital-only debit card with a card number, expiry, and CVV usable for online and contactless payments                |
| Instant issue   | A card issued immediately upon request without a delay for physical production                                          |
| PIN             | Personal Identification Number - a 4-digit code used to authenticate card-present transactions at ATMs and chip readers |
| PCI DSS         | Payment Card Industry Data Security Standard - the security standard governing card data handling                       |
| Banking partner | The FDIC-insured bank or card program manager that holds user deposits and issues cards on behalf of Vanteo             |
| Webhook         | An HTTP callback sent by the banking partner to Vanteo's backend when a card status event occurs                        |
