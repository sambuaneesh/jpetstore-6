```markdown
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---------------|-------------|---------------|-------------------|
| AccountActionBean | GET | /actions/Account.action | Display signon form (default action) |
| AccountActionBean | GET | /actions/Account.action?newAccount | Display new account registration form |
| AccountActionBean | POST | /actions/Account.action?newAccount | Submit new account registration |
| AccountActionBean | GET | /actions/Account.action?editAccount | Display account editing form |
| AccountActionBean | POST | /actions/Account.action?editAccount | Submit account updates |
| AccountActionBean | POST | /actions/Account.action?signon | User authentication/login |
| AccountActionBean | POST | /actions/Account.action?signoff | User logout/session termination |
| CatalogActionBean | GET | /actions/Catalog.action | Display main catalog homepage |
| CatalogActionBean | GET | /actions/Catalog.action?viewCategory={categoryId} | Display products by category |
| CatalogActionBean | GET | /actions/Catalog.action?viewProduct={productId} | Display product details |
| CatalogActionBean | GET | /actions/Catalog.action?viewItem={itemId} | Display individual item details |
| CatalogActionBean | GET | /actions/Catalog.action?searchProducts={keywords} | Search products by keywords |
| CartActionBean | GET | /actions/Cart.action | Display shopping cart contents |
| CartActionBean | POST | /actions/Cart.action?addItemToCart={itemId} | Add item to shopping cart |
| CartActionBean | POST | /actions/Cart.action?removeItemFromCart={itemId} | Remove item from shopping cart |
| CartActionBean | POST | /actions/Cart.action?updateCartQuantities | Update item quantities in cart |
| CartActionBean | GET | /actions/Cart.action?checkout | Display checkout page |
| OrderActionBean | GET | /actions/Order.action?newOrder | Display new order form |
| OrderActionBean | POST | /actions/Order.action?newOrder | Submit new order creation |
| OrderActionBean | GET | /actions/Order.action?shipping | Display shipping form |
| OrderActionBean | POST | /actions/Order.action?shipping | Submit shipping information |
| OrderActionBean | GET | /actions/Order.action?confirmOrder | Display order confirmation page |
| OrderActionBean | POST | /actions/Order.action?confirmOrder | Submit order confirmation |
| OrderActionBean | GET | /actions/Order.action?listOrders | Display user's order history |
| OrderActionBean | GET | /actions/Order.action?viewOrder={orderId} | Display specific order details |
```