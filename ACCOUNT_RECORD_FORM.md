# Account Record Form Custom Lightning Type

## Overview
This custom Lightning type provides a comprehensive account record form for creating and editing Salesforce accounts. Built with `lightning-record-edit-form`, it offers a fully functional form with all account fields, validation, and a beautiful user interface. Perfect for Agentforce agents that need to collect account information or update existing records.

## 📁 Project Structure

```
force-app/main/default/
├── classes/
│   ├── AccountFormRequest.cls           # Input/Output model for form data
│   ├── AccountFormRequest.cls-meta.xml
│   ├── AccountFormAction.cls            # Agent action for form submission
│   └── AccountFormAction.cls-meta.xml
├── lightningTypes/
│   └── accountRecordForm/
│       ├── schema.json                  # Links to AccountFormRequest Apex class
│       └── lightningDesktopGenAi/
│           └── renderer.json            # References the LWC form component
└── lwc/
    └── accountRecordForm/
        ├── accountRecordForm.js         # Form component logic
        ├── accountRecordForm.html       # Form template
        ├── accountRecordForm.css        # Styling
        └── accountRecordForm.js-meta.xml # Component metadata
```

---

## 🎯 What's Included

### 1. **Apex Classes**

#### AccountFormRequest.cls (INPUT/OUTPUT)
Comprehensive model for account form data (both input and output).

```apex
@JsonAccess(serializable='always' deserializable='always')
global class AccountFormRequest {
    @AuraEnabled
    global String accountId;              // For editing existing accounts
    
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
    
    // Billing Address
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
    
    // Shipping Address
    @AuraEnabled
    global String shippingStreet;
    
    @AuraEnabled
    global String shippingCity;
    
    @AuraEnabled
    global String shippingState;
    
    @AuraEnabled
    global String shippingPostalCode;
    
    @AuraEnabled
    global String shippingCountry;
    
    @AuraEnabled
    global String description;
    
    @AuraEnabled
    global String rating;
    
    @AuraEnabled
    global String accountSource;
    
    @AuraEnabled
    global String sla;
}
```

#### AccountFormAction.cls
Agent action that creates or updates accounts based on form submission.

**Invocable Method:**
- Accepts `AccountFormRequest` as input
- Creates new account or updates existing account
- Returns success/failure response
- Handles all field validations

### 2. **Lightning Type Bundle**

#### schema.json
```json
{
  "title": "Account Record Form",
  "description": "Custom Lightning Type for creating/editing Account records",
  "lightning:type": "@apexClassType/c__AccountFormRequest"
}
```

Links the custom type to the `AccountFormRequest` Apex class.

#### renderer.json
```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/accountRecordForm"
      }
    }
  }
}
```

Specifies that `accountRecordForm` LWC should render the form interface.

### 3. **Lightning Web Component**

#### accountRecordForm
A comprehensive record form component with:

**Features:**
- ✅ Create new accounts
- ✅ Edit existing accounts
- ✅ All standard account fields
- ✅ Organized field sections (Company Info, Contact, Address)
- ✅ Billing and Shipping address forms
- ✅ "Copy Billing to Shipping" button
- ✅ Field-level validation
- ✅ Required field indicators
- ✅ Help text for fields
- ✅ Dependent picklists support
- ✅ Save and Cancel buttons
- ✅ Success/Error toast messages
- ✅ Loading spinner
- ✅ SLDS styling
- ✅ Responsive 2-column layout
- ✅ Accessibility compliant

**Form Sections:**

1. **Company Information**
   - Account Name (required)
   - Account Type
   - Industry
   - Rating
   - Account Source
   - SLA

2. **Contact Information**
   - Phone
   - Website
   - Annual Revenue
   - Number of Employees

3. **Billing Address**
   - Street
   - City
   - State/Province
   - Postal Code
   - Country

4. **Shipping Address**
   - Street
   - City
   - State/Province
   - Postal Code
   - Country
   - Copy from Billing button

5. **Additional Information**
   - Description (long text area)

---

## 🚀 Deployment

### Step 1: Deploy to Salesforce

```bash
# Deploy all metadata
sf project deploy start --manifest manifest/package.xml

# Or deploy specific components
sf project deploy start --source-dir force-app/main/default/classes
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountRecordForm
sf project deploy start --source-dir force-app/main/default/lwc/accountRecordForm
```

### Step 2: Verify Deployment

```bash
# Check deployed Lightning Types
sf data query --query "SELECT Id, DeveloperName FROM LightningTypeBundle WHERE DeveloperName = 'accountRecordForm'" --use-tooling-api
```

---

## 💡 Usage Examples

### Example 1: Create Account Action

