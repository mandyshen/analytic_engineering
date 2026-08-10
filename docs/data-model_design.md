# Delivery Analytics Data Model

## Overview

This data model supports a grocery-delivery platform where customers place orders with merchants, and drivers deliver those orders.

The model supports analysis of order volume, sales, item fulfillment, product replacements, store performance, driver activity, and customer behavior.

Intended users include data analysts, operations teams, and data scientists.

## Business Process

1. A customer places an order containing one or more items.
2. A merchant accepts the order.
3. The order is assigned to a driver.
4. Each ordered item is fulfilled, replaced, or marked unavailable.
5. The driver delivers the order.
6. The order reaches a final status, such as delivered, cancelled, or failed.

## Grain

| Fact table | Grain |
|---|---|
| `fact_orders` | One row per order. |
| `fact_order_items` | One row per ordered item within an order. |

| Dimension table | Grain |
|---|---|
| `dim_customer` | One row per customer. |
| `dim_driver` | One row per driver. |
| `dim_store` | One row per store or merchant location. |
| `dim_item` | One row per sellable item at a store. An item may be available at multiple stores. |

## Fact Tables

### fact_orders

Grain: one row per order.

Primary key: `order_id`

Foreign keys:
- `customer_id`
- `driver_id`

Important attributes and measures:
- `order_timestamp`
- `order_status`
- `currency`
- `order_total`

`order_status` contains the latest or final status of an order. This model does not track a full history of status changes.

### fact_order_items

Grain: one row per item within an order.

Primary key: (`order_id`, `order_item_id`)

Foreign keys:
- `order_id`
- `item_id`
- `store_id`

Important attributes and measures:
- `unit_price`: price charged for one unit of the item at the time of the order
- `quantity`
- `sales_amount`: `unit_price * quantity`
- `item_fulfillment_status`: fulfilled, replaced, or unavailable

The item price is stored in the fact table to preserve the historical price charged to the customer.

## Dimension Tables

### dim_customer

One row per customer.

Example attributes:
- `customer_id`
- `customer_name`
- `customer_join_date`
- `is_membership`

### dim_driver

One row per driver.

Example attributes:
- `driver_id`
- `driver_name`
- `driver_join_date`
- `employment_type`

### dim_store

One row per store or merchant location.

Example attributes:
- `store_id`
- `store_name`
- `store_address`
- `city_name`
- `country_name`
- `store_status`

### dim_item

One row per sellable item at a store.

Example attributes:
- `item_id`
- `store_id`
- `item_name`
- `item_type`
- `current_unit_price`
- `availability_status`

An item can represent a specific store’s listing of a grocery product. This makes the relationship between an item, its store, price, and availability clear.

## Model Diagram

The model includes two fact tables: one at the order level and one at the order-item level. `fact_order_items` connects each ordered item to its store and item details.

## Design Decisions

### Why are there two fact tables?

One order can contain multiple items. `fact_orders` stores order-level information, while `fact_order_items` stores details for each item in the order. Separating them prevents duplicated order-level measures and supports item-level analysis.

### Why are dimensions separated from facts?

Dimensions such as customer, driver, store, and item provide descriptive context. Fact tables capture business events and measurable values, such as orders, quantities, and sales.

### Why is store_id stored in fact_order_items?

An order may contain items from multiple stores. Therefore, store information belongs at the item level rather than the order level.

### How is item pricing handled?

`dim_item.current_unit_price` represents the current listed price. `fact_order_items.unit_price` represents the price charged at the time of the order. This preserves historical sales reporting.

### How is order status handled?

`fact_orders` stores only the latest or final order status. The record may be updated until the order reaches a terminal status, such as delivered or cancelled. A separate status-history table would be required if status-change analysis is needed later.

### How is private information handled?

Customer and driver email addresses and phone numbers are personally identifiable information (PII). Access should be restricted, and these fields should be excluded or masked in analytics models when they are not required.