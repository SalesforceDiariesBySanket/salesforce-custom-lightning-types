# Account Industry Filter Custom Lightning Type

## Overview
This custom Lightning type provides an interactive industry filter interface for Agentforce agents. It allows users to select multiple industries from a multi-select picklist and specify additional filtering criteria. This is an **INPUT** type that collects filter criteria from users and passes it to your agent action. The filtered results are then displayed using the Account Table Lightning Type.

## 📁 Project Structure

```
force-app/main/default/
├── classes/
│   ├── AccountFilterRequest.cls         # Input request model (filter criteria)
│   ├── AccountFilterRequest.cls-meta.xml
│   ├── AccountFilterResponse.cls        # Output response model (filtered results)
│   ├── AccountFilterResponse.cls-meta.xml
│   ├── AccountIndustryFilterAction.cls  # Agent action that uses filter
│   └── AccountIndustryFilterAction.cls-meta.xml
├── lightningTypes/
│   └── accountIndustryFilter/
│       ├── schema.json                  # Links to AccountFilterRequest Apex class
│       └── lightningDesktopGenAi/
│           └── renderer.json            # References the LWC editor component
└── lwc/
    ├── accountIndustryEditor/
    │   ├── accountIndustryEditor.js     # Filter editor component
    │   ├── accountIndustryEditor.html   # Editor template
    │   ├── accountIndustryEditor.css    # Styling
    │   └── accountIndustryEditor.js-meta.xml
    └── accountIndustryResults/
        ├── accountIndustryResults.js    # Results display component
        ├── accountIndustryResults.html  # Results template
        ├── accountIndustryResults.css   # Styling
        └── accountIndustryResults.js-meta.xml
```

---

## 🎯 What's Included

### 1. **Apex Classes**

#### AccountFilterRequest.cls (INPUT)
Request model that captures user's filter selections.

```apex
@JsonAccess(serializable='always' deserializable='always')
global class AccountFilterRequest {
    @AuraEnabled
    global List<String> selectedIndustries;  // Industry filter (multi-select)
    
    @AuraEnabled
    global Integer maxRecords;                // Maximum records to return
    
    @AuraEnabled
    global String accountType;                // Account type filter
}
```

**Fields:**
- `selectedIndustries`: List of selected industry values
- `maxRecords`: Limit number of results (default: 50)
- `accountType`: Optional account type filter (Customer, Prospect, etc.)

#### AccountFilterResponse.cls (OUTPUT)
Response model containing filtered account results.

```apex
@JsonAccess(serializable='always' deserializable='always')
global class AccountFilterResponse {
    @AuraEnabled
    global List<AccountData> accounts;       // Filtered accounts
    
    @AuraEnabled
    global List<String> appliedFilters;      // Applied industry filters
    
    @AuraEnabled
    global Integer totalRecords;             // Total count
    
    @AuraEnabled
    global String message;                   // Result message
}
```

#### AccountIndustryFilterAction.cls
Agent action that processes the filter and returns results.

**Invocable Method:**
- Accepts `AccountFilterRequest` as input
- Queries accounts matching selected industries
- Returns `AccountFilterResponse` with filtered data

### 2. **Lightning Type Bundle**

#### schema.json
```json
{
  "title": "Account Industry Filter",
  "description": "Custom Lightning Type for filtering accounts by industry",
  "lightning:type": "@apexClassType/c__AccountFilterRequest"
}
```

Links the custom type to the `AccountFilterRequest` Apex class (INPUT type).

#### renderer.json
```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/accountIndustryEditor"
      }
    }
  }
}
```

Specifies that `accountIndustryEditor` LWC should render the input form.

### 3. **Lightning Web Components**

#### accountIndustryEditor (INPUT Component)
Interactive filter form component with:

**Features:**
- ✅ Multi-select industry picklist
- ✅ Dynamic industry options from Schema
- ✅ Account type filter dropdown
- ✅ Max records slider/input
- ✅ Real-time validation
- ✅ Clear filters button
- ✅ Selected filters display
- ✅ SLDS styling for consistency
- ✅ Responsive design

