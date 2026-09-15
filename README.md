# Vehicle Transfer Feature

A feature built for the Embifi driver dashboard that enables clean transfer of a financed vehicle from one driver to another.

## What it does

When a driver defaults or exits, this feature allows an Embifi operations user to:
1. Select the existing driver's order
2. Enter the new driver's user ID and transfer fee
3. Trigger a full transfer — closing the old order and creating a fresh one for the new driver

## Why it was built

Previously, vehicle transfers were handled manually with no structured process. This feature automates the full workflow, ensures financial accuracy (fresh EMI, zero DPD for the new driver), and maintains a clean audit trail between the two orders.

## Files Changed

```
driver-dashboard/
└── src/
    └── app/
        └── orders/
            └── [orderId]/
                ├── page.tsx                  # Modified — added Transfer tab
                └── transfer/
                    └── transfer.tsx          # New file — Transfer form component
```

## Key Concepts

- **DPD (Days Past Due):** Tracks how many days late a driver is on EMI. New driver starts at 0.
- **TransferredToOrderID:** Field on Driver A's closed order, pointing to Driver B's new order.
- **TransferredFromOrderID:** Field on Driver B's new order, pointing back to Driver A's closed order.
- **History Isolation:** Driver A's payment history is fully preserved. Driver B starts with a clean slate.

## Branch

`feat/vehicle-transfer` on AWS CodeCommit

## Built by

Drishya Raj — Cloud & AI Automation Engineer, Embifi
