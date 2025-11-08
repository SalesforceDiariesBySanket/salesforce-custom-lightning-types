# Custom Lightning Types - Comprehensive Guide

## What are Custom Lightning Types?

Custom Lightning Types are a powerful feature in Salesforce that allow you to customize the appearance and behavior of user interfaces for **Agentforce Service agents** (via Enhanced Chat v2) and **Agentforce Employee agents** in Lightning Experience.

### Key Points:
- **Purpose**: Override default UI to manage complex interactions within Salesforce
- **Use Case**: Enhance agent action output UI for better user experience
- **Limitation**: Can only override UI for agent actions that use **Apex classes as input or output**
- **API Version**: Available in API version 64.0 and later

---

## Why Use Custom Lightning Types?

### 1. **Enhanced UI Customization**
Standard Lightning types have predefined UI components that don't always fit your design needs. Custom Lightning types give you full control over:
- Tailored components matching specific styling requirements
- Custom behavior requirements
- Exact interface appearance and functionality

### 2. **Handling Complex Data Structures**
Standard Lightning types sometimes can't handle complex data, but custom Lightning types can manage:
- Deeply nested objects
- Complex arrays
- Dynamic fields that change based on user input

### Before and After Comparison

| **Before (Without Custom Lightning Type)** | **After (With Custom Lightning Type)** |
|---------------------------------------------|----------------------------------------|
| ❌ No labels | ✅ Clear labels |
| ❌ Cluttered, unappealing layout | ✅ Appealing, organized layout |
| ❌ No interactive buttons | ✅ Prominent "Book Now" button |
| ❌ No visual highlights | ✅ Highlighted discounts |

---

## LightningTypeBundle Structure

Custom Lightning types are implemented using the **LightningTypeBundle** metadata type.

### Directory Structure

```
+--myMetadataPackage
    +--lightningTypes/                      (1)
        +--TypeName/                        (2)
           +--schema.json                   (3)
           +--lightningDesktopGenAi/        (4)
              +--editor.json                (5) OR
              +--renderer.json              (6)
```

### Components Explained

