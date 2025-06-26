# 🧾 POS App Schema Summary

A flexible multi-tenant POS schema suitable for:
- ☕ Coffee shops  
- 🍔 Food shops  
- 🍽️ Restaurants  

Supports products, ingredients, inventory tracking, orders, kitchen routing, and SaaS tenancy.


## 🏢 Tenancy

| Table                  | Purpose                                      |
|------------------------|----------------------------------------------|
| `tenants`              | Each business/storefront                     |
| `branches`             | Physical locations of each tenant            |
| `users`                | Staff/admins (linked to tenants)             |
| `roles` / `user_roles` | Role-based access control                    |
| `plans`                | Subscription plans (pricing tiers)           |
| `tenant_subscriptions` | Tracks each tenant's active plan & billing   |


## 🛒 Products / Menu Items

| Table                  | Purpose                                               |
|------------------------|-------------------------------------------------------|
| `products`             | Retail or prepared menu items                         |
| `product_categories`   | Group products (e.g., Beverages, Food)                |
| `product_modifiers`    | Available customizations per product                  |
| `modifiers`            | Global add-ons or options (e.g., "No ice")            |
| `product_ingredients`  | Recipes linking products to ingredients               |


## 📦 Inventory

| Table               | Purpose                                                 |
|---------------------|---------------------------------------------------------|
| `ingredients`        | Raw materials (e.g., sugar, coffee beans, beef)         |
| `inventory_movements`| Track all stock changes (retail + ingredients)         |
| `ingredient_stock`   | Optional: summarized stock per branch per ingredient   |
| `product_stock`      | Optional: summarized stock per branch per product      |


## 🍽️ Orders & Kitchen

| Table                  | Purpose                                             |
|------------------------|-----------------------------------------------------|
| `orders`               | Parent order (table service, takeaway, delivery)    |
| `order_items`          | Line items in an order                              |
| `order_item_modifiers` | Track item-level customizations                     |
| `kitchen_stations`     | Define kitchen areas (e.g., Grill, Barista)         |


## 💰 Sales & Payments

| Table       | Purpose                                            |
|-------------|----------------------------------------------------|
| `sales`     | Finalized sale transactions (invoice-level)        |
| `payments`  | Payment records (cash, card, QR, split payments)   |
| `refunds`   | Refund history linked to sales or order items      |


## 🧑‍💼 Customers (Optional)

| Table        | Purpose                                           |
|--------------|---------------------------------------------------|
| `customers`  | Track customers for loyalty, history, and delivery|


## ✅ Key Features Supported

- Multi-tenant SaaS
- Inventory tracking (products + ingredients)
- Modifiers and recipe management
- Kitchen printing or KDS support
- Role-based access control
- Flexible POS flows (dine-in, takeaway, delivery)
- Sales and payment tracking
- Optional customer and loyalty handling


---


# 🧭 Tenancy Flow

This flow outlines how a business owner (tenant admin) signs up, selects a plan, configures the system, and starts using the POS.


## 1. Sign Up
- A new user (usually the owner/admin) signs up.
- System creates:
  - A new `users` record.
  - A new `tenants` record linked to that user.


## 2. Select Plan
- The user selects a subscription plan (e.g. Free Trial, Basic, Pro).
- System creates:
  - A `tenant_subscriptions` record.
  - If payment is needed, `payment_status = 'pending'` until confirmed.


## 3. Create Branch
- The tenant sets up one or more `branches` (e.g. "Main Café", "Downtown Location").


## 4. Set Up Products and Ingredients
- Add menu items (`products`) and raw materials (`ingredients`).
- Define categories, recipes, and modifiers as needed.


## 5. Create Roles & Set Access Control
- The tenant admin creates custom roles (e.g., Manager, Cashier, Kitchen Staff).
- System stores:
  - Roles in `roles`.
  - Permissions in `role_permissions` or equivalent table.


## 6. Add Staff & Assign Roles
- The tenant invites or registers staff accounts.
- Staff are linked to:
  - `users` with the same `tenant_id`.
  - Assigned roles via `user_roles`.


## 7. Start Using POS
- Staff can now use the system based on their permissions.
- Orders (`orders`, `order_items`) are placed.
- Inventory is tracked via `inventory_movements`.


## 8. Manage Subscription
- The tenant can:
  - View usage limits.
  - Upgrade or cancel their subscription.
  - Renew payments or handle failures.
- Subscription status and plan features are enforced through `tenant_subscriptions`.


---


# 🛒 Products / Menu Items Flow

This flow describes how menu items and retail products are set up and managed in the POS system.


## 1. Create Product Categories
- Admin creates categories like:
  - "Beverages", "Food", "Snacks", "Desserts"
- Stored in `product_categories`
- Used for organizing menus and filtering in the POS UI


