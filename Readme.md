
# SUNRISE SUPER MARKET

- Names:Gakuru Elijah
- ID:28808
- Database Used :PostgreSQL

## Scenario 

Welcome to Sunrise Supermarket, your neighborhood grocery retailer! We proudly serve customers from across the region, making it easy to visit us in-store and purchase everything you need in a single order.

### Customer's order
>Listing every order with the customer's name, city, and order date.

```sql

select o.order_id,
       c.customer_name as name,
       c.city,
       o.order_date
  from orders o
 inner join customers c
on o.customer_id = c.customer_id;
```

![Customer Orders](http://github.com/gakuruElijah/assignment_1_Gakuru_Elijah-28808/blob/main/scrrenshots/join%20qst%201.png)

### Order Item 
>Listing every order item with product name, category, price, and quantity.

```sql 
select oi.order_item_id as id,
       p.product_name as product,
       p.category,
       p.price,
       oi.quantity
  from order_items oi
 inner join products p
on oi.product_id = p.product_id;
```

![Order Item Info](https://github.com/gakuruElijah/assignment_1_Gakuru_Elijah-28808/blob/main/scrrenshots/join%20qst%202.png)

### Customer With Orders
>Listing all customers and their orders where they exist, including customers with no orders.

```sql
select c.customer_id,
       c.customer_name as customer,
       c.city,
       o.order_id,
       o.order_date
  from customers c
  left join orders o
on c.customer_id = o.customer_id
 order by c.customer_id,
          o.order_date;
```
![Customer with Order](https://github.com/gakuruElijah/assignment_1_Gakuru_Elijah-28808/blob/main/scrrenshots/join%20qst%203.png)

### Spend Above average 
>Customer total spend (quantity × price) and showing customers above average spend. Using CTE.

```sql 
WITH customer_totals AS (
    SELECT
        o.customer_id,
        SUM(oi.quantity * p.price) AS total_spend
    FROM orders o
    JOIN order_items oi
        ON o.order_id = oi.order_id
    JOIN products p
        ON oi.product_id = p.product_id
    GROUP BY o.customer_id
)
SELECT
    c.customer_id,
    c.customer_name,
    ct.total_spend
FROM customer_totals ct
JOIN customers c
    ON c.customer_id = ct.customer_id
WHERE ct.total_spend > (
    SELECT AVG(total_spend)
    FROM customer_totals
)
ORDER BY ct.total_spend DESC;
```

![Spend Above Average](https://github.com/gakuruElijah/assignment_1_Gakuru_Elijah-28808/blob/main/scrrenshots/window%20function%201.png)



### Customer Order Number 
> Number each customer's orders in the order placed.

```sql

select o.customer_id,
       c.customer_name as customer,
       o.order_id,
       o.order_date,
       row_number()
       over(partition by o.customer_id
            order by o.order_date
       ) as order_number
  from orders o
  join customers c
on o.customer_id = c.customer_id
 order by o.customer_id,
          o.order_date;
```

![Order Numbers](https://github.com/gakuruElijah/assignment_1_Gakuru_Elijah-28808/blob/main/scrrenshots/window%20function%202.png)

### Revenue
>Showing a running total of revenue over time, ordered by order date.

```sql 
with order_totals as (
   select o.order_id,
          o.order_date,
          sum(oi.quantity * p.price) as order_total
     from orders o
     join order_items oi
   on o.order_id = oi.order_id
     join products p
   on oi.product_id = p.product_id
    group by o.order_id,
             o.order_date
)
select order_id,
       order_date,
       order_total,
       sum(order_total)
       over(
           order by order_date,
                    order_id
       ) as running_revenue
  from order_totals
 order by order_date,
          order_id;
```

![Revenue](https://github.com/gakuruElijah/assignment_1_Gakuru_Elijah-28808/blob/main/scrrenshots/window%20function%203%20(i).png)


### Days Between orders
>For each customer with more than one order, show days between the current and previous order.
```sql
SELECT
    customer_id,
    order_id,
    order_date,
    previous_order_date,
    order_date - previous_order_date AS days_between_orders
FROM (
    SELECT
        customer_id,
        order_id,
        order_date,
        LAG(order_date) OVER (
            PARTITION BY customer_id
            ORDER BY order_date
        ) AS previous_order_date
    FROM orders
) x
WHERE previous_order_date IS NOT NULL
ORDER BY customer_id, order_date;
 ```
![Days Between](https://github.com/gakuruElijah/assignment_1_Gakuru_Elijah-28808/blob/main/scrrenshots/window%20function%204%20(i).png)




