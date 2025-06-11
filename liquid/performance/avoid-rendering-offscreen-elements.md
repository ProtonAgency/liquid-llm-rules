# Avoid rendering offscreen elements

In Shopify Liquid, avoid rendering content that isn't immediately visible to users. Offscreen content like hidden menus, language selectors, and modal dialogs should be loaded conditionally or handled client-side to improve initial page load performance.

Bad:

```liquid
{% comment %}
  Language selector rendered in header, footer, and mobile menu
  with all options fully rendered in each location
{% endcomment %}

<div class="header-language-selector">
  <button class="language-toggle">{{ current_language.name }}</button>
  <ul class="language-dropdown">
    {% for language in shop.published_locales %}
      <li>
        <a href="{{ language.root_url }}" 
           hreflang="{{ language.iso_code }}"
           {% if language.active %}class="active"{% endif %}>
          <span class="language-flag">{% render 'flag', country_code: language.iso_country %}</span>
          <span class="language-name">{{ language.name }}</span>
        </a>
      </li>
    {% endfor %}
  </ul>
</div>

{% comment %} Same selector repeated in footer {% endcomment %}
<div class="footer-language-selector">
  <!-- Identical language selector code repeated -->
</div>

{% comment %} Same selector repeated in mobile menu {% endcomment %}
<div class="mobile-language-selector">
  <!-- Identical language selector code repeated -->
</div>
```

Good:

```liquid
{% comment %}
  Option 1: Render language selector once and use JavaScript to clone it
{% endcomment %}
<div id="primary-language-selector" class="header-language-selector">
  <button class="language-toggle">{{ current_language.name }}</button>
  <ul class="language-dropdown">
    {% for language in shop.published_locales %}
      <li>
        <a href="{{ language.root_url }}" 
           hreflang="{{ language.iso_code }}"
           {% if language.active %}class="active"{% endif %}>
          <span class="language-name">{{ language.name }}</span>
        </a>
      </li>
    {% endfor %}
  </ul>
</div>

<div id="footer-language-selector-container"></div>
<div id="mobile-language-selector-container"></div>

<script>
  // Clone the language selector to other locations client-side
  document.addEventListener('DOMContentLoaded', function() {
    const primarySelector = document.getElementById('primary-language-selector');
    
    if (primarySelector) {
      const footerContainer = document.getElementById('footer-language-selector-container');
      const mobileContainer = document.getElementById('mobile-language-selector-container');
      
      if (footerContainer) {
        const footerSelector = primarySelector.cloneNode(true);
        footerSelector.id = 'footer-language-selector';
        footerSelector.className = 'footer-language-selector';
        footerContainer.appendChild(footerSelector);
      }
      
      if (mobileContainer) {
        const mobileSelector = primarySelector.cloneNode(true);
        mobileSelector.id = 'mobile-language-selector';
        mobileSelector.className = 'mobile-language-selector';
        mobileContainer.appendChild(mobileSelector);
      }
    }
  });
</script>

{% comment %}
  Option 2: Lazy load language options when dropdown is opened
{% endcomment %}
<div class="header-language-selector">
  <button class="language-toggle" data-load-languages>{{ current_language.name }}</button>
  <div class="language-dropdown-container" data-languages-container></div>
</div>

{% comment %} Template for language options {% endcomment %}
<template id="language-options-template">
  {% for language in shop.published_locales %}
    <li>
      <a href="{{ language.root_url }}" 
         hreflang="{{ language.iso_code }}"
         {% if language.active %}class="active"{% endif %}>
        <span class="language-name">{{ language.name }}</span>
      </a>
    </li>
  {% endfor %}
</template>

<script>
  document.addEventListener('DOMContentLoaded', function() {
    const toggleButtons = document.querySelectorAll('[data-load-languages]');
    
    toggleButtons.forEach(button => {
      button.addEventListener('click', function() {
        const container = this.nextElementSibling;
        
        if (!container.hasChildNodes()) {
          const template = document.getElementById('language-options-template');
          const content = template.content.cloneNode(true);
          container.appendChild(content);
        }
        
        container.classList.toggle('active');
      });
    });
  });
</script>
```

## Why this matters

Rendering offscreen content has several performance implications:

1. **Increased server processing time**: Each Liquid operation adds to the server-side rendering time
2. **Larger HTML payload**: Duplicated content increases the page size, slowing down initial load
3. **Wasted resources**: Content that may never be seen still consumes rendering resources
4. **Reduced cache efficiency**: Larger pages are less efficient to cache and serve
