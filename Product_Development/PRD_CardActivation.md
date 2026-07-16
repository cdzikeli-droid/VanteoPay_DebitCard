**VanteoPay**

**Debit Card Onboarding & Activation**

_Product Requirements Document_

| **Field**      | **Value**                          |
| -------------- | ---------------------------------- |
| Feature ID     | DBT01                              |
| Feature Name   | Debit Card Onboarding & Activation |
| Product Area   | Debit Card / Banking               |
| Target Release | TBD                                |
| Author         | Christina Zikeli, Vanteo Product   |
| Date           | July 16, 2026                      |
| Version        | 2.0                                |

**Change Log**

| **Version** | **Date**      | **Author**       | **Summary**                                                                                                                                                                                                                                                                                                                                                                          |
| ----------- | ------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.0         | June 3, 2026  | Christina Zikeli | Initial draft of Debit Card Onboarding PRD                                                                                                                                                                                                                                                                                                                                           |
| 2.0         | July 16, 2026 | Christina Zikeli | Migrated content into the standard PRD Template (V3) and PRD Author Guide structure; introduced DBT01 Feature ID scheme; replaced "virtual card" terminology with Spidr's "Digital-First Debit Card"; removed the account-funding assumption; added Spidr Get Card Info, Get Card Image URL, and Activate Card APIs; added physical card activation flow; expanded Out of Scope list |

**Table of Contents**

