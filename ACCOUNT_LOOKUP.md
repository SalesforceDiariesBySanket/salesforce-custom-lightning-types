# Account Lookup Custom Lightning Type

## Overview
This custom Lightning type provides an interactive account lookup interface using Salesforce's `lightning-record-picker` component. It allows users to search and select accounts with autocomplete, then displays detailed account information in a beautiful card format. Perfect for Agentforce agents that need user input to select specific accounts.

## 📁 Project Structure

```
force-app/main/default/
├── classes/
│   ├── AccountLookupRequest.cls         # Input request model (selected account)
│   ├── AccountLookupRequest.cls-meta.xml
│   ├── AccountLookupResponse.cls        # Output response model (account details)
│   ├── AccountLookupResponse.cls-meta.xml
│   ├── AccountLookupAction.cls          # Agent action that uses lookup
│   └── AccountLookupAction.cls-meta.xml
├── lightningTypes/
│   └── accountLookup/
│       ├── schema.json                  # Links to AccountLookupRequest Apex class
│       └── lightningDesktopGenAi/
│           └── renderer.json            # References the LWC picker component
└── lwc/
    ├── accountLookupPicker/
    │   ├── accountLookupPicker.js       # Lookup picker component
    │   ├── accountLookupPicker.html     # Picker template
    │   ├── accountLookupPicker.css      # Styling
    │   └── accountLookupPicker.js-meta.xml
    └── accountLookupCard/
        ├── accountLookupCard.js         # Account detail card component
        ├── accountLookupCard.html       # Card template
        ├── accountLookupCard.css        # Styling
        └── accountLookupCard.js-meta.xml
```

---

## 🎯 What's Included

### 1. **Apex Classes**

#### AccountLookupRequest.cls (INPUT)
Request model that captures the selected account.

```apex
@JsonAccess(serializable='always' deserializable='always')
global class AccountLookupRequest {
    @AuraEnabled
    global String accountId;      // Selected account ID
    
    @AuraEnabled
    global String accountName;    // Selected account name
}
```

**Fields:**
- `accountId`: The 18-character Salesforce Account ID
- `accountName`: The name of the selected account (for display)

#### AccountLookupResponse.cls (OUTPUT)
Response model containing detailed account information.

```apex
@JsonAccess(serializable='always' deserializable='always')
global class AccountLookupResponse {
    @AuraEnabled
    global String accountId;
    
    @AuraEnabled
    global String accountName;
    
    @AuraEnabled
    global String accountType;
    
    @AuraEnabled
    global String industry;
    
    @AuraEnabled
    global String phone;
    
    @AuraEnabled
    global String website;
    
    @AuraEnabled
    global Decimal annualRevenue;
    
    @AuraEnabled
    global Integer numberOfEmployees;
    
    @AuraEnabled
    global String billingStreet;
    
    @AuraEnabled
    global String billingCity;
    
    @AuraEnabled
    global String billingState;
    
    @AuraEnabled
    global String billingPostalCode;
    
    @AuraEnabled
    global String billingCountry;
    
    @AuraEnabled
    global String rating;
    
    @AuraEnabled
    global String ownerName;
    
    @AuraEnabled
    global Date createdDate;
    
    @AuraEnabled
    global Integer totalContacts;
    
    @AuraEnabled
    global Integer totalOpportunities;
    
    @AuraEnabled
    global String message;
}
```

#### AccountLookupAction.cls
Agent action that retrieves account details based on lookup selection.

**Invocable Method:**
- Accepts `AccountLookupRequest` as input
- Queries full account details including related counts
- Returns `AccountLookupResponse` with complete information

### 2. **Lightning Type Bundle**

#### schema.json
```json
{
  "title": "Account Lookup",
  "description": "Account lookup using lightning-record-picker with detailed card display",
  "lightning:type": "@apexClassType/c__AccountLookupRequest"
}
```

Links the custom type to the `AccountLookupRequest` Apex class (INPUT type).

