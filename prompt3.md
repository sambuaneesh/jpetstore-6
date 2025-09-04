| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Account | GET | `/actions/Account.action?signonForm` | Displays the user sign-in page. |
| Account | POST | `/actions/Account.action?signon` | Processes user login credentials and authenticates the session. |
| Account | GET | `/actions/Account.action?signoff` | Logs out the current user and invalidates the session. |
| Account | GET | `/actions/Account.action?newAccountForm` | Displays the user registration form. |
| Account | POST | `/actions/Account.action?newAccount` | Submits and creates a new user account and profile. |
| Account | GET | `/actions/Account.action?editAccountForm` | Displays the form for the logged-in user to edit their profile information. |
| Account | POST | `/actions/Account.action?editAccount` | Submits and saves updates to the user's account and profile information. |
| Cart | GET | `/actions/Cart.action?viewCart` | Displays the contents of the current user's shopping cart. |
| Cart | GET | `/actions/Cart.action?addItemToCart` | Adds a specified item to the shopping cart. |
| Cart | GET | `/actions/Cart.action?removeItemFromCart` | Removes a specified item from the shopping cart. |
| Cart | POST | `/actions/Cart.action?updateCartQuantities` | Updates the quantities for all items currently in the shopping cart. |
| Cart | GET | `/actions/Cart.action?checkOut` | Displays the checkout summary page before starting the order process. |
| Catalog | GET | `/actions/Catalog.action` | Displays the main storefront page. |
| Catalog | GET | `/actions/Catalog.action?viewCategory` | Displays all products within a given category (e.g., FISH, DOGS). |
| Catalog | GET | `/actions/Catalog.action?viewProduct` | Displays all available items for a given product (e.g., Angelfish, Poodle). |
| Catalog | GET | `/actions/Catalog.action?viewItem` | Shows the detailed view for a single item (e.g., Large Angelfish). |
| Catalog | POST | `/actions/Catalog.action?searchProducts` | Processes a keyword search and displays a list of matching products. |
| Order | GET | `/actions/Order.action?listOrders` | Displays a list of all past orders for the currently logged-in user. |
| Order | GET | `/actions/Order.action?newOrderForm` | Displays the initial form to start a new order, collecting payment and billing info. |
| Order | POST | `/actions/Order.action?newOrder` | Processes the multi-step order creation including shipping, confirmation, and final submission. |
| Order | GET | `/actions/Order.action?viewOrder` | Displays the complete details for a specific, previously placed order. |