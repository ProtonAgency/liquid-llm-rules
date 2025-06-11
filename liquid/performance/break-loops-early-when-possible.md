# Always break loops early when possible

In Liquid always break a loop early when possible such as when assigning a variable from an iterated value.

Bad:

```liquid
  assign current_color_value = ''
  assign color_option_values = product.options_with_values[color_index].values
  for value in color_option_values
    if value.name == current_color
      assign current_color_value = value.swatch.color
    endif
  endfor
```

Good:

```liquid
  assign current_color_value = ''
  assign color_option_values = product.options_with_values[color_index].values
  for value in color_option_values
    if value.name == current_color
      assign current_color_value = value.swatch.color
      break
    endif
  endfor
```
