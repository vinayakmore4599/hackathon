# Retail Sales Performance Data Model

## Overview
This document details a **Dimensional (Star Schema)** data model for retail sales performance analytics, designed for OLAP and reporting. The model follows data warehouse best practices with dimension and fact tables.

---

## Entity Relationship Diagram (Star Schema)

```
        ┌─────────────────────┐
        │     CUSTOMER DIM    │
        │    (Dimension)      │
        ├─────────────────────┤
        │ customer_id (PK)    │
        │ name                │
        │ email               │
        │ phone               │
        │ address             │
        │ city                │
        │ state               │
        │ postal_code         │
        │ country             │
        │ age_group           │
        │ gender              │
        │ customer_segment    │
        │ created_date        │
        └─────────────────────┘
                  ▲
                  │ FK
                  │
    ┌─────────────┴──────────────┐
    │                            │
    │    ┌──────────────────┐    │
    │    │   PRODUCT DIM    │    │
    │    │  (Dimension)     │    │
    │    ├──────────────────┤    │
    │    │ product_id (PK)  │    │
    │    │ sku              │    │
    │    │ name             │    │
    │    │ description      │    │
    │    │ category         │    │
    │    │ supplier_name    │    │
    │    │ supplier_contact │    │
    │    │ supplier_email   │    │
    │    │ cost_price       │    │
    │    │ selling_price    │    │
    │    │ storage_location │    │
    │    │ reorder_level    │    │
    │    │ status           │    │
    │    │ created_date     │    │
    │    └──────────────────┘    │
    │            ▲                │
    │            │ FK             │
    │            │                │
    │      ┌─────────────────┐    │
    │      │  SALES (FACT)   │◄───┘
    │      ├─────────────────┤
    │      │ sales_id (PK)   │
    └─────►│ customer_id (FK)│
           │ product_id (FK) │
           │ order_date      │
           │ delivery_date   │
           │ quantity        │
           │ unit_price      │
           │ discount_amount │
           │ tax_amount      │
           │ sales_total     │
           │ sales_channel   │
           │ order_status    │
           │ is_returned (flag)
           │ return_date     │
           │ return_reason   │
           │ return_quantity │
           │ return_amount   │
           │ created_at      │
           └─────────────────┘
                    │ FK
                    │
           ┌────────▼─────────┐
           │  PAYMENT (FACT)  │
           ├──────────────────┤
           │ payment_id (PK)  │
           │ sales_id (FK)    │
           │ payment_type     │
           │ amount           │
           │ card_brand       │
           │ payment_status   │
           │ transaction_date │
           │ is_refunded (flag)
           │ refund_amount    │
           │ refund_date      │
           │ refund_reason    │
           │ refund_status    │
           │ created_at       │
           └──────────────────┘
```

---

## Entity Definitions

### 1. CUSTOMER (Dimension Table)
**Purpose:** Store customer attributes and demographics for analysis and filtering

| Column Name | Data Type | Constraints | Description |
|------------|-----------|-------------|-------------|
| customer_id | INT | PK, AUTO_INCREMENT | Unique customer identifier |
| name | VARCHAR(255) | NOT NULL | Full name of customer |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Email address |
| phone | VARCHAR(20) | | Phone number |
| address | VARCHAR(500) | | Street address |
| city | VARCHAR(100) | | City |
| state | VARCHAR(100) | | State/Province |
| postal_code | VARCHAR(20) | | Postal code |
| country | VARCHAR(100) | | Country |
| age_group | ENUM('18-25', '26-35', '36-45', '46-55', '56-65', '65+') | | Age demographic |
| gender | ENUM('M', 'F', 'Other', 'Prefer not to say') | | Gender |
| customer_segment | VARCHAR(50) | | Premium, Regular, Budget, etc. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Dimension effective date |
| updated_at | TIMESTAMP | ON UPDATE CURRENT_TIMESTAMP | Last update date |

