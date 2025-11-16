| Component Name | HTTP Method | Endpoint Path | Brief Description |
|----------------|-------------|---------------|-------------------|
| AccountActionBean | POST | `/actions/Account.action?signon` | User authentication and login |
| AccountActionBean | POST | `/actions/Account.action?newAccount` | Create new user account |
| AccountActionBean | POST | `/actions/Account.action?editAccount` | Update user account profile |
| AccountActionBean | GET/POST | `/actions/Account.action?signoff` | User logout and session cleanup |
| CatalogActionBean | GET | `/actions/Catalog.action?viewCategory&categoryId={id}` | Browse products by category |
| CatalogActionBean | GET | `/actions/Catalog.action?viewProduct&productId={id}` | View product details |
| CatalogActionBean | GET | `/actions/Catalog.action?viewItem&itemId={id}` | View specific item details |
| CatalogActionBean | GET | `/actions/Catalog.action?searchProducts&keyword={terms}` | Search products by keywords |
| CartActionBean | POST | `/actions/Cart.action?addItemToCart&workingItemId={id}` | Add item to shopping cart |
| CartActionBean | POST | `/actions/Cart.action?removeItemFromCart&workingItemId={id}` | Remove item from shopping cart |
| CartActionBean | POST | `/actions/Cart.action?updateCartQuantities` | Update item quantities in cart |
| CartActionBean | GET | `/actions/Cart.action?viewCart` | Display shopping cart contents |
| OrderActionBean | GET | `/actions/Order.action?newOrderForm` | Display order form for checkout |
| OrderActionBean | POST | `/actions/Order.action?newOrder` | Submit and process new order |
| OrderActionBean | GET | `/actions/Order.action?listOrders` | View user's order history |
| OrderActionBean | GET | `/actions/Order.action?viewOrder` | View specific order details |