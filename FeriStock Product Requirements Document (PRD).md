# Product Requirements Document (PRD): FeriStock

- **Product Name:** FeriStock
- **Target Market:** Retail shops and wholesale suppliers / distributors in Nepal
- **Platform Type:** Responsive Web Application (PWA) for mobile shopkeepers and desktop/mobile wholesalers
- **Payment Architecture:** Non-custodial direct buyer-to-supplier settlement (eSewa, Khalti, QR/Bank transfer, and credit / *udharo* tracking) with automated order & payment receipts

---

## 1. Problem Statement & Vision

### 1.1 Problem
In Nepal's retail and wholesale supply chain, shop owners and distributors still coordinate restocks through scattered phone calls, WhatsApp/Viber chats, paper notebooks, and manual screenshots. This creates avoidable stockouts, misquoted bulk prices or schemes, unclear *bilti* or van delivery updates, manual *udharo* (credit) tracking, and time-consuming receipt reconciliation.

### 1.2 Vision
Give retail shops and wholesale suppliers a single, fast mobile-first workspace to track live inventory, compare supplier terms, trigger 1-click, scheduled, or custom orders, track dispatches, pay suppliers directly, and generate clean receipts automatically—without FeriStock holding customer funds.

---

## 2. Target Users & Roles

1. **Retailer (Shop Owner / Store Manager):**
   - Operates primarily on an Android smartphone during shop hours.
   - Needs fast stock updates, low-stock alerts, quick reorders, recurring restock schedules, custom quote requests, shipment tracking, direct payment options, and downloadable receipts.
2. **Supplier (Wholesaler / Distributor / Manufacturer):**
   - Operates on desktop or mobile to manage catalogs, bulk pricing tiers, active schemes, custom quotes, delivery/dispatch logs, and direct payment gateways.
   - Needs clear incoming orders, *bilti*/van dispatch updates, *udharo* due tracking, and verified payment records.
3. **Dual-Role Organization (Optional):**
   - A business that both buys stock as a retailer and supplies stock to other shops.

---

## 3. Core Business Rules

1. **Non-Custodial Direct Settlement:** FeriStock never holds buyer funds in a platform escrow account. Digital wallet payments route using the selected supplier's own connected merchant credentials (or display the supplier's QR/bank details with proof verification).
2. **Server-Side Payment Verification:** A digital wallet order transitions to `PAID` only after backend status/lookup verification confirms the transaction status and exact NPR amount; browser redirects alone are not trusted.
3. **Supplier-Owned Fulfillment:** Suppliers control how goods move—Own Fleet / Van, Transport / *Bilti*, or Pickup—including minimum order quantities (MOQs) and delivery notes.
4. **Automated Receipt Integrity:** Every paid or confirmed order generates a structured **Order & Payment Receipt** containing supplier business details and PAN/VAT, buyer business details and PAN/VAT, itemized SKUs, payment reference ID, and NPR totals.

---

## 4. Functional Requirements (MVP)

### 4.1 Authentication & Organization Onboarding
- Users sign up/log in and create or join an organization (`retailer`, `supplier`, or `both`) with business name, PAN/VAT number, city/address, and contact details.
- Suppliers configure wholesale catalog basics, delivery modes, credit terms, and payment methods (eSewa merchant credentials, Khalti merchant credentials, QR/bank details).

### 4.2 Retailer Inventory & Reorder Triggers
- Retailers view a live SKU list showing current quantity, reorder threshold, status (`OK`, `Low Stock`, `Out of Stock`), and preferred supplier.
- Retailers can increment/decrement (`+` / `-`) or edit stock counts directly and toggle low-stock alerts.
- **Three Order Modes:**
  1. **1-Click / Low-Stock Reorder:** Add low or selected SKUs into a supplier-grouped cart and submit a Purchase Order (PO).
  2. **Scheduled Recurring Order:** Configure repeat restock rules (e.g., weekly or monthly on a chosen day/time) that generate draft or automatic orders.
  3. **Custom Order / Quote Request:** Submit custom quantities, unlisted items, or bulk pricing requests for supplier review/counter-offer.