**Indexes:**
- PRIMARY KEY: customer_id
- UNIQUE: email
- INDEX: city, country, customer_segment

---

### 2. PRODUCT (Dimension Table)
**Purpose:** Store product attributes for analysis and filtering

| Column Name | Data Type | Constraints | Description |
|------------|-----------|-------------|-------------|
| product_id | INT | PK, AUTO_INCREMENT | Unique product identifier |
| sku | VARCHAR(50) | UNIQUE, NOT NULL | Stock Keeping Unit |
| name | VARCHAR(255) | NOT NULL | Product name |
| description | TEXT | | Product description |
| category | VARCHAR(100) | | Product category (for analysis) |
| supplier_name | VARCHAR(255) | | Supplier company name |
| supplier_contact | VARCHAR(255) | | Supplier contact person |
| supplier_email | VARCHAR(255) | | Supplier email |
| supplier_phone | VARCHAR(20) | | Supplier phone |
| supplier_address | VARCHAR(500) | | Supplier address |
| cost_price | DECIMAL(10,2) | NOT NULL | Cost per unit |
| selling_price | DECIMAL(10,2) | NOT NULL | Selling price per unit |
| storage_location | VARCHAR(100) | | Warehouse location |
| reorder_level | INT | | Minimum stock level |
| unit_of_measure | VARCHAR(20) | | Unit (e.g., pieces, kg, liters) |
| status | ENUM('Active', 'Discontinued', 'Inactive') | | Product status |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Dimension effective date |
| updated_at | TIMESTAMP | ON UPDATE CURRENT_TIMESTAMP | Last update date |

**Indexes:**
- PRIMARY KEY: product_id
- UNIQUE: sku
- INDEX: category, status, storage_location

---

### 3. SALES (Fact Table)
**Purpose:** Central fact table storing all sales transactions with metrics, dimensions, and return flag

| Column Name | Data Type | Constraints | Description |
|------------|-----------|-------------|-------------|
| sales_id | INT | PK, AUTO_INCREMENT | Unique sales transaction identifier |
| customer_id | INT | FK → CUSTOMER, NOT NULL | Reference to customer dimension |
| product_id | INT | FK → PRODUCT, NOT NULL | Reference to product dimension |
| order_date | DATE | NOT NULL | Date of order placement (sortable key) |
| delivery_date | DATE | | Expected/actual delivery date |
| sales_channel | ENUM('Online', 'In-Store', 'Phone', 'Mobile App') | NOT NULL | Channel of sale |
| quantity | INT | NOT NULL | Quantity sold (additive measure) |
| unit_price | DECIMAL(10,2) | NOT NULL | Price per unit at time of sale (non-additive) |
| discount_percentage | DECIMAL(5,2) | DEFAULT 0 | Discount % on order |
| discount_amount | DECIMAL(10,2) | DEFAULT 0 | Total discount amount (additive measure) |
| tax_percentage | DECIMAL(5,2) | DEFAULT 0 | Tax % on order |
| tax_amount | DECIMAL(10,2) | DEFAULT 0 | Total tax amount (additive measure) |
| sales_total | DECIMAL(12,2) | NOT NULL | Total sales = (qty×price) - discount + tax (additive) |
| order_status | ENUM('Pending', 'Confirmed', 'Shipped', 'Delivered', 'Cancelled') | DEFAULT 'Pending' | Order fulfillment status |
| shipping_address | VARCHAR(500) | | Delivery address |
| shipping_city | VARCHAR(100) | | Delivery city |
| shipping_state | VARCHAR(100) | | Delivery state |
| shipping_postal_code | VARCHAR(20) | | Delivery postal code |
| shipping_cost | DECIMAL(10,2) | DEFAULT 0 | Shipping charges (additive) |
| is_returned | BOOLEAN | DEFAULT 0 | Flag: 1 if product was returned, 0 if not |
| return_date | DATE | | Date of return (if is_returned = 1) |
| return_reason | ENUM('Defective', 'Wrong Item', 'Not as Described', 'Changed Mind', 'Damaged in Shipping', 'Other') | | Reason for return (if applicable) |
| return_quantity | INT | | Quantity returned (if is_returned = 1) |
| return_amount | DECIMAL(12,2) | | Return/refund amount (if is_returned = 1) |
| notes | TEXT | | Transaction notes |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Fact record creation date |
| updated_at | TIMESTAMP | ON UPDATE CURRENT_TIMESTAMP | Last update date |

