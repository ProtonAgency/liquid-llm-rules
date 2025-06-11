# Avoid date operations

In Shopify Liquid, avoid date operations as they can be cache breaking.

Bad:

```liquid
{% if product.metafields.custom.drop_date != blank %}
  {% assign available = product.metafields.custom.drop_date | date: '%s' %}
  {% assign now = 'now' | date: '%s' %}
  {% if now > available %}
    {% assign days = now | minus: available | divided_by: 86400.00 %}
    {% if days < settings.new_badge_days and settings.enable_new_badge %}
      {% assign showNewBadge = true %}
    {% endif %}
  {% endif %}
{% endif %}
```

Good:

```liquid
{% comment %}
  Pre-calculate time-sensitive values server-side or use JavaScript
  for dynamic time-based functionality
{% endcomment %}
{% if product.metafields.custom.drop_date != blank %}
  {% assign drop_date_timestamp = product.metafields.custom.drop_date | date: '%s' %}
  {% comment %}
    Use a pre-calculated timestamp or handle time comparison client-side
  {% endcomment %}
  <div data-drop-date="{{ drop_date_timestamp }}" class="product-availability">
    <!-- Handle time-based logic with JavaScript -->
  </div>
{% endif %}
```

## Why this matters

Date operations using `now` break Shopify's caching mechanism because:

- The `now` filter returns the current timestamp, which changes every second
- This makes the rendered output different on each request
- Shopify cannot cache pages that produce different content on each render
- This significantly impacts page load performance and server resources

I suggest using Shopify Flow to automate time based content such as badges.