**Industry Options:**
- Agriculture
- Apparel
- Banking
- Biotechnology
- Chemicals
- Communications
- Construction
- Consulting
- Education
- Electronics
- Energy
- Engineering
- Entertainment
- Environmental
- Finance
- Food & Beverage
- Government
- Healthcare
- Hospitality
- Insurance
- Machinery
- Manufacturing
- Media
- Not For Profit
- Other
- Recreation
- Retail
- Shipping
- Technology
- Telecommunications
- Transportation
- Utilities

#### accountIndustryResults (OUTPUT Component)
Displays filtered results in a data table format (similar to accountDataTable).

---

## 🚀 Deployment

### Step 1: Deploy to Salesforce

```bash
# Deploy all metadata
sf project deploy start --manifest manifest/package.xml

# Or deploy specific components
sf project deploy start --source-dir force-app/main/default/classes
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountIndustryFilter
sf project deploy start --source-dir force-app/main/default/lwc/accountIndustryEditor
sf project deploy start --source-dir force-app/main/default/lwc/accountIndustryResults
```

### Step 2: Verify Deployment

```bash
# Check deployed Lightning Types
sf data query --query "SELECT Id, DeveloperName FROM LightningTypeBundle WHERE DeveloperName = 'accountIndustryFilter'" --use-tooling-api
```

---

## 💡 Usage Examples

### Example 1: Agent Action Implementation

The `AccountIndustryFilterAction.cls` demonstrates the complete flow:

```apex
@InvocableMethod(
    label='Filter Accounts by Industry' 
    description='Filters accounts based on selected industries and criteria'
    category='Agent Actions'
)
public static List<AccountFilterResponse> filterAccounts(
    List<AccountFilterRequest> requests
) {
    AccountFilterRequest request = requests[0];
    AccountFilterResponse response = new AccountFilterResponse();
    
    // Build dynamic SOQL query
    String soql = 'SELECT Id, Name, Type, Industry, Phone, Website, ' +
                  'AnnualRevenue, NumberOfEmployees, BillingCity, BillingState, ' +
                  'BillingCountry, Rating, CreatedDate FROM Account WHERE ';
    
    // Add industry filter
    if (request.selectedIndustries != null && !request.selectedIndustries.isEmpty()) {
        soql += 'Industry IN :selectedIndustries';
    } else {
        soql += 'Industry != null';
    }
    
    // Add account type filter
    if (String.isNotBlank(request.accountType)) {
        soql += ' AND Type = :accountType';
    }
    
    // Add limit
    Integer maxRecs = request.maxRecords != null ? request.maxRecords : 50;
    soql += ' LIMIT :maxRecs';
    
    // Execute query
    List<Account> accounts = Database.query(soql);
    
    // Transform to response
    response.accounts = new List<AccountData>();
    for (Account acc : accounts) {
        response.accounts.add(new AccountData(
            acc.Id, acc.Name, acc.Type, acc.Industry, acc.Phone, acc.Website,
            acc.AnnualRevenue, acc.NumberOfEmployees, acc.BillingCity,
            acc.BillingState, acc.BillingCountry, acc.Rating,
            Date.valueOf(acc.CreatedDate)
        ));
    }
    
    response.totalRecords = response.accounts.size();
    response.appliedFilters = request.selectedIndustries;
    response.message = 'Found ' + response.totalRecords + ' accounts matching filters';
    
    return new List<AccountFilterResponse>{ response };
}
```

### Example 2: Use in Agentforce

**Step-by-step Setup:**

1. **Create the Agent Action**
   - Go to Setup > Agentforce > Agent Actions
   - Click "New Agent Action"
   - Select "Apex" as the action type
   - Choose `AccountIndustryFilterAction.filterAccounts`

2. **Configure Input Type**
   - Input Type: `AccountFilterRequest`
   - The custom Lightning type will render the filter UI
   - Users will see the multi-select industry picker

3. **Configure Output Type**
   - Output Type: `AccountFilterResponse`
   - Link to Account Table Lightning Type for display
   - Results show in a beautiful data table

4. **Test the Flow**
   - Agent asks: "Show me Technology companies"
   - Filter UI appears with industries
   - User selects "Technology" and "Software"
   - Agent processes and returns filtered accounts
   - Results display in data table

### Example 3: Programmatic Usage

