| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| `CatalogActionBean` | GET | `/actions/Catalog.action` | Displays the main store page with featured categories (`viewMain`). |
| `CatalogActionBean` | GET | `/actions/Catalog.action` | Displays all products within a selected category (`viewCategory`). |
| `CatalogActionBean` | GET | `/actions/Catalog.action` | Displays all available items for a selected product (`viewProduct`). |
| `CatalogActionBean` | GET | `/actions/Catalog.action` | Displays the details for a single, specific item (`viewItem`). |
| `CatalogActionBean` | POST | `/actions/Catalog.action` | Processes a keyword search for products (`searchProducts`). |
| `AccountActionBean` | GET | `/actions/Account.action` | Displays the user sign-in form (`signonForm`). |
| `AccountActionBean` | POST | `/actions/Account.action` | Processes user credentials to log in (`signon`). |
| `AccountActionBean` | GET | `/actions/Account.action` | Logs the current user out and invalidates their session (`signoff`). |
| `AccountActionBean` | GET | `/actions/Account.action` | Displays the new user registration form (`newAccountForm`). |
| `AccountActionBean` | POST | `/actions/Account.action` | Processes form data to create a new user account (`newAccount`). |
| `AccountActionBean` | GET | `/actions/Account.action` | Displays the form for the logged-in user to edit their profile (`editAccountForm`). |
| `AccountActionBean` | POST | `/actions/Account.action` | Processes form data to update the user's account details (`editAccount`). |
| `CartActionBean` | POST | `/actions/Cart.action` | Adds a selected item to the user's shopping cart (`addItemToCart`). |
| `CartActionBean` | POST | `/actions/Cart.action` | Removes a selected item from the user's shopping cart (`removeItemFromCart`). |
| `CartActionBean` | POST | `/actions/Cart.action` | Updates the quantities of items in the shopping cart (`updateCartQuantities`). |
| `CartActionBean` | GET | `/actions/Cart.action` | Displays the contents of the user's shopping cart (`viewCart`). |
| `CartActionBean` | GET | `/actions/Cart.action` | Transitions the user from the cart to the order creation process (`checkOut`). |
| `OrderActionBean` | GET | `/actions/Order.action` | Displays a list of the current user's past orders (`listOrders`). |
| `OrderActionBean` | GET | `/actions/Order.action` | Displays the form for creating a new order with billing details (`newOrderForm`). |
| `OrderActionBean` | POST | `/actions/Order.action` | Processes the multi-step checkout to create and confirm a new order (`newOrder`). |
| `OrderActionBean` | GET | `/actions/Order.action` | Displays the details of a single, specific order (`viewOrder`). |