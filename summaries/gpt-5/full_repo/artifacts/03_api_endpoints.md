| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Catalog | GET | /actions/Catalog.action | Main storefront page (default handler: viewMain). |
| Catalog | GET | /actions/Catalog.action?viewCategory&categoryId={id} | List products in a category. |
| Catalog | GET | /actions/Catalog.action?viewProduct&productId={id} | List items for a product. |
| Catalog | GET | /actions/Catalog.action?viewItem&itemId={id} | View item detail. |
| Catalog | GET | /actions/Catalog.action?searchProducts&keyword={q} | Search products by keyword. |
| Account | GET | /actions/Account.action?signonForm | Render sign-in page. |
| Account | POST | /actions/Account.action?signon | Authenticate user and start session. |
| Account | GET | /actions/Account.action?signoff | Sign out and invalidate session. |
| Account | GET | /actions/Account.action?newAccountForm | Render user registration form. |
| Account | POST | /actions/Account.action?newAccount | Create a new user account. |
| Account | GET | /actions/Account.action?editAccountForm | Render account/profile edit form. |
| Account | POST | /actions/Account.action?editAccount | Update account/profile details. |
| Cart | GET | /actions/Cart.action?viewCart | View shopping cart. |
| Cart | GET | /actions/Cart.action?addItemToCart&workingItemId={itemId} | Add an item to cart (increments if already present). |
| Cart | GET | /actions/Cart.action?removeItemFromCart&workingItemId={itemId} | Remove an item from cart. |
| Cart | POST | /actions/Cart.action?updateCartQuantities | Update quantities for items in cart (form fields per itemId). |
| Cart | GET | /actions/Cart.action?checkOut | View checkout summary of current cart. |
| Order | GET | /actions/Order.action?listOrders | List orders for the signed-in user. |
| Order | GET | /actions/Order.action?newOrderForm | Start checkout; show payment form (requires sign-in). |
| Order | POST | /actions/Order.action?newOrder | Checkout flow: handle shipping address, confirm, and place order. |
| Order | GET | /actions/Order.action?viewOrder&orderId={id} | View details of a specific order (must belong to user). |
| Static Pages | GET | /index.html | Entry page (links to the store). |
| Static Pages | GET | /help.html | Help and usage documentation. |