```apex
// Create filter request
AccountFilterRequest filterReq = new AccountFilterRequest();
filterReq.selectedIndustries = new List<String>{ 'Technology', 'Banking' };
filterReq.accountType = 'Customer';
filterReq.maxRecords = 25;

// Call the action
List<AccountFilterResponse> results = 
    AccountIndustryFilterAction.filterAccounts(
        new List<AccountFilterRequest>{ filterReq }
    );

// Process results
AccountFilterResponse response = results[0];
System.debug('Total Records: ' + response.totalRecords);
System.debug('Message: ' + response.message);
System.debug('Applied Filters: ' + response.appliedFilters);

for (AccountData acc : response.accounts) {
    System.debug('Account: ' + acc.name + ' - ' + acc.industry);
}
```

---

## 🎨 Customization

### Add Custom Filter Fields

Extend `AccountFilterRequest.cls` with additional filters:

```apex
@AuraEnabled
global String minRevenue;

@AuraEnabled
global String rating;

@AuraEnabled
global List<String> selectedStates;
```

Then update the editor component to include new fields:

```html
<!-- Add rating filter -->
<lightning-combobox
    label="Rating"
    value={rating}
    options={ratingOptions}
    onchange={handleRatingChange}
></lightning-combobox>

<!-- Add revenue filter -->
<lightning-input
    type="number"
    label="Minimum Annual Revenue"
    value={minRevenue}
    onchange={handleRevenueChange}
></lightning-input>
```

### Customize Industry Options

Modify the industry picklist options in `accountIndustryEditor.js`:

```javascript
get industryOptions() {
    return [
        { label: 'Technology', value: 'Technology' },
        { label: 'Healthcare', value: 'Healthcare' },
        { label: 'Finance', value: 'Finance' },
        // Add your custom industries
        { label: 'Custom Industry', value: 'Custom Industry' }
    ];
}
```

Or dynamically fetch from Schema:

```javascript
@wire(getPicklistValues, {
    recordTypeId: '012000000000000AAA',
    fieldApiName: INDUSTRY_FIELD
})
wiredIndustries({ data, error }) {
    if (data) {
        this.industryOptions = data.values.map(item => ({
            label: item.label,
            value: item.value
        }));
    }
}
```

### Modify Results Display

Customize how results are displayed in `accountIndustryResults.html`:

```html
<!-- Add filter summary -->
<div class="slds-m-bottom_medium">
    <div class="slds-text-heading_small">Applied Filters:</div>
    <template for:each={appliedFilters} for:item="filter">
        <lightning-badge key={filter} label={filter}></lightning-badge>
    </template>
</div>

<!-- Display results -->
<lightning-datatable
    key-field="id"
    data={accounts}
    columns={columns}
    hide-checkbox-column
></lightning-datatable>
```

### Add Pre-set Filter Options

Create quick filter buttons:

```html
<div class="slds-m-bottom_small">
    <div class="slds-text-title">Quick Filters:</div>
    <lightning-button 
        label="Tech Companies" 
        onclick={handleTechFilter}
    ></lightning-button>
    <lightning-button 
        label="Healthcare Orgs" 
        onclick={handleHealthcareFilter}
    ></lightning-button>
    <lightning-button 
        label="Financial Services" 
        onclick={handleFinanceFilter}
    ></lightning-button>
</div>
```

```javascript
handleTechFilter() {
    this.selectedIndustries = ['Technology', 'Software', 'Electronics'];
    this.updateValue();
}
```

---

## 🔍 Testing

### Test in Developer Console

```apex
// Create test accounts with different industries
List<Account> testAccounts = new List<Account>{
    new Account(Name = 'Tech Corp', Industry = 'Technology', Type = 'Customer'),
    new Account(Name = 'Bank Inc', Industry = 'Banking', Type = 'Customer'),
    new Account(Name = 'Health Co', Industry = 'Healthcare', Type = 'Prospect')
};
insert testAccounts;

// Test the filter action
AccountFilterRequest request = new AccountFilterRequest();
request.selectedIndustries = new List<String>{ 'Technology' };
request.maxRecords = 10;

List<AccountFilterResponse> results = 
    AccountIndustryFilterAction.filterAccounts(
        new List<AccountFilterRequest>{ request }
    );

// Verify results
System.assertEquals(1, results[0].totalRecords);
System.assertEquals('Technology', results[0].accounts[0].industry);
System.debug('Message: ' + results[0].message);
```

