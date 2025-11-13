| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Catalog | GET | /jpetstore/actions/Catalog.action | Main catalog landing page |
| Catalog | GET | /jpetstore/actions/Catalog.action?viewCategory&categoryId={categoryId} | View a category and its products |
| Catalog | GET | /jpetstore/actions/Catalog.action?viewProduct&productId={productId} | View a product and its items |
| Catalog | GET | /jpetstore/actions/Catalog.action?viewItem&itemId={itemId} | View a specific item’s details |
| Catalog | GET | /jpetstore/actions/Catalog.action?searchProducts&keyword={keywords} | Search products by keyword(s) |
| Account | GET | /jpetstore/actions/Account.action?signonForm | Render sign-in form |
| Account | POST | /jpetstore/actions/Account.action?signon | Authenticate user and start session |
| Account | GET | /jpetstore/actions/Account.action?signoff | Sign out and invalidate session |
| Account | GET | /jpetstore/actions/Account.action?newAccountForm | Render registration form |
| Account | POST | /jpetstore/actions/Account.action?newAccount | Create a new user account |
| Account | GET | /jpetstore/actions/Account.action?editAccountForm | Render account edit form |
| Account | POST | /jpetstore/actions/Account.action?editAccount | Update account details |
| Cart | GET | /jpetstore/actions/Cart.action?viewCart | View shopping cart |
| Cart | POST | /jpetstore/actions/Cart.action?addItemToCart&workingItemId={itemId} | Add item to cart (or increment if present) |
| Cart | POST | /jpetstore/actions/Cart.action?removeItemFromCart&workingItemId={itemId} | Remove item from cart |
| Cart | POST | /jpetstore/actions/Cart.action?updateCartQuantities | Update quantities for items in cart |
| Cart | GET | /jpetstore/actions/Cart.action?checkOut | View checkout summary page |
| Order | GET | /jpetstore/actions/Order.action?listOrders | List orders for the current user |
| Order | GET | /jpetstore/actions/Order.action?newOrderForm | Start new order from cart (pre-fill billing/shipping) |
| Order | POST | /jpetstore/actions/Order.action?newOrder | Multi-step order submission (shipping/confirm/place) |
| Order | GET | /jpetstore/actions/Order.action?viewOrder&orderId={orderId} | View a specific order by ID |
| Web (Static) | GET | /jpetstore/index.html | Home page linking to the catalog |
| Web (Static) | GET | /jpetstore/help.html | Static help page |