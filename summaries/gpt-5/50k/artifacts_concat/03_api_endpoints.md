| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| AccountActionBean | GET | /actions/Account.action?signonForm | Display sign-in form (default view). |
| AccountActionBean | POST | /actions/Account.action?signon | Authenticate user with username/password; on success initializes session and redirects to catalog. |
| AccountActionBean | GET | /actions/Account.action?newAccountForm | Display registration form. |
| AccountActionBean | POST | /actions/Account.action?newAccount | Create a new account; expects username/password and account profile/preferences fields. |
| AccountActionBean | GET | /actions/Account.action?editAccountForm | Display account/profile edit form. |
| AccountActionBean | POST | /actions/Account.action?editAccount | Update account/profile; optionally updates password. |
| AccountActionBean | GET, POST | /actions/Account.action?signoff | Sign out; invalidates session and clears state. |
| CatalogActionBean | GET | /actions/Catalog.action?viewMain | Catalog homepage (default view). |
| CatalogActionBean | GET | /actions/Catalog.action?viewCategory&categoryId={categoryId} | View products within a category. |
| CatalogActionBean | GET | /actions/Catalog.action?viewProduct&productId={productId} | View items within a product. |
| CatalogActionBean | GET | /actions/Catalog.action?viewItem&itemId={itemId} | View item details (includes availability). |
| CatalogActionBean | GET, POST | /actions/Catalog.action?searchProducts&keyword={keyword} | Search products by keyword(s). |
| CartActionBean | POST | /actions/Cart.action?addItemToCart&workingItemId={itemId} | Add item to cart; increments quantity if already present. |
| CartActionBean | POST | /actions/Cart.action?removeItemFromCart&workingItemId={itemId} | Remove item from cart. |
| CartActionBean | POST | /actions/Cart.action?updateCartQuantities | Update cart quantities; expects form fields keyed by itemId with integer quantities (removes if < 1). |
| CartActionBean | GET | /actions/Cart.action?viewCart | View shopping cart. |
| CartActionBean | GET | /actions/Cart.action?checkOut | View checkout summary page. |
| OrderActionBean | GET | /actions/Order.action?listOrders | List orders for the authenticated user. |
| OrderActionBean | GET | /actions/Order.action?newOrderForm | Initialize new order from account and cart; displays payment/billing form. |
| OrderActionBean | POST | /actions/Order.action?newOrder | Multi-step order placement; on final confirmation persists order, decrements inventory, and clears cart. |
| OrderActionBean | GET | /actions/Order.action?viewOrder&order.orderId={orderId} | View order details; restricted to the owning user. |