[**1\. Overview** 3](#_Toc235075771)

[**Summary** 3](#_Toc235075772)

[**In Scope** 3](#_Toc235075773)

[**Out of Scope** 3](#_Toc235075774)

[**2\. Use Case** 3](#_Toc235075775)

[**Key Takeaways** 4](#_Toc235075776)

[**3\. Assumptions** 4](#_Toc235075777)

[**4\. UI Screen Inventory** 4](#_Toc235075778)

[**5\. User Flow** 5](#_Toc235075779)

[**Primary Flow** 5](#_Toc235075780)

[**Alternate Flows** 6](#_Toc235075781)

[**6\. Functional Requirements** 6](#_Toc235075782)

[**Feature Area: DBT01-FEA01 - Digital-First Debit Card Display** 6](#_Toc235075783)

[**Feature Area: DBT01-FEA02 - PIN Setup** 7](#_Toc235075784)

[**Feature Area: DBT01-FEA03 - Physical Card Request** 7](#_Toc235075785)

[**Feature Area: DBT01-FEA04 - Physical Card Activation** 8](#_Toc235075786)

[**7\. Non-Functional Requirements** 8](#_Toc235075787)

[**8\. Data Model / Versioning** 9](#_Toc235075788)

[**9\. Mobile App Navigation** 9](#_Toc235075789)

[**Navigation Rules** 9](#_Toc235075790)

[**Navigation Map** 10](#_Toc235075791)

[**10\. API Reference** 10](#_Toc235075792)

[**11\. Error States** 11](#_Toc235075793)

[**12\. Open Questions** 11](#_Toc235075794)

[**13\. Appendix** 12](#_Toc235075795)

[**Glossary** 12](#_Toc235075796)

[**References** 12](#_Toc235075797)

# **1\. Overview**

## **Summary**

Vanteo Connect is a mobile app that serves foreign nationals in the United States on employment or student visas. Many of these users arrive without a U.S. credit history or an established banking relationship. Upon opening a Vanteo bank account, the user is automatically issued a Visa-branded Digital-First Debit Card through Vanteo's card processing partner, Spidr - available immediately regardless of whether the account has been funded. This feature covers displaying that Digital-First Debit Card in-app, requesting a physical card, setting a PIN, and activating the physical card once it arrives.

## **In Scope**

| **Area**                         | **Description**                                                                                                                           |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Display Digital-First Debit Card | App displays the Visa Digital-First Debit Card image and status, available immediately upon account opening - no account funding required |
| Request physical debit card      | User can submit a request to mail a physical debit card to a U.S. address                                                                 |
| Update home address              | User can confirm the address on file or enter a new U.S. mailing address as part of the physical card request                             |
| Set debit card PIN               | User can set a 4-digit PIN for the physical card without leaving the app (initial set only)                                               |
| Activate physical debit card     | Once the physical card is delivered, user activates it in-app by confirming the card's printed expiration date                            |

## **Out of Scope**

| **Area**                                                  | **Description**                                                 |
| --------------------------------------------------------- | --------------------------------------------------------------- |
| Freeze a card                                             | Deferred to a future release                                    |
| Replace card                                              | Deferred to a future release                                    |
| Reissue card                                              | Deferred to a future release                                    |
| Change PIN                                                | Deferred to a future release (only initial PIN set is in scope) |
| Reset PIN                                                 | Deferred to a future release                                    |
| Provision to mobile wallet (e.g., Apple Pay / Google Pay) | Deferred to a future release                                    |
| International travel notifications/settings               | Deferred to a future release                                    |
| Dispute a transaction                                     | Deferred to a future release                                    |

# **2\. Use Case**

Give the user a Visa debit card they can see and start planning around the moment their account is opened - even before it's funded - and let them request a physical card, set a PIN, and activate that physical card entirely within the app, without a branch visit or a separate Spidr portal.

## **Key Takeaways**

| **Insight**                                                                                                   | **Product Impact**                                                                                            |
| ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| The Digital-First Debit Card is issued automatically at account opening and does not require a funded account | UI must never gate card display behind an account balance or funding check                                    |
| PIN data is highly sensitive and subject to PCI DSS                                                           | PIN must never be stored or logged by Vanteo; it is transmitted directly to Spidr                             |
| Card art must be shown per Spidr's display guidelines rather than rendering a raw card number client-side     | App uses Spidr's hosted card image (displayUrl) instead of building custom card-number UI, reducing PCI scope |
| A mailed physical card arrives inactive and must be explicitly activated                                      | App must support an in-app activation step using the Activate Card API                                        |
| Users may not have a permanent U.S. address yet                                                               | Address on file must be editable in-flow, with U.S.-only validation, before physical card request             |

# **3\. Assumptions**

- Primary user: a foreign national on an employment or student visa who has opened a Vanteo bank account (funding is not required)
- Secondary user: Compliance officer - needs assurance that PIN capture and card display meet PCI DSS requirements
- Business assumption: the bank account has been opened; the Digital-First Debit Card is issued automatically at account opening regardless of account balance
- Card assumption: the card is a Visa-branded Digital-First Debit Card issued and processed by Spidr; the app must display the card image per Spidr's display guidelines
- Technical assumption: Spidr's Get Card Info, Get Card Image URL, and Activate Card APIs are available and contracted for this program, including a configured card image configId
- Dependencies: Spidr Get Card Info API, Get Card Image URL API, Activate Card API, PIN Set API, physical card fulfillment API, and card status webhook must be available before build

# **4\. UI Screen Inventory**

| **UI ID**  | **Screen Name**            | **Purpose**                                                                                                         | **Entry Point**                                                  |
| ---------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| DBT01-UI01 | Bank Card Home             | Entry point showing card art, account summary, and access to View/Order/Settings actions                            | Home screen Account Balance card icon, or "View Debit Card" tile |
| DBT01-UI02 | Digital-First Card Display | Displays the Visa Digital-First Debit Card image (via Spidr displayUrl) and card status/details (via Get Card Info) | "View card" icon on Bank Card Home                               |
| DBT01-UI03 | Order a Physical Card      | Collects/confirms mailing address for physical card request                                                         | "Order a card" CTA on Bank Card Home                             |
| DBT01-UI04 | PIN Entry                  | Custom numeric keypad for initial 4-digit PIN entry                                                                 | Auto-advances after physical card order is submitted             |
| DBT01-UI05 | PIN Confirm                | Re-entry of the 4-digit PIN to confirm match                                                                        | Advances from PIN Entry                                          |
| DBT01-UI06 | Review Your Order          | Confirms name, address, and estimated delivery window before submission                                             | Advances from PIN Confirm                                        |
| DBT01-UI07 | Order Confirmation         | Success screen with order reference and estimated delivery date range                                               | Advances after order submission                                  |
| DBT01-UI08 | Physical Card Activation   | Prompts user to enter the physical card's printed expiration date to activate it                                    | Triggered when card status webhook reports "Delivered"           |

# **5\. User Flow**

## **Primary Flow**

| **Step** | **User Action**                                                                                                 | **System Response**                                                                                                                                                                |
| -------- | --------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1        | User taps the card icon on the home screen Account Balance card, or the "View Debit Card" tile                  | App navigates to Bank Card Home                                                                                                                                                    |
| 2        | User taps "View card"                                                                                           | App calls Spidr Get Card Info (card status/details) and Get Card Image URL (displayUrl); the Visa Digital-First Debit Card image renders immediately - no account funding required |
| 3        | User taps "Order a card"                                                                                        | App displays the Order a Physical Card screen with the address on file pre-populated                                                                                               |
| 4        | User confirms the existing address or enters a new U.S. address (Street 1, Street 2 optional, City, State, ZIP) | App validates the address is a U.S. address                                                                                                                                        |
| 5        | User submits the order                                                                                          | Flow auto-advances to PIN setup                                                                                                                                                    |
| 6        | User enters a 4-digit PIN on the custom numeric keypad                                                          | PIN is masked with dots; screen advances to "Confirm your PIN"                                                                                                                     |
| 7        | User re-enters the same 4 digits                                                                                | If PINs match: success toast, flow advances to Review Your Order. If PINs do not match: inline error, both fields clear, user retries                                              |
| 8        | User reviews name, address, and estimated delivery window and confirms                                          | App submits the order to Spidr and displays the Order Confirmation screen with order reference and estimated delivery date range                                                   |
| 9        | Card status webhook reports "Delivered"                                                                         | App surfaces a prompt on Bank Card Home to activate the physical card                                                                                                              |
| 10       | User taps the activation prompt and enters the card's printed expiration date (MM/YY)                           | App calls Spidr Activate Card with the entered expiry date; on success, card status updates to "Active"                                                                            |

## **Alternate Flows**

- Back navigation during PIN entry: user is warned that PIN setup is incomplete; any entered PIN data is cleared on back navigation.
- Retry on card info/image load failure: inline error with a retry option and a support contact link is shown if Get Card Info or Get Card Image URL fails.
- Connectivity loss mid-flow: an offline error is shown; the flow resumes from the last completed step once connectivity is restored, with no partial state saved.
- Repeated PIN mismatch: after the 3rd consecutive mismatch, the session is invalidated and the user is prompted to restart from PIN Entry with a clear explanatory message.
- Cancel/timeout: no partial physical-card order or PIN state is persisted if the user exits the flow before final submission.
- Activation attempted before card is emboss-ready: if the card's emboss status is not yet sent_to_emboss, the app blocks activation and shows a "not ready yet" message rather than calling the Activate Card API.
- Activation expiry-date mismatch: if the entered expiration date does not match Spidr's records, show an inline error and allow retry.

# **6\. Functional Requirements**

## **Feature Area: DBT01-FEA01 - Digital-First Debit Card Display**

| **Requirement ID** | **Requirement**                                                                                                                                                               | **Acceptance Criteria**                                                                                                           | **Priority** |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| DBT01-FR01         | System must call Spidr Get Card Info (GET /v1/card/{cardId}) to retrieve card status and emboss details                                                                       | Card status and details render on the Digital-First Card Display screen; card is shown whether or not the account is funded       | High         |
| DBT01-FR02         | System must call Spidr Get Card Image URL (GET /v1/card/{cardId}/displayUrl) and render the returned image as the Visa Digital-First Debit Card, per Spidr display guidelines | Card image loads using the program's configured configId; app never constructs its own card-number UI in place of the Spidr image | High         |
| DBT01-FR03         | If Get Card Info or Get Card Image URL fails, or the returned image URL expires, the app must surface an inline error and allow retry                                         | Error state matches DBT01-ERR01; retrying re-issues both API calls                                                                | Medium       |

## **Feature Area: DBT01-FEA02 - PIN Setup**

| **Requirement ID** | **Requirement**                                                                                | **Acceptance Criteria**                                                                                | **Priority** |
| ------------------ | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ------------ |
| DBT01-FR04         | PIN must be exactly 4 numeric digits, entered on a custom in-app keypad (no system keyboard)   | Entry is rejected/blocked for non-numeric input or fewer than 4 digits                                 | High         |
| DBT01-FR05         | PIN must be confirmed with a second entry                                                      | Mismatch clears both fields and surfaces an inline error; match advances the flow with a success toast | High         |
| DBT01-FR06         | PIN transmission to Spidr must use end-to-end encryption; Vanteo must not log or store the PIN | Security review confirms no PIN value appears in logs, analytics events, or persistent storage         | High         |
| DBT01-FR07         | This flow supports initial PIN set only; change PIN and reset PIN are explicitly out of scope  | No UI entry point exists for changing or resetting an existing PIN                                     | High         |

## **Feature Area: DBT01-FEA03 - Physical Card Request**

| **Requirement ID** | **Requirement**                                                                                                             | **Acceptance Criteria**                                                                                                      | **Priority** |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------ |
| DBT01-FR08         | Mailing address must be pre-populated from account data                                                                     | Address on file is displayed by default when the screen loads                                                                | High         |
| DBT01-FR09         | User may enter a new U.S. address if needed; the new address is stored to the user's profile                                | New address passes U.S.-address validation and is persisted to the profile on submission                                     | High         |
| DBT01-FR10         | Submission must call Spidr's physical card fulfillment API to enqueue card production and mailing                           | API call succeeds and returns an order reference; UI disables the submit CTA after first tap to prevent duplicate submission | High         |
| DBT01-FR11         | System must display an order reference number and estimated delivery window upon success                                    | Confirmation screen shows both values                                                                                        | High         |
| DBT01-FR12         | Card status (Processing, Mailed, Delivered, Active) must be surfaced in the card section and updated via webhook from Spidr | Status shown reflects the most recent webhook event within an agreed latency                                                 | Medium       |

## **Feature Area: DBT01-FEA04 - Physical Card Activation**

| **Requirement ID** | **Requirement**                                                                                                                        | **Acceptance Criteria**                                                                                                    | **Priority** |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------ |
| DBT01-FR13         | Once card status is "Delivered," app must prompt the user to activate the physical card                                                | Activation prompt appears on Bank Card Home after the "Delivered" webhook event                                            | High         |
| DBT01-FR14         | User must enter the card's printed expiration date to activate                                                                         | App calls Spidr Activate Card (POST /v1/card/{cardId}/activate) with cardExpiryDate in YYYY-MM format                      | High         |
| DBT01-FR15         | Activation must only be attempted when the card's emboss status is sent_to_emboss                                                      | App checks emboss status before enabling the activation CTA; otherwise shows a "not ready yet" message                     | High         |
| DBT01-FR16         | On successful activation, card status must update to "Active" and the activation prompt must clear                                     | Bank Card Home reflects "Active" status immediately after a successful API response                                        | High         |
| DBT01-FR17         | This endpoint must not be called for the initial Digital-First card or for Digital-First replacement cards, per Spidr's usage guidance | Activation flow is only reachable from a "Delivered" physical-card state, never from the Digital-First Card Display screen | High         |

# **7\. Non-Functional Requirements**

| **Requirement ID** | **Requirement**                                                                                                                                                       | **Acceptance Criteria**                                                                                                            |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| DBT01-NFR01        | Performance: Get Card Info and Get Card Image URL response ≤ 3s P99; Activate Card and PIN Set response ≤ 5s P99                                                      | Load/perf testing confirms P99 thresholds are met                                                                                  |
| DBT01-NFR02        | Security: PCI DSS compliance for PIN handling                                                                                                                         | PIN is captured on-device and transmitted via Spidr's tokenization/SDK approach; Vanteo servers never receive or log the PIN value |
| DBT01-NFR03        | Security: card number is masked by Spidr unless the program is PCI-compliant; the app relies on Spidr's hosted card image for display rather than rendering a raw PAN | No raw, unmasked PAN or CVV is written to logs, analytics events, or local storage                                                 |
| DBT01-NFR04        | Brand compliance: card art and activation experience must meet Visa network program requirements and Spidr's display guidelines                                       | Design review sign-off from Compliance and Spidr program requirements                                                              |
| DBT01-NFR05        | Session security: sensitive card actions (view, activate) may require Spidr ZTM session checks per program configuration                                              | sessionKey / bypassZtm handling confirmed with Spidr for this program (see Open Questions)                                         |
| DBT01-NFR06        | Accessibility: WCAG 2.1 AA                                                                                                                                            | Custom numpad supports VoiceOver / TalkBack; audit confirms AA conformance                                                         |
| DBT01-NFR07        | Localization: English (en-US) at launch                                                                                                                               | Architecture supports adding locales without rework                                                                                |
| DBT01-NFR08        | Offline / connectivity: graceful degradation                                                                                                                          | Appropriate error is shown if network is unavailable; no partial state is saved                                                    |
| DBT01-NFR09        | Analytics: step completion and drop-off instrumentation                                                                                                               | All step completions and drop-offs, including activation, are instrumented with Segment events                                     |
| DBT01-NFR10        | Regulatory: physical card issuance and mailing comply with Reg E and CARD Act obligations                                                                             | Spidr attests to compliant card delivery under its Reg E / CARD Act obligations                                                    |
| DBT01-NFR11        | Data residency: card-related API traffic routes through U.S.-based infrastructure                                                                                     | Architecture review confirms routing meets Spidr's data residency requirements                                                     |

# **8\. Data Model / Versioning**

| **Field / Object**                  | **Current State**                           | **New State**                                                                                             | **Notes**                                                                                     |
| ----------------------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Mailing address                     | Stored on user profile from account opening | Editable and re-stored to profile when user submits a new address during physical card request            | Open question: confirm new addresses are stored with a timestamp/history (see Section 12, Q1) |
| Card status                         | Not tracked in-app                          | New field: Processing / Mailed / Delivered / Active, updated via Spidr webhook                            | Surfaced in Bank Card Home                                                                    |
| Card image URL                      | N/A                                         | New, ephemeral: time-limited Spidr-hosted displayUrl, not persisted                                       | Re-fetched each time the Digital-First Card Display screen loads                              |
| Card expiry date (activation input) | N/A                                         | New, transient: entered by user during physical card activation, sent to Spidr, not stored by Vanteo      | Used only in the Activate Card request                                                        |
| PIN                                 | N/A                                         | Not stored - transmitted directly to Spidr via encrypted channel; no PIN field persists in Vanteo systems | PCI DSS scope boundary                                                                        |

# **9\. Mobile App Navigation**

## **Navigation Rules**

- Entry points: home screen Account Balance card icon, or the "View Debit Card" tile - both route to Bank Card Home.
- Back navigation during PIN Entry or PIN Confirm clears any entered PIN data and warns the user that setup is incomplete.
- Cancel/back before final order submission discards the in-progress physical card order; no partial state is saved.
- The Physical Card Activation prompt is only reachable from Bank Card Home once card status is "Delivered."
- Exit points: Order Confirmation's "Go to my account" CTA, and the Activation success state, both return the user to Bank Card Home / account home.

## **Navigation Map**

_Home → Bank Card Home → \[View Card → Digital-First Card Display\] or \[Order a Card → Order a Physical Card → PIN Entry → PIN Confirm → Review Your Order → Order Confirmation\] → (on "Delivered" webhook) → Physical Card Activation → Bank Card Home (Active)_

# **10\. API Reference**

| **API ID**  | **API Name**                  | **Method**     | **Endpoint**                  | **Related FR ID** | **Notes**                                                                                                                                                                              |
| ----------- | ----------------------------- | -------------- | ----------------------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DBT01-API01 | Get Card Info                 | GET            | /v1/card/{cardId}             | DBT01-FR01        | Returns card status and emboss records; card number masked unless program is PCI-compliant; expiry generated during the daily emboss process for new cards                             |
| DBT01-API02 | Get Card Image URL            | GET            | /v1/card/{cardId}/displayUrl  | DBT01-FR02        | Returns a time-limited Spidr-hosted card image URL; requires a Spidr-configured configId for this program's card design                                                                |
| DBT01-API03 | Activate Card                 | POST           | /v1/card/{cardId}/activate    | DBT01-FR14-FR17   | Only valid for Physical-Only cards and Digital-First reissued cards, and only when emboss status is sent_to_emboss; must not be used for the initial or replacement Digital-First card |
| DBT01-API04 | PIN Set API                   | TBD            | Spidr endpoint TBD            | DBT01-FR04-FR06   | Exact Spidr endpoint to be confirmed (see Open Questions)                                                                                                                              |
| DBT01-API05 | Physical Card Fulfillment API | TBD            | Spidr endpoint TBD            | DBT01-FR08-FR10   | Enqueues physical card production and mailing; exact Spidr endpoint to be confirmed                                                                                                    |
| DBT01-API06 | Card Status Webhook           | POST (inbound) | Vanteo receiving endpoint TBD | DBT01-FR12        | Delivers real-time status updates (Processing, Mailed, Delivered, Active)                                                                                                              |
| DBT01-API07 | Email Notification Service    | POST           | Internal service endpoint TBD | DBT01-FR01        | Notifies user of card status changes, including when the physical card is ready to activate                                                                                            |

**Base URL (sandbox):** <https://api.sb.spidrsrv.com>

**Reference docs:** [Get Card Info](https://docs.gospidr.com/reference/getv1cardcardid) • [Get Card Image URL](https://docs.gospidr.com/reference/getv1cardcardiddisplayurl) • [Activate Card](https://docs.gospidr.com/reference/postv1cardcardidactivate)

# **11\. Error States**

| **Error ID** | **Scenario**                                                | **User Message**                                | **Recovery Action**                                                           |
| ------------ | ----------------------------------------------------------- | ----------------------------------------------- | ----------------------------------------------------------------------------- |
| DBT01-ERR01  | Get Card Info or Get Card Image URL call fails              | Inline error banner                             | Allow retry up to 3 times; offer support contact link after the 3rd failure   |
| DBT01-ERR02  | User loses connectivity mid-flow                            | Offline error message                           | Resume from the last completed step once connectivity restores                |
| DBT01-ERR03  | PIN confirmation mismatch (3rd consecutive attempt)         | Session invalidated message                     | Prompt user to restart the flow from PIN Entry                                |
| DBT01-ERR04  | Physical card fulfillment API failure after PIN success     | Inline error                                    | Allow retry without requiring PIN re-entry; Digital-First card remains usable |
| DBT01-ERR05  | User navigates back during PIN entry                        | Warning that PIN setup is incomplete            | PIN data cleared on back navigation                                           |
| DBT01-ERR06  | Duplicate card request (user submits twice)                 | N/A - prevented at UI layer                     | Spidr API must be idempotent; CTA disabled after first tap                    |
| DBT01-ERR07  | Mailing address fails U.S. address validation               | Address validation error                        | User corrects the address fields inline before resubmitting                   |
| DBT01-ERR08  | Activation attempted before emboss status is sent_to_emboss | "Your card isn't ready to activate yet" message | Activation CTA disabled; user is notified via webhook/notification once ready |
| DBT01-ERR09  | Entered expiration date does not match Spidr's record       | Inline "expiration date doesn't match" error    | User retries entry; escalate to support after repeated failures               |

# **12\. Open Questions**

| **#** | **Question**                                                                                                                                               | **Owner**                | **Status** |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ | ---------- |
| 1     | Confirm new addresses are stored with a timestamp/history in the backend                                                                                   | Engineering              | Open       |
| 2     | Should the address be checked/validated against standardized mailing address data (e.g., USPS Address Validation API) to validate and standardize entries? | Engineering              | Open       |
| 3     | What is Spidr's exact PIN Set endpoint, and does it require a client-side SDK or a standard REST call with client-side encryption?                         | Engineering              | Open       |
| 4     | Can push notifications be enabled for card status updates (Processing, Mailed, Delivered, Active)?                                                         | Engineering              | Open       |
| 5     | What configId should be used for this program's Digital-First Debit Card image, and who owns configuring it with Spidr?                                    | Product / Engineering    | Open       |
| 6     | Is our program configured for Spidr ZTM (sessionKey / bypassZtm)? If so, which actions require a ZTM session?                                              | Engineering / Compliance | Open       |
| 7     | Does Get Card Info return an unmasked PAN/CVV for our program, or is card number display fully replaced by the Spidr-hosted card image?                    | Engineering / Compliance | Open       |

# **13\. Appendix**

## **Glossary**

| **Term**                      | **Definition**                                                                                                                                                   |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Digital-First Debit Card      | A Visa-branded debit card issued automatically at account opening, available for use before a physical card arrives, and displayed via Spidr's hosted card image |
| Spidr (referred to as "SPDR") | Vanteo's card processing partner; issues and manages debit cards, PINs, and card status on behalf of Vanteo                                                      |
| Emboss / emboss status        | The daily process by which Spidr generates physical card details (e.g., expiry); sent_to_emboss indicates a physical card is ready to be activated               |
| PIN                           | Personal Identification Number - a 4-digit code used to authenticate card-present transactions at ATMs and chip readers                                          |
| PCI DSS                       | Payment Card Industry Data Security Standard - the security standard governing card data handling                                                                |
| ZTM                           | Spidr's session-based security check (sessionKey / bypassZtm); applicability to this program to be confirmed (see Open Questions)                                |
| Webhook                       | An HTTP callback sent by Spidr to Vanteo's backend when a card status event occurs                                                                               |

## **References**

- Design links: TBD
  - API documentation: [Get Card Info](https://docs.gospidr.com/reference/getv1cardcardid), [Get Card Image URL](https://docs.gospidr.com/reference/getv1cardcardiddisplayurl), [Activate Card](https://docs.gospidr.com/reference/postv1cardcardidactivate)
- Supporting references: v1.0 Debit Card Onboarding PRD (June 3, 2026)
