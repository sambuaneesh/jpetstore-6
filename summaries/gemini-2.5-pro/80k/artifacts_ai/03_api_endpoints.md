| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| `CatalogActionBean` | GET | `/actions/Catalog.action` | Displays the main store page (`viewMain`). |
| `CatalogActionBean` | GET | `/actions/Catalog.action` | Displays products for a selected category (`viewCategory`). |
| `CatalogActionBean` | GET | `/actions/Catalog.action` | Displays items for a selected product (`viewProduct`). |
| `CatalogActionBean` | GET | `/actions/Catalog.action` | Displays details for a specific item (`viewItem`). |
| `CatalogActionBean` | POST | `/actions/Catalog.action` | Searches for products based on keywords and displays results (`searchProducts`). |
| `AccountActionBean` | GET | `/actions/Account.action` | Displays the user sign-on form (`signonForm`). |
| `AccountActionBean` | POST | `/actions/Account.action` | Processes user credentials to sign them in (`signon`). |
| `AccountActionBean` | GET | `/actions/Account.action` | Signs the current user out of their session (`signoff`). |
| `AccountActionBean` | GET | `/actions/Account.action` | Displays the new account registration form (`newAccountForm`). |
| `AccountActionBean` | POST | `/actions/Account.action` | Creates a new user account from submitted form data (`newAccount`). |
| `AccountActionBean` | GET | `/actions/Account.action` | Displays the form to edit the current user's account details (`editAccountForm`). |
| `AccountActionBean` | POST | `/actions/Account.action` | Updates the current user's account from submitted form data (`editAccount`). |
| `CartActionBean` | POST | `/actions/Cart.action` | Adds a selected item to the user's shopping cart (`addItemToCart`). |
| `CartActionBean` | POST | `/actions/Cart.action` | Removes a selected item from the user's shopping cart (`removeItemFromCart`). |
| `CartActionBean` | POST | `/actions/Cart.action` | Updates the quantities of items in the shopping cart (`updateCartQuantities`). |
| `CartActionBean` | GET | `/actions/Cart.action` | Displays the contents of the user's shopping cart (`viewCart`). |
| `CartActionBean` | POST | `/actions/Cart.action` | Initiates the checkout process from the cart view (`checkOut`). |
| `OrderActionBean` | GET | `/actions/Order.action` | Displays a list of the current user's past orders (`listOrders`). |
| `OrderActionBean` | GET | `/actions/Order.action` | Displays the initial form for creating a new order (`newOrderForm`). |
| `OrderActionBean` | POST | `/actions/Order.action` | Processes and confirms a new order through a multi-step flow (`newOrder`). |
| `OrderActionBean` | GET | `/actions/Order.action` | Displays the detailed view of a specific past order (`viewOrder`). |