#### renderer.json
```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/accountLookupPicker"
      }
    }
  }
}
```

Specifies that `accountLookupPicker` LWC should render the input interface.

### 3. **Lightning Web Components**

#### accountLookupPicker (INPUT Component)
Interactive account lookup component with:

**Features:**
- ✅ Lightning Record Picker with autocomplete
- ✅ Type-ahead search across account names
- ✅ Recent accounts displayed by default
- ✅ Matching info display (shows search criteria)
- ✅ Clear selection button
- ✅ Selected account badge
- ✅ Real-time validation
- ✅ Placeholder text and help text
- ✅ SLDS styling
- ✅ Responsive design

**Search Capabilities:**
- Search by account name
- Filter by record type (optional)
- Show recently viewed accounts
- Display matching criteria

#### accountLookupCard (OUTPUT Component)
Displays selected account details in a rich card format:

**Features:**
- ✅ Account header with name and rating badge
- ✅ Type and industry pills
- ✅ Contact information (phone, website, email)
- ✅ Company information (revenue, employees)
- ✅ Complete billing address
- ✅ Owner information
- ✅ Related record counts (contacts, opportunities)
- ✅ Quick actions (view record, edit)
- ✅ Created date
- ✅ Clickable links (website, record)
- ✅ Formatted currency and phone numbers
- ✅ Color-coded rating indicators
- ✅ Empty state handling

---

## 🚀 Deployment

### Step 1: Deploy to Salesforce

```bash
# Deploy all metadata
sf project deploy start --manifest manifest/package.xml

# Or deploy specific components
sf project deploy start --source-dir force-app/main/default/classes
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountLookup
sf project deploy start --source-dir force-app/main/default/lwc/accountLookupPicker
sf project deploy start --source-dir force-app/main/default/lwc/accountLookupCard
```

### Step 2: Verify Deployment

```bash
# Check deployed Lightning Types
sf data query --query "SELECT Id, DeveloperName FROM LightningTypeBundle WHERE DeveloperName = 'accountLookup'" --use-tooling-api
```

---

## 💡 Usage Examples

### Example 1: Agent Action Implementation

The `AccountLookupAction.cls` demonstrates the complete flow:

```apex
@InvocableMethod(
    label='Get Account Details' 
    description='Retrieves detailed account information based on lookup selection'
    category='Agent Actions'
)
public static List<AccountLookupResponse> getAccountDetails(
    List<AccountLookupRequest> requests
) {
    AccountLookupRequest request = requests[0];
    AccountLookupResponse response = new AccountLookupResponse();
    
    if (String.isBlank(request.accountId)) {
        response.message = 'Please select an account';
        return new List<AccountLookupResponse>{ response };
    }
    
    // Query account with all details
    List<Account> accounts = [
        SELECT Id, Name, Type, Industry, Phone, Website,
               AnnualRevenue, NumberOfEmployees,
               BillingStreet, BillingCity, BillingState, 
               BillingPostalCode, BillingCountry,
               Rating, Owner.Name, CreatedDate,
               (SELECT Id FROM Contacts),
               (SELECT Id FROM Opportunities)
        FROM Account
        WHERE Id = :request.accountId
        LIMIT 1
    ];
    
    if (accounts.isEmpty()) {
        response.message = 'Account not found';
        return new List<AccountLookupResponse>{ response };
    }
    
    Account acc = accounts[0];
    
    // Populate response
    response.accountId = acc.Id;
    response.accountName = acc.Name;
    response.accountType = acc.Type;
    response.industry = acc.Industry;
    response.phone = acc.Phone;
    response.website = acc.Website;
    response.annualRevenue = acc.AnnualRevenue;
    response.numberOfEmployees = acc.NumberOfEmployees;
    response.billingStreet = acc.BillingStreet;
    response.billingCity = acc.BillingCity;
    response.billingState = acc.BillingState;
    response.billingPostalCode = acc.BillingPostalCode;
    response.billingCountry = acc.BillingCountry;
    response.rating = acc.Rating;
    response.ownerName = acc.Owner.Name;
    response.createdDate = Date.valueOf(acc.CreatedDate);
    response.totalContacts = acc.Contacts.size();
    response.totalOpportunities = acc.Opportunities.size();
    response.message = 'Account details retrieved successfully';
    
    return new List<AccountLookupResponse>{ response };
}
```

