## Create tables and loading data

Documentation: Hive Documentation --> User Documentation --> DDL

create database retail_db;  
use retail_db;

create table orders (  
  order_id int,  
  order_date string,  
  order_customer_id int,    
  order_status string    
)  
row format delimited fields terminated by ','  
stored as textfile;  

load data local inpath '/data/retail_db/orders' into table orders;  
insert into table orders select * from  retail_db.orders;  



##  Spark SQL – Queries 

User Documentation --> Queries (select) -->  More Select Syntax: Join

select order_status,  
       case    
            when order_status IN ('CLOSED', 'COMPLETE') then ‘Okay’   
            when order_status IN ('PAYMENT_REVIEW', 'PENDING', 'PENDING_PAYMENT',') then ‘Check’  
            else 'others'  
       end   
from orders;  



select o.order_id, o.order_date, o.order_status, sum(oi.order_item_subtotal)  as order_revenue  
from orders o   
join order_items oi  
on o.order_id = oi.order_item_order_id  
where o.order_status in ('COMPLETE')  
group by o.order_id, o.order_date, o.order_status  
having sum(oi.order_item_subtotal) >= 1000;  
 
