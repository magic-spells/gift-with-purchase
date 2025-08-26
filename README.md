# Gift with Purchase

A powerful, e-commerce web component for automatic gift-with-purchase threshold promotions. Seamlessly integrates with Shopify and automatically manages gift items in the cart based on spending thresholds.

[**Live Demo**](https://magic-spells.github.io/gift-with-purchase/demo/)

## Features

- 🎁 **Automatic Gift Management** - Adds/removes gifts based on cart thresholds using smart pricing logic
- 🛒 **Shopify Integration** - Built-in Cart API support with proper line item properties
- 📱 **Cart Panel Sync** - Automatically syncs with cart-dialog components using `calculated_subtotal`
- 🎨 **Highly Customizable** - CSS custom properties and swappable content elements
- ⚡ **Event-Driven** - Custom events for gift addition, removal, and errors
- 🔧 **Flexible Content** - Data attributes for dynamic image, title, and variant updates
- 📱 **Responsive** - Mobile-optimized with responsive design

## Installation

```bash
npm install @magic-spells/gift-with-purchase
```

## Basic Usage

```html
<gift-with-purchase
	threshold="75.00"
	current="45.00"
	variant-id="12345678"
	message-above="🎉 Congratulations! You've qualified for your FREE gift!"
	message-below="Add ${ amount } more to unlock your free gift! 🎁">
	<div class="gwp-product">
		<img src="gift-image.jpg" alt="Free Gift" />
		<div class="gwp-content">
			<h4 class="gwp-title">Free Sample Set</h4>
			<p class="gwp-variant">Travel Size Collection</p>
		</div>
	</div>
	<p data-content-gwp-message class="text-sm font-medium"></p>
</gift-with-purchase>
```

```css
@import '@magic-spells/gift-with-purchase/css';
```

## Cart Integration

The component automatically listens for cart data changes when placed inside a `<cart-dialog>` component from the `@magic-spells/cart-panel` package:

```html
<cart-dialog>
	<gift-with-purchase
		threshold="75.00"
		variant-id="12345678"
		message-above="🎉 Congratulations! You've qualified for your FREE gift!"
		message-below="Add ${ amount } more to unlock your free gift! 🎁">
		<!-- Gift content -->
	</gift-with-purchase>
</cart-dialog>
```

When the cart-dialog emits a `cart-dialog:data-changed` event (typically from Shopify cart updates), the gift component will automatically:

- Update the current cart amount using `calculated_subtotal` for accurate threshold calculation
- Check if the threshold is met (excludes other gifts and honors pricing exclusions)
- Add the gift to cart if threshold is reached
- Remove the gift if cart falls below threshold

### Smart Pricing Logic

The component uses intelligent threshold calculation:
- **Uses `calculated_subtotal`** from cart-panel which properly handles item exclusions
- **Excludes gifts**: Other gifts with purchase won't count toward this threshold
- **Includes bundle items**: Hidden bundle components that should count are included
- **Backwards compatible**: Falls back to `total_price` if `calculated_subtotal` unavailable

## JavaScript API

```javascript
const gwp = document.querySelector('gift-with-purchase');

// Update cart amount
gwp.setCurrentAmount(85.5);

// Change threshold
gwp.setThreshold(100.0);

// Update variant ID
gwp.setVariantId('87654321');

// Get current state
const state = gwp.getState();
console.log(state.isActive, state.isAdded, state.remainingAmount);

// Manual message updates (component handles this automatically)
const messageEl = gwp.querySelector('[data-content-gwp-message]');
if (messageEl) {
	messageEl.textContent = 'Custom message';
}
```

## Attributes

| Attribute       | Description                                                          | Example                                                      |
| --------------- | -------------------------------------------------------------------- | ------------------------------------------------------------ |
| `threshold`     | Spending threshold to unlock the gift                                | `"75.00"`                                                    |
| `current`       | Current cart amount                                                  | `"45.00"`                                                    |
| `variant-id`    | Shopify variant ID for the gift product                              | `"12345678"`                                                 |
| `message-above` | Message shown when threshold is met                                  | `"🎉 Congratulations! You've qualified for your FREE gift!"` |
| `message-below` | Message shown when below threshold (uses `{ amount }` placeholder) | `"Add ${ amount } more to unlock your free gift! 🎁"`      |

## Message Element

The component requires a message element to display threshold messages:

```html
<gift-with-purchase threshold="75.00" variant-id="12345678">
	<!-- Your gift content with any styling -->
	<div class="gift-content">
		<h4 class="font-bold">Free Gift</h4>
		<p class="text-gray-600">Sample Description</p>
	</div>

	<!-- Required: Element where messages will be injected -->
	<p data-content-gwp-message class="bg-green-100 p-2 rounded"></p>
</gift-with-purchase>
```

## Events

The component emits custom events for integration:

```javascript
// Gift successfully added to cart
gwp.addEventListener('gwp:added', (event) => {
	console.log('Gift added:', event.detail.variantId);
});

// Gift successfully removed from cart
gwp.addEventListener('gwp:removed', (event) => {
	console.log('Gift removed:', event.detail.variantId);
});

// Error occurred during add/remove
gwp.addEventListener('gwp:error', (event) => {
	console.error('Error:', event.detail.error);
	console.log('Action:', event.detail.action); // 'add' or 'remove'
});
```

## Customization

Use CSS custom properties to customize the appearance:

```css
gift-with-purchase {
	--gwp-border-radius: 12px;
	--gwp-padding: 1.5rem;
	--gwp-bg-active: #e8f5e8;
	--gwp-border-active: #28a745;
	--gwp-text-active: #155724;
	--gwp-image-size: 80px;
}
```

## Available CSS Custom Properties

| Property              | Description            | Default   |
| --------------------- | ---------------------- | --------- |
| `--gwp-border-radius` | Border radius          | `8px`     |
| `--gwp-padding`       | Internal padding       | `1rem`    |
| `--gwp-bg-active`     | Background when active | `#e8f5e8` |
| `--gwp-bg-added`      | Background when added  | `#d4edda` |
| `--gwp-border-active` | Border when active     | `#28a745` |
| `--gwp-border-added`  | Border when added      | `#155724` |
| `--gwp-text-active`   | Text color when active | `#155724` |
| `--gwp-text-added`    | Text color when added  | `#155724` |
| `--gwp-gap`           | Gap between elements   | `1rem`    |

## Component States

The component automatically applies a `state` attribute based on its current condition:

- `state="active"` - Threshold met, gift available to add
- `state="added"` - Gift successfully added to cart
- `state="ended"` - Promo ended (component hidden)

Note: There is no `inactive` state since the component would typically not be loaded at all when below threshold in most Shopify implementations.

## Shopify Integration Details

### Cart API Calls

The component uses Shopify's Cart API endpoints:

- `POST /cart/add.js` - Adds the gift to cart
- `GET /cart.js` - Gets current cart state
- `POST /cart/change.js` - Removes gift from cart

### Line Item Properties

When adding gifts to the cart, the component automatically sets these properties for proper cart integration:

```javascript
{
  id: variantId,
  quantity: 1,
  properties: {
    _gwp_item: "true",
    _hide_in_cart: "true",
    _ignore_price_in_subtotal: "true"
  }
}
```

**Property Functions:**
- **`_gwp_item: "true"`** - Identifies gift items in cart templates and for removal logic
- **`_hide_in_cart: "true"`** - Hides gifts from cart display (handled by `@magic-spells/cart-panel`)
- **`_ignore_price_in_subtotal: "true"`** - Excludes gift price from subtotal calculations and threshold checks
- **Automatic management** - These properties ensure gifts don't interfere with other promotions or cart totals

### Cart Template Integration

In your Shopify cart template, you can identify and handle gift items:

```liquid
{% for item in cart.items %}
  {% if item.properties._gwp_item == 'true' %}
    <!-- This is a gift item -->
    <div class="cart-gift-item">
      <span class="gift-badge">FREE GIFT</span>
      {{ item.product.title }}
    </div>
  {% else %}
    <!-- Regular cart item -->
  {% endif %}
{% endfor %}
```

## Advanced Usage

### Multiple Thresholds

```html
<!-- Low tier gift -->
<gift-with-purchase
	threshold="50.00"
	variant-id="111111"
	message-above="🎉 You've unlocked a free tote bag!"
	message-below="Add ${ amount } more for a free tote bag! 👜">
	<div class="gwp-product">
		<img src="tote-bag.jpg" alt="Free Tote Bag" />
		<h4 class="gwp-title">Free Tote Bag</h4>
	</div>
	<p data-content-gwp-message class="message-low-tier"></p>
</gift-with-purchase>

<!-- High tier gift -->
<gift-with-purchase
	threshold="100.00"
	variant-id="222222"
	message-above="✨ Amazing! You've earned our premium gift set!"
	message-below="Spend ${ amount } more for our exclusive premium gift set! ✨">
	<div class="gwp-product">
		<img src="gift-set.jpg" alt="Premium Gift Set" />
		<h4 class="gwp-title">Premium Gift Set</h4>
	</div>
	<p data-content-gwp-message class="message-premium"></p>
</gift-with-purchase>
```

### Custom Styling Examples

```css
/* Minimal style */
.gwp-minimal {
	--gwp-padding: 0.75rem;
	--gwp-border-radius: 4px;
	--gwp-bg-active: #f0f8f0;
}

/* Bold style */
.gwp-bold {
	--gwp-padding: 1.5rem;
	--gwp-border-radius: 12px;
	--gwp-bg-active: #28a745;
	--gwp-text-active: white;
}

/* Compact mobile style */
@media (max-width: 768px) {
	gift-with-purchase {
		--gwp-image-size: 40px;
		--gwp-padding: 0.5rem;
		--gwp-gap: 0.5rem;
	}
}
```

## Browser Support

- Chrome/Edge 88+
- Firefox 85+
- Safari 14+
- All modern browsers with Custom Elements support

## TypeScript Support

Type definitions are included in the package:

```typescript
interface GiftWithPurchaseState {
	currentAmount: number;
	threshold: number;
	variantId: string | null;
	isActive: boolean;
	isAdded: boolean;
	remainingAmount: number;
}

// Component automatically handles message injection
// Style your content and message elements with any CSS classes
```

## Contributing

Contributions are welcome! Please read our contributing guidelines and submit pull requests to our repository.

## License

MIT License - see LICENSE file for details.