### 4.3 Supplier Catalog, Pricing & Schemes
- Suppliers create and edit products with SKU, unit, base wholesale price (NPR), volume/bulk pricing tiers, active schemes/offers, and MOQ.
- Suppliers can inspect incoming POs and custom quote requests, accept or adjust items, and move orders through fulfillment stages (`Pending` $\rightarrow$ `Accepted` $\rightarrow$ `Packed` $\rightarrow$ `Dispatched` $\rightarrow$ `Delivered`).

### 4.4 Shipment & Dispatch Tracking
- When marking an order `Dispatched`, suppliers record fulfillment method (Own Van, Transport *Bilti*, or Pickup), driver/courier details or *bilti* reference, and optional *bilti* photo upload.
- Retailers view live shipment status and tap `Confirm Delivery` upon arrival to add received quantities back into live shop stock.

### 4.5 Direct Payments, Credit (*Udharo*) & Receipts
- Retailers choose to pay immediately via the supplier's connected eSewa/Khalti checkout, submit QR/bank payment proof for supplier confirmation, or record/settle approved credit (*udharo*) balances by due date.
- The system generates a printable/downloadable **Order & Payment Receipt** linked to the order and payment reference.

---

## 5. Recommended Data Entities (PostgreSQL)

- `organizations`: `id`, `name`, `role`, `pan_vat_no`, `city`, `address`, `phone`
- `supplier_payment_configs`: `org_id`, `esewa_merchant_id`, `esewa_secret_enc`, `khalti_secret_enc`, `qr_image_url`, `bank_details_json`
- `products`: `id`, `supplier_org_id`, `sku`, `name`, `unit`, `price_paisa`, `bulk_tiers_json`, `scheme_note`, `moq`
- `retailer_inventory`: `id`, `retailer_org_id`, `product_id`, `current_qty`, `reorder_threshold`, `auto_reorder`, `preferred_supplier_id`
- `orders`: `id`, `order_no`, `retailer_org_id`, `supplier_org_id`, `order_type`, `status`, `total_paisa`, `payment_status`, `due_date`
- `shipments`: `id`, `order_id`, `delivery_mode`, `carrier_or_van`, `bilti_ref`, `bilti_photo_url`, `dispatched_at`, `delivered_at`
- `payment_records`: `id`, `order_id`, `method`, `transaction_ref`, `amount_paisa`, `status`, `receipt_url`, `paid_at`

---

## 6. Non-Functional Requirements

- **Mobile-First Responsiveness:** Core shop stock updates, order approvals, and payment flows must work cleanly on small Android screens as well as desktop browsers.
- **Security:** Encrypt sensitive supplier gateway credentials at rest and enforce strict role-based access so organizations only read/mutate their own records.
- **Auditability:** Preserve immutable payment references, order status timestamps, and generated receipt records for accounting and dispute clarity.

---

## 7. Phased Rollout & Success Metrics

### MVP Scope (Phase 1)
- Organization onboarding, supplier catalog + bulk tiers, retailer stock tracking + low-stock/custom orders, dispatch/*bilti* logging, direct eSewa/Khalti/QR payment flow, and PDF/printable receipt generation.

### Phase 2 Enhancements
- Automated recurring order execution jobs, multi-branch retailer stock sync, WhatsApp/SMS notifications, and supplier demand/scheme insights.

### Key Success Metrics
- **Activation:** % of onboarded shops and suppliers that complete at least one real order and receipt within 7 days.
- **Repeat Usage:** Weekly active shops updating stock or placing reorders and % of orders completed end-to-end (Ordered $\rightarrow$ Dispatched $\rightarrow$ Paid/Settled $\rightarrow$ Stock Synced).
- **Reliability:** 100% reconciliation between verified payment callbacks and generated receipts.