## 2. Create Products
- Add products/menu items:
  - Example: “Cappuccino”, “Grilled Chicken”, “Bottled Water”
- Stored in the `products` table
- Key fields:
  - `name`, `type` (`retail` or `prepared`)
  - `sale_price`, `category_id`, `kitchen_required`, `track_inventory`


## 3. (Optional) Define Modifiers
- Create global modifiers like:
  - "Extra shot", "No sugar", "Less ice", "Spicy"
- Stored in `modifiers`
- Linked to applicable products via `product_modifiers`


## 4. (Optional) Define Product Recipes
- For prepared items:
  - Link to raw ingredients (e.g. 10g coffee beans, 200ml milk)
- Stored in `product_ingredients`
- Enables auto-deducting ingredients during order fulfillment


## 5. Set Kitchen Routing (if needed)
- For prepared items that require kitchen printing/KDS:
  - Assign `kitchen_station_id` (e.g. Grill, Barista, Fryer)
- Enables backend to route tickets correctly


## 6. Track Product Stock (Optional)
- If item is stock-tracked (e.g., bottled drinks):
  - Enable `track_inventory = true`
  - System tracks stock via `inventory_movements`


## 7. Use in POS
- Products appear in POS menu by category
- Staff can select product, choose modifiers, and add to order
- When order is submitted:
  - Inventory is deducted
  - Kitchen station is notified if needed


---


# 📦 Inventory Flow

This flow describes how inventory is managed for both retail products and ingredients in the POS system.


## 1. Add Ingredients
- Admin adds raw materials like:
  - `Coffee beans`, `Milk`, `Sugar`, `Beef`, `Bread`
- Stored in the `ingredients` table
- Includes fields like:
  - `name`, `unit`, `cost_price`, `low_stock_threshold`


## 2. (Optional) Add Stocked Products
- For items that are sold directly (e.g., bottled water, snacks)
- Use `products` table with `track_inventory = true`


## 3. Add Initial Stock
- For each `branch`, initial stock is added via:
  - `inventory_movements` with `type = 'restock'` and `quantity > 0`
- Applies to both ingredients and stocked products


## 4. Track Stock Changes Automatically

Stock is adjusted automatically in these cases:

| Action         | What Happens                                                |
|----------------|-------------------------------------------------------------|
| **Sale**       | `inventory_movements` entry with `type = 'sale'` or `usage` |
| **Return**     | Quantity added back with `type = 'return'`                  |
| **Recipe Use** | Ingredients deducted per recipe on product sale             |
| **Adjustment** | Manual corrections using `type = 'adjustment'`              |

All changes are stored in the `inventory_movements` table.


## 5. Check Current Stock Levels
- Use a summary view or query on `inventory_movements` to calculate stock:
  ```sql
  SUM(quantity) per ingredient/product per branch
  ```
  - Optionally store in:
    - ingredient_stock for fast read access
    - product_stock for tracked product items

## 6. Receive New Stock (Restock)
- When new inventory arrives, staff adds it via:
  - Manual input
  - Linked purchase/order system (optional)

- Logged in inventory_movements with type = 'restock'

## 7. Low Stock Alerts
- System compares stock levels with low_stock_threshold from ingredients or products
- Sends alerts or shows warnings in admin dashboard

## 🧠 Notes
- All inventory activity (retail or ingredients) is tracked in one table: inventory_movements
- Separation is managed by product_id vs ingredient_id (only one is filled per row)


---


# 🍽️ Orders & Kitchen Flow

This flow describes how orders are handled in the POS system, including routing items to the kitchen and tracking preparation progress.


## 1. Create Order
- A cashier/waiter creates a new order via POS (dine-in, takeaway, delivery).
- Stored in the `orders` table.
  - Fields: `tenant_id`, `branch_id`, `table_id` (if dine-in), `status`, `customer_id` (optional)


## 2. Add Order Items
- Each selected product is stored in the `order_items` table.
  - Fields: `product_id`, `quantity`, `notes`, `kitchen_status`
- If a product has modifiers (e.g., "No sugar", "Extra cheese"):
  - Modifier selections are saved in `order_item_modifiers`.


## 3. Kitchen Routing (If Required)
- For items that require preparation:
  - Check `products.kitchen_required = true`
  - Each product is linked to a `kitchen_station_id` (e.g., Barista, Grill)
- System routes items to the appropriate kitchen station via:
  - Kitchen printer
  - Kitchen Display System (KDS)
  - Real-time kitchen dashboard


## 4. Track Kitchen Status
- `order_items.kitchen_status` tracks each item's preparation progress:
  - `pending` → `preparing` → `ready` → `served`
- Staff update statuses via kitchen UI or device


