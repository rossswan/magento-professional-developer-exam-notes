# Checkout and Sales Notes

## Cart

### Cart Components
- **Shopping Cart** - Structure that holds items a customer is going to buy
  - Shopping Cart Functionality:
    - **Add to Cart** - The most straight forward way. Customer simply clicks 'add to cart'
    - **Reorder Button** - This is found in *MyAccount*. Adds previously ordered items to cart. May present issues if item has changed
    - **Wishlist** - Another option for adding to cart. Pretty straightforward.
    - **Merge Quotes** - Happens when a customer adds an item to cart and then logs into their account. Items from the old (unlogged in) cart are merged into the new (logged in) cart
  - Represented by two classes
    - `Magento\checkout\Model\Cart` - Think of it as the "wrapper"
    - `Magento\Quote\Model\Quote` - Class that performs all operations
      - Adding to cart (also deleting/updating cart)
      - Recollecting totals
      - Keeping track of items and addresses, including which should be separate line items and which should be included in another existing line item (i.e. increasing quantity)
      - Owning a `Payment` object
- **Cart Items** - A customer can see what items are currently in their cart
  - Items are represented by `Magento\Quote\Model\Quote\Item` which has a few important methods
    - `representProduct()` - determines if two items should be joined (i.e. increased in quantity) or separated
    - `setProduct()` - Defines which product attributes are available
  - Cart layout is defined in`Magento/Checkout/view/frontend/layout/checkout_cart_item_renderers.xml`
  - There are several product types to be aware of
    - simple product - think a single item. Mapped 1:1 with a product on the merchant's shelves
    - configurable product - has two parts:
      - configurable sku (the parent, i.e. T-shirt)
      - selected sku (the child, i.e. XL Blue T-shirt)
    - Bundle - a group of items; has one bundle sku, and a sku for each item in bundle
    - virtual - Similar to a simple product, but intangible so it cannot map 1:1 with physical inventory
- **Shipping Address** - Fields to enter zip code/shipping address, triggers a shipping cost calculation
  - Represented by `Magento\Quote\Model\Quote\Address` (as is Billing address). This class hold items to be delivered and also collects shipping rates
- **Discounts** - Field to add promo codes. Managed by *promo rules* in admin panel
  - These are applied via `Magento/SalesRule/etc/sales.xml`
  - Uses the same engine as Catalog rules
  - Had two core elements
    - Conditions - tool to configure boolean function that defines when a rule should be applied
      - Can use EAV attributes to trigger a rule if the "Used in Promo Rule Conditions" property is set to true
    - Actions - what happens wht a condition is true. There are 4 possible options
      - Percent of product price discount
      - Fixed amount discount
      - Fixed amount discount for whole cart
      - Buy X get Y free
- **Totals** - Shows total price accounting for shipping, tax, and discounts
  - Total models can be found in `Magento\Quote\etc\sales.xml`; a Total model does two things:
    - `collect()` - calculates its own total amount
    - `fetch()` - defines what data is needed to render a total

Additional Shopping Cart/Checkout configurations (not exhaustive):
- Enable Onepage checkout
- Enable Multishipping Checkout
- Allow guest checkout
- Display mini cart

### The checkout flow
Two steps:
- Shipping
  - Two substeps:
    - Shipping Address
    - Shipping methods
- Billing
  - four substeps:
    - billing address definition
    - payment method selection
    - submitting
    - order
      - Final step in the checkout process. Sequence of 3 parts:
        - Convert Quote to Order using `Magento\Quote\Model\QuoteManagement` class's `submitOrder()` method
        - Submit payment for order - this places the order
        - Save order/invalidate quote/send configuration email/sometimes create invoice
          - fires event `checkout_submit_all_after`
- Checkout steps are rendered with JS found in `Magento/Checkout/view/frontend/web/js/`
  - because mostly done by JS, we use mixins to customize
  - Follows a modified MVC framework
    - M - `./js/model/`
    - V - `./js/view/`
    - C - `./js/action/` - actions send data (rather than receive them as is typical in MVC)

### The Quote object
`Magento\Quote\Model\Quote` methods to be aware of:
- `beforeSave()`
- `load()`
- `assignCustomer()`
- `getBillingAddress()`
- `getShippingAddress()`
- `getAllShippingAddresses()`
- `getAllItems()`
- `getAllVisibleItems()`
- `addItem()`
- `addProduct()`
- `updateItem()`
- `getIsVirtual()`
- `merge()`
- `collectTotals()`
  
Question? What is the difference between an item and a product? (re: addItem()/addProduct())
- A Product is a general representation of an item
- An item is a specific product (i.e. the one that is in your cart and will be purchased)


We usually get a Quote object using `Magento\Checkout\Model\Session::getQuote()`

### The Address object
A Quote can have only one billing address, though (if multishipping is enabled) it may have multiple shipping addresses.  If the item is virtual, it may even have no shipping address at all.

Totals are calculated with the Address object's `getTotals()` method.

### Items
Items may belong to either the Quote or Address object, however, in most situations you will use the Quote object to get them
- `getAllItems()` - returns all items in a collection after removing deleted items
- `getAllVisibleItems()` - returns all items in a collection after removing all deleted items and all items with a parent item
- `getItemsCollection()` - returns a collection of items

Not all information about an item is stored on the item. Custom options and some product type info is stored in the `quote_item_option` table.

Magento will load some product information when creating the collection. The attributes to be loaded are listed in `etc/catalog_attributes.xml`

### Shipping Method Configuration
Shipping methods are defined as either Offline or Online
- Offline methods are more or less 1st-party shipping methods 
  - FlatRate
  - TableRage - requires special CSV uploaded to system configs
  - Free Shipping
  - In Store delivery
- Online methods use 3rd parties and generally require API calls to obtain shipping rates

### Payment Method Configurations
There are three types of payment methods
- Offline
  - These are all methods that do nothing at checkout
  - Basically, Magento just takes your word for it that a payment happened
  - Check/Money Order, Zero subtotal checkout, COD, etc
- Gateway
  - methods that send a request to a payment provider
- Hosted
  - Methods that redirect the customer to another website to complete payment, then redirect back (i.e. PayPal Express payment)

**Gateway and Hosted methods**
- In its core, Magento only has PayPal as a method
- Other methods would have to be configured with credentials (i.e. API keys)
- There are three principle options when using these methods
  - Authorize Only - Authorizes funds, but an invoice will have to be created manually by an admin, which will then execute the payment and charge the card
  - Capture - Will charge the card and automatically create an invoice
  - Order - Does nothing, Admin must create invoice and charge the card

### Tax and Currency Rules
To configure tax rules you must configure the following:
- Product Tax Classes
- Customer Tax Classes
- Tax rates - tied to a physical location
- Rules relating to the above rates and classes

To configure currencies, you must consider the following:
- Currency Scope - should the product have one global price, or separate prices for each different website (i.e. UK site vs US site)
- Store/Display Currency - Currency that the customer sees

