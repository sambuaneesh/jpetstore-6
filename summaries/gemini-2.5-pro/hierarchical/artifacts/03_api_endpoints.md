| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Product Catalog | GET | `/actions/Catalog.action` | Display the main product catalog page. |
| Product Catalog | GET | `/actions/Catalog.action?viewCategory` | View all products within a specific category. |
| Product Catalog | GET | `/actions/Catalog.action?viewProduct` | View the details of a specific product. |
| Product Catalog | GET | `/actions/Catalog.action?viewItem` | View the details of a specific item (SKU). |
| Product Catalog | POST | `/actions/Catalog.action?searchProducts` | Search for products based on a keyword. |
| Shopping Cart | GET | `/actions/Cart.action` | Display the contents of the current shopping cart. |
| Shopping Cart | GET | `/actions/Cart.action?addItemToCart` | Add a specified item to the shopping cart. |
| Shopping Cart | GET | `/actions/Cart.action?removeItemFromCart` | Remove a specified item from the shopping cart. |
| Shopping Cart | POST | `/actions/Cart.action?updateCartQuantities` | Update the quantities of multiple items in the shopping cart. |
| Shopping Cart | GET | `/actions/Cart.action?checkout` | Initiate the checkout process from the cart view. |
| Order Management | GET | `/actions/Order.action?newOrderForm` | Display the initial checkout form to place a new order. |
| Order Management | POST | `/actions/Order.action?newOrder` | Submit and process a new order from the checkout form. |
| Order Management | GET | `/actions/Order.action?listOrders` | Display the order history for the logged-in user. |
| Order Management | GET | `/actions/Order.action?viewOrder` | View the details of a specific past order. |
| Account Management | GET | `/actions/Account.action?signonForm` | Display the user login form. |
| Account Management | POST | `/actions/Account.action?signon` | Process user login credentials for authentication. |
| Account Management | GET | `/actions/Account.action?signoff` | Log out the current user and invalidate the session. |
| Account Management | GET | `/actions/Account.action?newAccountForm` | Display the new user registration form. |
| Account Management | POST | `/actions/Account.action?newAccount` | Process and create a new customer account. |
| Account Management | GET | `/actions/Account.action?editAccountForm` | Display the form to edit the current user's profile. |
| Account Management | POST | `/actions/Account.action?editAccount` | Process and update the current user's profile information. |