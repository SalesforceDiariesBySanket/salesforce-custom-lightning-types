# Account Table Custom Lightning Type

## Overview
This custom Lightning type displays a list of Salesforce accounts in a beautiful, interactive Lightning Data Table. Perfect for Agentforce agent actions that return account information.

## 📁 Project Structure

```
force-app/main/default/
├── classes/
│   ├── AccountList.cls                  # Wrapper class for account list
│   ├── AccountList.cls-meta.xml
│   ├── AccountData.cls                  # Individual account data model
│   ├── AccountData.cls-meta.xml
│   ├── AccountAgentAction.cls           # Example agent action (optional)
│   └── AccountAgentAction.cls-meta.xml
├── lightningTypes/
│   └── accountTable/
│       ├── schema.json                  # Links to AccountList Apex class
│       └── lightningDesktopGenAi/
│           └── renderer.json            # References the LWC component
└── lwc/
    └── accountDataTable/
        ├── accountDataTable.js          # Component logic
        ├── accountDataTable.html        # Component template
        ├── accountDataTable.css         # Styling
        └── accountDataTable.js-meta.xml # Component metadata
```

---

## 🎯 What's Included

### 1. **Apex Classes**

#### AccountList.cls
Wrapper class that holds the list of accounts and metadata.

```apex
@JsonAccess(serializable='always' deserializable='always')
global class AccountList {
    @AuraEnabled
    global List<AccountData> accounts;      // List of account data
    
    @AuraEnabled
    global Integer totalRecords;             // Total count
    
    @AuraEnabled
    global String message;                   // Display message
}
```

#### AccountData.cls
Represents individual account information with all relevant fields.

**Fields included:**
- Account ID, Name, Type
- Industry, Phone, Website
- Annual Revenue, Number of Employees
- Billing Address (City, State, Country)
- Rating, Created Date

### 2. **Lightning Type Bundle**

#### schema.json
```json
{
  "title": "Account List Table",
  "description": "Custom Lightning Type for displaying accounts in a data table",
  "lightning:type": "@apexClassType/c__AccountList"
}
```

Links the custom type to the `AccountList` Apex class.

#### renderer.json
```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/accountDataTable"
      }
    }
  }
}
```

Specifies that `accountDataTable` LWC should render the output.

### 3. **Lightning Web Component**

#### accountDataTable
A sophisticated data table component with:

**Features:**
- ✅ Sortable columns
- ✅ Row numbers
- ✅ Clickable account names (links to record)
- ✅ Formatted currency, phone, and dates
- ✅ Color-coded ratings (Hot=Red, Warm=Orange, Cold=Gray)
- ✅ Location concatenation
- ✅ Empty state handling
- ✅ Record count badge
- ✅ Responsive design

**Columns:**
1. Account Name (clickable link)
2. Type
3. Industry
4. Phone
5. Website (clickable link)
6. Annual Revenue (currency)
7. Employees (number)
8. Rating (color-coded)
9. Location (City, State, Country)
10. Created Date

---

## 🚀 Deployment

### Step 1: Deploy to Salesforce

```bash
# Deploy all metadata
sf project deploy start --manifest manifest/package.xml

# Or deploy specific components
sf project deploy start --source-dir force-app/main/default/classes
sf project deploy start --source-dir force-app/main/default/lightningTypes
sf project deploy start --source-dir force-app/main/default/lwc
```

### Step 2: Verify Deployment

```bash
# Check deployed Lightning Types
sf data query --query "SELECT Id, DeveloperName FROM LightningTypeBundle WHERE DeveloperName = 'accountTable'" --use-tooling-api
```

Or use the REST API:
```
GET /services/data/v64.0/connect/lightning-types
```

---

## 💡 Usage Examples

### Example 1: Simple Agent Action

Create an agent action that returns `AccountList`:

```apex
@InvocableMethod(label='Get All Accounts' category='Agent Actions')
public static List<AccountList> getAllAccounts() {
    AccountList result = new AccountList();
    result.accounts = new List<AccountData>();
    
    // Query accounts
    List<Account> accts = [
        SELECT Id, Name, Type, Industry, Phone, Website,
               AnnualRevenue, NumberOfEmployees, 
               BillingCity, BillingState, BillingCountry,
               Rating, CreatedDate
        FROM Account
        LIMIT 50
    ];
    
    // Transform to AccountData
    for (Account acc : accts) {
        result.accounts.add(new AccountData(
            acc.Id,
            acc.Name,
            acc.Type,
            acc.Industry,
            acc.Phone,
            acc.Website,
            acc.AnnualRevenue,
            acc.NumberOfEmployees,
            acc.BillingCity,
            acc.BillingState,
            acc.BillingCountry,
            acc.Rating,
            Date.valueOf(acc.CreatedDate)
        ));
    }
    
    result.totalRecords = result.accounts.size();
    result.message = 'Found ' + result.totalRecords + ' accounts';
    
    return new List<AccountList>{ result };
}
```

### Example 2: Filtered Agent Action

Use the included `AccountAgentAction.cls`:

```apex
// This invocable method accepts filters:
// - Account Type (Customer, Prospect, etc.)
// - Industry
// - Minimum Revenue
// - Max Records

AccountAgentAction.AccountRequest request = new AccountAgentAction.AccountRequest();
request.accountType = 'Customer';
request.industry = 'Technology';
request.minRevenue = 1000000;
request.maxRecords = 25;

List<AccountList> results = AccountAgentAction.getAccounts(
    new List<AccountAgentAction.AccountRequest>{ request }
);
```

