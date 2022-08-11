# EAV and Database Notes

## EAV
The EAV framework is what allows entities to have different values for given properties. 
It is deigned to make attaching or detatching attributes to and from objects simple, with low overhead.


Three components:
- ### EAV Entities
  - The "things" in your db (i.e. Customer, Product)
  - It is difficult (and rarely necessary) to create new EAV Entities

Out of the box EAV entities
1. `customer`
2. `customer_address`
3. `catalog_category`
4. `catalog_product`
5. `order`
6. `invoice`
7. `creditmemo`
8. `shipment`

*FYI: `catalog_category` and `catalog_product` are the only "full-fledged" entity types*
- ### EAV Attributes
  - The elements that make a thing what it is (i.e. Name, Price)
  - Attributes are grouped in **attribute sets** which relate to an entity
    - i.e. attributes relating to products are grouped together as "product attributes"
    - Different products will have different attribute sets
      - this makes sense since a T-shirt has different attributes than a kayak (or whatever)
      - This could be the case for customers and categories, but in practice likely will not be (since one type of category will probably have the same attributes as another)
    - Attribute sets are found in the `eav_attribute_set` table
    - Attribute sets can be created in the admin panel or programmatically
      - When creating attribute sets programmatically use `initFromSeleton()` to make sure all necessary attributes (like price, for example) are included
  - Attributes may also be grouped in **attribute groups**
    - A group maps to one parent attribute set
    - Attribute groups are used for rendering attributes on an edit page
  - New attributes should be added via a data patch (they may be added in the admin panel, but this is not recommended)
  - Certain attributes may relate to certain behaviors, for example:
    - `frontend_model`
      - This must be set to a class that extends `Magento\Eav\Model\Entity\Attribute\Frontend\[FrontendInterface || AbstractFrontend]`
      - This model renders an attribute on the storefront using its `getValue()` method which takes an entity model as a parameter.
    - Source Model
      - Provides a list of acceptable options for an attribute
      - Must implement `Magento\Eav\Model\Entity\Attribute\Source\[SourceInterface || AbstractSource]`
    - Backend Model
      - Controls how attribute values are saved to the db
      - Must implement `Magento\Eav\Model\Entity\Attribute\Backend\[BackendInterface || AbstractBackend]`
- ### EAV Values
  - What a given entity's given attribute is (i.e. Rick, $3.50)
  - Stored in tables corresponding to entity type and data tyoe
    - i.e. `catalog_product_entity_datetime` - for all datetime attribute values that relate to product entities
    - Each of these tables has 4 columns
      - `entity_id`
      - `attribute_id`
      - `value`
      - `store_id`


## Database
Database tables can be created and altered using the `etc/db_schema.xml` file, which looks like this:
```
<schema ...>
    <table name="table_name">
        <column xsi:type="int" name="id" ... />
        <column xsi:type="varchar" name="name" ... />
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="id"/>
        </constraint>
    </table>
</schema>
```
Available xsi:types for columns:
- blob
- boolean
- date
- datetime
- decimal
- float
- int
- json
- real
- smallint
- text
- timestamp
- varbinary
- varchar

Some additional column attributes:
- unsigned: (if true, value must be a positive number)
- nullable: (if true, value can be null)
- identity: (if true, column value auto-increments)
- length: specifies maxlength of column
- default: specifies a default value
- onCreate: tells Magento to perform an action when this column is created
  - ex: To rename a column use `onCreate="migrateDataFrom([existing_column])`

Adding a table to `db_schema` will add the table to the db, likewise removing a table from `db_schema` will remove it from the database (upon running `bin/magento setup:upgrate`)

When changing (or deleting columns) the changes must be reflected in `db_schema_whitelist.json`


