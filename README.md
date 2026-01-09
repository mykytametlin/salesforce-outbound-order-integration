# Outbound Order Integration (Salesforce)

## Overview
This repository contains a Salesforce outbound integration implemented using Apex Triggers and asynchronous processing.

The solution reacts to order confirmation events in Salesforce and sends order data, including related line items, to an external REST API.  
The integration is designed to be **bulk-safe**, **asynchronous**, and **configuration-driven**.

---

## Business Case
When an order is confirmed in Salesforce, it must be sent to an external system such as an ERP, billing, or fulfillment service.

**Integration is triggered when:**
- an `Order__c` record is inserted with status `Confirmed`
- an existing `Order__c` record changes its status to `Confirmed`

Once triggered, Salesforce sends a structured JSON payload with order and item data to an external endpoint.

---

## Architecture Overview

- Order__c (Status = Confirmed)
- Apex Trigger
- Trigger Handler
- Integration Gate
- Custom Metadata
- Queueable Job
- Payload Builder
- REST API Callout

---

## Why Apex Triggers
This integration is driven by a **data change event** (order confirmation).

Apex Triggers were selected because:
- the logic must respond directly to DML events
- asynchronous HTTP callouts are required
- payload creation involves multiple related objects
- the solution must remain testable and maintainable

---

## Asynchronous Processing
All HTTP callouts are executed using `Queueable` to:
- comply with Salesforce governor limits
- avoid callouts inside triggers
- support bulk processing

---

## Configuration & Feature Toggle
Integration behavior is controlled via **Custom Metadata**.

The `Outbound_Integration_Setting__mdt` metadata type allows:
- enabling or disabling the integration without redeployment
- defining the endpoint via Named Credentials
- controlling retry and logging behavior

This approach separates configuration from code.

---

## Data Model

### Order__c
- `Order_Number__c` (Text, Unique)
- `Account__c` (Lookup → Account)
- `Status__c` (Draft / Confirmed / Cancelled)
- `Total_Amount__c` (Currency)
- `CurrencyIsoCode`
- `Confirmed_Date__c` (DateTime)

### Order_Item__c
- `Order__c` (Master-Detail)
- `Product_Code__c`
- `Quantity__c`
- `Unit_Price__c`
- `Line_Amount__c` (Formula)

---

## Payload Example

```json
{
  "orderNumber": "ORD-10045",
  "status": "Confirmed",
  "confirmedDate": "2026-01-05T12:30:00Z",
  "account": {
    "id": "001XXXXXXXXXXXX",
    "name": "Acme Corp"
  },
  "currency": "USD",
  "totalAmount": 1200.00,
  "items": [
    {
      "productCode": "SKU-01",
      "quantity": 2,
      "unitPrice": 300.00,
      "lineAmount": 600.00
    }
  ]
}
```

---

## Testing
- Trigger behavior is covered for insert and update scenarios
- HTTP callouts are mocked using HttpCalloutMock
- Both success and error responses are tested
- Bulk scenarios are validated to ensure governor-limit safety

## Key Takeaways
This project demonstrates:
- clean trigger architecture (Trigger → Handler → Service)
- asynchronous outbound integrations
- configuration-driven behavior using Custom Metadata
- separation of concerns and production-oriented Apex design
