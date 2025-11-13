| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| `AccountActionBean` | GET | `/jpetstore/actions/Account.action` | Show login page (default action). |
| `AccountActionBean` | POST | `/jpetstore/actions/Account.action` | Process user login. |
| `AccountActionBean` | GET | `/jpetstore/actions/Account.action?signoff` | Log out user. |
| `AccountActionBean` | GET | `/jpetstore/actions/Account.action?newAccountForm` | Show new account registration form. |
| `AccountActionBean` | POST | `/jpetstore/actions/Account.action` | Create a new user account. |
| `AccountActionBean` | GET | `/jpetstore/actions/Account.action?editAccountForm` | Show edit account form for logged-in user. |
| `AccountActionBean` | POST | `/jpetstore/actions/Account.action` | Update user account details. |
| `CatalogActionBean` | GET | `/jpetstore/actions/Catalog.action` | Show the main storefront page (default action). |
| `CatalogActionBean` | GET | `/jpetstore/actions/Catalog.action?viewCategory=&categoryId={id}` | View all products in a specific category. |
| `CatalogActionBean` | GET | `/jpetstore/actions/Catalog.action?viewProduct=&productId={id}` | View all items for a specific product. |
| `CatalogActionBean` | GET | `/jpetstore/actions/Catalog.action?viewItem=&itemId={id}` | View details of a single item. |
| `CatalogActionBean` | POST | `/jpetstore/actions/Catalog.action` | Search for products by keyword. |
| `CartActionBean` | GET | `/jpetstore/actions/Cart.action` | Display the contents of the shopping cart (default action). |
| `CartActionBean` | GET | `/jpetstore/actions/Cart.action?addItemToCart=&workingItemId={id}` | Add a selected item to the cart. |
| `CartActionBean` | GET | `/jpetstore/actions/Cart.action?removeItemFromCart=&cartItem={id}` | Remove an item from the cart. |
| `CartActionBean` | POST | `/jpetstore/actions/Cart.action` | Update quantities of items in the cart. |
| `CartActionBean` | POST | `/jpetstore/actions/Cart.action` | Proceed to the checkout process. |
| `OrderActionBean` | GET | `/jpetstore/actions/Order.action?newOrderForm` | Prepare a new order from the current cart. |
| `OrderActionBean` | POST | `/jpetstore/actions/Order.action` | Submit a new order. |
| `OrderActionBean` | GET | `/jpetstore/actions/Order.action?listOrders` | View history of the user's past orders. |
| `OrderActionBean` | GET | `/jpetstore/actions/Order.action?viewOrder=&orderId={id}` | View the details of a specific past order. |