```apex
@InvocableMethod(
    label='Create Account' 
    description='Creates a new account with provided information'
    category='Agent Actions'
)
public static List<AccountFormResponse> createAccount(
    List<AccountFormRequest> requests
) {
    AccountFormRequest request = requests[0];
    AccountFormResponse response = new AccountFormResponse();
    
    try {
        // Create new account
        Account newAccount = new Account();
        newAccount.Name = request.accountName;
        newAccount.Type = request.accountType;
        newAccount.Industry = request.industry;
        newAccount.Phone = request.phone;
        newAccount.Website = request.website;
        newAccount.AnnualRevenue = request.annualRevenue;
        newAccount.NumberOfEmployees = request.numberOfEmployees;
        newAccount.BillingStreet = request.billingStreet;
        newAccount.BillingCity = request.billingCity;
        newAccount.BillingState = request.billingState;
        newAccount.BillingPostalCode = request.billingPostalCode;
        newAccount.BillingCountry = request.billingCountry;
        newAccount.ShippingStreet = request.shippingStreet;
        newAccount.ShippingCity = request.shippingCity;
        newAccount.ShippingState = request.shippingState;
        newAccount.ShippingPostalCode = request.shippingPostalCode;
        newAccount.ShippingCountry = request.shippingCountry;
        newAccount.Description = request.description;
        newAccount.Rating = request.rating;
        newAccount.AccountSource = request.accountSource;
        newAccount.SLA__c = request.sla;
        
        insert newAccount;
        
        response.success = true;
        response.accountId = newAccount.Id;
        response.message = 'Account created successfully: ' + newAccount.Name;
        
    } catch (Exception e) {
        response.success = false;
        response.message = 'Error creating account: ' + e.getMessage();
    }
    
    return new List<AccountFormResponse>{ response };
}
```

### Example 2: Update Account Action

```apex
@InvocableMethod(
    label='Update Account' 
    description='Updates an existing account'
    category='Agent Actions'
)
public static List<AccountFormResponse> updateAccount(
    List<AccountFormRequest> requests
) {
    AccountFormRequest request = requests[0];
    AccountFormResponse response = new AccountFormResponse();
    
    if (String.isBlank(request.accountId)) {
        response.success = false;
        response.message = 'Account ID is required for update';
        return new List<AccountFormResponse>{ response };
    }
    
    try {
        // Query existing account
        Account existingAccount = [
            SELECT Id FROM Account WHERE Id = :request.accountId LIMIT 1
        ];
        
        // Update fields
        existingAccount.Name = request.accountName;
        existingAccount.Type = request.accountType;
        existingAccount.Industry = request.industry;
        existingAccount.Phone = request.phone;
        existingAccount.Website = request.website;
        existingAccount.AnnualRevenue = request.annualRevenue;
        existingAccount.NumberOfEmployees = request.numberOfEmployees;
        existingAccount.BillingStreet = request.billingStreet;
        existingAccount.BillingCity = request.billingCity;
        existingAccount.BillingState = request.billingState;
        existingAccount.BillingPostalCode = request.billingPostalCode;
        existingAccount.BillingCountry = request.billingCountry;
        existingAccount.ShippingStreet = request.shippingStreet;
        existingAccount.ShippingCity = request.shippingCity;
        existingAccount.ShippingState = request.shippingState;
        existingAccount.ShippingPostalCode = request.shippingPostalCode;
        existingAccount.ShippingCountry = request.shippingCountry;
        existingAccount.Description = request.description;
        existingAccount.Rating = request.rating;
        existingAccount.AccountSource = request.accountSource;
        
        update existingAccount;
        
        response.success = true;
        response.accountId = existingAccount.Id;
        response.message = 'Account updated successfully';
        
    } catch (Exception e) {
        response.success = false;
        response.message = 'Error updating account: ' + e.getMessage();
    }
    
    return new List<AccountFormResponse>{ response };
}
```

### Example 3: Use in Agentforce

**Create Account Workflow:**

1. **Agent Prompt**: "Create a new customer account"
2. **Form Displays**: Lightning form with all account fields
3. **User Fills Form**: Enters company information
4. **Form Submits**: Agent action processes the data
5. **Success**: Account created, confirmation shown

**Edit Account Workflow:**

1. **Agent Prompt**: "Update account information"
2. **Account Selected**: Via lookup or ID
3. **Form Loads**: Pre-populated with existing data
4. **User Edits**: Modifies fields as needed
5. **Form Submits**: Updates saved to Salesforce

### Example 4: Programmatic Usage

