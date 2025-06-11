# Avoid unnecessary JSON operations

In Shopify Liquid, avoid using the `json` filter to serialize entire large objects when only a subset of data is needed. This creates unnecessary processing overhead and bloats the page with unused data.

Bad:

```liquid
{% comment %}
  Serializing the entire product object when only price is needed
{% endcomment %}
<div class="product-card" data-product="{{ product | json }}">
  <h2>{{ product.title }}</h2>
  <div class="price" data-price-container>
    <span class="current-price">{{ product.price | money }}</span>
  </div>
  <button class="add-to-cart-button">Add to Cart</button>
</div>

<script>
  document.addEventListener('DOMContentLoaded', function() {
    const productCards = document.querySelectorAll('.product-card');
    
    productCards.forEach(card => {
      const productData = JSON.parse(card.dataset.product);
      // Only using the price from the entire product object
      const priceContainer = card.querySelector('[data-price-container]');
      
      // Update price on variant change
      function updatePrice(variantId) {
        const variant = productData.variants.find(v => v.id === variantId);
        if (variant) {
          // Only using price, but serialized the entire product object
          priceContainer.innerHTML = formatMoney(variant.price);
        }
      }
    });
  });
</script>
```

Good:

```liquid
{% comment %}
  Option 1: Serialize only the specific data needed
{% endcomment %}
<div class="product-card" 
     data-product-id="{{ product.id }}"
     data-product-title="{{ product.title | escape }}"
     data-product-price="{{ product.price | money_without_currency | strip_html }}"
     data-product-url="{{ product.url | within: collection }}">
  <h2>{{ product.title }}</h2>
  <div class="price" data-price-container>
    <span class="current-price">{{ product.price | money }}</span>
  </div>
  <button class="add-to-cart-button">Add to Cart</button>
</div>

{% comment %}
  Option 2: Create a minimal JSON object with only required data
{% endcomment %}
{% assign product_price_data = product.variants | map: 'id' | zip: product.variants | map: 'price' %}
{% assign minimal_product_data = '{
  "id": "' | append: product.id | append: '",
  "title": "' | append: product.title | escape | append: '",
  "url": "' | append: product.url | within: collection | append: '",
  "current_variant_id": "' | append: product.selected_or_first_available_variant.id | append: '",
  "variants": {' %}

{% for variant in product.variants %}
  {% assign variant_id = variant.id | append: '' %}
  {% assign variant_price = variant.price | money_without_currency | strip_html | append: '' %}
  {% assign variant_data = '"' | append: variant_id | append: '":{"price":"' | append: variant_price | append: '"}' %}
  
  {% if forloop.last %}
    {% assign minimal_product_data = minimal_product_data | append: variant_data %}
  {% else %}
    {% assign minimal_product_data = minimal_product_data | append: variant_data | append: ',' %}
  {% endif %}
{% endfor %}

{% assign minimal_product_data = minimal_product_data | append: '}}' %}

<div class="product-card" data-product-minimal='{{ minimal_product_data }}'>
  <h2>{{ product.title }}</h2>
  <div class="price" data-price-container>
    <span class="current-price">{{ product.price | money }}</span>
  </div>
  <button class="add-to-cart-button">Add to Cart</button>
</div>
```

## Why this matters

Using the `json` filter on large objects has several performance implications:

1. **Increased server processing time**: Serializing large objects is CPU-intensive
2. **Bloated HTML**: The serialized JSON can be many kilobytes or even megabytes for complex objects
3. **Slower page load**: Larger HTML payloads take longer to download and parse
4. **Memory usage**: Browser must parse and store large JSON objects in memory
5. **Wasted bandwidth**: Sending data that's never used wastes user bandwidth
