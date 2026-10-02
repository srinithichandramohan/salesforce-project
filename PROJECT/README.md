# WhatNext Vision Motors

**Shaping the Future of Mobility with Innovation and Excellence**

A Salesforce CRM-based Vehicle Order and Dealership Management System built on the Salesforce Platform (Developer Edition).

---

## Table of Contents

1. [Overview](#overview)
2. [Objectives](#objectives)
3. [Features](#features)
4. [Technology Stack](#technology-stack)
5. [Architecture](#architecture)
6. [Data Model](#data-model)
7. [Project Structure](#project-structure)
8. [Setup and Installation](#setup-and-installation)
9. [Automation](#automation)
10. [Apex Code](#apex-code)
11. [Batch and Scheduled Jobs](#batch-and-scheduled-jobs)
12. [Reports and Dashboards](#reports-and-dashboards)
13. [Testing](#testing)
14. [Security](#security)
15. [Future Enhancements](#future-enhancements)
16. [Documentation](#documentation)

---

## Overview

WhatNext Vision Motors brings the whole vehicle sales cycle onto one cloud platform. Vehicles, dealers, customers, orders and test drives are stored as connected records. Flows and Apex handle order processing, and reports and dashboards give managers live visibility into sales and stock.

## Objectives

- Build a centralized data model for vehicles, dealers, customers and orders.
- Provide a Lightning app with a tab for each business object.
- Automate order handling with Flow.
- Block orders for out-of-stock vehicles using an Apex trigger.
- Confirm pending orders in bulk with a scheduled Batch Apex job.
- Provide reports and a dashboard for management.

## Features

| Area | Feature |
|---|---|
| Data management | Custom objects for Vehicle, Dealer, Customer, Vehicle Order and Test Drive |
| Navigation | Object tabs and a **Vision Motors** Lightning app |
| Automation | Screen flow (Create Vehicle Order) and record-triggered flow (Order Confirmation Update) |
| Business rules | Apex trigger validates stock and reduces it when an order is confirmed |
| Bulk processing | Batch Apex job confirms Pending orders, run daily by a scheduler |
| Analytics | Reports and the Vision Motors Sales Overview dashboard |

## Technology Stack

- **Platform:** Salesforce Platform (Developer Edition)
- **UI:** Lightning Experience, Lightning App Builder
- **Declarative automation:** Flow Builder
- **Programming:** Apex (triggers, classes, Batch Apex, Schedulable Apex)
- **Tooling:** Setup, Object Manager, Developer Console
- **Analytics:** Salesforce Reports and Dashboards

## Architecture

The solution has four layers:

1. **Users:** dealers, sales executives, inventory managers, management
2. **Presentation:** Lightning app, object tabs, screen flow
3. **Business logic:** record-triggered flow, Apex trigger and handler, batch class, scheduler
4. **Data:** custom objects connected by lookup relationships

Reports and dashboards read from the data layer. Profiles, roles, permission sets, field-level security and sharing rules apply across all layers.

## Data Model

### Objects

| Object | API Name | Purpose |
|---|---|---|
| Vehicle | `Vehicle__c` | Models the company sells |
| Dealer | `Dealer__c` | Dealerships supplying vehicles |
| Customer | `Vehicle_Customer__c` | Buyer details |
| Vehicle Order | `Vehicle_Order__c` | Orders placed for vehicles |
| Test Drive | `Test_Drive__c` | Test drive bookings |

### Key fields

- **Vehicle:** Vehicle Name, Model, Price (Currency), Stock Quantity (Number), Status (Available / Low Stock / Sold Out), Dealer (Lookup)
- **Dealer:** Dealer Name, Location, Phone, Email, Active (Checkbox)
- **Customer:** Customer Name, Email, Phone
- **Vehicle Order:** Order Date, Quantity, Order Status (Pending / Confirmed / Delivered / Cancelled), Customer (Lookup), Vehicle (Lookup)
- **Test Drive:** Drive Date, Customer (Lookup), Vehicle (Lookup)

### Relationships

```
Dealer ──1:N──► Vehicle
Customer ──1:N──► Vehicle Order ◄──N:1── Vehicle
Customer ──1:N──► Test Drive ◄──N:1── Vehicle
```

> Object and field API names above are the ones used in the project report. Adjust them if your org uses different names.

## Project Structure

```
WhatNext-Vision-Motors/
├── README.md
├── docs/
│   ├── WhatNext_Vision_Motors_Project_Report.docx
│   ├── Figure2_Solution_Architecture.png
│   └── Figure3_ER_Diagram.png
└── force-app/main/default/        (optional, if you export the metadata)
    ├── classes/
    │   ├── VehicleOrderTriggerHandler.cls
    │   ├── VehicleOrderBatch.cls
    │   └── VehicleOrderScheduler.cls
    ├── triggers/
    │   └── VehicleOrderTrigger.trigger
    ├── flows/
    ├── objects/
    └── applications/
```

## Setup and Installation

The project is built directly in a Salesforce Developer Edition org.

1. **Create a Developer org** at the Salesforce Developer Edition sign-up page.
2. **Verify the account** from the email Salesforce sends, then set a password.
3. **Create the objects** in Setup → Object Manager → Create → Custom Object: Vehicle, Dealer, Customer, Vehicle Order, Test Drive.
4. **Create the fields** on each object, including the picklists above.
5. **Create the relationship fields** as Lookup Relationships (see the relationships list).
6. **Create the tabs** in Setup → Tabs → Custom Object Tabs.
7. **Create the Lightning app** in Setup → App Manager → New Lightning App, named *Vision Motors*, and add all tabs plus Reports and Dashboards.
8. **Build the flows** in Setup → Flows (see Automation).
9. **Add the Apex classes and trigger** in the Developer Console (see Apex Code).
10. **Schedule the batch job** (see Batch and Scheduled Jobs).
11. **Create the reports and dashboard** from the Reports and Dashboards tabs.

## Automation

| Flow | Type | Trigger / Entry | Action |
|---|---|---|---|
| Create Vehicle Order | Screen Flow | Launched by the user | Collects Customer, Vehicle, Quantity and Order Date, then creates an order with status Pending |
| Order Confirmation Update | Record-Triggered Flow | Vehicle Order created or updated, Order Status = Confirmed | Updates the related Vehicle and sends an email alert to the customer |

## Apex Code

| Component | Type | Role |
|---|---|---|
| `VehicleOrderTrigger` | Trigger on `Vehicle_Order__c` | Before: validates stock. After: reduces stock |
| `VehicleOrderTriggerHandler` | Class | `validateStock()` blocks confirmation when stock is too low; `reduceStock()` reduces stock and sets Status to Sold Out when it reaches zero |

The code is bulkified: no queries or DML inside loops, with maps and sets used to stay within governor limits.

## Batch and Scheduled Jobs

| Component | Role |
|---|---|
| `VehicleOrderBatch` (Batchable) | Queries Pending orders and sets them to Confirmed; uses `Database.update(scope, false)` so one failure does not stop the batch |
| `VehicleOrderScheduler` (Schedulable) | Runs the batch with a scope size of 200 |

Schedule it from **Setup → Apex Classes → Schedule Apex**, or run this in the Developer Console:

```apex
String cron = '0 0 1 * * ?';   // every day at 1:00 AM
System.schedule('Vehicle Order Daily Job', cron, new VehicleOrderScheduler());
```

Run it once manually:

```apex
Database.executeBatch(new VehicleOrderBatch(), 200);
```

## Reports and Dashboards

| Report | Purpose |
|---|---|
| Orders by Status | Pending, Confirmed and Delivered counts |
| Vehicles by Dealer | Stock distribution across dealers |
| Low Stock Vehicles | Vehicles that are Low Stock or Sold Out |
| Orders by Customer | Order history per customer |

The **Vision Motors Sales Overview** dashboard combines these in a donut chart, a bar chart, a table and a total-orders metric.

## Testing

| Test | Expected result |
|---|---|
| Create a Vehicle with all fields | Saves successfully |
| Link a Vehicle to a Dealer | Lookup shows on the Vehicle |
| Create an order with the screen flow | Order created as Pending |
| Confirm an order with enough stock | Stock reduces; flow notifies the customer |
| Confirm an order with insufficient stock | Error blocks the save |
| Order reduces stock to zero | Vehicle status becomes Sold Out |
| Run the batch job | Pending orders are processed |
| Check Scheduled Jobs | Job is listed |

Apex test classes create their own data. Run them from **Setup → Apex Test Execution**; the target is at least 75 percent coverage.

## Security

- Access is controlled with profiles, roles and permission sets.
- Field-level security protects sensitive fields such as Price and Stock Quantity.
- Sharing rules control which records each user can see.

## Future Enhancements

- Customer self-service portal with Experience Cloud
- Approval process for high-value orders
- ERP and payment gateway integration
- Salesforce mobile app access
- Sales forecasting with Einstein analytics

## Documentation

The full project report, covering ideation, requirements, design, planning, development, testing, and reports, is in `docs/WhatNext_Vision_Motors_Project_Report.docx`.

## Author

- **Name:** _your name_
- **Institution:** _your institution_
- **Date:** _date_