```apex
// Create account request
AccountFormRequest createReq = new AccountFormRequest();
createReq.accountName = 'Tech Innovations Inc';
createReq.accountType = 'Customer';
createReq.industry = 'Technology';
createReq.phone = '555-0100';
createReq.website = 'www.techinnovations.com';
createReq.annualRevenue = 10000000;
createReq.numberOfEmployees = 250;
createReq.billingStreet = '123 Tech Street';
createReq.billingCity = 'San Francisco';
createReq.billingState = 'CA';
createReq.billingPostalCode = '94105';
createReq.billingCountry = 'USA';
createReq.rating = 'Hot';

// Call create action
List<AccountFormResponse> results = 
    AccountFormAction.createAccount(
        new List<AccountFormRequest>{ createReq }
    );

System.debug('Success: ' + results[0].success);
System.debug('Account ID: ' + results[0].accountId);
System.debug('Message: ' + results[0].message);
```

---

## 🎨 Customization

### Add Custom Fields

Add your org's custom fields to the form:

1. **Update AccountFormRequest.cls**:
```apex
@AuraEnabled
global String customField1__c;

@AuraEnabled
global Decimal customField2__c;
```

2. **Add to Form HTML**:
```html
<div class="slds-col slds-size_1-of-2">
    <lightning-input-field 
        field-name="CustomField1__c"
        value={formData.customField1__c}
    ></lightning-input-field>
</div>
```

3. **Update Agent Action**:
```apex
newAccount.CustomField1__c = request.customField1__c;
```

### Make Fields Required

Add validation to required fields:

```html
<lightning-input-field 
    field-name="Phone"
    value={formData.phone}
    required
></lightning-input-field>
```

Or in JavaScript:

```javascript
validateForm() {
    const requiredFields = ['accountName', 'phone', 'industry'];
    let isValid = true;
    
    requiredFields.forEach(field => {
        if (!this.formData[field]) {
            this.showToast('Error', `${field} is required`, 'error');
            isValid = false;
        }
    });
    
    return isValid;
}
```

### Add Field Sections

Organize fields into collapsible sections:

```html
<lightning-accordion active-section-name={activeSections}>
    <lightning-accordion-section 
        name="company" 
        label="Company Information"
    >
        <!-- Company fields -->
    </lightning-accordion-section>
    
    <lightning-accordion-section 
        name="contact" 
        label="Contact Information"
    >
        <!-- Contact fields -->
    </lightning-accordion-section>
    
    <lightning-accordion-section 
        name="address" 
        label="Address Information"
    >
        <!-- Address fields -->
    </lightning-accordion-section>
</lightning-accordion>
```

### Add Copy Address Functionality

Already included! Copy billing to shipping:

```javascript
handleCopyBillingToShipping() {
    this.formData.shippingStreet = this.formData.billingStreet;
    this.formData.shippingCity = this.formData.billingCity;
    this.formData.shippingState = this.formData.billingState;
    this.formData.shippingPostalCode = this.formData.billingPostalCode;
    this.formData.shippingCountry = this.formData.billingCountry;
    
    this.showToast('Success', 'Shipping address copied from billing', 'success');
}
```

### Add Field Dependencies

Implement dependent picklists:

```javascript
handleAccountTypeChange(event) {
    this.formData.accountType = event.detail.value;
    
    // Update dependent fields based on type
    if (this.formData.accountType === 'Customer') {
        this.showCustomerFields = true;
    } else {
        this.showCustomerFields = false;
    }
}
```

### Add Pre-population

Pre-fill form with default values:

```javascript
connectedCallback() {
    // Set defaults for new records
    if (!this.recordId) {
        this.formData = {
            accountType: 'Prospect',
            rating: 'Warm',
            billingCountry: 'USA',
            shippingCountry: 'USA',
            numberOfEmployees: 1
        };
    }
}
```

---

## 🔍 Testing

### Test in Developer Console

```apex
// Test account creation
AccountFormRequest createRequest = new AccountFormRequest();
createRequest.accountName = 'Test Company Ltd';
createRequest.accountType = 'Customer';
createRequest.industry = 'Technology';
createRequest.phone = '555-1234';
createRequest.website = 'www.testcompany.com';
createRequest.annualRevenue = 5000000;
createRequest.numberOfEmployees = 100;
createRequest.billingCity = 'New York';
createRequest.billingState = 'NY';
createRequest.billingCountry = 'USA';
createRequest.rating = 'Hot';

List<AccountFormResponse> createResults = 
    AccountFormAction.createAccount(
        new List<AccountFormRequest>{ createRequest }
    );

System.assert(createResults[0].success, 'Account creation failed');
System.debug('Created Account ID: ' + createResults[0].accountId);

// Test account update
AccountFormRequest updateRequest = createRequest;
updateRequest.accountId = createResults[0].accountId;
updateRequest.phone = '555-5678'; // Change phone
updateRequest.rating = 'Cold'; // Change rating

List<AccountFormResponse> updateResults = 
    AccountFormAction.updateAccount(
        new List<AccountFormRequest>{ updateRequest }
    );

System.assert(updateResults[0].success, 'Account update failed');
System.debug('Update Message: ' + updateResults[0].message);

// Verify changes
Account updatedAccount = [
    SELECT Phone, Rating 
    FROM Account 
    WHERE Id = :createResults[0].accountId
];
System.assertEquals('555-5678', updatedAccount.Phone);
System.assertEquals('Cold', updatedAccount.Rating);
```

