| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| AccountActionBean | GET | `/jpetstore/actions/Account.action?signonForm=` | Show the user login page. |
| AccountActionBean | POST | `/jpetstore/actions/Account.action?signon=` | Process user login credentials. |
| AccountActionBean | GET | `/jpetstore/actions/Account.action?signoff=` | Log out the current user. |
| AccountActionBean | GET | `/jpetstore/actions/Account.action?newAccountForm=` | Display the new account registration form. |
| AccountActionBean | POST | `/jpetstore/actions/Account.action?newAccount=` | Create a new user account from submitted form data. |
| AccountActionBean | GET | `/jpetstore/actions/Account.action?editAccountForm=` | Display the form to edit the current user's profile. |
| AccountActionBean | POST | `/jpetstore/actions/Account.action?editAccount=` | Update the current user's account details. |
| CatalogActionBean | GET | `/jpetstore/actions/Catalog.action` | Show the main storefront page (default view). |
| CatalogActionBean | GET | `/jpetstore/actions/Catalog.action?viewCategory=&categoryId={id}` | View all products in a specific category. |
| CatalogActionBean | GET | `/jpetstore/actions/Catalog.action?viewProduct=&productId={id}` | View all items available for a specific product. |
| CatalogActionBean | GET | `/jpetstore/actions/Catalog.action?viewItem=&itemId={id}` | View the details of a single item. |
| CatalogActionBean | POST | `/jpetstore/actions/Catalog.action?searchProducts=` | Search for products by keyword. |
| CartActionBean | GET | `/jpetstore/actions/Cart.action?viewCart=` | Display the contents of the shopping cart. |
| CartActionBean | GET | `/jpetstore/actions/Cart.action?addItemToCart=&workingItemId={id}` | Add a selected item to the shopping cart. |
| CartActionBean | GET | `/jpetstore/actions/Cart.action?removeItemFromCart=&cartItem={id}` | Remove an item from the shopping cart. |
| CartActionBean | POST | `/jpetstore/actions/Cart.action?updateCartQuantities=` | Update quantities of items in the shopping cart. |
| CartActionBean | GET | `/jpetstore/actions/Cart.action?checkOut=` | Initiate the checkout process. |
| OrderActionBean | GET | `/jpetstore/actions/Order.action?newOrderForm=` | Display the initial form for a new order (billing/shipping info). |
| OrderActionBean | POST | `/jpetstore/actions/Order.action?newOrder=` | Submit and create a new order based on the cart. |
| OrderActionBean | GET | `/jpetstore/actions/Order.action?listOrders=` | Display a list of the current user's past orders. |
| OrderActionBean | GET | `/jpetstore/actions/Order.action?viewOrder=&orderId={id}` | View the details of a specific past order. |