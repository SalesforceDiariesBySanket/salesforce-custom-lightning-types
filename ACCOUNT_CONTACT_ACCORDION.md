# Account Contact Accordion Custom Lightning Type

## Overview
This custom Lightning type displays accounts with their related contacts in an elegant accordion interface. Each account can be expanded to reveal its associated contacts in a formatted card layout. Perfect for Agentforce agent actions that need to show hierarchical account-contact relationships.

## 📁 Project Structure

```
force-app/main/default/
├── classes/
│   ├── AccountContactList.cls           # Wrapper class for account-contact list
│   ├── AccountContactList.cls-meta.xml
│   ├── AccountWithContacts.cls          # Account with contacts data model
│   ├── AccountWithContacts.cls-meta.xml
│   ├── ContactData.cls                  # Individual contact data model
│   ├── ContactData.cls-meta.xml
│   ├── AccountContactAgentAction.cls    # Example agent action (optional)
│   └── AccountContactAgentAction.cls-meta.xml
├── lightningTypes/
│   └── accountContactAccordion/
│       ├── schema.json                  # Links to AccountContactList Apex class
│       └── lightningDesktopGenAi/
│           └── renderer.json            # References the LWC component
└── lwc/
    └── accountContactAccordion/
        ├── accountContactAccordion.js   # Component logic
        ├── accountContactAccordion.html # Component template
        ├── accountContactAccordion.css  # Styling
        └── accountContactAccordion.js-meta.xml # Component metadata
```

---

## 🎯 What's Included

### 1. **Apex Classes**

#### AccountContactList.cls
Wrapper class that holds the list of accounts with their contacts.

```apex
@JsonAccess(serializable='always' deserializable='always')
global class AccountContactList {
    @AuraEnabled
    global List<AccountWithContacts> accounts;  // List of accounts with contacts
    
    @AuraEnabled
    global Integer totalAccounts;               // Total account count
    
    @AuraEnabled
    global Integer totalContacts;               // Total contact count
    
    @AuraEnabled
    global String message;                      // Display message
}
```

#### AccountWithContacts.cls
Represents an account with its related contacts.

**Account Fields:**
- Account ID, Name, Type
- Industry, Phone, Website
- Annual Revenue, Number of Employees
- Billing City, State, Country
- Rating

**Contact List:**
- List of ContactData objects for each account

#### ContactData.cls
Represents individual contact information.

**Fields included:**
- Contact ID, First Name, Last Name
- Email, Phone, Mobile Phone
- Title, Department
- Mailing City, State, Country

### 2. **Lightning Type Bundle**

#### schema.json
```json
{
  "title": "Account Contact Accordion",
  "description": "Custom Lightning Type for displaying accounts with related contacts in an accordion",
  "lightning:type": "@apexClassType/c__AccountContactList"
}
```

Links the custom type to the `AccountContactList` Apex class.

#### renderer.json
```json
{
  "renderer": {
    "componentOverrides": {
      "$": {
        "definition": "c/accountContactAccordion"
      }
    }
  }
}
```

Specifies that `accountContactAccordion` LWC should render the output.

### 3. **Lightning Web Component**

#### accountContactAccordion
A sophisticated accordion component with:

**Features:**
- ✅ Expandable/collapsible accordion sections
- ✅ Account summary cards with key information
- ✅ Contact cards with detailed information
- ✅ Clickable account and contact names (links to records)
- ✅ Formatted phone numbers and emails
- ✅ Color-coded ratings
- ✅ Contact count badges
- ✅ Empty state handling
- ✅ Summary statistics (total accounts and contacts)
- ✅ Responsive design with SLDS styling

**Account Display:**
- Account name with record link
- Type, Industry, Rating badges
- Phone, Website (clickable)
- Annual Revenue (formatted)
- Employee count
- Location information

**Contact Display:**
- Full name with record link
- Title and Department
- Email (clickable mailto link)
- Phone and Mobile (formatted)
- Mailing address

---

## 🚀 Deployment

### Step 1: Deploy to Salesforce

```bash
# Deploy all metadata
sf project deploy start --manifest manifest/package.xml

# Or deploy specific components
sf project deploy start --source-dir force-app/main/default/classes
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountContactAccordion
sf project deploy start --source-dir force-app/main/default/lwc/accountContactAccordion
```

### Step 2: Verify Deployment

```bash
# Check deployed Lightning Types
sf data query --query "SELECT Id, DeveloperName FROM LightningTypeBundle WHERE DeveloperName = 'accountContactAccordion'" --use-tooling-api
```

---

## 💡 Usage Examples

### Example 1: Simple Agent Action