### Example 2: Use in Agentforce

**Step-by-step Setup:**

1. **Create the Agent Action**
   - Go to Setup > Agentforce > Agent Actions
   - Click "New Agent Action"
   - Select "Apex" as the action type
   - Choose `AccountLookupAction.getAccountDetails`

2. **Configure Input Type**
   - Input Type: `AccountLookupRequest`
   - The custom Lightning type will render the lookup UI
   - Users will see the record picker with search

3. **Configure Output Type**
   - Output Type: `AccountLookupResponse`
   - Use the account card component to display results
   - Shows rich account details

4. **Test the Flow**
   - Agent asks: "Tell me about an account"
   - Lookup picker appears
   - User searches and selects "Acme Corporation"
   - Agent retrieves and displays full account details
   - Card shows all information beautifully formatted

### Example 3: Programmatic Usage

```apex
// Create lookup request
AccountLookupRequest lookupReq = new AccountLookupRequest();
lookupReq.accountId = '001xx000003DGb2AAG';
lookupReq.accountName = 'Acme Corporation';

// Call the action
List<AccountLookupResponse> results = 
    AccountLookupAction.getAccountDetails(
        new List<AccountLookupRequest>{ lookupReq }
    );

// Process results
AccountLookupResponse response = results[0];
System.debug('Account: ' + response.accountName);
System.debug('Industry: ' + response.industry);
System.debug('Revenue: ' + response.annualRevenue);
System.debug('Total Contacts: ' + response.totalContacts);
System.debug('Message: ' + response.message);
```

### Example 4: Chained Actions

Use account lookup as input for other actions:

```apex
// Step 1: User selects account via lookup
AccountLookupRequest lookupReq = new AccountLookupRequest();
lookupReq.accountId = '001xx000003DGb2AAG';

// Step 2: Get account details
List<AccountLookupResponse> accountDetails = 
    AccountLookupAction.getAccountDetails(
        new List<AccountLookupRequest>{ lookupReq }
    );

// Step 3: Use account info for related actions
// - Get contacts for this account
// - Get opportunities for this account
// - Create case for this account
// - etc.
```

---

## 🎨 Customization

### Add Additional Search Filters

Modify `accountLookupPicker.html` to add filters:

```html
<!-- Add record type filter -->
<lightning-combobox
    label="Account Type Filter"
    value={accountTypeFilter}
    options={accountTypeOptions}
    onchange={handleTypeFilterChange}
    class="slds-m-bottom_small"
></lightning-combobox>

<lightning-record-picker
    label="Search Accounts"
    object-api-name="Account"
    value={selectedAccountId}
    onchange={handleAccountChange}
    filter={accountFilter}
></lightning-record-picker>
```

```javascript
get accountFilter() {
    if (this.accountTypeFilter) {
        return {
            criteria: [
                {
                    fieldPath: 'Type',
                    operator: 'eq',
                    value: this.accountTypeFilter
                }
            ]
        };
    }
    return null;
}
```

### Customize Card Display

Add or remove fields in `accountLookupCard.html`:

```html
<!-- Add custom fields -->
<div class="slds-col slds-size_1-of-2">
    <div class="slds-text-body_small">
        <strong>SLA:</strong> {accountData.sla}
    </div>
</div>

<div class="slds-col slds-size_1-of-2">
    <div class="slds-text-body_small">
        <strong>Account Source:</strong> {accountData.accountSource}
    </div>
</div>
```

### Add Quick Actions

Include action buttons in the card:

