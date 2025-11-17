```markdown
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|----------------|-------------|---------------|-------------------|
| CatalogActionBean | GET | /catalog/viewMain | Display main catalog page |
| CatalogActionBean | GET | /catalog/viewCategory | View a category and its products |
| CatalogActionBean | GET | /catalog/viewProduct | View a product and its items |
| CatalogActionBean | GET | /catalog/viewItem | View an individual item |
| CatalogActionBean | GET | /catalog/searchProducts | Search products by keyword |
| OrderActionBean | GET | /order/listOrders | List orders for the current user |
| OrderActionBean | GET | /order/viewOrder | View a specific order |
| OrderActionBean | POST | /order/newOrder | Start a new order (from cart) |
| OrderActionBean | POST | /order/confirmOrder | Confirm the order (after shipping address) |
| OrderActionBean | POST | /order/shippingForm | Show shipping form (or handle shipping address) |
| AccountActionBean | GET | /account/signonForm | Show signon form |
| AccountActionBean | POST | /account/signon | Sign in |
| AccountActionBean | GET | /account/signoff | Sign out |
| AccountActionBean | GET | /account/newAccountForm | Show registration form |
| AccountActionBean | POST | /account/newAccount | Create new account |
| AccountActionBean | GET | /account/editAccountForm | Show edit account form |
| AccountActionBean | POST | /account/editAccount | Update account |
| CartActionBean | GET | /cart/viewCart | View the cart |
| CartActionBean | POST | /cart/addItemToCart | Add an item to the cart |
| CartActionBean | POST | /cart/removeItemFromCart | Remove an item from the cart |
| CartActionBean | POST | /cart/updateCartQuantities | Update item quantities in the cart |
| CartActionBean | GET | /cart/checkout | Proceed to checkout |
```