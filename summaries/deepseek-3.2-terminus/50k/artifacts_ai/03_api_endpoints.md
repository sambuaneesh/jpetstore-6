```markdown
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---------------|-------------|---------------|-------------------|
| CatalogActionBean | GET | `/actions/Catalog.action?viewCategory&categoryId={categoryId}` | View products in a specific category |
| CatalogActionBean | GET | `/actions/Catalog.action?viewProduct&productId={productId}` | View details of a specific product |
| CatalogActionBean | GET | `/actions/Catalog.action?viewItem&itemId={itemId}` | View details of a specific item |
| CatalogActionBean | GET | `/actions/Catalog.action?searchProducts&keyword={keyword}` | Search products by keywords |
| CartActionBean | POST | `/actions/Cart.action?addItemToCart&workingItemId={itemId}` | Add an item to the shopping cart |
| CartActionBean | POST | `/actions/Cart.action?removeItemFromCart&workingItemId={itemId}` | Remove an item from the shopping cart |
| CartActionBean | POST | `/actions/Cart.action?updateCartQuantities` | Update quantities of items in the cart |
| CartActionBean | GET | `/actions/Cart.action?viewCart` | View the current shopping cart |
| OrderActionBean | GET | `/actions/Order.action?newOrderForm` | Display new order creation form |
| OrderActionBean | POST | `/actions/Order.action?newOrder` | Submit and process a new order |
| OrderActionBean | GET | `/actions/Order.action?listOrders` | List order history for current user |
| OrderActionBean | GET | `/actions/Order.action?viewOrder` | View details of a specific order |
| AccountActionBean | POST | `/actions/Account.action?signon` | Authenticate user login |
| AccountActionBean | GET | `/actions/Account.action?signoff` | Log out current user |
| AccountActionBean | POST | `/actions/Account.action?newAccount` | Create a new user account |
| AccountActionBean | POST | `/actions/Account.action?editAccount` | Update user account information |
```