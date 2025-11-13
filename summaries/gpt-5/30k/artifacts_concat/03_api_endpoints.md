| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| AccountActionBean | GET | /actions/Account.action | Display sign-in form (default handler: signonForm). |
| AccountActionBean | POST | /actions/Account.action?event=signon | Authenticate user; on success sets session and redirects to catalog; on failure returns to sign-in form. |
| AccountActionBean | GET | /actions/Account.action?event=signoff | Sign out; invalidate session and redirect to catalog. |
| AccountActionBean | GET | /actions/Account.action?event=newAccountForm | Display registration form. |
| AccountActionBean | POST | /actions/Account.action?event=newAccount | Create a new account; set authenticated and redirect to catalog. |
| AccountActionBean | GET | /actions/Account.action?event=editAccountForm | Display account edit form. |
| AccountActionBean | POST | /actions/Account.action?event=editAccount | Update account/profile (and password if provided); redirect to catalog. |
| CatalogActionBean | GET | /actions/Catalog.action | Show main catalog page (default handler: viewMain). |
| CatalogActionBean | GET | /actions/Catalog.action?event=viewCategory | Show products for a category (param: categoryId). |
| CatalogActionBean | GET | /actions/Catalog.action?event=viewProduct | Show items for a product (param: productId). |
| CatalogActionBean | GET | /actions/Catalog.action?event=viewItem | Show item details (param: itemId). |
| CatalogActionBean | GET | /actions/Catalog.action?event=searchProducts | Search products (param: keyword). |
| CartActionBean | GET | /actions/Cart.action?event=viewCart | View cart contents. |
| CartActionBean | POST | /actions/Cart.action?event=addItemToCart | Add an item to the cart (param: workingItemId). |
| CartActionBean | GET | /actions/Cart.action?event=removeItemFromCart | Remove an item from the cart (param: workingItemId). |
| CartActionBean | POST | /actions/Cart.action?event=updateCartQuantities | Update cart item quantities (form inputs named by itemId). |
| CartActionBean | GET | /actions/Cart.action?event=checkOut | Show checkout summary page. |
| OrderActionBean | GET | /actions/Order.action?event=listOrders | List orders for the signed-in user. |
| OrderActionBean | GET | /actions/Order.action?event=newOrderForm | Start new order; initialize from account/cart; requires sign-in. |
| OrderActionBean | POST | /actions/Order.action?event=newOrder | Submit order flow steps (shipping address as needed, confirm, and place order). |
| OrderActionBean | GET | /actions/Order.action?event=viewOrder | View a specific order (param: orderId; restricted to owner). |