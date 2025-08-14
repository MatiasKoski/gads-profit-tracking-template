# Technical Documentation - Item Value (Firestore) GTM Template

## Architecture Overview

The Item Value (Firestore) template is a Google Tag Manager variable that implements a distributed calculation pattern for e-commerce profit tracking. It combines real-time Firestore data with event-driven calculations to provide accurate profit values for conversion tracking.

## System Components

### Core Dependencies
- **Firestore**: Google's NoSQL database for product data storage
- **GTM Server-Side**: Processing environment for template execution
- **Event Data**: Items array from e-commerce events
- **Configuration**: Template parameters for calculation logic

### Data Flow Architecture
```
Event Data → Item Validation → Firestore Lookup → Value Calculation → Aggregation → Cost Adjustment → Output
```

## Function Specifications

### `sumValues(values: Array<number>): string`

**Purpose**: Aggregates item values and applies shipping/fulfillment costs.

**Algorithm**:
```javascript
total = 0
for each value in values:
  if value is number:
    total += value
  else:
    log warning and skip

if shippingCost > 0:
  total += shippingCost

if fulfillmentCost > 0:
  total -= fulfillmentCost

if total < 0:
  total = 0

return total as string
```

**Parameters**:
- `values`: Array of calculated item values

**Returns**: String representation of total profit

**Time Complexity**: O(n) where n is number of items
**Space Complexity**: O(1)

### `getItemValues(items: Array<Object>): Promise<Array<number>>`

**Purpose**: Processes all items in parallel to fetch their values from Firestore.

**Algorithm**:
```javascript
promises = []
for each item in items:
  promises.push(getFirestoreValue(item))

return Promise.all(promises)
```

**Parameters**:
- `items`: Array of item objects from event data

**Returns**: Promise resolving to array of calculated values

**Time Complexity**: O(n) parallel operations
**Space Complexity**: O(n) for promise array

### `getDefaultValue(item: Object): number`

**Purpose**: Calculates fallback value when Firestore document is not found.

**Fallback Logic**:
- **zero**: Always returns 0
- **revenue**: `item.price × item.quantity`
- **percent**: `item.price × fallBackPercent × item.quantity`

**Parameters**:
- `item`: Item object containing price and quantity

**Returns**: Calculated fallback value

**Edge Cases**:
- Missing quantity defaults to 1
- Invalid price defaults to 0
- Invalid percentage defaults to 0.1

### `calculateValue(item: Object, fsDocument: Object): number`

**Purpose**: Calculates final value based on Firestore data and configuration.

**Calculation Methods**:

1. **valueQuantity**
   ```
   result = documentValue × quantity
   ```

2. **returnRate**
   ```
   returnRate = fsDocument.data[returnRateField]
   result = (1 - returnRate) × documentValue × quantity
   result = round(result, 2)
   ```

3. **valueWithDiscount**
   ```
   discount = item.discount || 0
   result = (documentValue - discount) × quantity
   ```

**Parameters**:
- `item`: Item object with quantity and optional discount
- `fsDocument`: Firestore document data

**Returns**: Calculated value

### `roundValue(value: number): number`

**Purpose**: Rounds a number to 2 decimal places.

**Algorithm**:
```javascript
return Math.round(value * 100) / 100
```

**Parameters**:
- `value`: Number to round

**Returns**: Rounded number

### `getFirestoreValue(item: Object): Promise<number>`

**Purpose**: Retrieves and calculates value for a single item from Firestore.

**Algorithm**:
```javascript
defaultValue = getDefaultValue(item)

if !item.item_id:
  log warning
  return Promise.resolve(defaultValue)

path = collectionId + "/" + item.item_id
firestore = getFirestoreInstance()

return firestore.read(path, { projectId: gcpProjectId })
  .then(document => calculateValue(item, document))
  .catch(error => {
    log error
    return defaultValue
  })
```

**Parameters**:
- `item`: Item object with item_id

**Returns**: Promise resolving to calculated value

## Data Models

### Item Object
```typescript
interface Item {
  item_id: string;           // Required: Product identifier
  price: number;             // Required: Item price
  quantity?: number;         // Optional: Defaults to 1
  discount?: number;         // Optional: Item discount
}
```

### Firestore Document
```typescript
interface FirestoreDocument {
  data: {
    [valueField]: number;    // Required: Base value field
    [returnRateField]?: number; // Optional: Return rate (0-1)
    [key: string]: any;      // Additional fields
  };
}
```

### Configuration Object
```typescript
interface Config {
  gcpProjectId: string;
  collectionId: string;
  valueField: string;
  valueCalculation: 'valueQuantity' | 'returnRate' | 'valueWithDiscount';
  returnRateField?: string;
  fallbackValueIfNotFound: 'zero' | 'revenue' | 'percent';
  fallBackPercent?: number;
  shippingCost: number;
  fulfillmentCost: number;
}
```

## Performance Characteristics

### Time Complexity Analysis
- **Best Case**: O(n) for n items (all cached/default values)
- **Worst Case**: O(n) for n items (all require Firestore calls)
- **Average Case**: O(n) with some Firestore calls

