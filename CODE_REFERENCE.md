# Code Reference - Item Value (Firestore) GTM Template

## Template Structure

The Item Value (Firestore) template is organized into several sections:

1. **Template Metadata** - Configuration and display information
2. **Template Parameters** - User-configurable settings
3. **JavaScript Code** - Core calculation logic
4. **Server Permissions** - Required GTM permissions
5. **Tests** - Unit test scenarios

## Template Metadata

```json
{
  "type": "MACRO",
  "id": "cvt_temp_public_id",
  "version": 1,
  "securityGroups": [],
  "displayName": "Item Value (Firestore)",
  "description": "Variable that retrieves values from Firestore for each item_id in the items array of the event data.",
  "containerContexts": ["SERVER"]
}
```

## Core Dependencies

The template uses these GTM server-side APIs:

```javascript
const Firestore = require("Firestore");
const Promise = require("Promise");
const getEventData = require("getEventData");
const logToConsole = require("logToConsole");
const makeNumber = require("makeNumber");
const makeString = require("makeString");
const Math = require("Math");
const getType = require("getType");
```

## Function Reference

### `sumValues(values)`

**Purpose**: Aggregates item values and applies shipping/fulfillment costs.

**Parameters**:
- `values` (Array<number>): Array of calculated item values

**Returns**: String representation of total profit

**Algorithm**:
```javascript
function sumValues(values) {
  var total = 0;

  // Sum all valid numeric values
  for (var i = 0; i < values.length; i++) {
    var itemValue = values[i];
    if (getType(itemValue) === "number") {
      total = total + itemValue;
    } else {
      logToConsole("Skipping invalid item value:", itemValue);
    }
  }

  // Add shipping cost if provided
  var shippingCost = makeNumber(data.shippingCost);
  if (getType(shippingCost) === "number" && shippingCost > 0) {
    total = total + shippingCost;
  }

  // Subtract fulfillment cost if provided
  var fulfillmentCost = makeNumber(data.fulfillmentCost);
  if (getType(fulfillmentCost) === "number" && fulfillmentCost > 0) {
    total = total - fulfillmentCost;
  }

  // Ensure non-negative result
  if (total < 0) {
    total = 0;
  }

  return makeString(total);
}
```

**Key Features**:
- Validates each value is numeric before adding
- Logs warnings for invalid values
- Applies shipping and fulfillment costs
- Clamps result to 0 or greater
- Returns string for GTM compatibility

### `getItemValues(items)`

**Purpose**: Processes all items in parallel to fetch their values from Firestore.

**Parameters**:
- `items` (Array<Object>): Array of item objects from event data

**Returns**: Promise<Array<number>>

**Algorithm**:
```javascript
function getItemValues(items) {
  const valueRequests = [];
  for (const item of items) {
    valueRequests.push(getFirestoreValue(item));
  }
  return Promise.all(valueRequests);
}
```

**Key Features**:
- Creates parallel requests for all items
- Uses Promise.all for concurrent execution
- Returns array of calculated values

### `getDefaultValue(item)`

**Purpose**: Calculates fallback value when Firestore document is not found.

**Parameters**:
- `item` (Object): Item object containing price and quantity

**Returns**: number

**Algorithm**:
```javascript
function getDefaultValue(item) {
  let value;
  const quantity = item.hasOwnProperty("quantity") ? item.quantity : 1;

  switch (data.fallbackValueIfNotFound) {
    case "zero":
      value = 0;
      break;

    case "revenue":
      value = makeNumber(item.price) * makeNumber(quantity);
      break;

    case "percent":
      const percent = makeNumber(data.fallBackPercent);
      value = makeNumber(item.price) * percent * makeNumber(quantity);
      value = roundValue(value);
      break;
  }
  return value;
}
```

**Fallback Strategies**:
- **zero**: Always returns 0
- **revenue**: Uses item price × quantity
- **percent**: Uses item price × percentage × quantity

**Key Features**:
- Handles missing quantity (defaults to 1)
- Uses makeNumber() for type safety
- Applies rounding for percent calculations

