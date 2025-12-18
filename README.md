## Project as part of the “Route 256: Go-middle developer” course from Ozon Tech


### Microservices

1. LOMS (Logistics and Order Management System) - service responsible for order tracking and logistics.
2. Checkout - service responsible for the user's shopping cart and order processing.
3. Notifications - service responsible for sending notifications.
4. ProductService - an external service that provides information about products.

### The path to purchasing goods
* Checkout.addToCart  
    * add to cart and check availability
    * We can delete from the cart
    * We can receive a list of items in the shopping cart
    * The name and price are retrieved from ProductService.get_product.
* We purchase goods through Checkout.purchase
    * Go to LOMS.createOrder and create an order.
    * The order has the status new
    * LOMS reserves the required number of items
    * If the reservation fails, the order status changes to failed.
    * If successful, we fall into the status of awaiting payment.
* Purchase the order
    * Use LOMS.orderPayed
    * Reserves are transferred to write-offs of goods from the warehouse.
    * The order is in the paid status.
* You can cancel your order before payment.
    * Call LOMS.cancelOrder
    * All reservations for the order are canceled, and the items are once again available to other users.
    * The order changes to cancelled status.
    * LOMS should cancel orders itself after a timeout if they have not been paid for within 10 minutes.


## Database schema
<img width="1253" alt="image" src="https://github.com/deerc-dev/openedx-admin/assets/45228812/beb54696-c907-41e5-ab69-6302a6d53801">

# Local development

1. Add a .env file to the deployments folder, similar to example.env, which specifies the parameters for connecting to databases via microservices. 
2. Add config.yaml files to the checkout, loms, and notifications folders, similar to config.example.yaml.
3. Add your SSL certificate to the certs folder.
4. Start the environment for monitoring logs and wait until everything is up and running:
> make run-log-env
5. If all services started without errors in the previous step, start the service environment in a separate terminal:
> make run-services


## Make migrations
1. To perform database migration, you need to install the goose library:
> go install github.com/pressly/goose/v3/cmd/goose@latest

2. Go to the migration folder of the corresponding microservice and run the command:
> goose create \*migration name\* sql

3. In the terminal, import the corresponding environment variable:
> CH_POSTGRES_URL  # connection string for checkout service  
> LOMS_POSTGRES_URL # connection string for loms service  
> NOTIF_POSTGRES_URL # connection string for notifications service
4. do the command:
> make migrate

## Logs monitoring
1. Access Graylog via your browser (default port 7555).
2. Click on “Systems/Inputs” and go to “Inputs.”
3. Select “GELF TCP” and click on the “Launch new input” button.
4. Enable “Global.”
5. In the “Title” field, enter the desired input name (for example, checkout or loms).
6. For checkout, leave port 12201, for loms change to 12202, for notifications change to 12203.

## Trace monitoring
Jaeger is available on port 16686.

## Metrics monitoring: Prometheus + Grafana
1. Go to http://localhost:3000
2. Create a new Data source:
    a. In the Prometheus server URL field, enter http://localhost:9090,
    b. Set the scrape interval to 5s (as in prometheus.yml)
3. Create a new Dashboard and select Prometheus as the data source
4. Select the metric you are interested in