```html
<div class="slds-card__footer">
    <div class="slds-button-group" role="group">
        <lightning-button 
            label="Create Contact" 
            onclick={handleCreateContact}
            icon-name="utility:new"
        ></lightning-button>
        <lightning-button 
            label="Create Opportunity" 
            onclick={handleCreateOpportunity}
            icon-name="utility:opportunity"
        ></lightning-button>
        <lightning-button 
            label="Create Case" 
            onclick={handleCreateCase}
            icon-name="utility:case"
        ></lightning-button>
    </div>
</div>
```

```javascript
handleCreateContact() {
    this[NavigationMixin.Navigate]({
        type: 'standard__objectPage',
        attributes: {
            objectApiName: 'Contact',
            actionName: 'new'
        },
        state: {
            defaultFieldValues: `AccountId=${this.accountId}`
        }
    });
}
```

### Add Related Lists

Display related records inline:

```html
<!-- Add after main card content -->
<div class="slds-m-top_medium">
    <lightning-card title="Recent Opportunities">
        <lightning-datatable
            key-field="id"
            data={opportunities}
            columns={opportunityColumns}
            hide-checkbox-column
        ></lightning-datatable>
    </lightning-card>
</div>

<div class="slds-m-top_medium">
    <lightning-card title="Recent Contacts">
        <lightning-datatable
            key-field="id"
            data={contacts}
            columns={contactColumns}
            hide-checkbox-column
        ></lightning-datatable>
    </lightning-card>
</div>
```

### Customize Matching Info

Configure what information shows in the picker:

```html
<lightning-record-picker
    label="Search Accounts"
    placeholder="Type to search accounts..."
    object-api-name="Account"
    value={selectedAccountId}
    onchange={handleAccountChange}
    display-info="{!v.displayInfo}"
    matching-info={matchingInfo}
></lightning-record-picker>
```

```javascript
get matchingInfo() {
    return {
        primaryField: { fieldPath: 'Name' },
        additionalFields: [
            { fieldPath: 'Type' },
            { fieldPath: 'Industry' },
            { fieldPath: 'BillingCity' }
        ]
    };
}
```

---

## 🔍 Testing

### Test in Developer Console

```apex
// Create test account
Account testAcc = new Account(
    Name = 'Test Account Inc',
    Type = 'Customer',
    Industry = 'Technology',
    Phone = '555-0100',
    Website = 'www.test.com',
    AnnualRevenue = 5000000,
    NumberOfEmployees = 100,
    BillingCity = 'San Francisco',
    BillingState = 'CA',
    BillingCountry = 'USA',
    Rating = 'Hot'
);
insert testAcc;

// Create related contacts
List<Contact> testContacts = new List<Contact>{
    new Contact(AccountId = testAcc.Id, FirstName = 'John', LastName = 'Doe'),
    new Contact(AccountId = testAcc.Id, FirstName = 'Jane', LastName = 'Smith')
};
insert testContacts;

// Test the lookup action
AccountLookupRequest request = new AccountLookupRequest();
request.accountId = testAcc.Id;
request.accountName = testAcc.Name;

List<AccountLookupResponse> results = 
    AccountLookupAction.getAccountDetails(
        new List<AccountLookupRequest>{ request }
    );

// Verify results
AccountLookupResponse response = results[0];
System.assertEquals('Test Account Inc', response.accountName);
System.assertEquals('Technology', response.industry);
System.assertEquals(2, response.totalContacts);
System.debug('Full Response: ' + JSON.serializePretty(response));
```

### Test in Agentforce

1. Create an Agentforce agent
2. Add the `AccountLookupAction` as an action
3. Test prompts:
   - "Show me details for an account"
   - "Look up account information"
   - "Tell me about [Account Name]"
4. Verify lookup picker appears
5. Search and select an account
6. Verify detailed card displays correctly

---

## 📊 Features Breakdown