**Indexes:**
- PRIMARY KEY: sales_id
- FK: customer_id, product_id
- INDEX: order_date, sales_channel, order_status, is_returned
- INDEX: customer_id, product_id (for joins)

**Measures (Additive Metrics):**
- Quantity
- Discount Amount
- Tax Amount
- Sales Total
- Shipping Cost
- Return Quantity (if returned)
- Return Amount (if returned)

**Dimension Keys:**
- customer_id (FK to CUSTOMER)
- product_id (FK to PRODUCT)

---

### 4. PAYMENT (Fact Table)
**Purpose:** Track all payment transactions with refund status and details

| Column Name | Data Type | Constraints | Description |
|------------|-----------|-------------|-------------|
| payment_id | INT | PK, AUTO_INCREMENT | Unique payment identifier |
| sales_id | INT | FK → SALES, NOT NULL | Reference to sales transaction |
| payment_type | ENUM('Credit Card', 'Debit Card', 'Bank Transfer', 'Digital Wallet', 'Cash', 'Check') | NOT NULL | Payment method |
| card_last_4_digits | VARCHAR(4) | | Last 4 digits of card (masked) |
| card_brand | ENUM('Visa', 'Mastercard', 'American Express', 'Discover', 'Other') | | Card brand |
| card_token | VARCHAR(255) | | Tokenized card reference (never store full card) |
| amount | DECIMAL(12,2) | NOT NULL | Payment amount (additive measure) |
| payment_status | ENUM('Pending', 'Authorized', 'Captured', 'Failed', 'Cancelled') | DEFAULT 'Pending' | Payment status |
| transaction_date | TIMESTAMP | NOT NULL | Date/time of payment transaction |
| authorization_code | VARCHAR(100) | | Payment gateway authorization code |
| gateway_response | TEXT | | Payment gateway response details |
| currency | VARCHAR(3) | DEFAULT 'USD' | Currency code |
| is_refunded | BOOLEAN | DEFAULT 0 | Flag: 1 if payment was refunded, 0 if not |
| refund_amount | DECIMAL(12,2) | DEFAULT 0 | Amount refunded (if is_refunded = 1) (additive) |
| refund_reason | VARCHAR(255) | | Reason for refund (if applicable) |
| refund_status | ENUM('Initiated', 'Processing', 'Completed', 'Failed', 'Cancelled', 'Pending') | DEFAULT 'Pending' | Refund processing status (if applicable) |
| refund_date | TIMESTAMP | | Date refund was initiated (if is_refunded = 1) |
| gateway_refund_id | VARCHAR(100) | | Payment gateway refund reference |
| notes | TEXT | | Payment/refund notes |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Record creation date |

**Indexes:**
- PRIMARY KEY: payment_id
- FK: sales_id
- INDEX: transaction_date, payment_status, is_refunded
- INDEX: refund_status, refund_date (for refund analytics)

**Measures (Additive Metrics):**
- Amount (original payment)
- Refund Amount (if refunded)

---

## Key Relationships (Star Schema Pattern)

| Relationship | Type | Cardinality | Purpose |
|-------------|------|-------------|---------|
| CUSTOMER → SALES | Dimension to Fact | 1:N | Each customer can have multiple sales |
| PRODUCT → SALES | Dimension to Fact | 1:N | Each product can appear in multiple sales |
| SALES → PAYMENT | Fact to Fact | 1:N | Each sale can have multiple payments (partial/split payments) |

