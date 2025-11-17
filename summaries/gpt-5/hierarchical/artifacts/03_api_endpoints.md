| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| CatalogActionBean | GET | /actions/Catalog.action | Catalog landing/main page |
| CatalogActionBean | GET | /actions/Catalog.action?categoryId={categoryId} [event=viewCategory] | View a category and its products |
| CatalogActionBean | GET | /actions/Catalog.action?productId={productId} [event=viewProduct] | View a product and its item list |
| CatalogActionBean | GET | /actions/Catalog.action?itemId={itemId} [event=viewItem] | View item details |
| CatalogActionBean | GET | /actions/Catalog.action?keyword={keyword} [event=searchProducts] | Search products by keyword |
| CartActionBean | GET | /actions/Cart.action [event=viewCart] | View the shopping cart |
| CartActionBean | POST | /actions/Cart.action?workingItemId={itemId} [event=addItemToCart] | Add an item to the cart (increments if present) |
| CartActionBean | POST | /actions/Cart.action [event=updateCartQuantities] | Update cart item quantities |
| CartActionBean | POST | /actions/Cart.action?workingItemId={itemId} [event=removeItemFromCart] | Remove an item from the cart |
| CartActionBean | POST | /actions/Cart.action [event=clear] | Clear all items from the cart |
| AccountActionBean | GET | /actions/Account.action [event=signonForm] | Display the sign-in page |
| AccountActionBean | POST | /actions/Account.action [event=signon] | Authenticate user and start session |
| AccountActionBean | POST | /actions/Account.action [event=signoff] | Sign out and invalidate session |
| AccountActionBean | GET | /actions/Account.action [event=newAccountForm] | Display the registration form |
| AccountActionBean | POST | /actions/Account.action [event=newAccount] | Create a new user account |
| AccountActionBean | GET | /actions/Account.action [event=editAccountForm] | Display the account edit form |
| AccountActionBean | POST | /actions/Account.action [event=editAccount] | Update account profile/preferences |
| OrderActionBean | GET | /actions/Order.action [event=newOrderForm] | Begin checkout; initialize order from account and cart |
| OrderActionBean | POST | /actions/Order.action [event=newOrder] | Create order in session and proceed to shipping |
| OrderActionBean | GET | /actions/Order.action [event=shippingForm] | Collect/confirm shipping address details |
| OrderActionBean | POST | /actions/Order.action [event=confirmOrder] | Confirm order details prior to submission |
| OrderActionBean | POST | /actions/Order.action [event=submitOrder] | Persist order, update inventory, clear cart |
| OrderActionBean | GET | /actions/Order.action?orderId={orderId} [event=viewOrder] | View a specific order (owner-only) |
| OrderActionBean | GET | /actions/Order.action [event=listOrders] | List current user’s order history |