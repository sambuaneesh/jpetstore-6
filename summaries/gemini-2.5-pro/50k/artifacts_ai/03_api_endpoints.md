| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| **Account Management** | GET/POST | `/actions/Account.action?signon` | Displays login form and handles user authentication. |
| **Account Management** | GET | `/actions/Account.action?signoff` | Logs the current user out of the application. |
| **Account Management** | GET/POST | `/actions/Account.action?newAccount` | Displays the registration form and handles new user creation. |
| **Account Management** | GET/POST | `/actions/Account.action?editAccount` | Displays the user profile form and processes profile updates. |
| **Catalog & Inventory** | GET | `/actions/Catalog.action?viewCategory` | Displays all products within a given category. |
| **Catalog & Inventory** | GET | `/actions/Catalog.action?viewProduct` | Displays all available items for a given product. |
| **Catalog & Inventory** | GET | `/actions/Catalog.action?viewItem` | Displays the details, price, and inventory status of a single item. |
| **Catalog & Inventory** | GET/POST | `/actions/Catalog.action?searchProducts` | Processes a keyword search and displays matching products. |
| **Shopping Cart Management** | GET | `/actions/Cart.action?addItemToCart` | Adds a specified item to the user's session-based shopping cart. |
| **Shopping Cart Management** | GET | `/actions/Cart.action?removeItemFromCart` | Removes a specified item from the shopping cart. |
| **Shopping Cart Management** | POST | `/actions/Cart.action?updateCartQuantities`| Updates the quantity for one or more items in the cart. |
| **Shopping Cart Management** | GET | `/actions/Cart.action?checkOut` | Finalizes the cart and transitions the user to the order process. |
| **Order Management** | GET/POST | `/actions/Order.action?newOrder` | Displays the order confirmation form and submits a new order. |
| **Order Management** | GET | `/actions/Order.action?listOrders` | Displays a list of all orders placed by the current user. |
| **Order Management** | GET | `/actions/Order.action?viewOrder` | Displays the complete details for a single, specific order. |