1. **lightningTypes/** - Contains folders for each custom Lightning type
2. **TypeName/** - Individual folder for each custom Lightning type
3. **schema.json** - Defines the JSON schema for validation (REQUIRED)
4. **lightningDesktopGenAi/** - Optional folder for UI overrides
5. **editor.json** - Custom UI for INPUT (editing/entering data)
6. **renderer.json** - Custom UI for OUTPUT (displaying data)

---

## The Three Configuration Files

### 1. **schema.json** (Required)
Defines the data structure and links to an Apex class.

**Example:**
```json
{
  "title": "My Flight Response",
  "description": "My Flight Response",
  "lightning:type": "@apexClassType/c__AvailableFlight"
}
```

**Purpose:**
- Maps custom Lightning type to an Apex class
- Uses `@apexClassType` reference format
- `c__` prefix indicates namespace (c = local org)

### 2. **renderer.json** (Optional)
Defines custom UI for DISPLAYING output data from agent actions.

**Example:**
```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/flightDetails"
      }
    }
  }
}
```

**Purpose:**
- Override how data is RENDERED/DISPLAYED to users
- References a Lightning Web Component (LWC)
- Used for OUTPUT of agent actions

### 3. **editor.json** (Optional)
Defines custom UI for COLLECTING input data for agent actions.

**Example:**
```json
{
  "editor": {
    "componentOverrides": {
      "$": {
        "definition": "c/flightRequestFilter"
      }
    }
  }
}
```

**Purpose:**
- Override how users INPUT/EDIT data
- References a Lightning Web Component (LWC)
- Used for INPUT to agent actions

---

## Namespace Prefix Rules

### For Local Org Metadata
Use the **"c"** namespace prefix, even if your org has its own namespace.

**Example:**
- LWC Component: `flightDetails` → Reference as `c/flightDetails`
- Custom Label: `FlightTitle` → Reference as `{!$Label.c.FlightTitle}`
- Apex Class: `AvailableFlight` → Reference as `@apexClassType/c__AvailableFlight`

### For Managed Package Metadata
Use the package's unique namespace as the prefix.

**Example:**
- Package namespace: `isv`
- LWC Component: `flightDetails` → Reference as `isv/flightDetails`

---

## Complete Example: Flight Booking System

### Scenario
Create a custom Lightning type for displaying available flights with booking capability.

### 1. **Apex Classes** (Data Model)

#### Flight.cls
```apex
@JsonAccess(serializable='always' deserializable='always')
global class Flight {
    @AuraEnabled
    global String flightId;
    
    @AuraEnabled
    global Integer numLayovers;
    
    @AuraEnabled
    global Boolean isPetAllowed;
    
    @AuraEnabled
    global Long price;
    
    @AuraEnabled
    global Double discountPercentage;
    
    @AuraEnabled
    global Integer durationInMin;
}
```

#### AvailableFlight.cls
```apex
@JsonAccess(serializable='always' deserializable='always')
global class AvailableFlight {
    @AuraEnabled
    global List<Flight> flights;
}
```

#### FlightRequestFilter.cls (Input)
```apex
@JsonAccess(serializable='always' deserializable='always')
global class FlightRequestFilter {
    @AuraEnabled
    global Long price;
    
    @AuraEnabled
    global Double discountPercentage;
}
```

### 2. **Custom Lightning Type: Flight Response (Output)**

#### Directory Structure
```
lightningTypes/
  flightResponse/
    schema.json
    lightningDesktopGenAi/
      renderer.json
```

#### schema.json
```json
{
  "title": "My Flight Response",
  "description": "My Flight Response",
  "lightning:type": "@apexClassType/c__AvailableFlight"
}
```

#### renderer.json
```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/flightDetails"
      }
    }
  }
}
```

#### LWC: flightDetails.js
```javascript
import { LightningElement, api } from 'lwc';

export default class FlightDetails extends LightningElement {
    @api value
    flightData = [];

    formattedDuration(durationInMin) {
        if (durationInMin) {
            const hours = Math.floor(durationInMin / 60);
            const minutes = durationInMin % 60;
            return `${hours} hr ${minutes} min`
        }
        return;
    }

    arrivalTime(durationInMin) {
        const hours = 7, minutes = 0;
        const departureDate = new Date(2025, 0, 1, hours, minutes);
        const arrivalDate = new Date(departureDate.getTime() + durationInMin * 60000);
        const arrivalHours = String(arrivalDate.getHours()).padStart(2, '0');
        const arrivalMinutes = String(arrivalDate.getMinutes()).padStart(2, '0');
        return `${arrivalHours}:${arrivalMinutes}`;
    }

    connectedCallback() {
        const flights = this.value?.flights || [];
        this.flightData = flights.map((flight) => ({
            ...flight,
            petAllowedStatus: flight.isPetAllowed ? 'Yes' : 'No',
            durationInHr: this.formattedDuration(flight.durationInMin),
            departureTime: '07:00',
            arrivalInHr: this.arrivalTime(flight.durationInMin)
        }));
    }
}
```

**Key Points:**
- `@api value` receives data from the agent action
- `value.flights` contains the list of flights from `AvailableFlight` Apex class
- Helper methods format data for display

### 3. **Custom Lightning Type: Flight Filter (Input)**

#### Directory Structure
```
lightningTypes/
  flightFilter/
    schema.json
    lightningDesktopGenAi/
      editor.json
```

#### schema.json
```json
{
  "title": "Request Filters",
  "description": "Flight Request Filters",
  "lightning:type": "@apexClassType/c__FlightRequestFilter"
}
```

#### editor.json
```json
{
  "editor": {
    "componentOverrides": {
      "$": {
        "definition": "c/flightRequestFilter"
      }
    }
  }
}
```

#### LWC: flightRequestFilter.js
```javascript
import { api, LightningElement } from 'lwc';

export default class FlightRequestFilter extends LightningElement {
    @api
    get readOnly() {
        return this._readOnly;
    }
    set readOnly(value) {
        this._readOnly = value;
    }
    _readOnly = false;
    _value;

    @api
    get value() {
        return this._value;
    }
    set value(value) {
        this._value = value;
    }

    price;
    discountPercentage;

    connectedCallback() {
        if (this.value) {
            this.price = this.value?.price || '';
            this.discountPercentage = this.value?.discountPercentage || '';
        }
    }

    handleInputChange(event) {
        event.stopPropagation();
        const { name, value } = event.target;
        this[name] = value;

        this.dispatchEvent(new CustomEvent('valuechange', {
          detail: {
            value: {
              price: this.price,
              discountPercentage: this.discountPercentage
            }
          }
        }));
    }
}
```

**Key Points:**
- `@api value` - Gets/sets the current filter values
- `@api readOnly` - Controls whether fields are editable
- `valuechange` event - Dispatches updated values back to the system

---

## Deployment

### Using package.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>flightResponse</members>
        <members>flightFilter</members>
        <name>LightningTypeBundle</name>
    </types>
    <version>64.0</version>
</Package>
```

### Using SF Commands

```bash
# Retrieve custom Lightning types
sf project retrieve start -m LightningTypeBundle

# Deploy custom Lightning types
sf project deploy start -m LightningTypeBundle

# Deploy specific type
sf project deploy start -m LightningTypeBundle:flightResponse
```

### Deleting Custom Lightning Types

To delete, use a destructiveChanges package:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>flightResponse</members>
        <name>LightningTypeBundle</name>
    </types>
</Package>
```

---

## Packaging Support

Custom Lightning types can be included in:
- **First-Generation (1GP) packages**
- **Second-Generation (2GP) packages**

---

## Type System Connect REST API

Get a list of all custom and standard Lightning types in your org:

```
GET /connect/lightning-types
```

This REST API endpoint returns metadata about all available Lightning types.

---

## Key Concepts Summary

| Concept | Description |
|---------|-------------|
| **LightningTypeBundle** | Metadata type for custom Lightning types |
| **schema.json** | Links custom type to Apex class (required) |
| **editor.json** | Custom UI for INPUT data |
| **renderer.json** | Custom UI for OUTPUT data |
| **@apexClassType** | Reference format for Apex classes in schema |
| **Namespace "c"** | Local org metadata prefix |
| **@api value** | Property to receive/send data in LWC |
| **valuechange** | Event to dispatch updated input values |

---

## Best Practices

1. **Always use @JsonAccess** on Apex classes for proper serialization
2. **Mark fields with @AuraEnabled** to expose them to LWC
3. **Use namespace "c"** for local org metadata references
4. **Dispatch valuechange events** in editor components to update values
5. **Handle null/undefined** values gracefully in LWC components
6. **Format data** in LWC for better user experience
7. **Test thoroughly** before deploying to production
8. **Use semantic naming** for custom Lightning types

---

## Common Use Cases

1. **Custom Data Display** - Show complex data in user-friendly formats
2. **Interactive Forms** - Create custom input forms for agent actions
3. **Data Filtering** - Allow users to filter/refine agent requests
4. **Rich Visualizations** - Display charts, timelines, or custom layouts
5. **Action Buttons** - Add interactive buttons for booking, approvals, etc.

---

## Resources

- [Official Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/lightning-types-custom.html)
- [Metadata API Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/)
- [Salesforce CLI Command Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/)
- [Type System REST API](https://developer.salesforce.com/docs/atlas.en-us.chatterapi.meta/chatterapi/connect_resources_type_system_resources.htm)

---

## Quick Checklist

- [ ] Create Apex class(es) with @JsonAccess and @AuraEnabled
- [ ] Create lightningTypes folder structure
- [ ] Create schema.json linking to Apex class
- [ ] Create renderer.json (for output) or editor.json (for input)
- [ ] Create corresponding LWC component(s)
- [ ] Add to package.xml
- [ ] Deploy using SF commands or Metadata API
- [ ] Test with Agentforce agents

---

*Created: November 7, 2025*
*API Version: 64.0+*