---

## Normalization

- **Star Schema:** ✅ Denormalized by design for analytical efficiency
- **Dimension Tables:** CUSTOMER, PRODUCT (slowly changing dimensions)
- **Fact Table:** SALES (event-based, granular transactions)
- **Bridge Table:** RETURN_PAYMENT (links returns and payments to sales)

**Note:** This dimensional model is optimized for OLAP queries and reporting, not OLTP operations.

---

## Data Quality Rules

### Mandatory Validations:
1. Customer email must be unique
2. Product SKU must be unique
3. Sales total = (quantity × unit_price) - discount_amount + tax_amount + shipping_cost
4. Selling price > Cost price
5. Reorder level > 0
6. Return quantity ≤ quantity in original sale
7. Return amount + Refund amount ≤ original sales total
8. Fact table transactions should only be inserted (immutable history)

### Status Flow Rules:
- **Order Status:** Pending → Confirmed → Shipped → Delivered (or → Cancelled)
- **Payment Status:** Pending → Authorized → Captured (or → Failed/Cancelled/Refunded/Partially Refunded)
- **Refund Status:** Initiated → Processing → Completed (or → Failed/Cancelled)
- **Return Status:** Pending → Approved → Processed (or → Rejected)

---

## Analytics Dimensions & Metrics

### Key Metrics:
- Total Sales Revenue
- Average Order Value (AOV)
- Customer Lifetime Value (CLV)
- Return Rate (%)
- Payment Success Rate (%)
- Sales by Channel
- Sales by Product Category
- Sales by Geography
- Customer Acquisition Cost (CAC)
- Repeat Customer Rate

### Analytical Queries:

### Sales by Channel
```sql
SELECT s.sales_channel, COUNT(s.sales_id) as transaction_count, 
       SUM(s.sales_total) as total_revenue, AVG(s.sales_total) as avg_order_value
FROM SALES s
WHERE s.order_date BETWEEN '2025-01-01' AND '2025-12-31'
GROUP BY s.sales_channel
ORDER BY total_revenue DESC;
```

#### Top 10 Customers by Revenue
```sql
SELECT c.customer_id, c.name, c.customer_segment,
       COUNT(s.sales_id) as order_count, SUM(s.sales_total) as total_revenue
FROM CUSTOMER c
JOIN SALES s ON c.customer_id = s.customer_id
WHERE s.order_date >= DATE_SUB(NOW(), INTERVAL 1 YEAR)
GROUP BY c.customer_id, c.name, c.customer_segment
ORDER BY total_revenue DESC LIMIT 10;
```

#### Product Performance & Return Rate
```sql
SELECT p.product_id, p.name, p.category,
       COUNT(s.sales_id) as units_sold, SUM(s.quantity) as total_quantity,
       SUM(CASE WHEN s.is_returned = 1 THEN 1 ELSE 0 END) as return_count,
       ROUND(SUM(CASE WHEN s.is_returned = 1 THEN 1 ELSE 0 END) / COUNT(s.sales_id) * 100, 2) as return_rate,
       SUM(s.sales_total) as total_sales_revenue,
       SUM(CASE WHEN s.is_returned = 1 THEN s.return_amount ELSE 0 END) as total_returns
FROM PRODUCT p
JOIN SALES s ON p.product_id = s.product_id
GROUP BY p.product_id, p.name, p.category
ORDER BY total_sales_revenue DESC;
```

