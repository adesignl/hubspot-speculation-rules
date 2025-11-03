# HubSpot Speculation Rules API Implementation Guide

This guide explains how to implement the Speculation Rules API in your HubSpot CMS using HubL (HubSpot's templating language).

## Installation Methods

### Method 1: Add to Theme Settings (Recommended)

1. Go to **Design Manager** in HubSpot
2. Navigate to your theme
3. Click **Theme Settings** or create/edit your theme's **header.html**
4. Add the HubL code before the closing `</head>` tag
5. Save and publish

### Method 2: Add to Individual Template

1. Go to **Design Manager**
2. Edit your page template or create a new one
3. Paste the HubL code in the header section
4. Save changes

### Method 3: Add as a Module

1. Create a new module file (e.g., `speculation-rules.module`)
2. Paste the HubL code
3. Add the module to your theme or templates as needed

## Usage

### Basic Implementation

Once installed, the code works automatically. No additional configuration needed.

### Prevent Prefetching on Specific Links

Add the `no-prefetch` CSS class to any link:

```html
<a href="/path-to-page" class="no-prefetch">Link text</a>
```

### Prevent Prerendering on Specific Links

Add the `no-prerender` CSS class to any link:

```html
<a href="/path-to-page" class="no-prerender">Link text</a>
```

### Custom Exclusions

To add custom URL patterns or exclusions, modify the `speculation_rules` object:

```hubl
{% set speculation_rules = {
  "prefetch": [
    {
      "source": "document",
      "where": {
        "and": [
          { "href_matches": "/*" },
          { "not": { "href_matches": "/*.zip" } },
          { "not": { "href_matches": "/*.pdf" } },
          { "not": { "href_matches": "/*checkout*" } },  {# Add custom exclusion #}
          { "not": { "selector_matches": ".no-prefetch" } }
        ]
      },
      "eagerness": "moderate"
    }
  ],
  "prerender": [
    {
      "source": "document",
      "where": {
        "and": [
          { "href_matches": "/*" },
          { "not": { "href_matches": "/*checkout*" } },  {# Add custom exclusion #}
          { "not": { "selector_matches": ".no-prerender" } }
        ]
      },
      "eagerness": "conservative"
    }
  ]
} %}
```

## Default Exclusions

The HubL code automatically excludes:

- ZIP files (`*.zip`)
- PDF files (`*.pdf`)
- Links with `.no-prefetch` class
- Links with `.no-prerender` class

## Eagerness Levels Explained

### Prefetch - "moderate"
- Downloads HTML in background without rendering
- Good for likely navigation targets
- More aggressive than prerender

### Prerender - "conservative"
- Downloads AND renders the page in background
- Only for very confident predictions
- More conservative with resources

Adjust these values based on your site's traffic patterns and device considerations.

## HubL-Specific Notes

### tojson Filter
The `|tojson` filter converts the HubL dictionary to valid JSON format that browsers understand.

### Variable Scoping
The `{% set %}` tag defines a scoped variable available within the template. It won't leak into other templates.

### Comment Syntax
- HubL comments: `{# comment #}`
- HTML comments: `<!-- comment -->`

## Integration with HubSpot Modules

If you're creating a module with this functionality:

```hubl
{# speculation-rules.module #}

{# Module fields #}
{% if enable_prefetch %}
  {% set prefetch_enabled = true %}
{% endif %}

{% if enable_prerender %}
  {% set prerender_enabled = true %}
{% endif %}

{# Speculation rules setup #}
{% if prefetch_enabled or prerender_enabled %}
  <script type="speculationrules">
    {{ speculation_rules|tojson }}
  </script>
{% endif %}
```

## Performance Monitoring

### In HubSpot Analytics
1. Check **Page Performance** reports
2. Monitor metrics like bounce rate and time on page
3. Compare performance before/after implementation

### In Browser DevTools
1. Open **Chrome DevTools** (F12)
2. Go to **Application** → **Speculation Rules**
3. Verify your rules are loaded correctly

## Browser Support

Currently supported in:
- Chrome 121+
- Edge 121+
- Other Chromium-based browsers

Other browsers safely ignore the script tag with no negative effects.

## Troubleshooting

### Rules Not Showing in DevTools

**Solution**: Verify the code is in your published theme template and you're viewing a published page (not draft).

### Links Not Being Prefetched

**Causes**:
- Browser doesn't support the API (only Chrome/Edge 121+)
- Link has `.no-prefetch` class
- URL matches an exclusion pattern
- JavaScript console errors

**Solution**: Check browser console for errors and verify link eligibility.

### Performance Issues

**If pages are loading slower**:
1. Change `eagerness` to `"low"` or remove prerender
2. Add `.no-prefetch` classes to resource-heavy pages
3. Add custom exclusions for problematic URLs

## HubSpot-Specific Considerations

### CMS Hub Pages
- Automatically works with standard HubSpot links
- Compatible with HubSpot CMS navigation menus
- Works with dynamic content blocks

### Contact-Specific Personalization
The code works at the browser level, so it doesn't interfere with HubSpot's personalization engine.

### Portal Settings
- No special portal settings needed
- Works across all domain types
- Compatible with subdomains

## Best Practices

1. **Start conservative**: Begin with prefetch only, add prerender later if needed
2. **Monitor metrics**: Track bounce rate, time on page, and conversion metrics
3. **Exclude checkouts**: Add checkout/payment pages to exclusions
4. **Test thoroughly**: Verify in Chrome DevTools before production deployment
5. **Custom classes**: Use `.no-prefetch` for pages with sensitive operations

## Advanced Configuration

### Conditional Rules by Page Type

```hubl
{% if page.is_landing_page %}
  {# More aggressive prefetching for landing pages #}
  "eagerness": "aggressive"
{% else %}
  {# Conservative for other pages #}
  "eagerness": "moderate"
{% endif %}
```

### Per-Module Exclusions

```hubl
{# In your module #}
{% if module.disable_speculation %}
  class="no-prefetch no-prerender"
{% endif %}
```

## References

- [HubSpot Developer Docs](https://developers.hubspot.com/docs/cms/guides/page-templates)
- [HubL Syntax Reference](https://developers.hubspot.com/docs/cms/hubl/intro-to-hubl)
- [Speculation Rules API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Speculation_Rules_API)
- [W3C Specification](https://wicg.github.io/nav-speculation/speculation-rules.html)

## Support

For HubSpot-specific issues, contact HubSpot support. For Speculation Rules API issues, check browser DevTools and the W3C specification.
