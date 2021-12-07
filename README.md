** START by running the commands in the clearSandbox.sql file in the postgres.devmountain.com database **

Picking up students from School Example

Martha picks up students each day
-- Authorized to get them from school if they are sick or injured

-- To keep track of this, we could set up two tables

```
CREATE TABLE parent(
	parent_id SERIAL PRIMARY KEY,
	parent_name VARCHAR(20)
);
```
```
INSERT INTO parent
(parent_name)
VALUES
('Martha'), ('Johnathan'), ('Bernadette');
```

We can run a query to allow us to see the parents and their associated IDs

```
SELECT * FROM parent;
```

Once we have those IDs, we can create our child table, and add children to it

```
CREATE TABLE child(
	child_id SERIAL PRIMARY KEY,
	child_name VARCHAR(20), 
	parent_id INT REFERENCES parent(parent_id)
);
```
```
INSERT INTO child
(child_name, parent_id)
VALUES
('Joey', 1), ('Brittney', 1), ('Carlos', 2), ('Samantha', 3);
```

When we run our query to get this data, we can see each child, and the ID of the person authorized to pick them up

```
SELECT * FROM child;
```

This works well, until we take into consideration that their father, Scott, sometimes
needs to pick up the kids. 

We COULD just add a row to the child table, but what if Scott and Martha split up, and Scott gets remarried? 
We'd then have to add another column to the child table for his new partner. Since parents can have multiple 
(or "MANY") children, and children can have multiple (or "MANY") parents, this can get very confusing very fast.

It's much easier to do a Many-to-Many OR "Bridge" Table to sort out this data.

I present: The parent_child table!

Instead of adding parent IDs directly to the child table, we can create two independent tables, and then BRIDGE 
the difference between them. 

The Parent Table gets created the same way: 

```
CREATE TABLE parent(
	parent_id SERIAL PRIMARY KEY,
	parent_name VARCHAR(20)
);
```
```
INSERT INTO parent
(parent_name)
VALUES
('Martha'), ('Johnathan'), ('Bernadette');
```

But the Child table no longer requires the parent_id

```
CREATE TABLE child(
	child_id SERIAL PRIMARY KEY,
	child_name VARCHAR(20)
);
```
```
INSERT INTO child
(child_name)
VALUES
('Joey'), ('Brittney'), ('Carlos'), ('Samantha');
```

Now, we create/add information to our parent_child table

```
CREATE TABLE parent_child(
    parent_child_id SERIAL PRIMARY KEY,
    parent_id INT REFERENCES parent(parent_id),
    child_id INT REFERENCES child(child_id)
);
```

We get our parent_id from the parent table, and the child_id from the child table

```
INSERT INTO parent_child
(parent_id, child_id)
VALUES
(1, 1), (1, 2), (2, 1), (2, 2), (3, 3), (3, 4);
```

We can see the full list by running a query:

```
SELECT * FROM parent_child;
```

Now, when someone comes to pick up their kids, we can run a quick JOIN query to see who they are authorized to pick up.

```
SELECT pc.parent_child_id, c.child_id, c.child_name, p.parent_id, p.parent_name
FROM parent_child pc
JOIN parent p ON pc.parent_id = p.parent_id
JOIN child c ON pc.child_id = c.child_id
WHERE parent_id = 4;
```

This table BRIDGES the distance between the Parent and Child tables. 


Another common example would be any company that wants to track their products, invoices, and invoice details.

Product table creation

```
CREATE TABLE product(
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(50)
);
```
```
INSERT INTO product
(product_name)
VALUES
('Motorcycle Grips'), ('Life-size Avatar Aang Cutout'), ('Wendys 4 for 4 Voucher'), ('Mechanical Keyboard'), 
('24-pack Monster: Zero Ultra'), ('Magic: The Gathering booster pack');
```

Invoice creation

```
CREATE TABLE invoice(
    invoice_id SERIAL PRIMARY KEY,
    invoice_date VARCHAR(50)
);
```
```
INSERT INTO invoice
(invoice_date)
VALUES
('2/3/2021'), ('3/5/2021'), ('4/4/2021'), ('6/13/2021'), ('10/5/2021');
```


Invoice Details creation

```
CREATE TABLE invoice_details(
    invoice_details_id SERIAL PRIMARY KEY,
    product_id INT REFERENCES product(product_id),
    invoice_id INT REFERENCES invoice(invoice_id)
);
```
```
INSERT INTO invoice_details
(product_id, invoice_id)
VALUES
(1, 1), (1, 3), (2, 2), (2, 3), (3, 5), (3, 4);
```
```
SELECT * FROM invoice_details;
```

JOIN Query to view all data together

```
SELECT invoice_details_id, p.product_id, p.product_name, i.invoice_id, i.invoice_date
FROM invoice_details id
JOIN invoice i ON id.invoice_id = i.invoice_id
JOIN product p on id.product_id = p.product_id;
```