### Test in Agentforce

1. Create an Agentforce agent
2. Add the `AccountFormAction` as an action
3. Test prompts:
   - "Create a new account"
   - "Add a new customer"
   - "Register a company"
4. Fill out the form
5. Verify account is created in Salesforce
6. Test update functionality

---

## 📊 Features Breakdown

### Form Features
- **Full Field Coverage**: All standard Account fields
- **Organized Layout**: Logical sections for easy completion
- **Copy Address**: One-click billing to shipping copy
- **Field Validation**: Built-in and custom validation
- **Required Indicators**: Visual cues for required fields
- **Help Text**: Field-level guidance
- **Picklist Support**: All standard picklists included

### User Experience
- **Responsive Layout**: 2-column design on desktop, stacked on mobile
- **Loading States**: Spinner during save operations
- **Success Messages**: Toast notifications on save
- **Error Handling**: Clear error messages
- **Cancel Support**: Discard changes option
- **Pre-population**: Default values for new records
- **Edit Mode**: Load existing records for editing

### Developer Features
- **Extensible Model**: Easy to add custom fields
- **Reusable Component**: Use in multiple contexts
- **Event Handling**: Custom events for integration
- **SLDS Compliant**: Follows Salesforce design system
- **API Version 64.0+**: Latest features and best practices

---

## 🛠️ Troubleshooting

### Form Not Saving

1. Check field-level security for all fields
2. Verify user has create/edit permission on Account
3. Review validation rules on Account object
4. Check browser console for JavaScript errors
5. Review Apex debug logs for DML errors

### Fields Not Displaying

1. Verify field API names are correct
2. Check field-level security
3. Ensure fields exist in your org
4. Check page layout configuration
5. Verify component API version

### Copy Address Not Working

1. Check that all address field API names match
2. Verify fields are properly bound in template
3. Test binding in browser console
4. Ensure fields are editable

### Picklist Values Not Showing

1. Verify picklist field has values defined
2. Check record type picklist value assignments
3. Verify field-level security on picklist
4. Check for dependent picklist configurations

---

## 📚 Additional Resources

- [Custom Lightning Types Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/lightning-types-custom.html)
- [Lightning Record Edit Form](https://developer.salesforce.com/docs/component-library/bundle/lightning-record-edit-form)
- [Lightning Input Field](https://developer.salesforce.com/docs/component-library/bundle/lightning-input-field)
- [Account Object Reference](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_account.htm)

---

## ✅ Checklist

- [x] AccountFormRequest.cls model created
- [x] AccountFormAction.cls agent action created
- [x] schema.json linking to Apex class
- [x] renderer.json linking to form LWC
- [x] accountRecordForm LWC component
- [x] All standard Account fields included
- [x] Billing and Shipping address sections
- [x] Copy billing to shipping functionality
- [x] Field validation
- [x] Save and cancel buttons
- [x] Success/error toast messages
- [x] Responsive 2-column layout
- [x] SLDS styling
- [x] package.xml manifest
- [x] Documentation

---

## 🎉 You're Ready!

Your custom Lightning type for account record forms is complete and ready to deploy. This form provides a comprehensive interface for creating and editing accounts in Agentforce!

**Next Steps:**
1. Deploy to your Salesforce org
2. Create Agentforce actions for create/update
3. Test with sample data
4. Add custom fields specific to your org
5. Customize validation rules
6. Add to your agent workflows!

---

## 💡 Pro Tips

1. **Use for Onboarding**: Perfect for new customer registration
2. **Add Lookup Fields**: Link to parent accounts, owners
3. **Implement Auto-save**: Save as user types
4. **Add File Upload**: Attach documents to account
5. **Multi-step Form**: Break into wizard for better UX
6. **Duplicate Detection**: Check for existing accounts before create
7. **Address Validation**: Integrate with address validation API

---

*Created: November 8, 2025*  
*API Version: 64.0+*  
*Metadata Type: LightningTypeBundle*  
*Type: INPUT (Editor/Form)*
