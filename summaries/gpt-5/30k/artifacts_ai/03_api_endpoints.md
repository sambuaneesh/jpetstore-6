| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| AccountActionBean | GET | /actions/Account.action | Display sign-in form (default handler) |
| AccountActionBean | POST | /actions/Account.action?signon | Authenticate user and start session |
| AccountActionBean | GET | /actions/Account.action?signoff | Sign out and invalidate session |
| AccountActionBean | GET | /actions/Account.action?newAccountForm | Display new account creation form |
| AccountActionBean | POST | /actions/Account.action?newAccount | Create a new account |
| AccountActionBean | GET | /actions/Account.action?editAccountForm | Display edit account form |
| AccountActionBean | POST | /actions/Account.action?editAccount | Update existing account details |
| CatalogActionBean | GET | /actions/Catalog.action | Show main catalog page (default handler) |
| CatalogActionBean | GET | /actions/Catalog.action?viewCategory&categoryId={categoryId} | View a category and its products |
| CatalogActionBean | GET | /actions/Catalog.action?viewProduct&productId={productId} | View a product and its items |
| CatalogActionBean | GET | /actions/Catalog.action?viewItem&itemId={itemId} | View item details |
| CatalogActionBean | GET | /actions/Catalog.action?searchProducts&keyword={term} | Search products by keyword(s) |
| CartActionBean | GET | /actions/Cart.action?viewCart | View shopping cart |
| CartActionBean | GET | /actions/Cart.action?addItemToCart&workingItemId={itemId} | Add an item to the cart |
| CartActionBean | GET | /actions/Cart.action?removeItemFromCart&workingItemId={itemId} | Remove an item from the cart |
| CartActionBean | POST | /actions/Cart.action?updateCartQuantities | Update cart item quantities |
| CartActionBean | GET | /actions/Cart.action?checkOut | Begin checkout (review cart) |
| OrderActionBean | GET | /actions/Order.action?listOrders | List current user’s orders |
| OrderActionBean | GET | /actions/Order.action?newOrderForm | Start new order (enter details) |
| OrderActionBean | POST | /actions/Order.action?newOrder | Progress/submit new order (shipping/confirm/place) |
| OrderActionBean | GET | /actions/Order.action?viewOrder&orderId={orderId} | View order details |