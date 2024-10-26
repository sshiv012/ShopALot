# ShopALot
Learning AsterixDB, SQL++ with an ecommerce dataset

This README provides instructions to create a sample AsterixDB database named `ShopALot`, set up various datasets, and load data from JSON files.

## Prerequisites

Ensure AsterixDB is installed and running locally (Run http://localhost:19006/ on your web browser). To load data, have your JSON files downloaded from the github repository.

## Database Schema

### Dataverse and Datasets

The `ShopALot` dataverse contains the following datasets:

1. **Users**
    - Stores user information, including IDs, names, email addresses, and phone numbers.
    - Primary Key: `user_id`

2. **Stores**
    - Contains details about stores, including addresses, categories, and geolocation data.
    - Primary Key: `store_id`

3. **Products**
    - Describes products with details like category, name, and description.
    - Primary Key: `product_id`

4. **Orders**
    - Holds information about orders, including order ID, user ID, store ID, total price, and items purchased.
    - Primary Key: `order_id`

5. **StockedBy**
    - Tracks product availability at each store, including quantity available.
    - Primary Key: `(product_id, store_id)`

## Creating the Dataverse and Datasets

Run the following AsterixDB SQL++ commands to create the dataverse, types, and datasets:

```sql
CREATE DATAVERSE ShopALot;
USE ShopALot;

CREATE TYPE UsersType AS {
    user_id: string,
    email: string?,
    name: {
        first: string,
        last: string
    },
    phones: [{
        kind: string,
        number: string
    }]?
};
CREATE DATASET Users(UsersType)
   PRIMARY KEY user_id;

CREATE TYPE StoresType AS {
    store_id: string,
    name: string,
    address: {
        city: string,
        street: string,
        state: string,
        zip_code: string,
        location: geometry?
    },
    phone: string,
    categories: [string]
};
CREATE DATASET Stores(StoresType)
   PRIMARY KEY store_id;

CREATE TYPE ProductsType AS {
    product_id: string,
    category: string,
    name: string,
    description: string
};
CREATE DATASET Products(ProductsType)
   PRIMARY KEY product_id;

CREATE TYPE OrdersType AS {
    order_id: string,
    user_id: string,
    store_id: string,
    total_price: float,
    time_placed: datetime,
    pickup_time: datetime?,
    time_fulfilled: datetime?,
    items: [{
        item_id: string,
        qty: integer,
        selling_price: float,
        product_id: string
    }]
};
CREATE DATASET Orders(OrdersType)
   PRIMARY KEY order_id;

CREATE TYPE StockedByType AS {
    product_id: string,
    store_id: string,
    qty: integer
};
CREATE DATASET StockedBy(StockedByType)
   PRIMARY KEY product_id, store_id;
```

## Loading Data

To load data into AsterixDB, use the `LOAD DATASET` commands. Update the file paths to point to your dataset's location.

### Example Paths for Mac/Linux

The following commands demonstrate loading data from JSON files on a Mac/Linux system. Replace the paths with the actual location of your files.

```sql
LOAD DATASET ShopALot.Users USING localfs (("path"="localhost:///home/user/ShopALot/users.json"), ("format"="json"));
LOAD DATASET ShopALot.Stores USING localfs (("path"="localhost:///home/user/ShopALot/stores.json"), ("format"="json"));
LOAD DATASET ShopALot.Products USING localfs (("path"="localhost:///home/user/ShopALot/products.json"), ("format"="json"));
LOAD DATASET ShopALot.Orders USING localfs (("path"="localhost:///home/user/ShopALot/orders.json"), ("format"="json"));
LOAD DATASET ShopALot.StockedBy USING localfs (("path"="localhost:///home/user/ShopALot/stockedby.json"), ("format"="json"));
```

### Example Paths for Windows

For Windows, specify paths using Windows syntax. Replace the paths with the actual location of your files.

```sql
LOAD DATASET ShopALot.Users USING localfs (("path"="localhost:///C:/Users/YourUsername/ShopALot/users.json"), ("format"="json"));
LOAD DATASET ShopALot.Stores USING localfs (("path"="localhost:///C:/Users/YourUsername/ShopALot/stores.json"), ("format"="json"));
LOAD DATASET ShopALot.Products USING localfs (("path"="localhost:///C:/Users/YourUsername/ShopALot/products.json"), ("format"="json"));
LOAD DATASET ShopALot.Orders USING localfs (("path"="localhost:///C:/Users/YourUsername/ShopALot/orders.json"), ("format"="json"));
LOAD DATASET ShopALot.StockedBy USING localfs (("path"="localhost:///C:/Users/YourUsername/ShopALot/stockedby.json"), ("format"="json"));
```

Replace `YourUsername` with your actual Windows username and adjust the file paths as needed.

---
**Warning:**

The `INSERT INTO` statements provided below should only be executed **after bulk loading all initial data**. If records are inserted before completing the bulk load, the bulk load process will fail due to existing records. **In such cases, you will need to drop the dataverse and recreate all datasets and types from scratch to proceed with bulk loading.**

### Insert Statements

To add new records after completing bulk loading, use the following:
**Reminder:** Execute these inserts only after completing bulk loading.

```sql
INSERT INTO Users (
[
  {"user_id": "user008",
   "email": "lara.croft@adventure.com",
   "name": {"first": "Lara", "last": "Croft"},
   "phones": [{"kind": "HOME", "number": "555-876-4321"}]
  },
  {"user_id": "user009",
   "name": {"first": "Sherlock", "last": "Holmes"},
   "phones": [{"kind": "WORK", "number": "123-456-7890"}]
  },
  {"user_id": "user010",
   "email": "tony.stark@starkindustries.com",
   "name": {"first": "Tony", "last": "Stark"},
   "phones": [{"kind": "MOBILE", "number": "999-555-1111"}]
  }
]
);

INSERT INTO Stores (
    {
        "store_id": "S001",
        "name": "Healthy Foods Market",
        "address": {
            "city": "Los Angeles",
            "street": "123 Wellness St",
            "state": "CA",
            "zip_code": "90001",
            "location": st_make_point(-118.2437, 34.0522)
        },
        "phone": "123-456-7890",
        "categories": ["Groceries", "Organic", "Health"]
    }
);
```

## Verifying Data Counts

After loading data, run the following SQL++ query to verify record counts in each dataset:

```sql
SELECT VALUE {
    "usersCount": (SELECT VALUE COUNT(*) FROM Users),
    "storesCount": (SELECT VALUE COUNT(*) FROM Stores),
    "productsCount": (SELECT VALUE COUNT(*) FROM Products),
    "ordersCount": (SELECT VALUE COUNT(*) FROM Orders),
    "stockedByCount": (SELECT VALUE COUNT(*) FROM StockedBy)
};
```

The query will return a JSON object with the count of records in each dataset. Everything will be greater than 0.

---

# Queries for ShopALot Database

### Query 1

```sql
SELECT * FROM Products WHERE list_price > 10;
```
- Retrieves all columns for products with a `list_price` greater than 10.

---

### Query 2

```sql
SELECT VALUE name FROM Products WHERE list_price > 10;
```
- Returns only the `name` field of products with a `list_price` greater than 10.

---

### Query 3

```sql
SELECT VALUE name FROM Products WHERE list_price > 10 LIMIT 10;
```
- Retrieves the `name` of up to 10 products with a `list_price` greater than 10.

---

### Query 4

```sql
SELECT VALUE name FROM Products WHERE list_price > 10 LIMIT 10 OFFSET 5;
```
- Retrieves the `name` of up to 10 products with a `list_price` greater than 10, skipping the first 5 records.

---

### Query 5

```sql
SELECT user_id, email FROM Users WHERE email LIKE "%gmail.com" LIMIT 3;
```
- Selects the `user_id` and `email` fields of up to 3 users whose email addresses contain "gmail.com".

---

### Query 6

```sql
SELECT VALUE product_id FROM StockedBy WHERE store_id = "C4N2L";
```
- Retrieves the `product_id` of all products stocked by the store with ID "C4N2L".

---

### Query 7

```sql
SELECT VALUE { "StoreName": s.name, "Quantity": sb.qty } 
FROM StockedBy sb, Stores s 
WHERE sb.store_id = s.store_id AND sb.store_id = "C4N2L";
```
- Returns a JSON object containing the store name and quantity for products in stock at the store with ID "C4N2L".

---

### Query 8

```sql
SELECT s.name AS StoreName, sb.qty AS Quantity 
FROM StockedBy sb, Stores s 
WHERE sb.store_id = s.store_id AND sb.store_id = "C4N2L";
```
- Retrieves the store name and stock quantity for products at the store with ID "C4N2L".

---

### Query 9

```sql
SELECT VALUE { 
    "StoreName": s.name, 
    "Stocks": (SELECT VALUE sb.product_id FROM StockedBy sb WHERE sb.store_id = s.store_id) 
} 
FROM Stores s 
WHERE s.store_id = "C4N2L";
```
- Returns a JSON object with the store name and a list of `product_id`s for all products stocked by the store with ID "C4N2L".

---

### Query 10

```sql
SELECT o.order_id, o.user_id, i.product_id AS product, i.qty AS quantity 
FROM Orders o UNNEST o.items i 
WHERE i.qty > 30;
```
- Retrieves order ID, user ID, product ID, and quantity for items within orders where the item quantity is greater than 30.

---

### Query 11

```sql
SELECT DISTINCT VALUE o.user_id 
FROM Orders o 
WHERE SOME i IN o.items SATISFIES i.selling_price >= 80.00;
```
- Returns unique user IDs for orders where at least one item has a selling price of 80.00 or more.

---

### Query 12

```sql
SELECT DISTINCT VALUE o.user_id 
FROM Orders o 
WHERE EVERY i IN o.items SATISFIES i.selling_price >= 70.00;
```
- Retrieves unique user IDs for orders where every item has a selling price of at least 70.00.

---

### Query 13

```sql
SELECT DISTINCT VALUE o.user_id 
FROM Orders o 
WHERE array_count(o.items) > 0 
AND (EVERY i IN o.items SATISFIES i.selling_price >= 70.00);
```
- Returns unique user IDs for orders that contain items, and where every item has a selling price of 70.00 or more.

---

### Query 14

```sql
SELECT u.name 
FROM Users u 
WHERE u.user_id IN (
    SELECT DISTINCT VALUE o.user_id 
    FROM Orders o 
    WHERE EVERY i IN o.items SATISFIES i.selling_price >= 70.00 
    AND ARRAY_COUNT(o.items) > 0
);
```
- Retrieves the name of users whose orders contain items, and where every item in those orders has a selling price of at least 70.00.

---

### Query 15

```sql
SELECT o.order_id, o.time_placed, o.time_fulfilled, o.total_price, o.user_id 
FROM Orders o 
WHERE total_price > 150.00 AND o.time_fulfilled IS MISSING;
```
- Retrieves order ID, time placed, time fulfilled, total price, and user ID for orders with a total price greater than 150.00, where `time_fulfilled` is missing.

---

### Query 16

```sql
SELECT VALUE { 
    "order_id": o.order_id, 
    "time_placed": o.time_placed, 
    "time_fulfilled": CASE WHEN o.time_fulfilled IS MISSING THEN "TBD" ELSE o.time_fulfilled END, 
    "total_price": o.total_price, 
    "user_id": o.user_id 
} 
FROM Orders o 
WHERE user_id = "QREX9" 
LIMIT 3;
```
- Retrieves a JSON object containing the order ID, time placed, total price, and user ID for orders from user "QREX9". If `time_fulfilled` is missing, it displays "TBD"; otherwise, it shows the actual fulfillment time. Limits results to 3 records.

---

### Query 17

```sql
SELECT name, total_price FROM Orders ORDER BY total_price DESC;
```
- Retrieves the `name` and `total_price` from the `Orders` dataset, ordered by `total_price` in descending order.

---

### Query 18

```sql
SELECT u.name, o.order_id FROM Users AS u, Orders AS o WHERE u.user_id = o.user_id;
```
- Selects the `name` from `Users` and `order_id` from `Orders` for records where the `user_id` matches in both datasets.

---

### Query 19

```sql
SELECT u.email, o.time_placed
FROM Users u JOIN Orders o
  ON u.user_id = o.user_id
WHERE o.total_price > 200
ORDER BY o.total_price DESC
LIMIT 3;
```
- Retrieves the `email` of users and the `time_placed` for orders where the total price is over 200. Orders are sorted by `total_price` in descending order, limiting results to the top 3.

---

### Query 20

```sql
SELECT * FROM Stores s GROUP BY s.address.city;
```
- Groups all `Stores` by their `city` field within the `address` attribute. This structure allows querying store groups based on city.

---

### Query 21

```sql
SELECT category, COUNT(*) AS total_products FROM Products GROUP BY category;
```
- Counts the total products within each category in the `Products` dataset and returns the category name and product count.

---

### Query 22

```sql
SELECT u.email,
       ARRAY_COUNT(o.items) AS order_size
FROM Users AS u, Orders AS o
WHERE u.user_id = o.user_id
ORDER BY order_size DESC
LIMIT 3;
```
- Returns user emails and the size of each order (i.e., the count of items per order), sorted in descending order of order size, limiting to the top 3.

---

### Query 23

```sql
ARRAY_MAX(
  (SELECT VALUE list_price
   FROM Products
   WHERE is_number(list_price))
);
```
- Finds the maximum `list_price` among products with a numeric value for `list_price`.

---

### Query 24

```sql
SELECT s.address.state, COUNT(*) AS cnt
FROM Stores AS s, Orders AS o
WHERE s.store_id = o.store_id
GROUP BY s.address.state;
```
- Counts the number of orders per state, grouping by the `state` attribute in the `Stores` dataset where orders are placed.

---

### Query 25

```sql
SELECT s.address.state, g
FROM Stores AS s, Orders AS o
WHERE s.store_id = o.store_id
GROUP BY s.address.state GROUP AS g;
```
- Groups `Stores` and `Orders` by `state` and creates a grouped alias `g` that contains information related to each state’s stores and orders.

---

### Query 26

```sql
FROM Stores AS s, Orders AS o
WHERE s.store_id = o.store_id
GROUP BY s.address.state GROUP AS g
SELECT s.address.state,
       (SELECT g.s.store_id, g.s.name, g.o.order_id FROM g) AS so_pairs;
```
- Groups `Stores` and `Orders` by `state`, and for each state, generates a list of pairs that includes each store’s `store_id` and `name`, along with the `order_id` for orders made at that store.

## Notes

- Ensure paths are accessible from the machine running AsterixDB.
- Data is expected to be in JSON format.
- This setup assumes basic familiarity with AsterixDB and SQL++ commands.
---
