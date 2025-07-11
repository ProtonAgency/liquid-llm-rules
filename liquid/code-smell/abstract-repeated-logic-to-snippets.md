# Abstract repeated logic to snippets

When writing Liquid, **avoid large monolithic sections with repeated or complex logic.** Use snippets to keep your templates modular, maintainable, and testable.

---

## Bad:

```liquid
# snippets/product-badges.liquid

{% comment %}
  Accepts:
  - variant: {Object} variant Liquid object
{% endcomment %}

{% liquid
  assign badge_container_classes = badge_container_classes | default: 'absolute gap-x-2.5 top-3 left-3'
  assign badge_classes = badge_classes | default: 'px-2 py-1 rounded font-medium'
%}

<div class="flex flex-wrap {{ badge_container_classes }}" data-badges>
  
  {% if product.metafields.display.badge.value != blank %}
    {% liquid
      assign bg_color = product.metafields.display.badge.value.color
      assign text_color = product.metafields.display.badge.value.text_color
      assign text = product.metafields.display.badge.value.text
      assign show_badge = false
      if text != blank
        if product.metafields.display.badge.value.end_date != null
          assign current_date = 'now' | date: '%s'
          assign start_date = product.metafields.display.badge.value.start_date | default: current_date | date: '%s' | plus: 0
          assign end_date = product.metafields.display.badge.value.end_date | default: current_date | date: '%s' | plus: 0
          assign current_date = current_date | plus: 0
          if current_date >= start_date and current_date < end_date
            assign show_badge = true
          endif
        else
          assign show_badge = true
        endif
      endif
    %}
    {% if show_badge %}
      <span class="flex text-[--text-color] leading-none bg-[--bg-color] w-max {{ badge_classes }}" style="--bg-color: {{ bg_color | default: '#B9C9D5' }}; --text-color: {{ text_color | default: 'black' }}">
        {{- text -}}
      </span>
    {% endif %}
  {% endif %}

  {% for badge in variant.metafields.display.badges.value %}
    {% liquid
      assign bg_color = badge.color.value.color
      assign text_color = badge.text_color
      assign text = badge.text
      assign show_badge = false
      if text != blank
        if badge.end_date != null
          assign current_date = 'now' | date: '%s'
          assign start_date = badge.start_date | default: current_date | date: '%s' | plus: 0
          assign end_date = badge.end_date | default: current_date | date: '%s' | plus: 0
          assign current_date = current_date | plus: 0
          if current_date >= start_date and current_date < end_date
            assign show_badge = true
          endif
        else
          assign show_badge = true
        endif
      endif
    %}
    {% if show_badge %}
      <span class="flex text-[--text-color] leading-none bg-[--bg-color] w-max {{ badge_classes }}" style="--bg-color: {{ bg_color | default: '#B9C9D5' }}; --text-color: {{ text_color | default: 'black' }}">
        {{- text -}}
      </span>
    {% endif %}
  {% endfor %}

</div>

```


## Good:

```liquid
# snippets/product-badges.liquid
{% comment %}
  Accepts:
  - variant: {Object} variant Liquid object
{% endcomment %}

{% liquid
  assign badge_container_classes = badge_container_classes | default: 'absolute gap-x-2.5 top-3 left-3'
  assign badge_classes = badge_classes | default: 'px-2 py-1 rounded font-medium'
%}

<div class="flex flex-wrap {{ badge_container_classes }}" data-badges>
  
  {% if product.metafields.display.badge.value != blank %}
    {% render 'badge', badge: product.metafields.display.badge.value, badge_classes: badge_classes %}
  {% endif %}

  {% for badge in variant.metafields.display.badges.value %}
    {% render 'badge', badge: badge, badge_classes: badge_classes %}
  {% endfor %}
</div>

# snippets/badge.liquid
{% comment %}
  Renders a single badge with optional date-based visibility.
  - badge: {Object}
  - badge_classes: {String}
{% endcomment %}

{% liquid
  assign bg_color = badge.color.value.color
  assign text_color = badge.text_color
  assign text = badge.text
  assign show_badge = false

  if text != blank
    if badge.end_date != null
      assign current_date = 'now' | date: '%s'
      assign start_date = badge.start_date | default: current_date | date: '%s' | plus: 0
      assign end_date = badge.end_date | default: current_date | date: '%s' | plus: 0
      if current_date >= start_date and current_date < end_date
        assign show_badge = true
      endif
    else
      assign show_badge = true
    endif
  endif
%}

{% if show_badge %}
  <span class="flex text-[--text-color] leading-none bg-[--bg-color] w-max {{ badge_classes }}" style="--bg-color: {{ bg_color | default: '#B9C9D5' }}; --text-color: {{ text_color | default: 'black' }}">
    {{- text -}}
  </span>
{% endif %}
```

## Why is this better?

- Badge rendering logic is centralized in a snippet.
- Easier to maintain and update styles or rules.
- Reduces duplication in sections.
- Improves theme consistency and reusability.