### Input Features (Picker)
- **Type-Ahead Search**: Real-time search as you type
- **Recent Accounts**: Shows recently accessed accounts
- **Matching Info**: Displays relevant account fields
- **Clear Selection**: Easy reset button
- **Validation**: Ensures account is selected
- **Placeholder Text**: Helpful search hints
- **Mobile Responsive**: Works on all devices

### Output Features (Card)
- **Rich Detail Display**: Comprehensive account information
- **Formatted Data**: Currency, phone, dates properly formatted
- **Clickable Links**: Website and record navigation
- **Rating Indicators**: Color-coded rating badges
- **Related Counts**: Shows number of contacts and opportunities
- **Owner Information**: Account owner name
- **Full Address**: Complete billing address display
- **Quick Actions**: View and edit buttons

### User Experience
- **Intuitive Search**: Easy to find and select accounts
- **Visual Feedback**: Selected account clearly indicated
- **Loading States**: Spinner during data retrieval
- **Error Handling**: User-friendly error messages
- **Accessibility**: Keyboard navigation and screen reader support
- **SLDS Styling**: Consistent Salesforce design

---

## 🛠️ Troubleshooting

### Record Picker Not Showing Results

1. Check user permissions on Account object
2. Verify sharing rules allow account visibility
3. Check filter criteria (may be too restrictive)
4. Review browser console for JavaScript errors

### Account Details Not Loading

1. Verify account ID is valid 18-character format
2. Check field-level security on Account fields
3. Ensure user has read access to related objects (Contact, Opportunity)
4. Review Apex debug logs for SOQL errors

### Card Not Displaying

1. Check that `accountLookupCard` component is properly deployed
2. Verify value binding is correct
3. Ensure all required fields are populated
4. Check browser console for rendering errors

### Styling Issues

1. Use SLDS utility classes for consistency
2. Test in both Classic and Lightning Experience
3. Check CSS scoping (component isolation)
4. Verify component API version compatibility

---

## 📚 Additional Resources

- [Custom Lightning Types Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/lightning-types-custom.html)
- [Lightning Record Picker Documentation](https://developer.salesforce.com/docs/component-library/bundle/lightning-record-picker)
- [Lightning Card Documentation](https://developer.salesforce.com/docs/component-library/bundle/lightning-card)
- [Agentforce Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/get-started-agents.html)

---

## ✅ Checklist

- [x] AccountLookupRequest.cls input model created
- [x] AccountLookupResponse.cls output model created
- [x] schema.json linking to input class
- [x] renderer.json linking to picker LWC
- [x] accountLookupPicker LWC component (input)
- [x] accountLookupCard LWC component (output)
- [x] Lightning Record Picker integration
- [x] Account detail card with formatting
- [x] AccountLookupAction.cls agent action
- [x] Related record counts (contacts, opportunities)
- [x] Styling with SLDS
- [x] Quick actions (view, edit)
- [x] package.xml manifest
- [x] Documentation

---

## 🎉 You're Ready!

Your custom Lightning type for account lookup is complete and ready to deploy. This INPUT type provides an intuitive search interface and displays rich account information, perfect for Agentforce agents that need user-selected account context!

**Next Steps:**
1. Deploy to your Salesforce org
2. Create an Agentforce action with input/output types
3. Test account search and selection
4. Customize card display for your needs
5. Add quick actions for common tasks
6. Chain with other actions for workflows!

---

## 💡 Pro Tips

1. **Use with Workflows**: Chain lookup → get details → perform action
2. **Add Filters**: Filter by type, industry, or custom criteria
3. **Show Related Data**: Display contacts, opportunities inline
4. **Add Quick Actions**: Create contact, opportunity from card
5. **Performance**: Use selective queries to improve speed
6. **Mobile Friendly**: Test on Salesforce Mobile App

---

*Created: November 8, 2025*  
*API Version: 64.0+*  
*Metadata Type: LightningTypeBundle*  
*Type: INPUT (Editor) with OUTPUT (Display)*
