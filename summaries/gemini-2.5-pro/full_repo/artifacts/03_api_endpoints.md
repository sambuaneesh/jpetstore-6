| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Account | GET | `/actions/Account.action` | Displays the user sign-in form. |
| Account | POST | `/actions/Account.action?signon` | Processes user login credentials. |
| Account | GET | `/actions/Account.action?signoff` | Logs the current user out of the application. |
| Account | GET | `/actions/Account.action?newAccountForm` | Displays the new user registration form. |
| Account | POST | `/actions/Account.action?newAccount` | Processes the submission of a new user account. |
| Account | GET | `/actions/Account.action?editAccountForm` | Displays the form for the user to edit their account details. |
| Account | POST | `/actions/Account.action?editAccount` | Processes the submission of updated user account details. |
| Catalog | GET | `/actions/Catalog.action` | Displays the main storefront page. |
| Catalog | GET | `/actions/Catalog.action?viewCategory` | Displays all products within a specified category. |
| Catalog | GET | `/actions/Catalog.action?viewProduct` | Displays all available items for a specified product. |
| Catalog | GET | `/actions/Catalog.action?viewItem` | Displays the details for a single item. |
| Catalog | GET/POST | `/actions/Catalog.action?searchProducts` | Searches the product catalog based on a keyword. |
| Cart | GET | `/actions/Cart.action?viewCart` | Displays the contents of the user's shopping cart. |
| Cart | GET | `/actions/Cart.action?addItemToCart` | Adds a specified item to the shopping cart. |
| Cart | GET | `/actions/Cart.action?removeItemFromCart` | Removes a specified item from the shopping cart. |
| Cart | POST | `/actions/Cart.action?updateCartQuantities` | Updates the quantities of items in the shopping cart. |
| Cart | GET | `/actions/Cart.action?checkOut` | Displays the checkout summary page. |
| Order | GET | `/actions/Order.action?listOrders` | Displays a list of all orders for the logged-in user. |
| Order | GET | `/actions/Order.action?newOrderForm` | Displays the initial form for creating a new order. |
| Order | POST | `/actions/Order.action?newOrder` | Submits a new order and guides the user through shipping and confirmation. |
| Order | GET | `/actions/Order.action?viewOrder` | Displays the details of a specific, previously placed order. |