Create an agent action that returns `AccountContactList`:

```apex
@InvocableMethod(label='Get Accounts with Contacts' category='Agent Actions')
public static List<AccountContactList> getAccountsWithContacts() {
    AccountContactList result = new AccountContactList();
    result.accounts = new List<AccountWithContacts>();
    
    // Query accounts
    List<Account> accts = [
        SELECT Id, Name, Type, Industry, Phone, Website,
               AnnualRevenue, NumberOfEmployees, 
               BillingCity, BillingState, BillingCountry,
               Rating,
               (SELECT Id, FirstName, LastName, Email, Phone, MobilePhone,
                       Title, Department, MailingCity, MailingState, MailingCountry
                FROM Contacts
                LIMIT 10)
        FROM Account
        WHERE Id IN (SELECT AccountId FROM Contact)
        LIMIT 20
    ];
    
    Integer totalContacts = 0;
    
    // Transform to AccountWithContacts
    for (Account acc : accts) {
        AccountWithContacts accWithContacts = new AccountWithContacts();
        accWithContacts.accountId = acc.Id;
        accWithContacts.accountName = acc.Name;
        accWithContacts.accountType = acc.Type;
        accWithContacts.industry = acc.Industry;
        accWithContacts.phone = acc.Phone;
        accWithContacts.website = acc.Website;
        accWithContacts.annualRevenue = acc.AnnualRevenue;
        accWithContacts.numberOfEmployees = acc.NumberOfEmployees;
        accWithContacts.billingCity = acc.BillingCity;
        accWithContacts.billingState = acc.BillingState;
        accWithContacts.billingCountry = acc.BillingCountry;
        accWithContacts.rating = acc.Rating;
        
        accWithContacts.contacts = new List<ContactData>();
        for (Contact con : acc.Contacts) {
            ContactData contactData = new ContactData();
            contactData.contactId = con.Id;
            contactData.firstName = con.FirstName;
            contactData.lastName = con.LastName;
            contactData.email = con.Email;
            contactData.phone = con.Phone;
            contactData.mobilePhone = con.MobilePhone;
            contactData.title = con.Title;
            contactData.department = con.Department;
            contactData.mailingCity = con.MailingCity;
            contactData.mailingState = con.MailingState;
            contactData.mailingCountry = con.MailingCountry;
            
            accWithContacts.contacts.add(contactData);
            totalContacts++;
        }
        
        result.accounts.add(accWithContacts);
    }
    
    result.totalAccounts = result.accounts.size();
    result.totalContacts = totalContacts;
    result.message = 'Found ' + result.totalAccounts + ' accounts with ' + totalContacts + ' total contacts';
    
    return new List<AccountContactList>{ result };
}
```

### Example 2: Filtered Agent Action

Use the included `AccountContactAgentAction.cls`:

```apex
// This invocable method accepts filters:
// - Account Type
// - Industry
// - Has Contacts flag
// - Max Accounts

AccountContactAgentAction.getAccountsWithContacts(new List<AccountContactAgentAction.Request>{
    new AccountContactAgentAction.Request('Customer', 'Technology', true, 15)
});
```

### Example 3: Use in Agentforce

1. **Create an Agentforce Action** pointing to your invocable method
2. **Set the Output Type** to `AccountContactList`
3. **The custom Lightning type will automatically render** the accordion interface
4. **Agent will display** accounts with expandable contact sections

---

## 🎨 Customization

### Modify Account Display Fields

Edit `accountContactAccordion.js` to add/remove account fields:

```javascript
// In the template HTML, add custom fields:
<div class="slds-col slds-size_1-of-2">
    <div class="slds-text-body_small">
        <strong>Custom Field:</strong> {account.customField}
    </div>
</div>
```

Don't forget to:
1. Add field to `AccountWithContacts.cls`
2. Query the field in your Apex action
3. Map the data in component logic

### Modify Contact Display

Add or remove contact fields in the template:

```html
<!-- Add to contact card -->
<div class="slds-m-top_x-small">
    <lightning-formatted-phone value={contact.otherPhone}></lightning-formatted-phone>
</div>
```

### Modify Styling

Edit `accountContactAccordion.css`:

```css
/* Customize accordion header */
.account-header {
    background: linear-gradient(to right, #f3f3f3, #ffffff);
}

/* Customize contact cards */
.contact-card {
    border-left: 4px solid #0070d2;
}
```

### Add Statistics Panel

Add a summary panel showing aggregate data:

