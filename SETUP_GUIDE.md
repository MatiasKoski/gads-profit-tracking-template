# Setup Guide - Item Value (Firestore) GTM Template

## Prerequisites

Before setting up the Item Value (Firestore) template, ensure you have:

1. **Google Tag Manager** account with admin access
2. **Google Cloud Platform** project with Firestore enabled
3. **GTM Server-Side Container** (not client-side)
4. **Firestore collection** with product data
5. **E-commerce events** with items array

## Step 1: Prepare Firestore Database

### Create Firestore Collection

1. Go to [Firestore Console](https://console.firebase.google.com/)
2. Select your GCP project
3. Navigate to Firestore Database
4. Create a new collection (e.g., `products`)

### Add Product Documents

Create documents with the following structure:

```json
{
  "value": 25.99,
  "return_rate": 0.05,
  "category": "electronics",
  "lastUpdated": "2024-01-15T10:30:00Z"
}
```

**Required Fields:**
- `value`: Base profit value for the product

**Optional Fields:**
- `return_rate`: Return rate as decimal (0.05 = 5%)
- `category`: Product category for organization
- `lastUpdated`: Timestamp for data freshness

### Document ID Strategy

Use your product SKU or item ID as the document ID:
- Document ID: `prod_123`
- This will be referenced as `item_id` in GTM events

### Firestore Security Rules

Ensure your Firestore security rules allow read access:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /products/{document} {
      allow read: if true; // Adjust based on your security needs
    }
  }
}
```

## Step 2: Import GTM Template

### Method 1: Community Gallery (Recommended)

1. In GTM, go to **Templates** → **Tag Templates**
2. Click **Search Gallery**
3. Search for "Item Value (Firestore)"
4. Click **Add to Workspace**
5. Select your container and click **Add**

### Method 2: Manual Import

1. In GTM, go to **Templates** → **Tag Templates**
2. Click **New** → **Import**
3. Upload the `firestore_item_profits.tpl` file
4. Review and save the template

## Step 3: Configure Template Parameters

### Required Parameters

| Parameter | Display Name | Type | Description |
|-----------|--------------|------|-------------|
| `gcpProjectId` | GCP Project ID | Text | Your Google Cloud Project ID |
| `collectionId` | Firestore Collection ID | Text | Collection name (default: "products") |
| `valueField` | Value Field | Text | Field name for value data (default: "value") |

### Calculation Method Parameters

| Parameter | Display Name | Type | Description |
|-----------|--------------|------|-------------|
| `valueCalculation` | Value Calculation | Select | Calculation method to use |
| `returnRateField` | Return Rate Field | Text | Field name for return rate (if using return rate calculation) |

**Calculation Methods:**
- **Value**: Simple value × quantity
- **Return Rate**: Value × (1 - return rate) × quantity
- **Value with Discount**: (Value - discount) × quantity

### Fallback Parameters

| Parameter | Display Name | Type | Description |
|-----------|--------------|------|-------------|
| `fallbackValueIfNotFound` | Fallback Value Calculation | Select | Strategy when product not found |
| `fallBackPercent` | Percentage | Text | Percentage for fallback (0.1 = 10%) |

**Fallback Strategies:**
- **Zero**: Use 0 when product not found
- **Revenue**: Use item price × quantity
- **Percent**: Use item price × percentage × quantity

### Cost Parameters

| Parameter | Display Name | Type | Description |
|-----------|--------------|------|-------------|
| `shippingCost` | Shipping Cost | Text | Amount to add to final value |
| `fulfillmentCost` | Fulfillment Cost | Text | Amount to subtract from final value |

## Step 4: Create GTM Variable

### Add Variable

1. Go to **Variables** → **User-Defined Variables**
2. Click **New** → **Custom Template**
3. Select "Item Value (Firestore)" template

### Configure Variable Settings

Fill in the configuration fields:

```javascript
{
  "gcpProjectId": "your-project-id-123",
  "collectionId": "products",
  "valueField": "value",
  "valueCalculation": "returnRate",
  "returnRateField": "return_rate",
  "fallbackValueIfNotFound": "percent",
  "fallBackPercent": "0.15",
  "shippingCost": "5.99",
  "fulfillmentCost": "2.50"
}
```

### Test Configuration

1. Click **Test** to validate your configuration
2. Check the console for any error messages
3. Verify Firestore connection is working

## Step 5: Set Up Event Data

### Ensure Event Data Structure

Your GTM events should include an `items` array with this structure:

```javascript
{
  "items": [
    {
      "item_id": "prod_123",
      "price": 29.99,
      "quantity": 2,
      "discount": 5.00
    },
    {
      "item_id": "prod_456",
      "price": 15.50,
      "quantity": 1
    }
  ]
}
```

### Common Event Types

This template works with various e-commerce events:

- **Purchase**: Complete transaction data
- **Add to Cart**: Cart addition events
- **View Item**: Product view events
- **Begin Checkout**: Checkout initiation

### Data Layer Example

```javascript
dataLayer.push({
  'event': 'purchase',
  'ecommerce': {
    'items': [
      {
        'item_id': 'prod_123',
        'price': 29.99,
        'quantity': 2,
        'discount': 5.00
      }
    ]
  }
});
```

## Step 6: Create Tags and Triggers

### Create Conversion Tag

1. Go to **Tags** → **New**
2. Choose your preferred tag type (GA4, GA3, etc.)
3. Add the Item Value variable to your tag
4. Configure other tag settings as needed

### Example GA4 Purchase Tag

```javascript
{
  "event_name": "purchase",
  "value": "{{Item Value Variable}}",
  "currency": "USD",
  "items": "{{Event Items}}"
}
```

### Set Up Triggers

Create triggers for events that should calculate profit:

- **Purchase** events
- **Add to Cart** events
- **Custom events** with item data

### Trigger Configuration Example

```javascript
{
  "name": "Purchase Event",
  "type": "Custom Event",
  "eventName": "purchase",
  "conditions": [
    {
      "type": "eventName",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "purchase"
        }
      ]
    }
  ]
}
```

## Step 7: Test Implementation

### Preview Mode Testing

1. Enable **Preview Mode** in GTM
2. Trigger an event with item data
3. Check the **Variables** tab for your Item Value variable
4. Verify the calculated value is correct

### Debug Steps

1. **Check Console Logs**: Look for any error messages
2. **Verify Firestore Data**: Ensure documents exist with correct IDs
3. **Test Event Data**: Confirm items array is properly formatted
4. **Check Configuration**: Validate all required fields are set

### Common Test Scenarios

- **Single Item Purchase**: Basic functionality test
- **Multiple Items**: Verify batch processing
- **Missing Item ID**: Test fallback behavior
- **Firestore Error**: Test error handling
- **Invalid Data**: Test data validation

### Sample Test Data

```javascript
// Test with existing product
{
  "items": [
    {
      "item_id": "prod_123",
      "price": 29.99,
      "quantity": 1
    }
  ]
}