### `calculateValue(item, fsDocument)`

**Purpose**: Calculates final value based on Firestore data and configuration.

**Parameters**:
- `item` (Object): Item object with quantity and optional discount
- `fsDocument` (Object): Firestore document data

**Returns**: number

**Algorithm**:
```javascript
function calculateValue(item, fsDocument) {
  let value;
  const quantity = item.hasOwnProperty("quantity") ? item.quantity : 1;
  const documentValue = makeNumber(fsDocument.data[data.valueField]);

  switch (data.valueCalculation) {
    case "valueQuantity":
      value = documentValue * makeNumber(quantity);
      break;

    case "returnRate":
      const returnRate = makeNumber(fsDocument.data[data.returnRateField]);
      value = (1 - returnRate) * documentValue * quantity;
      value = roundValue(value);
      break;

    case "valueWithDiscount":
      const discount = item.hasOwnProperty("discount") ? item.discount : 0;
      value = (documentValue - discount) * quantity;
      break;
  }
  return value;
}
```

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

**Key Features**:
- Handles missing quantity (defaults to 1)
- Uses makeNumber() for type safety
- Applies rounding for return rate calculations
- Handles missing discount (defaults to 0)

### `roundValue(value)`

**Purpose**: Rounds a number to 2 decimal places.

**Parameters**:
- `value` (number): Number to round

**Returns**: number

**Algorithm**:
```javascript
function roundValue(value) {
  return Math.round(value * 100) / 100;
}
```

**Key Features**:
- Rounds to 2 decimal places for currency precision
- Uses standard JavaScript Math.round()

### `getFirestoreValue(item)`

**Purpose**: Retrieves and calculates value for a single item from Firestore.

**Parameters**:
- `item` (Object): Item object with item_id

**Returns**: Promise<number>

**Algorithm**:
```javascript
function getFirestoreValue(item) {
  let value = getDefaultValue(item);

  if (!item.item_id) {
    logToConsole("No item ID in item");
    return value;
  }

  const path = data.collectionId + "/" + item.item_id;

  let firestore = Firestore;
  if (getType(Firestore) === "function") {
    firestore = Firestore();
  }

  return Promise.create((resolve) => {
    return firestore.read(path, { projectId: data.gcpProjectId })
      .then((fsDocument) => {
        value = calculateValue(item, fsDocument, value);
      })
      .catch((error) => {
        logToConsole("Error retrieving Firestore document `" + path + "`", error);
      })
      .finally(() => {
        resolve(value);
      });
  });
}
```

**Key Features**:
- Gets default value as fallback
- Validates item has item_id
- Constructs Firestore document path
- Handles Firestore instance creation
- Uses Promise.create for async operations
- Logs errors but continues with default value
- Always resolves with a value

## Entry Point

```javascript
// Entry point
const items = getEventData("items");
logToConsole("items", items);
return getItemValues(items)
  .then(sumValues)
  .catch((error) => {
    logToConsole("Error", error);
  });
```

**Flow**:
1. Gets items array from event data
2. Logs items for debugging
3. Processes all items to get values
4. Sums values and applies costs
5. Returns final result as string
6. Logs any errors that occur

## Configuration Parameters

### Required Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `gcpProjectId` | TEXT | Google Cloud Project ID | - |
| `collectionId` | TEXT | Firestore collection name | "products" |
| `valueField` | TEXT | Field name for value data | "value" |

### Calculation Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `valueCalculation` | SELECT | Calculation method | "valueQuantity" |
| `returnRateField` | TEXT | Field name for return rate | "return_rate" |

### Fallback Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `fallbackValueIfNotFound` | SELECT | Fallback strategy | "percent" |
| `fallBackPercent` | TEXT | Percentage for fallback | 0.1 |

### Cost Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `shippingCost` | TEXT | Shipping cost to add | 0 |
| `fulfillmentCost` | TEXT | Fulfillment cost to subtract | 0 |

## Error Handling

