# Item Value (Firestore) - GTM Variable Template

A Google Tag Manager variable template that retrieves values from Firestore for each item_id in the items array of event data. This template calculates profit margins, handles return rates, applies discounts, and manages shipping/fulfillment costs.

## 📋 Overview

This template is designed for e-commerce businesses that need to calculate accurate profit values by fetching product-specific data from Firestore. It supports multiple calculation methods and provides robust fallback strategies when product data is not available.

## 🚀 Quick Start

1. **Import the template** into your GTM container
2. **Configure Firestore settings** (GCP Project ID, Collection ID)
3. **Set up your calculation method** (Value, Return Rate, or Value with Discount)
4. **Configure fallback strategies** for missing products
5. **Test with your event data**

## 🔧 Configuration Options

### Required Settings
- **GCP Project ID**: Your Google Cloud Platform project where Firestore is located
- **Firestore Collection ID**: The collection containing your product data (default: "products")
- **Value Field**: Field name in Firestore document containing the value data (default: "value")

### Calculation Methods
- **Value**: Simple value × quantity calculation
- **Return Rate**: Value × (1 - return rate) × quantity (accounts for product returns)
- **Value with Discount**: (Value - discount) × quantity (applies item-level discounts)

### Fallback Strategies
- **Zero**: Use 0 when product not found
- **Revenue**: Use item price × quantity
- **Percent**: Use item price × percentage × quantity

### Cost Adjustments
- **Shipping Cost**: Amount to add to final order value
- **Fulfillment Cost**: Amount to subtract from final order value

## 📊 Example Usage

### Firestore Document Structure
```json
{
  "value": 25.99,
  "return_rate": 0.05,
  "category": "electronics"
}
```

### Event Data Structure
```json
{
  "items": [
    {
      "item_id": "prod_123",
      "price": 29.99,
      "quantity": 2,
      "discount": 5.00
    }
  ]
}
```

### Sample Configuration
```javascript
{
  "gcpProjectId": "my-project-123",
  "collectionId": "products",
  "valueField": "value",
  "valueCalculation": "returnRate",
  "returnRateField": "return_rate",
  "fallbackValueIfNotFound": "percent",
  "fallBackPercent": 0.15,
  "shippingCost": 5.99,
  "fulfillmentCost": 2.50
}
```

## 🔍 How It Works

1. **Event Trigger**: GTM event fires with items array
2. **Item Processing**: Each item is processed to fetch its value from Firestore
3. **Value Calculation**: Values are calculated based on the selected method
4. **Fallback Handling**: Default values are used if Firestore document is not found
5. **Aggregation**: All item values are summed together
6. **Cost Adjustment**: Shipping and fulfillment costs are applied
7. **Output**: Final calculated value is returned as a string

## 🛠️ Setup Instructions

### 1. Prepare Firestore Data
- Create a collection in Firestore (e.g., "products")
- Add documents with your product data
- Use product SKUs as document IDs
- Include required fields (value, return_rate if needed)

### 2. Configure GTM Template
- Import the template into your GTM container
- Set required configuration fields
- Configure calculation method and fallback strategy
- Set shipping and fulfillment costs

### 3. Create Variable
- Create a new variable using this template
- Configure with your specific settings
- Test with sample event data

### 4. Use in Tags
- Add the variable to your conversion tracking tags
- Use for purchase events, add to cart, etc.
- Monitor performance and adjust as needed

## 📈 Performance Considerations

- **Parallel Processing**: All Firestore requests are made concurrently
- **Error Isolation**: Individual item failures don't affect others
- **Fallback Protection**: Default values prevent complete failures
- **Efficient Calculations**: Optimized for minimal processing time

## 🚨 Error Handling

The template includes comprehensive error handling:
- **Missing item IDs**: Logs warning and uses default value
- **Firestore errors**: Logs error and uses default value
- **Invalid data**: Skips invalid values in calculations
- **Configuration errors**: Provides helpful error messages

## 🔧 Advanced Features

### Return Rate Calculations
Account for product return rates in profit calculations:
```
Final Value = Base Value × (1 - Return Rate) × Quantity
```

### Discount Handling
Apply item-level discounts to calculations:
```
Final Value = (Base Value - Discount) × Quantity
```

### Flexible Fallback Strategies
Multiple options when product data is unavailable:
- Use zero for conservative estimates
- Use revenue for maximum value
- Use percentage of price for estimated margins

## 📝 Best Practices

1. **Use meaningful field names** in Firestore documents
2. **Set appropriate fallback strategies** for your business
3. **Monitor logs** for Firestore errors and missing items
4. **Keep Firestore data current** and accurate
5. **Test with various scenarios** to ensure fallback logic works
6. **Use consistent item_id formats** across your system

## 🐛 Troubleshooting

### Common Issues
- **"No item ID in item"**: Ensure event data includes item_id for each item
- **Firestore errors**: Check collection ID, project ID, and document structure
- **Zero values**: Verify fallback configuration and item data
- **Calculation errors**: Ensure Firestore documents contain required fields

### Debug Steps
1. Check GTM console logs for error messages
2. Verify Firestore document exists and has correct structure
3. Confirm configuration variables are set correctly
4. Test with simple item data to isolate issues

## 📚 Additional Resources

- [GTM Template Documentation](https://developers.google.com/tag-manager/templates)
- [Firestore Documentation](https://firebase.google.com/docs/firestore)
- [GTM Community](https://support.google.com/tagmanager/community)

## 🔄 Version History

- **v3.1.2**: Current version with enhanced error handling and documentation
- **v3.0**: Added return rate calculations and improved fallback strategies
- **v2.0**: Added discount handling and shipping/fulfillment costs
- **v1.0**: Initial release with basic Firestore integration 