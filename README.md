# Monetization Dashboard
## Final Dashboard
Final dashboard done in Looker Studio can be found [here.](https://lookerstudio.google.com/reporting/5fa15879-49d3-4d8a-bc72-cace8e3df367) 
For analysis I used data starting from 2017 January, because data for 2016 wasn't providing useful insights. 
Main KPIs are compared 2017 Jan-Aug to 2018 Jan-Aug. 
## Dataset
This dashboard was made using Brazilian ecommerce public dataset of orders made at Olist Store. The dataset has information of 100k orders from 2016 to 2018 made at multiple marketplaces in Brazil. Its features allows viewing an order from multiple dimensions: from order status, price, payment and freight performance to customer location and product categories. Also geolocation dataset relased that relates Brazilian zip codes to lat/lng coordinates.
Dataset can be found [here.](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
## Project Task
Change overall look of the dashboard. Provide analytical insights, what are the drawbacks of this analysis, what further analysis could you recommend?

![Original Dashboard (2)](https://user-images.githubusercontent.com/116706695/237001968-be506d77-180e-40b0-8205-16c10cab02df.jpg)

## Preparing tables
### Customer Table
First I decided to create customer table, pulling out order_id,customer_id,customer_state and purchase timestamp. 
```
SELECT order_id,
       order_purchase_timestamp,
       customers.customer_id,
       CASE	
         WHEN customer_state = 'RR' THEN 'Roraima'	
         WHEN customer_state = 'RN' THEN 'Rio Grande do Norte'	
         WHEN customer_state = 'CE' THEN 'Ceará'	
         WHEN customer_state = 'RS' THEN 'Rio Grande do Sul'	
         WHEN customer_state = 'SC' THEN 'Santa Catarina'	
         WHEN customer_state = 'SP' THEN 'São Paulo'	
         WHEN customer_state = 'MG' THEN 'Minas Gerais'	
         WHEN customer_state = 'BA' THEN 'Bahia'	
         WHEN customer_state = 'RJ' THEN 'Rio de Janeiro'	
         WHEN customer_state = 'GO' THEN 'Goiás'	
         WHEN customer_state = 'MA' THEN 'Maranhão'	
         WHEN customer_state = 'PE' THEN 'Pernambuco'	
         WHEN customer_state = 'PB' THEN 'Paraíba'	
         WHEN customer_state = 'ES' THEN 'Espírito Santo'	
         WHEN customer_state = 'PR' THEN 'Paraná'	
         WHEN customer_state = 'RO' THEN 'Rondônia'	
         WHEN customer_state = 'MS' THEN 'Mato Grosso do Sul'	
         WHEN customer_state = 'PA' THEN 'Pará'	
         WHEN customer_state = 'TO' THEN 'Tocantins'	
         WHEN customer_state = 'MT' THEN 'Mato Grosso'	
         WHEN customer_state = 'PI' THEN 'Piauí'	
         WHEN customer_state = 'AL' THEN 'Alagoas'	
         WHEN customer_state = 'AM' THEN 'Amazonas'	
         WHEN customer_state = 'DF' THEN 'Federal District'	
         WHEN customer_state = 'SE' THEN 'Sergipe'	
         WHEN customer_state = 'AP' THEN 'Amapá'	
         WHEN customer_state = 'AC' THEN 'Acre'	
         ELSE customer_state	
       END AS customer_state,
FROM `olist_db.olist_orders_dataset` orders 
JOIN `olist_db.olist_customesr_dataset` customers
ON orders.customer_id = customers.customer_id
```
Table result:
<img width="700" alt="Customers table" src="https://user-images.githubusercontent.com/116706695/237003775-d9357d34-8a10-4dca-926e-2ac0eebbd96a.PNG">

### Sellers Table
Then I prepared sellers table, joining items to have order_id, because I will need it when working on my final dashboard. 

```
SELECT order_id, 
       sellers.seller_id,
       CASE	
         WHEN seller_state = 'RR' THEN 'Roraima'	
         WHEN seller_state = 'RN' THEN 'Rio Grande do Norte'	
         WHEN seller_state = 'CE' THEN 'Ceará'	
         WHEN seller_state = 'RS' THEN 'Rio Grande do Sul'	
         WHEN seller_state = 'SC' THEN 'Santa Catarina'	
         WHEN seller_state = 'SP' THEN 'São Paulo'	
         WHEN seller_state = 'MG' THEN 'Minas Gerais'	
         WHEN seller_state = 'BA' THEN 'Bahia'	
         WHEN seller_state = 'RJ' THEN 'Rio de Janeiro'	
         WHEN seller_state = 'GO' THEN 'Goiás'	
         WHEN seller_state = 'MA' THEN 'Maranhão'	
         WHEN seller_state = 'PE' THEN 'Pernambuco'	
         WHEN seller_state = 'PB' THEN 'Paraíba'	
         WHEN seller_state = 'ES' THEN 'Espírito Santo'	
         WHEN seller_state = 'PR' THEN 'Paraná'	
         WHEN seller_state = 'RO' THEN 'Rondônia'	
         WHEN seller_state = 'MS' THEN 'Mato Grosso do Sul'	
         WHEN seller_state = 'PA' THEN 'Pará'	
         WHEN seller_state = 'TO' THEN 'Tocantins'	
         WHEN seller_state = 'MT' THEN 'Mato Grosso'	
         WHEN seller_state = 'PI' THEN 'Piauí'	
         WHEN seller_state = 'AL' THEN 'Alagoas'	
         WHEN seller_state = 'AM' THEN 'Amazonas'	
         WHEN seller_state = 'DF' THEN 'Federal District'	
         WHEN seller_state = 'SE' THEN 'Sergipe'	
         WHEN seller_state = 'AP' THEN 'Amapá'	
         WHEN seller_state = 'AC' THEN 'Acre'	
         ELSE seller_state	
         END AS seller_state,
FROM `olist_db.olist_sellers_dataset` sellers
JOIN `olist_db.olist_order_items_dataset` items 
ON sellers.seller_id = items.seller_id
```
Sellers table result: 
<img width="600" alt="Sellers table" src="https://user-images.githubusercontent.com/116706695/237004346-c29a26e6-642c-4fc4-a67f-1130646a0f2d.PNG">

### Orders, Revenue table
My third table is about orders, revenue, freight price.
```
WITH products AS (
              SELECT orders.order_id,
                     products.product_id,
                     order_purchase_timestamp,
                     order_approved_at,
                     order_delivered_customer_date,
                     order_estimated_delivery_date,
                     freight_value,
                     payment_value,
                     product_category_name,
                     string_field_1
              FROM `olist_db.product_category_name_translation` translation 
              LEFT JOIN `olist_db.olist_products_dataset` products
              ON translation.string_field_0 = products.product_category_name
              LEFT JOIN `olist_db.olist_order_items_dataset` items 
              ON products.product_id = items.product_id
              LEFT JOIN `olist_db.olist_orders_dataset` orders
              ON items.order_id = orders.order_id
              JOIN `olist_db.olist_order_payments_dataset` payments 
              ON orders.order_id = payments.order_id
              WHERE order_purchase_timestamp > '2016-12-31 23:59:59 UTC'
                ),
row_table AS (
           SELECT *,
                  ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY order_purchase_timestamp) AS row_nmr 
           FROM products)
SELECT order_id,
       order_purchase_timestamp,
       order_approved_at,
       order_delivered_customer_date,
       order_estimated_delivery_date,
       freight_value,
       payment_value,
       product_category_name,
       string_field_1
FROM row_table
WHERE row_nmr = 1
```
Orders / Revenue table results: 
<img width="1000" alt="Revenue Orders" src="https://user-images.githubusercontent.com/116706695/237006726-ea7cd9c8-df96-4ee5-b4aa-f86a0eb40568.PNG">

## Final Dashboard
### KPIs and Orders
We can see good overall performace of the marketplace. Revenue and orders increased significantly. The average payment has a slight increase. Our average delivery days have decreased by 3% compared to 2017, which means we are going in the right direction to deliver orders faster. Unfortunately in 2018 by 4.5% we had an increase in approving orders. We will change further to see what are the reasons for the change. 
When we look at the orders performance table we can see a growing trend with higher spikes during festive periods like Christmas and famous carnival weeks.
Maps show us, that the majority of our orders and sellers are coming from the south of the country, we do not have much business in the north. What might be the reason? Is the business in the north affected because we do not have enough sellers there?

![KPIs Orders-page-001](https://user-images.githubusercontent.com/116706695/237009318-b78b01ab-87a7-40ff-b9c3-4e02ce53e027.jpg)

### Shipping 
When we look at the Shipping page we see, that during festive weeks our average delivery time is increasing, there is a correlation with the amount of orders, so the more customers order - the longer we deliver. Knowing this tendency we could prepare for the upcoming festive season, allocating additional power to the delivery. When I check the average hours to approve the chart, I could not find the reason for the drastic spikes going up in 2018 and going low in 2017. To investigate this, I recommend going deeper into data and checking what states and customers are approving orders longer than on average. Starting from there we can see what was happening locally, maybe some state holidays influenced order approving time. Furthermore, we can see that states located far from the south are waiting longer for orders to be delivered and on average they are paying higher freight prices. This is why our north business is not performing as well as the south. In order to fix it we should onboard more sellers from the north. It will help decreaseTo freight price and delivery time, and as a result, we will increase our revenue.
![Shipping-page-001](https://user-images.githubusercontent.com/116706695/237009317-1b9fa98e-4908-46a0-9a8f-b761d99bafae.jpg)

### Products and Payment 
Pareto chart shows us, that out of 70 categories that we have, only 17 are bringing us 80% of total revenue. And TOP 5 categories are bringing 40% of sales. This is very useful information because we know what category of sellers we have to focus on first, and when will start expansion to the north. We need to find not only new sellers in the north, but we have to find popular category sellers if we want to succeed. 

When we look at payment types charts, we see that boleto and credit card payments are going in a similar direction. What is interesting here is that on June 2018 we had 3 times higher revenue coming from debit cards and it followed even with a higher increase in July. We have to check further what has caused this high change and how we could support debit card payers. 

![Products Payment Types-page-001](https://user-images.githubusercontent.com/116706695/237009315-1aa165af-5155-40d5-a1fd-7c512c3c12e8.jpg)
