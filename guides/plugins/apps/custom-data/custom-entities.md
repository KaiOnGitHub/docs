# Custom entities

In addition to [Custom fields](custom-fields.md), you can create completely own entities in the system, named custom entities.
Unlike [Custom fields](custom-fields.md), you can generate completely custom data structures with custom relations, which can then be maintained by the admin.
To make use of the custom entities register your entities in your `entities.xml` file, which is located in the `Resources` directory of your app.

{% code title="<app root>/Resources/entities.xml" %}

```xml
<?xml version="1.0" encoding="utf-8" ?>
<entities xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="https://raw.githubusercontent.com/shopware/platform/trunk/src/Core/System/CustomEntity/Xml/entity-1.0.xsd">
    <entity name="custom_entity_bundle">
        <fields>
            <string name="name" required="true" translatable="true" store-api-aware="true" />
            <price name="discount" required="true" store-api-aware="true"/>
            <many-to-many name="products" reference="product" store-api-aware="true" />
        </fields>
    </entity>
</entities>
```

{% endcode %}

For a complete reference of the structure of the entities file take a look at the [Custom entity xml reference](../../../../resources/references/app-reference/entities-reference.md).

## Functionality

All registered entities will get an automatically registered repository. It is also available in the [App scripts](../app-scripts/README.md) section, in case you are allowed to access the repository service inside the hook.
{% raw %}

```twig
{% set blogs = services.repository.search('custom_entity_blog', criteria) %}
```

{% endraw %}

Additionally, to the repository you can also access your custom entities via [Admin api](../../../../concepts/api/admin-api.md).

```bash
POST /api/search/custom-entity-blog
```

## Custom Fields

By default, it is not possible to create a custom field of type "Entity Select" which references a custom entity. However, you can opt in to this behavior. You will need to add the `custom-fields-aware` & `label-property` attributes to your entity definition:

{% code title="Resources/entities.xml" %}

```xml
<?xml version="1.0" encoding="utf-8" ?>
<entities xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="https://raw.githubusercontent.com/shopware/platform/trunk/src/Core/System/CustomEntity/Xml/entity-1.0.xsd">
    <entity name="custom_entity_bundle" custom-fields-aware="true" label-property="name">
        <fields>
            <string name="name" required="true" translatable="true" store-api-aware="true" />
            <price name="discount" required="true" store-api-aware="true"/>
            <many-to-many name="products" reference="product" store-api-aware="true" />
        </fields>
    </entity>
</entities>
```

{% endcode %}

First we set `custom-fields-aware` to true. Then we specify a field to use as a label for the entity when selecting via the custom field. In this case, we use the `name` field. The field must exist in the `fields` section of the entity definition and it must be of type `string`.

Now you will find your entity in the "Entity Type" select when creating a custom field of type "Entity Select". Without a snippet label for the entity it will display as `custom_entity_bundle.label`. You can create a snippet to add a label like so:

{% code title="Resources/app/administration/snippet/en-GB.json" %}

```json
{
  "custom_entity_bundle": {
    "label": "My Custom Entity"
  }
}
```

{% endcode %}

## Permissions

Unlike core entities, your app directly has full access rights to your own custom entities. However, if your entity has associations that reference core tables,
you need the appropriate [permissions](../../../../resources/references/app-reference/manifest-reference.md) to load and write these associations.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:noNamespaceSchemaLocation="https://raw.githubusercontent.com/shopware/platform/trunk/src/Core/Framework/App/Manifest/Schema/manifest-2.0.xsd">
    <meta>
        <!-- ... -->
    </meta>
    <permissions>
        <read>product</read>
<!--    <read>custom_entity_blog</read>   < permissions for own entities are automatically set  -->
    </permissions>
</manifest>
```

## Shorthand prefix

Since v6.4.15.0 it is possible to also use the `ce_` shorthand prefix for your custom entities to prevent problems with length restrictions of names inside the DB.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<entities xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="https://raw.githubusercontent.com/shopware/platform/trunk/src/Core/System/CustomEntity/Xml/entity-1.0.xsd">
    <entity name="ce_bundle">
        <fields>
            ...
        </fields>
    </entity>
</entities>
```

If you use the shorthand in the entity definition, you also need to use it if you use the repository or the API.

```twig
{% set blogs = services.repository.search('ce_blog', criteria) %}
```

```bash
POST /api/search/ce_blog
```

{% hint style="warning" %}
Note that you can't rename existing custom entities as that would lead to the deletion of all existing data.
{% endhint %}