### Space Complexity
- **Memory**: O(n) for storing item values array
- **Network**: O(n) concurrent Firestore requests

### Latency Considerations
- **Firestore Read**: ~50-200ms per document
- **Parallel Processing**: All requests made concurrently
- **Total Time**: Max(single Firestore read time) + calculation time

## Error Handling Strategy

### Error Categories

1. **Missing Item ID**
   - **Detection**: `!item.item_id`
   - **Recovery**: Use default value
   - **Logging**: Warning message

2. **Firestore Connection Failure**
   - **Detection**: Promise rejection
   - **Recovery**: Use default value
   - **Logging**: Error with path and details

3. **Invalid Document Data**
   - **Detection**: Missing required fields or invalid types
   - **Recovery**: Use default value
   - **Logging**: Error with document path

4. **Configuration Errors**
   - **Detection**: Missing required config fields
   - **Recovery**: Template may fail completely
   - **Logging**: Error during execution

### Error Recovery Mechanisms
- **Graceful Degradation**: Individual failures don't affect others
- **Default Values**: Fallback strategies prevent complete failures
- **Comprehensive Logging**: Detailed error information for debugging
- **Type Safety**: Input validation prevents runtime errors

## Security Considerations

### Data Access
- **Firestore Security Rules**: Must allow read access to collection
- **Project ID Validation**: Must be valid and accessible
- **Path Construction**: No path traversal vulnerabilities

### Input Validation
- **Item ID**: Should be validated for format/size
- **Numeric Values**: All calculations use `makeNumber()` for safety
- **String Output**: Final result converted to string for GTM compatibility

### Permission Requirements
- **Firestore Read Access**: Template requires read permissions
- **Event Data Access**: Template reads from event data
- **Logging Permissions**: Template logs for debugging

## Monitoring & Observability

### Logging Points
1. **Item Processing**: Log items array for debugging
2. **Missing IDs**: Warning for items without item_id
3. **Firestore Errors**: Error details with document path
4. **Invalid Values**: Warning for non-numeric values in summation
5. **Final Result**: Log total calculated value

### Metrics to Track
- **Success Rate**: Percentage of successful Firestore reads
- **Fallback Usage**: How often default values are used
- **Processing Time**: Total time for calculation
- **Error Types**: Distribution of different error scenarios

### Debug Information
- **Event Data**: Complete items array logged
- **Configuration**: Template settings logged
- **Firestore Paths**: Document paths being accessed
- **Calculation Results**: Individual item calculations

## Scalability Considerations

### Current Limitations
- **No Caching**: Each request hits Firestore
- **No Rate Limiting**: Could overwhelm Firestore with many items
- **No Batching**: Individual requests for each item
- **Memory**: All values held in memory during processing

### Potential Bottlenecks
- **Firestore Quotas**: Read operations per second
- **Network Latency**: Firestore response times
- **Memory Usage**: Large item arrays
- **GTM Timeout**: Template execution time limits

### Optimization Opportunities
- **Implement Caching**: Reduce Firestore calls
- **Batch Processing**: Group multiple requests
- **Rate Limiting**: Prevent quota exhaustion
- **Memory Management**: Optimize for large datasets

## Integration Points

### GTM Integration
- **Event Data**: Uses `getEventData("items")`
- **Return Format**: String value for GTM variables
- **Error Handling**: Returns "0" on complete failure

### Firestore Integration
- **Authentication**: Uses GTM's Firestore service
- **Read Operations**: Single document reads by item_id
- **Error Handling**: Graceful degradation to defaults

### Configuration Integration
- **Template Fields**: All config via `data` object
- **Validation**: Runtime validation of required fields
- **Defaults**: Sensible defaults for optional fields

## Testing Strategy

### Unit Tests
The template includes comprehensive test scenarios:
- **Collection ID Usage**: Verifies correct Firestore path construction
- **Value Field Parsing**: Tests field name configuration
- **Calculation Methods**: Tests all three calculation types
- **Fallback Strategies**: Tests all fallback options
- **Rounding**: Tests decimal precision handling

### Test Coverage
- **Happy Path**: Normal operation with valid data
- **Error Scenarios**: Missing data, Firestore errors
- **Edge Cases**: Zero values, large numbers, invalid types
- **Configuration**: Different parameter combinations

### Mock Framework
- **Event Data Mocking**: Simulates GTM event data
- **Firestore Mocking**: Simulates Firestore responses
- **Promise Handling**: Tests async operations
- **Error Simulation**: Tests error conditions

## Deployment Considerations

### Environment Setup
- **GTM Container**: Server-side container required
- **Firestore Database**: Must be accessible from GTM
- **Security Rules**: Proper read permissions configured
- **Monitoring**: Logging and error tracking enabled

### Configuration Management
- **Environment Variables**: GCP Project ID can use environment variable
- **Default Values**: Sensible defaults for optional fields
- **Validation**: Runtime validation of required fields
- **Documentation**: Clear parameter descriptions

### Maintenance Tasks
- **Regular Testing**: Test with new products and scenarios
- **Data Updates**: Keep Firestore data current
- **Configuration Review**: Periodically review template settings
- **Error Monitoring**: Watch for increased error rates 