| Component Name | HTTP Method | Endpoint Path                             | Brief Description                                                 |
|----------------|-------------|-------------------------------------------|-------------------------------------------------------------------|
| Account        | GET         | `/actions/Account.action?signonForm`      | Displays the user login form.                                     |
| Account        | POST        | `/actions/Account.action?signon`          | Authenticates a user with submitted credentials.                  |
| Account        | GET         | `/actions/Account.action?signoff`         | Logs out the current user and invalidates the session.            |
| Account        | GET         | `/actions/Account.action?newAccountForm`  | Displays the new account registration form.                       |
| Account        | POST        | `/actions/Account.action?newAccount`      | Creates a new user account from submitted form data.              |
| Account        | GET         | `/actions/Account.action?editAccountForm` | Displays the form to edit the current user's account details.     |
| Account        | POST        | `/actions/Account.action?editAccount`     | Updates the current user's account from submitted form data.      |
| Catalog        | GET         | `/actions/Catalog.action`                 | Displays the main catalog page (homepage).                        |
| Catalog        | GET         | `/actions/Catalog.action?viewCategory`    | Displays all products within a specified category.                |
| Catalog        | GET         | `/actions/Catalog.action?viewProduct`     | Displays all items available for a specified product.             |
| Catalog        | GET         | `/actions/Catalog.action?viewItem`        | Displays the details of a single, specific item.                  |
| Catalog        | GET         | `/actions/Catalog.action?searchProducts`  | Searches for products based on a submitted keyword.               |
| Cart           | GET         | `/actions/Cart.action?viewCart`           | Displays the contents of the user's shopping cart.                |
| Cart           | GET         | `/actions/Cart.action?addItemToCart`      | Adds a specific item to the shopping cart.                        |
| Cart           | GET         | `/actions/Cart.action?removeItemFromCart` | Removes a specific item from the shopping cart.                   |
| Cart           | POST        | `/actions/Cart.action?updateCartQuantities` | Updates the quantities for items in the cart from a form submission. |
| Cart           | GET         | `/actions/Cart.action?checkOut`           | Initiates the checkout process for the items in the cart.         |
| Order          | GET         | `/actions/Order.action?listOrders`        | Displays the current user's order history.                        |
| Order          | GET         | `/actions/Order.action?newOrderForm`      | Displays the initial form for placing a new order.                |
| Order          | POST        | `/actions/Order.action?newOrder`          | Submits, confirms, and places a new order.                        |
| Order          | GET         | `/actions/Order.action?viewOrder`         | Displays the details of a specific, previously placed order.      |