```html
<div class="slds-box slds-theme_shade slds-m-bottom_medium">
    <div class="slds-grid slds-wrap">
        <div class="slds-col slds-size_1-of-3">
            <div class="slds-text-heading_small">{totalAccounts}</div>
            <div class="slds-text-body_small">Total Accounts</div>
        </div>
        <div class="slds-col slds-size_1-of-3">
            <div class="slds-text-heading_small">{totalContacts}</div>
            <div class="slds-text-body_small">Total Contacts</div>
        </div>
        <div class="slds-col slds-size_1-of-3">
            <div class="slds-text-heading_small">{avgContactsPerAccount}</div>
            <div class="slds-text-body_small">Avg Contacts/Account</div>
        </div>
    </div>
</div>
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
    Rating = 'Hot'
);
insert testAcc;

Contact testCon1 = new Contact(
    AccountId = testAcc.Id,
    FirstName = 'John',
    LastName = 'Doe',
    Email = 'john.doe@example.com',
    Title = 'CEO'
);
Contact testCon2 = new Contact(
    AccountId = testAcc.Id,
    FirstName = 'Jane',
    LastName = 'Smith',
    Email = 'jane.smith@example.com',
    Title = 'CTO'
);
insert new List<Contact>{ testCon1, testCon2 };

// Test the action
List<AccountContactList> results = AccountContactAgentAction.getAccountsWithContacts(
    new List<AccountContactAgentAction.Request>{
        new AccountContactAgentAction.Request(null, null, true, 10)
    }
);

System.debug('Total Accounts: ' + results[0].totalAccounts);
System.debug('Total Contacts: ' + results[0].totalContacts);
System.debug('Message: ' + results[0].message);
```

### Test in Agentforce

1. Create an Agentforce agent
2. Add action using `AccountContactAgentAction`
3. Configure action inputs (optional filters)
4. Test in Agent Builder
5. Verify the accordion displays and expands correctly

---

## 📊 Features Breakdown

### Visual Indicators
- **Rating Colors**:
  - 🔴 Hot = Red badge
  - 🟠 Warm = Orange badge
  - ⚫ Cold = Gray badge
- **Contact Count Badges**: Shows number of contacts per account
- **Summary Statistics**: Total accounts and contacts at the top
- **Info Banner**: Displays custom messages

### Interactions
- **Expandable Accordion**: Click to expand/collapse account sections
- **Clickable Names**: Navigate to account/contact records
- **Formatted Communications**: Click-to-call phone, click-to-email
- **Responsive Layout**: Adapts to different screen sizes

### Data Formatting
- **Phone Numbers**: Formatted with lightning-formatted-phone
- **Emails**: Formatted with lightning-formatted-email
- **Currency**: Annual Revenue formatted with currency symbol
- **Location**: City, State, Country intelligently concatenated
- **URLs**: Website as clickable link

---

## 🛠️ Troubleshooting

### Accordion Not Expanding

1. Check browser console for JavaScript errors
2. Verify `activeSectionName` is properly bound
3. Ensure each accordion section has unique `name` attribute
4. Check that `lightning-accordion` component is properly imported

### Contacts Not Displaying

1. Verify SOQL query includes Contact subquery
2. Check `@AuraEnabled` on all ContactData properties
3. Ensure parent-child relationship is properly maintained
4. Verify contact data is being mapped correctly in Apex

### Styling Issues

1. Use SLDS utility classes for consistency
2. Check for CSS specificity conflicts
3. Test in different browsers
4. Verify component isolation (CSS scoping)

---

## 📚 Additional Resources

- [Custom Lightning Types Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/lightning-types-custom.html)
- [Lightning Accordion Documentation](https://developer.salesforce.com/docs/component-library/bundle/lightning-accordion)
- [SOQL Parent-Child Relationships](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql_relationships.htm)
- [Agentforce Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/get-started-agents.html)

---

## ✅ Checklist

- [x] AccountContactList.cls wrapper class created
- [x] AccountWithContacts.cls data model created
- [x] ContactData.cls data model created
- [x] schema.json linking to Apex class
- [x] renderer.json linking to LWC
- [x] accountContactAccordion LWC component
- [x] Accordion functionality implemented
- [x] Contact cards with formatting
- [x] Styling with SLDS
- [x] package.xml manifest
- [x] Example agent action
- [x] Documentation

---

## 🎉 You're Ready!

Your custom Lightning type for account-contact accordion is complete and ready to deploy. Use it in your Agentforce actions to create beautiful, hierarchical displays of accounts and their related contacts!

**Next Steps:**
1. Deploy to your Salesforce org
2. Create an Agentforce action
3. Test with sample account-contact data
4. Customize fields and styling to your needs
5. Share with your team!

---

*Created: November 8, 2025*  
*API Version: 64.0+*  
*Metadata Type: LightningTypeBundle*
