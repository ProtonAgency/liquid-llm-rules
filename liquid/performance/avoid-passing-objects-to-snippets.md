# Avoid passing objects to nested snippets

In Shopify Liquid, avoid passing entire objects to snippets. Instead, pass only the primitive values needed. This improves caching efficiency, reduces processing overhead, and makes snippets more reusable.

Bad:

```liquid
{% comment %}
  Passing entire variant object to price snippet
{% endcomment %}

{% comment %} Main product template {% endcomment %}
<div class="product-details">
  {% for variant in product.variants %}
    <div class="variant-option">
      {% render 'price-display', variant: variant %}
    </div>
  {% endfor %}
</div>

{% comment %} snippets/price-display.liquid {% endcomment %}
<div class="price-container">
  {% if variant.compare_at_price > variant.price %}
    <span class="price-sale">{{ variant.price | money }}</span>
    <span class="price-compare">{{ variant.compare_at_price | money }}</span>
    <span class="price-savings">
      Save {{ variant.compare_at_price | minus: variant.price | money }}
    </span>
  {% else %}
    <span class="price-regular">{{ variant.price | money }}</span>
  {% endif %}
</div>
```

Good:

```liquid
{% comment %}
  Passing only primitive values needed by price snippet
{% endcomment %}

{% comment %} Main product template {% endcomment %}
<div class="product-details">
  {% for variant in product.variants %}
    <div class="variant-option">
      {% render 'price-display', 
          price: variant.price, 
          compare_at_price: variant.compare_at_price %}
    </div>
  {% endfor %}
</div>

{% comment %} snippets/price-display.liquid {% endcomment %}
<div class="price-container">
  {% if compare_at_price and compare_at_price > price %}
    <span class="price-sale">{{ price | money }}</span>
    <span class="price-compare">{{ compare_at_price | money }}</span>
    <span class="price-savings">
      Save {{ compare_at_price | minus: price | money }}
    </span>
  {% else %}
    <span class="price-regular">{{ price | money }}</span>
  {% endif %}
</div>
```

## Why this matters

Passing objects to snippets has several performance implications:

1. **Increased processing overhead**: Liquid must process and pass entire object structures
2. **Reduced caching efficiency**: Objects change more frequently than primitive values, breaking cache
3. **Memory usage**: Larger objects consume more memory during template processing
4. **Slower rendering**: More data to serialize and process in nested contexts
5. **Reduced reusability**: Snippets tied to specific objects can't be reused in different contexts