### Test in Agentforce

1. Create an Agentforce agent
2. Add the `AccountIndustryFilterAction` as an action
3. Test prompts:
   - "Show me technology companies"
   - "Find all banking customers"
   - "List healthcare prospects"
4. Verify filter UI appears
5. Select industries and submit
6. Verify results display correctly

---

## 📊 Features Breakdown

### Input Features
- **Multi-Select Picklist**: Choose multiple industries at once
- **Account Type Filter**: Filter by Customer, Prospect, Partner, etc.
- **Max Records Control**: Limit result size with slider
- **Clear Filters**: Reset all selections with one click
- **Selected Filters Display**: Visual chips showing active filters
- **Validation**: Ensures valid input before submission

### Output Features
- **Filtered Results Table**: Clean data table display
- **Applied Filters Summary**: Shows which filters were used
- **Record Count**: Total matching records
- **Success Message**: Confirmation of filter application
- **Empty State**: Helpful message when no results found

### User Experience
- **Responsive Design**: Works on desktop and mobile
- **SLDS Styling**: Consistent Salesforce look and feel
- **Loading States**: Spinner during processing
- **Error Handling**: User-friendly error messages
- **Accessibility**: Keyboard navigation and screen reader support

---

## 🛠️ Troubleshooting

### Filter Not Showing

1. Verify Lightning Type deployment:
   ```bash
   sf data query --query "SELECT Id, DeveloperName FROM LightningTypeBundle WHERE DeveloperName = 'accountIndustryFilter'" --use-tooling-api
   ```

2. Check agent action input type is set to `AccountFilterRequest`

3. Verify `renderer.json` points to correct LWC component

### No Results Returned

1. Check SOQL query in `AccountIndustryFilterAction.cls`
2. Verify accounts exist with selected industries
3. Check field-level security on Account fields
4. Review debug logs for query errors

### Picklist Values Not Loading

1. Verify industry field API name is correct
2. Check field permissions
3. Ensure Schema describe is accessible
4. Fall back to hardcoded values if needed

### Styling Issues

1. Use SLDS utility classes consistently
2. Test in both Classic and Lightning Experience
3. Check CSS scoping (component isolation)
4. Verify SLDS version compatibility

---

## 📚 Additional Resources

- [Custom Lightning Types Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/lightning-types-custom.html)
- [Lightning Combobox Documentation](https://developer.salesforce.com/docs/component-library/bundle/lightning-combobox)
- [Dynamic SOQL](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dynamic_soql.htm)
- [Agentforce Agent Actions](https://developer.salesforce.com/docs/einstein/genai/guide/agent-actions.html)

---

## ✅ Checklist

- [x] AccountFilterRequest.cls input model created
- [x] AccountFilterResponse.cls output model created
- [x] schema.json linking to input class
- [x] renderer.json linking to editor LWC
- [x] accountIndustryEditor LWC component (input)
- [x] accountIndustryResults LWC component (output)
- [x] Multi-select industry picklist
- [x] Additional filter fields
- [x] AccountIndustryFilterAction.cls agent action
- [x] Dynamic SOQL query building
- [x] Styling with SLDS
- [x] package.xml manifest
- [x] Documentation

---

## 🎉 You're Ready!

Your custom Lightning type for account industry filtering is complete and ready to deploy. This INPUT type allows users to interactively filter accounts, making your Agentforce agents more dynamic and user-friendly!

**Next Steps:**
1. Deploy to your Salesforce org
2. Create an Agentforce action with input/output types
3. Test with various industry combinations
4. Customize filters for your business needs
5. Combine with Account Table Lightning Type for display!

---

## 💡 Pro Tips

1. **Combine with Other Lightning Types**: Use this filter as input, Account Table for output
2. **Add More Filters**: Extend with revenue, employee count, location filters
3. **Save Filter Presets**: Store commonly used filter combinations
4. **Add Analytics**: Track which industries are searched most
5. **Performance**: Add indexes on Industry field for faster queries

---

*Created: November 8, 2025*  
*API Version: 64.0+*  
*Metadata Type: LightningTypeBundle*  
*Type: INPUT (Editor)*