// Test with missing product
{
  "items": [
    {
      "item_id": "missing_product",
      "price": 29.99,
      "quantity": 1
    }
  ]
}
```

## Step 8: Monitor and Optimize

### Set Up Monitoring

1. **GTM Debug Mode**: Regular testing in preview mode
2. **Firestore Logs**: Monitor read operations and errors
3. **Analytics**: Track profit values in your analytics platform

### Performance Optimization

1. **Firestore Indexes**: Ensure proper indexing for your queries
2. **Data Structure**: Optimize document structure for quick reads
3. **Caching Strategy**: Consider implementing caching if needed

### Maintenance Tasks

1. **Regular Testing**: Test with new products and scenarios
2. **Data Updates**: Keep Firestore data current
3. **Configuration Review**: Periodically review template settings
4. **Error Monitoring**: Watch for increased error rates

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "No item ID in item" | Missing item_id in event data | Ensure events include item_id |
| Firestore errors | Invalid project ID or permissions | Check GCP project and Firestore rules |
| Zero values | Missing Firestore documents | Create documents for all products |
| Calculation errors | Invalid field names | Verify field names match Firestore data |
| Template not found | Template not imported | Import template from gallery or file |

### Debug Checklist

- [ ] GCP project ID is correct
- [ ] Firestore collection exists
- [ ] Product documents are created
- [ ] Event data includes items array
- [ ] Items have valid item_id values
- [ ] Template configuration is complete
- [ ] Firestore security rules allow reads
- [ ] GTM has proper permissions

### Error Messages

**"Error retrieving Firestore document"**
- Check if document exists in Firestore
- Verify collection ID and item_id
- Check Firestore security rules

**"No item ID in item"**
- Ensure event data includes item_id
- Check data layer structure
- Verify event triggering

**"Skipping invalid item value"**
- Check Firestore document structure
- Verify value field exists and is numeric
- Review calculation method configuration

## Advanced Configuration

### Multiple Collections

For different product types, you can create multiple variables:

```javascript
// Electronics products
{
  "collectionId": "electronics",
  "valueCalculation": "returnRate"
}

// Clothing products
{
  "collectionId": "clothing",
  "valueCalculation": "valueWithDiscount"
}
```

### Dynamic Configuration

Use GTM variables to make configuration dynamic:

```javascript
{
  "collectionId": "{{Product Category}}",
  "valueCalculation": "{{Calculation Method}}"
}
```

### Environment-Specific Settings

Use different configurations for development/production:

```javascript
// Development
{
  "gcpProjectId": "dev-project",
  "collectionId": "dev-products"
}

// Production
{
  "gcpProjectId": "prod-project",
  "collectionId": "products"
}
```

## Support Resources

- [GTM Template Documentation](https://developers.google.com/tag-manager/templates)
- [Firestore Documentation](https://firebase.google.com/docs/firestore)
- [GTM Community](https://support.google.com/tagmanager/community)
- [Server-Side GTM Guide](https://developers.google.com/tag-platform/tag-manager/server-side) 