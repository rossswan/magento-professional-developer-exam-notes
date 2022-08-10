# Catalog Notes

## Categories
A category is

We can create a category by implementing `\Magento\Catalog\Api\Data\CategoryInterface`.
This interface gives us access to a lot of helpful methods, notably `getCatagoryPath`.
Category objects are then instantiated with the `CategoryInterfaceFactory` which because of preferences
implements a `\Magento\Catalog\Model\CategoryFactory`. 

### Assigning products to categories
You have a few options
- Admin panel.
- Category save controller
  - load a category
  - get the product assignment list with `getProductsPosition()`
  - add product by passing key-value array to `setPostedProducts()`
  - save the category (and wait for indexer to run)
    - product urls are set when the category is saved

Products may be grouped together to display similar products together.
(i.e. pipes of separate guages)

### `base_` prefixed columns
Base attributes are attributes (like currency) that can be converted to different units corresponding to a stores needs
(i.e. a US store will have base prices mapped to USD, but an Indian store will have base prices mapped to INR)
This is useful for a company that may sell products in different locations with differing currencies, but want all values tracked in their local currency on the backend for bookkeeping

### The Product Model and Product Type Model
**Product Model** - contains *data* from db as it pertains to a specific product
**Product Type Model** - Contains *methods* that apply to a given product type
- `getChildrenIDs()`
- `getParentIdsByChild()`
- `getAssociatedProducts()` - loads all products associated with a given parent

### Custom Shopping Carts
- Create this file: `app/code/[vendor]/[module}/view/frontend/layout/checkout_cart_item_renderers.xml`
- reference this block: `checkout.cart.item.renderers`
- Create new block using the product type as the `as` field
- Set the template (`Magento_Checkout::cart/item/default.phtml` is a common option)
- Make sure block extends `\Magento\Checkout\Block\Cart\Item\Renderer`

## Prices
Prices will be rendered using the `\Magento\Catalog\Model\Product\Type\Price`. This is where calculations occur.
Customization via plugins can be done with a `afterCalculatePrice()` method (or by replacing the price calculation class entirely)

### Types of Prices
**Price visible on product**
- Base Price
- Special Pricing
- Catalog Rule

**Price in shopping cart**
- tiered pricing
- Options price
- Tax/VAT
- Shopping cart rules


### Price Rules
Price rules are found in `catalogrule_product_price`

Price rules can be created in admin panel: *Marketing > Catalog Price Rule*

Catalog price rules are useful for global price updates (like sales) because they apply to a set of products