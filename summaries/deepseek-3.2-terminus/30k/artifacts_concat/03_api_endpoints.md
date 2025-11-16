```markdown
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---------------|-------------|---------------|-------------------|
| AccountActionBean | GET | `/actions/Account.action` | Default signon form display |
| AccountActionBean | GET | `/actions/Account.action?newAccount` | Display new account registration form |
| AccountActionBean | POST | `/actions/Account.action?newAccount` | Submit new account registration |
| AccountActionBean | GET | `/actions/Account.action?editAccount` | Display account editing form |
| AccountActionBean | POST | `/actions/Account.action?editAccount` | Submit account updates |
| AccountActionBean | POST | `/actions/Account.action?signon` | User authentication/login |
| AccountActionBean | GET | `/actions/Account.action?signoff` | User logout/session termination |
| CatalogActionBean | GET | `/actions/Catalog.action` | Main catalog homepage/default view |
| CatalogActionBean | GET | `/actions/Catalog.action?viewCategory` | Display products by category |
| CatalogActionBean | GET | `/actions/Catalog.action?viewProduct` | Display product details |
| CatalogActionBean | GET | `/actions/Catalog.action?viewItem` | Display individual item details |
| CatalogActionBean | GET | `/actions/Catalog.action?searchProducts` | Search products by keywords |
| CartActionBean | POST | `/actions/Cart.action?addItemToCart` | Add item to shopping cart |
| CartActionBean | POST | `/actions/Cart.action?removeItemFromCart` | Remove item from shopping cart |
| CartActionBean | POST | `/actions/Cart.action?updateCartQuantities` | Update item quantities in cart |
| CartActionBean | GET | `/actions/Cart.action?checkout` | Initiate checkout process |
| OrderActionBean | GET | `/actions/Order.action?newOrder` | Start new order creation |
| OrderActionBean | GET | `/actions/Order.action?shippingForm` | Display shipping address form |
| OrderActionBean | POST | `/actions/Order.action?confirmOrder` | Confirm order details |
| OrderActionBean | POST | `/actions/Order.action?submitOrder` | Final order submission |
| OrderActionBean | GET | `/actions/Order.action?listOrders` | Display user order history |
| OrderActionBean | GET | `/actions/Order.action?viewOrder` | View specific order details |
| Web Interface | GET | `/order/ViewOrder.jsp` | Direct order details display page |
| Web Interface | GET | `/Main.jsp` | Application homepage |
| Web Interface | GET | `/Category.jsp` | Category products listing |
| Web Interface | GET | `/Product.jsp` | Product details page |
| Web Interface | GET | `/Item.jsp` | Individual item view |
| Web Interface | GET | `/SearchProducts.jsp` | Search results page |
| Web Interface | GET | `/Cart.jsp` | Shopping cart management page |
| Web Interface | GET | `/Checkout.jsp` | Order summary and checkout page |
| Web Interface | GET | `/NewOrderForm.jsp` | Billing/shipping information form |
| Web Interface | GET | `/ShippingForm.jsp` | Separate shipping address form |
| Web Interface | GET | `/ConfirmOrder.jsp` | Order confirmation page |
| Web Interface | GET | `/ListOrders.jsp` | User order history page |
| Web Interface | GET | `/SignonForm.jsp` | User login page |
| Web Interface | GET | `/NewAccountForm.jsp` | User registration page |
| Web Interface | GET | `/EditAccountForm.jsp` | Account profile management page |
| Web Interface | GET | `/Error.jsp` | Error handling and display page |
```