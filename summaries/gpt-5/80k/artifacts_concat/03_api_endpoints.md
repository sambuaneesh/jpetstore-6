| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Static Pages | GET | /index.html | Entry page linking to the store (navigates to Catalog main). |
| Static Pages | GET | /help.html | Static help page. |
| AccountActionBean | GET | /actions/Account.action | Show sign-in form (default handler). |
| AccountActionBean | POST | /actions/Account.action?signon | Authenticate user; establish session; redirect to catalog. |
| AccountActionBean | GET | /actions/Account.action?signoff | Sign out; invalidate session; redirect to catalog. |
| AccountActionBean | GET | /actions/Account.action?newAccountForm | Render registration form. |
| AccountActionBean | POST | /actions/Account.action?newAccount | Create new account; auto sign-in; redirect to catalog. |
| AccountActionBean | GET | /actions/Account.action?editAccountForm | Render account edit form. |
| AccountActionBean | POST | /actions/Account.action?editAccount | Update account/profile (and password if provided); redirect to catalog. |
| CatalogActionBean | GET | /actions/Catalog.action | Main catalog page. |
| CatalogActionBean | GET | /actions/Catalog.action?viewCategory&categoryId={id} | View product list for the specified category. |
| CatalogActionBean | GET | /actions/Catalog.action?viewProduct&productId={id} | View item list for the specified product. |
| CatalogActionBean | GET | /actions/Catalog.action?viewItem&itemId={id} | View item details. |
| CatalogActionBean | GET, POST | /actions/Catalog.action?searchProducts | Search products by keyword. |
| CartActionBean | GET | /actions/Cart.action?viewCart | Display cart contents. |
| CartActionBean | GET | /actions/Cart.action?checkOut | Show checkout summary page. |
| CartActionBean | GET | /actions/Cart.action?addItemToCart&workingItemId={itemId} | Add item to cart (increments if already present). |
| CartActionBean | GET | /actions/Cart.action?removeItemFromCart&workingItemId={itemId} | Remove item from cart. |
| CartActionBean | POST | /actions/Cart.action?updateCartQuantities | Update quantities for cart items (fields named by itemId). |
| OrderActionBean | GET | /actions/Order.action?listOrders | List past orders for the signed-in user. |
| OrderActionBean | GET | /actions/Order.action?newOrderForm | Start checkout; initialize order from account and cart. |
| OrderActionBean | GET, POST | /actions/Order.action?newOrder | Multi-step order submission: shipping address, confirm, place order. |
| OrderActionBean | GET | /actions/Order.action?viewOrder&orderId={id} | View order details (requires ownership). |