# Item Value (Firestore) - GTM Variable Template

A Google Tag Manager variable template that retrieves values from Firestore for each item_id in the items array of event data. This template calculates profit margins, handles return rates, applies discounts, and manages shipping/fulfillment costs.

## 🎯 Purpose: POAS (Profit on Ad Spend) Optimization

This template is specifically designed for **POAS (Profit on Ad Spend) optimization** in digital advertising campaigns. It enables advertisers to send accurate profit data to advertising platforms like Google Ads, Facebook Ads, and other ad networks to optimize campaigns based on actual profitability rather than just revenue.

### Why POAS Optimization Matters

- **Better ROAS**: Focus on profit instead of just revenue
- **Improved Campaign Performance**: Optimize for actual business value
- **Accurate Bidding**: Bid based on real profit margins
- **Cost Efficiency**: Reduce wasted ad spend on low-profit products
- **Competitive Advantage**: Outperform competitors using revenue-only optimization

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

### Google Ads Tag Example
```javascript
// GA4 Purchase Tag with Profit Value
{
  "event_name": "purchase",
  "value": "{{Item Value Variable}}", // Profit value instead of revenue
  "currency": "USD",
  "items": "{{Event Items}}"
}
```

### Facebook Ads Tag Example
```javascript
// Facebook Conversion API with Profit Value
{
  "event_name": "Purchase",
  "value": "{{Item Value Variable}}", // Profit value for optimization
  "currency": "USD",
  "content_ids": "{{Item IDs}}",
  "content_type": "product"
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

## 📊 Advertising Platform Integration

### Google Ads Integration
Use this template to send profit data to Google Ads for:
- **Conversion Value**: Set profit as the conversion value instead of revenue
- **Smart Bidding**: Enable Target ROAS or Maximize Conversion Value bidding
- **Campaign Optimization**: Let Google Ads optimize for profit-based performance
- **Audience Insights**: Better understand which customers drive actual profit

### Facebook Ads Integration
Send profit data to Facebook Ads for:
- **Value Optimization**: Optimize for profit value instead of purchase events
- **Dynamic Ads**: Use profit data in dynamic product ads
- **Custom Audiences**: Create audiences based on profit-generating customers
- **Campaign Budget Optimization**: Distribute budget based on profit performance

### Other Advertising Platforms
This template works with any platform that accepts conversion value data:
- **Microsoft Advertising**
- **TikTok Ads**
- **LinkedIn Ads**
- **Amazon Advertising**
- **Custom tracking platforms**

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

## 💰 Business Benefits

### POAS Optimization Advantages
- **Higher Profit Margins**: Focus ad spend on high-profit products
- **Better Budget Allocation**: Distribute budget based on actual profitability
- **Improved Customer Lifetime Value**: Target customers who drive real profit
- **Reduced Customer Acquisition Cost**: Optimize for profitable conversions
- **Data-Driven Decisions**: Make campaign decisions based on profit data

### Competitive Edge
- **Outperform Revenue-Based Campaigns**: Beat competitors using basic ROAS
- **Smarter Bidding**: Bid based on actual profit potential
- **Product-Level Optimization**: Optimize individual product performance
- **Seasonal Profit Adjustments**: Account for varying profit margins
- **Return Rate Considerations**: Factor in product returns and refunds

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