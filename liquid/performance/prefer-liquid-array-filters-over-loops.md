# When possible use Liquid array filters instead of for loops

Liquid supports many array filters that can be used instead of looping over arrays.

Bad:

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

Good:

```liquid
  assign current_color_option = product.options_with_values[color_index].values | find: 'name', current_color
  assign current_color_value = current_color_option.swatch.color
```

Docs:

Array Filters in Shopify Liquid

Array filters are used to modify arrays in Shopify's Liquid templating language.

1. compact: Removes any `nil` items from an array.
   - Usage: `array | compact`
   - Example: `{% assign prices = collection.products | map: 'compare_at_price' | compact %}`

2. concat: Concatenates (combines) two arrays.
   - Usage: `array | concat: array`
   - Example: `{% assign combined = array1 | concat: array2 %}`

3. find: Returns the first item in an array with a specific property value.
   - Usage: `array | find: "property", "value"`
   - Example: `{% assign product = collection.products | find: "title", "Sample Product" %}`

4. find_index: Returns the index of the first item in an array with a specific property value.
   - Usage: `array | find_index: "property", "value"`
   - Example: `{% assign index = collection.products | find_index: "title", "Sample Product" %}`

5. first: Returns the first item in an array.
   - Usage: `array | first`
   - Example: `{% assign first_product = collection.products | first %}`

6. has: Tests if any item in an array has a specific property value.
   - Usage: `array | has: "property", "value"`
   - Example: `{% assign has_product = collection.products | has: "title", "Sample Product" %}`

7. join: Combines all items in an array into a single string, separated by a specified delimiter.
   - Usage: `array | join: "delimiter"`
   - Example: `{% assign titles = collection.products | map: 'title' | join: ", " %}`

8. last: Returns the last item in an array.
   - Usage: `array | last`
   - Example: `{% assign last_product = collection.products | last %}`

9. map: Creates an array of values by extracting the named property from another array of objects.
   - Usage: `array | map: "property"`
   - Example: `{% assign titles = collection.products | map: 'title' %}`

10. pluck: Creates an array of values by extracting the named property from another array of objects.
    - Usage: `array | pluck: "property"`
    - Example: `{% assign titles = collection.products | pluck: 'title' %}`

11. reverse: Reverses the order of the items in an array.
    - Usage: `array | reverse`
    - Example: `{% assign reversed_products = collection.products | reverse %}`