### Example 3: Use in Agentforce

1. **Create an Agentforce Action** pointing to your invocable method
2. **Set the Output Type** to `AccountList`
3. **The custom Lightning type will automatically render** the account table
4. **Agent will display** the beautiful data table instead of raw JSON

---

## 🎨 Customization

### Modify Columns

Edit `accountDataTable.js` to add/remove columns:

```javascript
columns = [
    { label: 'Account Name', fieldName: 'accountId', type: 'url', ... },
    // Add your custom column:
    { label: 'Owner', fieldName: 'ownerName', type: 'text' },
    // ... rest of columns
];
```

Don't forget to:
1. Add field to `AccountData.cls`
2. Update `connectedCallback()` to map the data
3. Query the field in your Apex action

### Modify Styling

Edit `accountDataTable.css`:

```css
/* Change rating colors */
.slds-text-color_error {
    color: #ff0000;  /* Change Hot rating color */
}

/* Add custom table styles */
.custom-account-table {
    border: 2px solid #0070d2;  /* Custom border */
}
```

### Add Actions

Add action buttons to each row:

```javascript
columns = [
    // ... existing columns
    {
        type: 'action',
        typeAttributes: { 
            rowActions: [
                { label: 'View', name: 'view' },
                { label: 'Edit', name: 'edit' },
                { label: 'Delete', name: 'delete' }
            ]
        }
    }
];

// Handle actions
handleRowAction(event) {
    const actionName = event.detail.action.name;
    const row = event.detail.row;
    
    switch (actionName) {
        case 'view':
            // Navigate to record
            break;
        case 'edit':
            // Open edit modal
            break;
        case 'delete':
            // Delete record
            break;
    }
}
```

---

## 🔍 Testing

### Test in Developer Console

```apex
// Create test data
Account testAcc = new Account(
    Name = 'Test Account',
    Type = 'Customer',
    Industry = 'Technology',
    AnnualRevenue = 5000000,
    NumberOfEmployees = 150,
    Rating = 'Hot'
);
insert testAcc;

// Test the agent action
AccountAgentAction.AccountRequest request = new AccountAgentAction.AccountRequest();
request.maxRecords = 10;

List<AccountList> results = AccountAgentAction.getAccounts(
    new List<AccountAgentAction.AccountRequest>{ request }
);

System.debug('Total Accounts: ' + results[0].totalRecords);
System.debug('Message: ' + results[0].message);
```

### Test in Agentforce

1. Create an Agentforce agent
2. Add action using `AccountAgentAction.getAccounts`
3. Configure action inputs (optional filters)
4. Test in Agent Builder
5. Verify the custom table displays correctly

---

## 📊 Features Breakdown

### Data Formatting
- **Currency**: Annual Revenue displayed with currency symbol
- **Phone**: Phone numbers formatted as clickable links
- **URLs**: Website and Account Name as clickable links
- **Dates**: Created Date formatted as "MMM DD, YYYY"
- **Location**: City, State, Country concatenated intelligently

### Visual Indicators
- **Rating Colors**:
  - 🔴 Hot = Red (bold)
  - 🟠 Warm = Orange (bold)
  - ⚫ Cold = Gray
- **Record Count Badge**: Shows total at-a-glance
- **Info Banner**: Displays custom messages
- **Empty State**: User-friendly message when no data

### Interactions
- **Sortable Columns**: Click headers to sort
- **Row Numbers**: Auto-generated for easy reference
- **Clickable Names**: Navigate to account details
- **Responsive**: Adapts to screen size

---

## 🛠️ Troubleshooting

### Custom Type Not Appearing

1. Verify deployment:
   ```bash
   sf project deploy report --job-id <YOUR_JOB_ID>
   ```

2. Check the LightningTypeBundle exists:
   ```bash
   sf data query --query "SELECT Id, DeveloperName FROM LightningTypeBundle" --use-tooling-api
   ```

3. Ensure API version is 64.0+

### Data Not Displaying

1. Check `@AuraEnabled` on all Apex class properties
2. Verify `@JsonAccess` annotation on classes
3. Check browser console for JavaScript errors
4. Ensure `value` property is populated in LWC

### Styling Issues

1. Use SLDS utility classes
2. Test in different browsers
3. Check for CSS conflicts
4. Verify `lightning-datatable` is rendering

---

## 📚 Additional Resources

- [Custom Lightning Types Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/lightning-types-custom.html)
- [Lightning Data Table Documentation](https://developer.salesforce.com/docs/component-library/bundle/lightning-datatable)
- [Agentforce Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/get-started-agents.html)
- [LightningTypeBundle Metadata API](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/)

---

## ✅ Checklist

- [x] AccountList.cls wrapper class created
- [x] AccountData.cls data model created
- [x] schema.json linking to Apex class
- [x] renderer.json linking to LWC
- [x] accountDataTable LWC component
- [x] Styling with SLDS
- [x] package.xml manifest
- [x] Example agent action (AccountAgentAction.cls)
- [x] Documentation

---

## 🎉 You're Ready!

Your custom Lightning type for account tables is complete and ready to deploy. Use it in your Agentforce actions to create stunning, user-friendly account displays!

**Next Steps:**
1. Deploy to your Salesforce org
2. Create an Agentforce action
3. Test with sample data
4. Customize to your needs
5. Share with your team!

---

*Created: November 7, 2025*  
*API Version: 64.0+*  
*Metadata Type: LightningTypeBundle*
