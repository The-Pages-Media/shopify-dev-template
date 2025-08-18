# Shopify Development Best Practices and Conventions

This document outlines the coding standards and best practices to follow when developing for this Shopify theme.

## Theme-Specific Information

### Brand Colors and Typography


### Key Theme Classes


## File Structure and Organization

### Order of Code Sections
Files should be organized in the following order for optimal performance and maintainability:

1. **Liquid variable assignments and logic** (at the top)
2. **CSS/Styles** (`<style>` tags)
3. **HTML markup** (with Liquid templating)
4. **JavaScript** (`<script>` tags at the bottom)

### Example Structure:
```liquid
{% comment %} Liquid variable assignments {% endcomment %}
{% assign product_title = product.title %}
{% assign show_feature = true %}

{% comment %} CSS/Styles at the top {% endcomment %}
<style>
  .custom-class {
    /* styles here */
  }
</style>

{% comment %} HTML with Liquid {% endcomment %}
<div class="custom-class">
  {{ product_title }}
</div>

{% comment %} JavaScript at the bottom {% endcomment %}
<script>
  // JavaScript functionality here
</script>
```

## CSS Best Practices

1. **Always use CSS for styling** - Never use inline JavaScript for style changes (mouseover, mouseout, etc.)
2. **Use :hover pseudo-classes** instead of JavaScript event listeners for hover effects
3. **Place all CSS at the top of the file** after Liquid assignments
4. **Leverage existing theme classes** before creating new ones
5. **Use CSS variables** and theme colors consistently

### Theme-Specific CSS Examples


## JavaScript Best Practices

1. **Place all JavaScript at the bottom** of the file for better performance
2. **Use event delegation** when possible to reduce event listeners
3. **Avoid inline onclick handlers** when the functionality is complex
4. **Use semantic function names** that describe what they do
5. **Account for header offsets** when implementing scroll functionality
6. **Check for mobile app context** when implementing app-specific features

## Liquid Best Practices

1. **Assign variables at the top** of the file for clarity
2. **Use descriptive variable names**
3. **Comment complex logic** to explain the purpose
4. **Minimize logic in templates** - move complex logic to the top as variable assignments
5. **Use the {% liquid %} block tag** for multi-line Liquid code for better readability and maintainability
6. **Use proper variable assignment syntax** - ALWAYS initialize variables with default values first, then conditionally reassign them. Never use compound assignments or logical operators in variable declarations

### Proper Variable Assignment Pattern
**IMPORTANT:** Always initialize variables with default values first, then conditionally reassign:

**Correct approach:**
```liquid
{% liquid
  assign show_matching_lid = false
  if container_tag and matching_lid_text != blank
    assign show_matching_lid = true
  endif
  
  assign show_custom_branding = false
  if custom_branding_text != blank and custom_branding_link != blank
    assign show_custom_branding = true
  endif
%}
```

**NEVER do this (will break the page):**
```liquid
{% liquid
  # WRONG - This will cause Liquid syntax errors
  assign show_matching_lid = container_tag and matching_lid_text != blank
  assign show_custom_branding = custom_branding_text != blank and custom_branding_link != blank
%}
```

### Multi-line Liquid Block Usage
For multiple Liquid statements, use the `{% liquid %}` block tag instead of multiple separate tags:

**Preferred approach:**
```liquid
{% liquid
  assign is_available = product.available
  assign has_variants = product.variants.size > 1
  assign show_button = false
  
  if is_available and has_variants
    assign show_button = true
  endif
  
  if show_button
    assign button_text = 'Add to Cart'
  else
    assign button_text = 'Unavailable'
  endif
%}
```

**Avoid this approach:**
```liquid
{% assign is_available = product.available %}
{% assign has_variants = product.variants.size > 1 %}
{% assign show_button = is_available and has_variants %}
{% if show_button %}
  {% assign button_text = 'Add to Cart' %}
{% else %}
  {% assign button_text = 'Unavailable' %}
{% endif %}
```

The `{% liquid %}` tag provides cleaner code with less syntax overhead and improved performance. Reference: https://shopify.dev/docs/api/liquid/tags/liquid

## Performance Considerations

1. **Minimize DOM queries** - Cache selectors when used multiple times
2. **Use CSS transitions** instead of JavaScript animations
3. **Lazy load images** when appropriate
4. **Avoid unnecessary re-renders** by batching DOM updates

## Accessibility

1. **Always include proper ARIA labels** for interactive elements
2. **Ensure keyboard navigation** works for all interactive components
3. **Use semantic HTML elements** (button, nav, main, etc.)
4. **Provide alternative text** for images

## Code Comments

1. **Use Liquid comments** for Liquid-specific notes: `{% comment %} Note here {% endcomment %}`
2. **Use HTML comments** sparingly in production code
3. **Document complex functionality** with clear explanations
4. **Include TODO comments** for future improvements

## Testing Checklist

Before committing changes:
1. Test on mobile and desktop viewports
2. Verify JavaScript console has no errors
3. Check that styles don't conflict with existing theme styles
4. Ensure Liquid syntax is valid
5. Test with different product types/variants

## Common Patterns

### Metafield Access Pattern
```liquid
{% comment %} Shop metafields {% endcomment %}
{% assign plan_settings = shop.metafields.namespace.field_name %}

{% comment %} Product metafields {% endcomment %}
{% assign custom_data = product.metafields.custom.field_name %}

{% comment %} Safe access with default values {% endcomment %}
{% assign show_feature = false %}
{% if product.metafields.features.enabled %}
  {% assign show_feature = true %}
{% endif %}
```

## File Naming Conventions

1. Use **kebab-case** for file names: `product-form.liquid`
2. Prefix component files with `component-`: `component-product-card.liquid`
3. Use descriptive names that indicate the file's purpose

## Theme-Specific Conventions

### Typography


### Focus States


### Animation Patterns


### Responsive Breakpoints


### App-Specific Styling


---

Last Updated: {{ "now" | date: "%Y-%m-%d" }}