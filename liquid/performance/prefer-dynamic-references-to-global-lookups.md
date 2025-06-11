# Prefer dynamic references over global object lookups

In Shopify Liquid, avoid hardcoding direct references to global objects like metafields, settings, or specific product properties. Instead, use section settings with dynamic references to make themes more flexible and maintainable.

Bad:

```liquid
{% comment %}
  Hardcoded references to specific metafields and global objects
{% endcomment %}
<div class="product-hero">
  <h1>{{ product.title }}</h1>
  
  {% comment %} Hardcoded metafield reference {% endcomment %}
  {% if product.metafields.main.subcopy != blank %}
    <div class="product-subcopy">
      {{ product.metafields.main.subcopy | metafield_tag }}
    </div>
  {% endif %}
  
  {% comment %} Hardcoded settings reference {% endcomment %}
  {% if settings.show_vendor %}
    <div class="product-vendor">
      {{ product.vendor }}
    </div>
  {% endif %}
  
  {% comment %} Hardcoded collection reference {% endcomment %}
  {% if collections.featured-products.products contains product %}
    <div class="featured-badge">Featured Product</div>
  {% endif %}
</div>

{% comment %} Product description section {% endcomment %}
<div class="product-description">
  {% comment %} Hardcoded description reference {% endcomment %}
  {{ product.description }}
  
  {% comment %} Hardcoded metafield for additional content {% endcomment %}
  {% if product.metafields.custom.care_instructions != blank %}
    <div class="care-instructions">
      <h3>Care Instructions</h3>
      {{ product.metafields.custom.care_instructions | metafield_tag }}
    </div>
  {% endif %}
</div>
```

Good:

```liquid
{% comment %}
  Section with dynamic references and flexible settings
{% endcomment %}
<div class="product-hero">
  <h1>{{ product.title }}</h1>
  
  {% comment %} Dynamic subcopy with fallback options {% endcomment %}
  {% if section.settings.subcopy_content != blank %}
    <div class="product-subcopy">
      {{ section.settings.subcopy_content }}
    </div>
  {% endif %}
  
  {% comment %} Configurable vendor display {% endcomment %}
  {% if section.settings.show_vendor %}
    <div class="product-vendor">
      {% if section.settings.vendor_label != blank %}
        {{ section.settings.vendor_label }}: 
      {% endif %}
      {{ product.vendor }}
    </div>
  {% endif %}
  
  {% comment %} Dynamic badge with configurable collection {% endcomment %}
  {% if section.settings.featured_collection != blank and section.settings.show_featured_badge %}
    {% assign featured_collection = collections[section.settings.featured_collection] %}
    {% if featured_collection.products contains product %}
      <div class="featured-badge">
        {{ section.settings.featured_badge_text | default: 'Featured Product' }}
      </div>
    {% endif %}
  {% endif %}
</div>

{% comment %} Flexible content sections {% endcomment %}
{% for block in section.blocks %}
  {% case block.type %}
    {% when 'description' %}
      <div class="product-description" {{ block.shopify_attributes }}>
        {% if block.settings.content_source == 'description' %}
          {{ product.description }}
        {% elsif block.settings.content_source == 'custom' %}
          {{ block.settings.custom_content }}
        {% elsif block.settings.content_source == 'metafield' and block.settings.metafield_reference != blank %}
          {{ block.settings.metafield_reference }}
        {% endif %}
      </div>
      
    {% when 'additional_content' %}
      <div class="additional-content" {{ block.shopify_attributes }}>
        {% if block.settings.heading != blank %}
          <h3>{{ block.settings.heading }}</h3>
        {% endif %}
        
        {% if block.settings.content_type == 'metafield' and block.settings.metafield_reference != blank %}
          {{ block.settings.metafield_reference }}
        {% elsif block.settings.content_type == 'rich_text' %}
          {{ block.settings.rich_text_content }}
        {% elsif block.settings.content_type == 'html' %}
          {{ block.settings.html_content }}
        {% endif %}
      </div>
  {% endcase %}
{% endfor %}

{% schema %}
{
  "name": "Product Information",
  "settings": [
    {
      "type": "richtext",
      "id": "subcopy_content",
      "label": "Product subcopy",
      "info": "Rich text content to display below the product title",
      "default": "<p>Enter your product subcopy here</p>"
    },
    {
      "type": "checkbox",
      "id": "show_vendor",
      "label": "Show product vendor",
      "default": true
    },
    {
      "type": "text",
      "id": "vendor_label",
      "label": "Vendor label",
      "default": "Brand",
      "info": "Text to display before the vendor name"
    },
    {
      "type": "collection",
      "id": "featured_collection",
      "label": "Featured collection",
      "info": "Products in this collection will show a featured badge"
    },
    {
      "type": "checkbox",
      "id": "show_featured_badge",
      "label": "Show featured badge",
      "default": false
    },
    {
      "type": "text",
      "id": "featured_badge_text",
      "label": "Featured badge text",
      "default": "Featured Product"
    }
  ],
  "blocks": [
    {
      "type": "description",
      "name": "Product Description",
      "settings": [
        {
          "type": "select",
          "id": "content_source",
          "label": "Content source",
          "options": [
            {
              "value": "description",
              "label": "Product description"
            },
            {
              "value": "metafield",
              "label": "Metafield"
            },
            {
              "value": "custom",
              "label": "Custom content"
            }
          ],
          "default": "description"
        },
        {
          "type": "product_metafield",
          "id": "metafield_reference",
          "label": "Metafield",
          "info": "Select a metafield to display"
        },
        {
          "type": "richtext",
          "id": "custom_content",
          "label": "Custom content",
          "info": "Custom rich text content"
        }
      ]
    },
    {
      "type": "additional_content",
      "name": "Additional Content",
      "settings": [
        {
          "type": "text",
          "id": "heading",
          "label": "Heading",
          "default": "Additional Information"
        },
        {
          "type": "select",
          "id": "content_type",
          "label": "Content type",
          "options": [
            {
              "value": "metafield",
              "label": "Metafield"
            },
            {
              "value": "rich_text",
              "label": "Rich text"
            },
            {
              "value": "html",
              "label": "HTML"
            }
          ],
          "default": "rich_text"
        },
        {
          "type": "product_metafield",
          "id": "metafield_reference",
          "label": "Metafield",
          "info": "Select a metafield to display"
        },
        {
          "type": "richtext",
          "id": "rich_text_content",
          "label": "Rich text content"
        },
        {
          "type": "html",
          "id": "html_content",
          "label": "HTML content"
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "Product Information",
      "blocks": [
        {
          "type": "description"
        }
      ]
    }
  ]
}
{% endschema %}
```