#### Payment & Refund Analysis
```sql
SELECT p.payment_type, p.card_brand,
       COUNT(p.payment_id) as payment_count,
       SUM(p.amount) as total_paid,
       SUM(CASE WHEN p.payment_status = 'Captured' THEN p.amount ELSE 0 END) as captured_amount,
       SUM(CASE WHEN p.is_refunded = 1 THEN p.refund_amount ELSE 0 END) as total_refunded,
       SUM(CASE WHEN p.is_refunded = 1 THEN 1 ELSE 0 END) as refund_count,
       ROUND(SUM(CASE WHEN p.is_refunded = 1 THEN p.refund_amount ELSE 0 END) / 
             SUM(p.amount) * 100, 2) as refund_rate
FROM PAYMENT p
WHERE p.created_at >= DATE_SUB(NOW(), INTERVAL 1 MONTH)
GROUP BY p.payment_type, p.card_brand
ORDER BY total_paid DESC;
```

#### Customer Segment Analysis
```sql
SELECT c.customer_segment, c.age_group, COUNT(DISTINCT c.customer_id) as customer_count,
       COUNT(s.sales_id) as total_orders, AVG(s.sales_total) as avg_order_value,
       SUM(s.sales_total) as total_revenue
FROM CUSTOMER c
LEFT JOIN SALES s ON c.customer_id = s.customer_id AND s.order_date >= DATE_SUB(NOW(), INTERVAL 6 MONTH)
GROUP BY c.customer_segment, c.age_group
ORDER BY total_revenue DESC;
```

---

## Implementation Notes

1. **Star Schema Design:** Optimized for aggregations and slicing/dicing by dimensions
2. **Fact Table Granularity:** One row per sales transaction (customer + product combination)
3. **Return Flag in SALES:** `is_returned` boolean simplifies return analysis; return details nested in same row
4. **Payment Flag in PAYMENT:** `is_refunded` boolean tracks refund status; refund details in same row
5. **Additive Measures:** Quantity, discount_amount, tax_amount, sales_total, shipping_cost, return_amount (SALES); amount, refund_amount (PAYMENT)
6. **Non-Additive Measures:** unit_price (use AVG), rates (use weighted averages)
7. **Slowly Changing Dimensions:** CUSTOMER and PRODUCT may use Type 2 SCD (versioning) for historical analysis
8. **Security:** Never store full credit card details in PAYMENT; use tokenization
9. **Scalability:** Partition SALES and PAYMENT tables by date for large datasets
10. **Performance:** Pre-aggregated summary tables (cubes) recommended for BI tools
11. **Immutability:** Fact tables should support INSERT and SELECT only; avoid UPDATE/DELETE
12. **Audit Trail:** Consider adding created_by, updated_by fields for compliance

---

## CSV Import Mapping

### Customer CSV → CUSTOMER Dimension Table
```
customer_id, name, contact_details, demographics, email_address
→ CUSTOMER.customer_id, name, phone/address, age_group/gender, email
   (Dimensional attributes loaded once, updated when customer info changes)
```

### Product CSV → PRODUCT Dimension Table
```
product_name, sku, description, supplier, storage_details, selling_price, cost_price
→ PRODUCT.name, sku, description, supplier_name, storage_location, selling_price, cost_price
   (Dimensional attributes loaded; consider versioning for historical product prices)
```

### Sales & Return CSV → SALES Fact Table
```
sales_order_id, customer_id, product_id, quantities, prices, order_dates, mode_of_sales, return_flag, return_reason, return_date, return_quantity, return_amount
→ SALES.sales_id, customer_id, product_id, quantity, unit_price, order_date, sales_channel, 
   is_returned (boolean flag from return_flag), return_reason, return_date, return_quantity, return_amount
   (If is_returned = 1, populate return columns; otherwise leave NULL)
```

### Card Payment & Refund CSV → PAYMENT Fact Table
```
card_payment_id, sales_order_id, card_brand, amount, payment_date, card_refund_id, refund_amount, refund_date
→ PAYMENT.payment_id, sales_id, card_brand, amount, transaction_date, 
   is_refunded (boolean flag from presence of refund_id), refund_amount, refund_date, refund_status
   (If refund_id exists, set is_refunded = 1 and populate refund columns)
```