### Error Categories

1. **Missing Item ID**
   ```javascript
   if (!item.item_id) {
     logToConsole("No item ID in item");
     return value; // Returns default value
   }
   ```

2. **Firestore Errors**
   ```javascript
   .catch((error) => {
     logToConsole("Error retrieving Firestore document `" + path + "`", error);
   })
   ```

3. **Invalid Values**
   ```javascript
   if (getType(itemValue) === "number") {
     total = total + itemValue;
   } else {
     logToConsole("Skipping invalid item value:", itemValue);
   }
   ```

4. **Configuration Errors**
   - Handled by GTM template validation
   - Required fields must be provided

### Error Recovery

- **Graceful Degradation**: Individual failures don't affect others
- **Default Values**: Fallback strategies prevent complete failures
- **Comprehensive Logging**: Detailed error information for debugging
- **Type Safety**: Input validation prevents runtime errors

## Performance Characteristics

### Time Complexity
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

## Testing Framework

The template includes comprehensive unit tests covering:

### Test Scenarios
- **Collection ID Usage**: Verifies correct Firestore path construction
- **Value Field Parsing**: Tests field name configuration
- **Calculation Methods**: Tests all three calculation types
- **Fallback Strategies**: Tests all fallback options
- **Rounding**: Tests decimal precision handling

### Mock Framework
```javascript
// Event data mocking
function addMockEventData(items) {
  mock("getEventData", (data) => {
    if (data === "items") {
      return items;
    }
  });
}

// Firestore mocking
function addMockFirestore(firestoreDocs) {
  mock("Firestore", () => {
    return {
      "read": (path, options) => {
        const sku = path.replace(mockData.collectionId + "/", "");
        const doc = firestoreDocs[sku];
        return Promise.create((resolve) => {
          resolve(doc);
        });
      }
    };
  });
}
```

## Server Permissions

The template requires these GTM server-side permissions:

### Logging Permission
```json
{
  "instance": {
    "key": {
      "publicId": "logging",
      "versionId": "1"
    },
    "param": [
      {
        "key": "environments",
        "value": {
          "type": 1,
          "string": "all"
        }
      }
    ]
  }
}
```

### Firestore Permission
```json
{
  "instance": {
    "key": {
      "publicId": "access_firestore",
      "versionId": "1"
    },
    "param": [
      {
        "key": "allowedOptions",
        "value": {
          "type": 2,
          "listItem": [
            {
              "type": 3,
              "mapKey": [
                {"type": 1, "string": "projectId"},
                {"type": 1, "string": "path"},
                {"type": 1, "string": "operation"},
                {"type": 1, "string": "databaseId"}
              ],
              "mapValue": [
                {"type": 1, "string": "beredd-server-side-tracking"},
                {"type": 1, "string": "products/*"},
                {"type": 1, "string": "read"},
                {"type": 1, "string": "(default)"}
              ]
            }
          ]
        }
      }
    ]
  }
}
```

### Event Data Permission
```json
{
  "instance": {
    "key": {
      "publicId": "read_event_data",
      "versionId": "1"
    },
    "param": [
      {
        "key": "eventDataAccess",
        "value": {
          "type": 1,
          "string": "any"
        }
      }
    ]
  }
}
```

## Best Practices

### Code Organization
- **Modular Functions**: Each function has a single responsibility
- **Clear Naming**: Function names describe their purpose
- **Consistent Patterns**: Similar operations use consistent approaches
- **Error Handling**: Comprehensive error handling throughout

### Performance Optimization
- **Parallel Processing**: All Firestore requests made concurrently
- **Type Safety**: Input validation prevents runtime errors
- **Efficient Calculations**: Optimized for minimal processing time
- **Memory Management**: Minimal memory footprint

### Maintainability
- **Comprehensive Logging**: Detailed logging for debugging
- **Clear Documentation**: JSDoc comments for all functions
- **Test Coverage**: Extensive unit test coverage
- **Error Recovery**: Graceful handling of all error scenarios 