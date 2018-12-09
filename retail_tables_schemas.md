# customers-schema
customer_id INT(11) \
customer_fname VARCHAR(45) \
customer_lname VARCHAR(45) \
customer_email VARCHAR(45) \
customer_password VARCHAR(45) \
customer_street VARCHAR(45) \
customer_city VARCHAR(45) \
customer_state VARCHER(45) \
customer_zipcode VARCHAR(45)

# products-schema
product_id INT(11) \
product_category_id INT(11) \
product_name VARCHAR(45) \ 
product_description VARCHAR(255) \
product_price FLOAT \
product_image VARCHAR(255) 

# orders-schema
order_id INT(11) \
order_date DATETIME \
order_customer_id INT(11) \
order_status VARCHAR(45) 


# order-items schema
order_item_id INT(11) \
order_item_order_id INT(11) \
order_item_product_id INT(11) \
order_item_quantity  TINYINT(4) \
order_item_subtotal FLOAT \
order_item_product_price FLOAT \
