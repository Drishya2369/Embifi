# Vehicle Transfer Feature — Interview Prep Notes

## Part 1: Business Context

**Q: Why did we build the Vehicle Transfer feature?**
When a driver defaults or wants to exit, Embifi needs to move the vehicle to a new driver. Previously there was no structured way to do this — it was manual and messy. The feature automates the full transfer: closing the old driver's order cleanly and creating a fresh order for the new driver.

**Q: Why can't we just change the driver's name on the existing order?**
Because each driver goes through KYC (Know Your Customer) verification. The order is legally tied to a specific person's identity. You can't swap names — you need a new order for a new person.

**Q: What is DPD?**
DPD stands for Days Past Due. It tracks how many days late a driver is on their EMI payments. A driver with high DPD is a risk. After a transfer, Driver B starts with DPD = 0.

---

## Part 2: The 7-Step Backend Flow

Memory trick: **V C C C R N R** (Validate, Calculate, Close, Create, Record, Notify, Return)

| Step | Name | What happens |
|------|------|--------------|
| 1 | Validate | Check Driver B's user ID exists and is eligible |
| 2 | Calculate | Work out B's fresh EMI based on remaining loan amount and tenure |
| 3 | Close | Mark A's order as TRANSFERRED (not CLOSED — signals it was a transfer, not a default) |
| 4 | Create | Open a brand new order for B with zero DPD and empty payment history |
| 5 | Record | Write TransferredToOrderID on A's order and TransferredFromOrderID on B's order |
| 6 | Notify | Send notifications to relevant parties |
| 7 | Return | Return the new order details to the frontend |

**Q: Why TRANSFERRED and not CLOSED?**
CLOSED usually means the loan was fully paid off. TRANSFERRED means it ended because the vehicle moved to someone else. It's a different reason for closure and needs to be distinguishable in the system.

---

## Part 3: The 4 Frontend Code Changes

All 4 changes were made in `driver-dashboard/src/app/orders/[orderId]/page.tsx`

**Change 1 — Added `ArrowLeftRight` to lucide-react import**
lucide-react is the icon library the project uses. ArrowLeftRight (⇄) visually represents a transfer — two arrows facing each other. The Transfer tab needed an icon consistent with all other tabs.

**Change 2 — Added `import TransferTab from './transfer/transfer'`**
`TransferTab` is the new component built in `transfer/transfer.tsx`. It contains the full transfer form — Driver B's user ID field, transfer fee field, submit button, and success/error messages. `page.tsx` cannot see this component automatically; the import line introduces it and tells `page.tsx` where to find it.

**Change 3 — Added `<TabsTrigger value="transfer">`**
This is the clickable tab button that appears in the tab bar alongside Overview, Payments, and Documents. Without this, the Transfer tab has no entry point — the user would have no way to reach it.

**Change 4 — Added `<TabsContent value="transfer"><TransferTab orderId={orderId} /></TabsContent>`**
This is the content area that renders when the user clicks the Transfer tab. TabsTrigger = the switch, TabsContent = the light bulb. The `orderId={orderId}` passes the current order's ID into the TransferTab so the API call knows which order to transfer.

---

## Part 4: History Isolation

**Q: What happens to Driver A's payment history after the transfer?**
A's payment history stays fully preserved on their closed order. Nothing is deleted, nothing moves. Every payment A ever made is still visible. The order status changes to TRANSFERRED, and it has a `TransferredToOrderID` field pointing to B's new order. The history itself is completely untouched.

**Q: What does Driver B's payment history look like on day one after the transfer?**
B's history is completely empty. It's a fresh start — a clean chapter. Even though the vehicle came from A, nothing from A's history carries over. B's DPD is zero and their payment history has no entries.

**Q: If the histories are separate, how can Embifi trace that a transfer happened at all?**
That's exactly what the two linking fields are for. A's closed order has `TransferredToOrderID` pointing to B's new order. B's new order has `TransferredFromOrderID` pointing back to A. Anyone at Embifi can follow that chain — see the full transfer trail, who had the vehicle before, and when it was transferred. The histories are isolated but the link is always traceable.

---

## Key Fields to Remember

| Field | Lives on | Points to |
|-------|----------|-----------|
| `TransferredToOrderID` | Driver A's closed order | Driver B's new order |
| `TransferredFromOrderID` | Driver B's new order | Driver A's closed order |

---

## Files Changed

| File | What changed |
|------|-------------|
| `src/app/orders/[orderId]/transfer/transfer.tsx` | New file — the entire Transfer form component |
| `src/app/orders/[orderId]/page.tsx` | 4 changes — icon import, component import, tab trigger, tab content |

**Branch:** `feat/vehicle-transfer` on AWS CodeCommit
