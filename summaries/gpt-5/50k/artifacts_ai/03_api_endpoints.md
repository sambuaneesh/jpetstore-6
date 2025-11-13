| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| AccountActionBean | GET | /actions/Account.action?signonForm | Show sign-in form (default handler). |
| AccountActionBean | POST | /actions/Account.action?signon | Authenticate user with username/password; initializes session state. |
| AccountActionBean | GET | /actions/Account.action?newAccountForm | Show new account registration form. |
| AccountActionBean | POST | /actions/Account.action?newAccount | Create account/profile/signon in one transaction; logs user in. |
| AccountActionBean | GET | /actions/Account.action?editAccountForm | Show account edit form. |
| AccountActionBean | POST | /actions/Account.action?editAccount | Update account/profile; updates password if provided. |
| AccountActionBean | GET | /actions/Account.action?signoff | Sign out: invalidate session and clear bean state. |
| AccountActionBean | POST | /actions/Account.action?signoff | Sign out: invalidate session and clear bean state. |
| CatalogActionBean | GET | /actions/Catalog.action?viewMain | Main catalog landing page (default handler). |
| CatalogActionBean | GET | /actions/Catalog.action?viewCategory&categoryId={id} | View a category and its products. |
| CatalogActionBean | GET | /actions/Catalog.action?viewProduct&productId={id} | View a product and its items. |
| CatalogActionBean | GET | /actions/Catalog.action?viewItem&itemId={id} | View an item’s details and availability. |
| CatalogActionBean | GET | /actions/Catalog.action?searchProducts&keyword={q} | Search products by keyword(s). |
| CatalogActionBean | POST | /actions/Catalog.action?searchProducts | Search products by keyword(s) from form submission. |
| CartActionBean | GET | /actions/Cart.action?viewCart | View shopping cart contents. |
| CartActionBean | GET | /actions/Cart.action?checkOut | Checkout review page prior to order creation. |
| CartActionBean | POST | /actions/Cart.action?addItemToCart&workingItemId={itemId} | Add item to cart (increments if already present). |
| CartActionBean | POST | /actions/Cart.action?removeItemFromCart&workingItemId={itemId} | Remove item from cart. |
| CartActionBean | POST | /actions/Cart.action?updateCartQuantities | Update cart line quantities (body params: {itemId: qty}). |
| OrderActionBean | GET | /actions/Order.action?listOrders | List orders for the authenticated user. |
| OrderActionBean | GET | /actions/Order.action?newOrderForm | Begin new order from current cart; requires sign-in. |
| OrderActionBean | POST | /actions/Order.action?newOrder | Create/advance order; uses shippingAddressRequired and confirmed flags to proceed/finalize. |
| OrderActionBean | GET | /actions/Order.action?viewOrder&order.orderId={id} | View a specific order (only allowed for current user). |