## 5. Update Inventory
- When an order is submitted:
  - If the product is stock-tracked → deduct from `product_stock`
  - If the product uses ingredients → deduct from `ingredient_stock` via `product_ingredients`
- All deductions are logged in `inventory_movements`


## 6. Serve or Complete Order
- Once all kitchen items are `ready` or `served`, cashier can:
  - Print receipt
  - Mark `orders.status = 'completed'`


## 7. Optional: Link to Payment
- Finalized orders may be linked to `sales` and `payments` tables
  - Useful for receipts, reporting, and reconciliation


## ✅ Key Tables

| Table                  | Purpose                                               |
|------------------------|-------------------------------------------------------|
| `orders`               | Parent record for the whole order                     |
| `order_items`          | Line items (products) in the order                    |
| `order_item_modifiers` | Customizations per item                               |
| `kitchen_stations`     | Routing logic for prepared items                      |
| `inventory_movements`  | Deducts ingredients or stock as orders are fulfilled  |



---



# 💰 Sales & Payments Flow

This flow describes how finalized sales and payments are managed in the POS system, including support for multiple payment methods and receipts.

---

## 1. Start from an Order
- A completed order (`orders`) is ready to be paid.
- Typically triggered by cashier after kitchen is done (or instantly for retail/takeaway).


## 2. Create a Sale Record
- When payment is initiated:
  - Create a new record in `sales`
    - Links to `order_id`
    - Stores `total_amount`, `discount`, `tax`, `final_amount`
    - Can include `invoice_number`, `status`, `sale_type` (e.g., POS, online)


## 3. Record Payment
- Payment details go into `payments` table.
  - Linked to `sale_id`
  - Can support **multiple payment methods** per sale:
    - Cash, card, QRIS, digital wallet, split payment
  - Key fields:
    - `method`, `amount`, `payment_status`, `paid_at`, `reference_no`


## 4. Mark Order as Paid
- Once payment(s) are complete:
  - Set `sales.status = 'paid'`
  - Optionally update `orders.status = 'completed'`


## 5. Print Receipt or Invoice
- System generates receipt or invoice (thermal or A4 PDF)
  - Shows ordered items, total, payments, cashier info, and taxes
  - Optionally linked to customer (`customer_id`)


## 6. Optional: Handle Refunds
- If a customer returns or cancels:
  - Create a `refunds` record:
    - Links to `sale_id` or `order_item_id`
    - Stores refund amount, reason, and refunded_by
  - Adjust inventory via `inventory_movements` (e.g., return)


## 7. Sales Reporting & Cashier Summary
- At the end of the shift/day:
  - Total up `sales` and `payments`
  - Reconcile by `cashier_id`, `branch_id`, or `payment_method`
  - Include totals, refunds, discounts, taxes, etc.


## ✅ Key Tables

| Table      | Purpose                                              |
|------------|------------------------------------------------------|
| `sales`    | Finalized sale transactions (invoice-level)          |
| `payments` | Payment records (cash, card, QR, split payments)     |
| `refunds`  | Refund history linked to sales or order items        |



---


# 🧑‍💼 Customers Flow

This flow describes how customers are managed in the POS system, supporting loyalty, delivery, and purchase history tracking.


## 1. Customer Is Added
- Customer can be added manually by staff or captured during a transaction.
- Stored in the `customers` table.
  - Fields: `name`, `phone`, `email`, `notes`, `created_at`
  - Optional fields: `address`, `birth_date`, `gender`, `loyalty_points`


## 2. Customer Makes a Purchase
- When placing an order, staff can select or create a customer.
- The order (`orders`) is linked via `customer_id`.


## 3. Track Purchase History
- System tracks all past orders and payments linked to a customer.
  - Useful for receipts, refunds, and loyalty programs.
- Display recent purchases in customer detail view.


## 4. Offer Loyalty / Rewards (Optional)
- Use `loyalty_points`, `visits`, or `total_spent` to drive:
  - Discounts
  - Cashback
  - VIP tiers
- Loyalty programs can be stored in the `customers` table or in a separate `loyalty_programs` module.


## 5. Use for Delivery Orders
- If enabled, staff can search/select a customer for delivery.
  - Use saved address and contact info from the `customers` record.


## 6. Search / Filter Customers
- Search customers by phone, name, or email during checkout.
- List by last purchase date, high spenders, or birthdays.


## ✅ Summary of Key Features

| Feature              | Description                                              |
|----------------------|----------------------------------------------------------|
| Create/edit customer | Capture during sale or via admin                         |
| Link to order        | Store `customer_id` on `orders`                          |
| View history         | Show past orders, spend, items                           |
| Loyalty program      | Points, tiers, or coupons (optional)                     |
| Delivery use         | Store addresses for reuse                                |
