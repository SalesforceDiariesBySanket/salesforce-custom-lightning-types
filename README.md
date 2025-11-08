# 🚀 Salesforce Custom Lightning Types Collection

A comprehensive collection of **5 production-ready custom Lightning Types** for Salesforce Agentforce, demonstrating various UI patterns and agent action integrations. These components showcase how to create beautiful, functional interfaces for AI agents using Lightning Web Components and Apex.

[![Salesforce API](https://img.shields.io/badge/Salesforce%20API-v64.0+-blue.svg)](https://developer.salesforce.com/)
[![Lightning Web Components](https://img.shields.io/badge/LWC-enabled-brightgreen.svg)](https://developer.salesforce.com/docs/component-library/overview/components)
[![Custom Lightning Types](https://img.shields.io/badge/Custom%20Lightning%20Types-5-orange.svg)](https://developer.salesforce.com/docs/einstein/genai/guide/lightning-types-custom.html)

---

## 📋 Table of Contents

- [Overview](#overview)
- [What Are Custom Lightning Types?](#what-are-custom-lightning-types)
- [Lightning Types Included](#lightning-types-included)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Deployment Guide](#deployment-guide)
- [Usage Examples](#usage-examples)
- [Documentation](#documentation)
- [Key Features](#key-features)
- [Prerequisites](#prerequisites)
- [Learning Resources](#learning-resources)
- [Contributing](#contributing)
- [License](#license)

---

## 🎯 Overview

This repository contains **5 fully functional custom Lightning Types** designed for Salesforce Agentforce:

| # | Lightning Type | Type | Description | Use Case |
|---|---|---|---|---|
| 1 | **Account Table** | Output | Display accounts in a sortable data table | Show filtered or searched account lists |
| 2 | **Account Contact Accordion** | Output | Show accounts with related contacts in accordion | Display hierarchical account-contact data |
| 3 | **Account Industry Filter** | Input/Output | Multi-select industry filter with results | Collect user filter preferences |
| 4 | **Account Lookup** | Input/Output | Search and select accounts with detail card | Get user account selection |
| 5 | **Account Record Form** | Input | Create/edit accounts with comprehensive form | Collect account information |

Each Lightning Type includes:
- ✅ Complete Apex classes (data models and agent actions)
- ✅ Lightning Type bundle (schema.json and renderer.json)
- ✅ Lightning Web Components (fully styled with SLDS)
- ✅ Example Agentforce agent actions
- ✅ Comprehensive documentation
- ✅ Ready-to-deploy code

---

## 💡 What Are Custom Lightning Types?

**Custom Lightning Types** are a powerful feature in Salesforce that allow you to define how Agentforce agents display data and collect input from users. Instead of showing raw JSON or plain text, you can create beautiful, interactive UI components.

### Why Use Custom Lightning Types?

- 🎨 **Better UX**: Display data in tables, cards, forms instead of JSON
- 🤖 **AI-Powered**: Seamlessly integrate with Agentforce agents
- 🔄 **Reusable**: Define once, use across multiple agent actions
- 📱 **Responsive**: Built with Lightning Web Components (mobile-ready)
- ⚡ **Fast**: Leverage Lightning Data Service for optimal performance

### How They Work

```
┌─────────────────┐
│  Agent Action   │  (Apex Invocable Method)
│  Returns Data   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Apex Class Type │  (AccountList, AccountLookupRequest, etc.)
│   (Model)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Lightning Type  │  (schema.json + renderer.json)
│    Bundle       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│      LWC        │  (accountDataTable, accountLookupPicker, etc.)
│   Component     │
└─────────────────┘
```

---

## 📦 Lightning Types Included

### 1️⃣ Account Table (`accountTable`)

**Type:** Output Display  
**Use Case:** Display a list of accounts in a beautiful, sortable data table

**Features:**
- Sortable columns with visual indicators
- Clickable account names (navigate to records)
- Formatted currency, phone, dates
- Color-coded ratings (Hot, Warm, Cold)
- Location concatenation
- Record count badge
- Empty state handling
- 10 pre-configured columns

**Agent Action:** `AccountAgentAction.getAccounts`

📚 **[View Full Documentation →](./ACCOUNT_TABLE_LIGHTNING_TYPE.md)**

---

### 2️⃣ Account Contact Accordion (`accountContactAccordion`)

**Type:** Output Display  
**Use Case:** Show accounts with their related contacts in an expandable accordion

**Features:**
- Expandable/collapsible accordion sections
- Account summary cards with key metrics
- Contact cards with full details
- Clickable names for navigation
- Formatted emails and phone numbers
- Contact count badges per account
- Summary statistics (total accounts/contacts)
- Responsive card layout

**Agent Action:** `AccountContactAgentAction.getAccountsWithContacts`

📚 **[View Full Documentation →](./ACCOUNT_CONTACT_ACCORDION.md)**

---

### 3️⃣ Account Industry Filter (`accountIndustryFilter`)

**Type:** Input (Editor) + Output Display  
**Use Case:** Allow users to filter accounts by industry with multi-select picklist

**Features:**
- Multi-select industry picklist (30+ industries)
- Account type filter dropdown
- Max records slider control
- Clear filters button
- Selected filters display (chips)
- Real-time validation
- Dynamic SOQL query building
- Results displayed in data table

**Agent Action:** `AccountIndustryFilterAction.filterAccounts`

📚 **[View Full Documentation →](./ACCOUNT_INDUSTRY_FILTER.md)**

---

### 4️⃣ Account Lookup (`accountLookup`)

**Type:** Input (Editor) + Output Display  
**Use Case:** Search and select an account, then display detailed information

**Features:**
- Lightning Record Picker with autocomplete
- Type-ahead search across account names
- Recent accounts displayed
- Selected account detail card
- Complete account information display
- Related record counts (contacts, opportunities)
- Quick action buttons (view, edit)
- Formatted currency and contact info
- Color-coded rating indicators

**Agent Action:** `AccountLookupAction.getAccountDetails`

📚 **[View Full Documentation →](./ACCOUNT_LOOKUP.md)**

---

### 5️⃣ Account Record Form (`accountRecordForm`)

**Type:** Input (Editor/Form)  
**Use Case:** Create new accounts or edit existing accounts with a comprehensive form

**Features:**
- All standard Account fields
- Organized sections (Company, Contact, Address)
- Billing and Shipping address forms
- "Copy Billing to Shipping" button
- Field-level validation
- Required field indicators
- Help text for fields
- Save and Cancel buttons
- Success/Error toast messages
- Pre-population support
- Edit mode for existing records

**Agent Action:** `AccountFormAction.createAccount` / `updateAccount`

📚 **[View Full Documentation →](./ACCOUNT_RECORD_FORM.md)**

---

## 📁 Project Structure

```
salesforce-custom-lightning-types/
├── README.md                              # This file
├── ACCOUNT_TABLE_LIGHTNING_TYPE.md        # Account Table docs
├── ACCOUNT_CONTACT_ACCORDION.md           # Accordion docs
├── ACCOUNT_INDUSTRY_FILTER.md             # Industry Filter docs
├── ACCOUNT_LOOKUP.md                      # Account Lookup docs
├── ACCOUNT_RECORD_FORM.md                 # Record Form docs
├── CUSTOM_LIGHTNING_TYPES_EXPLAINED.md    # General guide
├── sfdx-project.json                      # SFDX project config
├── package.json                           # Node dependencies
├── jest.config.js                         # Jest testing config
├── eslint.config.js                       # ESLint config
│
├── force-app/main/default/
│   ├── classes/                           # All Apex classes
│   │   ├── AccountList.cls                # Table data model
│   │   ├── AccountData.cls                # Individual account model
│   │   ├── AccountAgentAction.cls         # Table agent action
│   │   ├── AccountContactList.cls         # Accordion data model
│   │   ├── AccountWithContacts.cls        # Account+contacts model
│   │   ├── ContactData.cls                # Contact data model
│   │   ├── AccountContactAgentAction.cls  # Accordion agent action
│   │   ├── AccountFilterRequest.cls       # Filter input model
│   │   ├── AccountFilterResponse.cls      # Filter output model
│   │   ├── AccountIndustryFilterAction.cls # Filter agent action
│   │   ├── AccountLookupRequest.cls       # Lookup input model
│   │   ├── AccountLookupResponse.cls      # Lookup output model
│   │   ├── AccountLookupAction.cls        # Lookup agent action
│   │   ├── AccountFormRequest.cls         # Form data model
│   │   └── AccountFormAction.cls          # Form agent action
│   │
│   ├── lightningTypes/                    # All Lightning Type bundles
│   │   ├── accountTable/
│   │   │   ├── schema.json
│   │   │   └── lightningDesktopGenAi/
│   │   │       └── renderer.json
│   │   ├── accountContactAccordion/
│   │   │   ├── schema.json
│   │   │   └── lightningDesktopGenAi/
│   │   │       └── renderer.json
│   │   ├── accountIndustryFilter/
│   │   │   ├── schema.json
│   │   │   └── lightningDesktopGenAi/
│   │   │       └── renderer.json
│   │   ├── accountLookup/
│   │   │   ├── schema.json
│   │   │   └── lightningDesktopGenAi/
│   │   │       └── renderer.json
│   │   └── accountRecordForm/
│   │       ├── schema.json
│   │       └── lightningDesktopGenAi/
│   │           └── renderer.json
│   │
│   └── lwc/                               # All Lightning Web Components
│       ├── accountDataTable/              # Table component
│       │   ├── accountDataTable.js
│       │   ├── accountDataTable.html
│       │   ├── accountDataTable.css
│       │   └── accountDataTable.js-meta.xml
│       ├── accountContactAccordion/       # Accordion component
│       │   ├── accountContactAccordion.js
│       │   ├── accountContactAccordion.html
│       │   ├── accountContactAccordion.css
│       │   └── accountContactAccordion.js-meta.xml
│       ├── accountIndustryEditor/         # Filter input component
│       │   ├── accountIndustryEditor.js
│       │   ├── accountIndustryEditor.html
│       │   ├── accountIndustryEditor.css
│       │   └── accountIndustryEditor.js-meta.xml
│       ├── accountIndustryResults/        # Filter results component
│       │   ├── accountIndustryResults.js
│       │   ├── accountIndustryResults.html
│       │   ├── accountIndustryResults.css
│       │   └── accountIndustryResults.js-meta.xml
│       ├── accountLookupPicker/           # Lookup input component
│       │   ├── accountLookupPicker.js
│       │   ├── accountLookupPicker.html
│       │   ├── accountLookupPicker.css
│       │   └── accountLookupPicker.js-meta.xml
│       ├── accountLookupCard/             # Lookup display component
│       │   ├── accountLookupCard.js
│       │   ├── accountLookupCard.html
│       │   ├── accountLookupCard.css
│       │   └── accountLookupCard.js-meta.xml
│       └── accountRecordForm/             # Form component
│           ├── accountRecordForm.js
│           ├── accountRecordForm.html
│           ├── accountRecordForm.css
│           └── accountRecordForm.js-meta.xml
│
├── manifest/
│   └── package.xml                        # Deployment manifest
│
└── config/
    └── project-scratch-def.json           # Scratch org definition
```

---

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/SalesforceDiariesBySanket/salesforce-custom-lightning-types.git
cd salesforce-custom-lightning-types
```

### 2. Authenticate to Your Org

```bash
# Authenticate to your Salesforce org
sf org login web --alias my-org

# Or create a scratch org
sf org create scratch --definition-file config/project-scratch-def.json --alias scratch-org --set-default
```

### 3. Deploy to Salesforce

```bash
# Deploy all metadata
sf project deploy start --manifest manifest/package.xml

# Or deploy specific components
sf project deploy start --source-dir force-app/main/default/classes
sf project deploy start --source-dir force-app/main/default/lightningTypes
sf project deploy start --source-dir force-app/main/default/lwc
```

### 4. Verify Deployment

```bash
# Check Lightning Type Bundles
sf data query --query "SELECT Id, DeveloperName, MasterLabel FROM LightningTypeBundle" --use-tooling-api

# Or via REST API
GET /services/data/v64.0/connect/lightning-types
```

### 5. Test in Agentforce

1. Go to **Setup** > **Agentforce** > **Agent Builder**
2. Create a new agent or edit existing one
3. Add an **Agent Action** using one of the included Apex actions
4. Set the **Input/Output Types** to the corresponding Lightning Types
5. **Test** in the Agent Builder
6. See your beautiful custom UI in action!

---

## 📚 Deployment Guide

### Option 1: Deploy Everything

```bash
sf project deploy start --manifest manifest/package.xml
```

### Option 2: Deploy Specific Lightning Type

**Account Table:**
```bash
sf project deploy start --source-dir force-app/main/default/classes/AccountList.cls,force-app/main/default/classes/AccountData.cls,force-app/main/default/classes/AccountAgentAction.cls
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountTable
sf project deploy start --source-dir force-app/main/default/lwc/accountDataTable
```

**Account Contact Accordion:**
```bash
sf project deploy start --source-dir force-app/main/default/classes/AccountContactList.cls,force-app/main/default/classes/AccountWithContacts.cls,force-app/main/default/classes/ContactData.cls
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountContactAccordion
sf project deploy start --source-dir force-app/main/default/lwc/accountContactAccordion
```

**Account Industry Filter:**
```bash
sf project deploy start --source-dir force-app/main/default/classes/AccountFilterRequest.cls,force-app/main/default/classes/AccountFilterResponse.cls,force-app/main/default/classes/AccountIndustryFilterAction.cls
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountIndustryFilter
sf project deploy start --source-dir force-app/main/default/lwc/accountIndustryEditor,force-app/main/default/lwc/accountIndustryResults
```

**Account Lookup:**
```bash
sf project deploy start --source-dir force-app/main/default/classes/AccountLookupRequest.cls,force-app/main/default/classes/AccountLookupResponse.cls,force-app/main/default/classes/AccountLookupAction.cls
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountLookup
sf project deploy start --source-dir force-app/main/default/lwc/accountLookupPicker,force-app/main/default/lwc/accountLookupCard
```

**Account Record Form:**
```bash
sf project deploy start --source-dir force-app/main/default/classes/AccountFormRequest.cls,force-app/main/default/classes/AccountFormAction.cls
sf project deploy start --source-dir force-app/main/default/lightningTypes/accountRecordForm
sf project deploy start --source-dir force-app/main/default/lwc/accountRecordForm
```

### Option 3: Deploy via VS Code

1. Right-click on `manifest/package.xml`
2. Select **SFDX: Deploy Source to Org**

---

## 💻 Usage Examples

### Example 1: Display Account List

```apex
// In your Apex class
@InvocableMethod(label='Get Technology Accounts')
public static List<AccountList> getTechAccounts() {
    AccountList result = new AccountList();
    result.accounts = new List<AccountData>();
    
    for (Account acc : [SELECT Id, Name, Industry, Phone FROM Account WHERE Industry = 'Technology' LIMIT 10]) {
        result.accounts.add(new AccountData(/* fields */));
    }
    
    result.totalRecords = result.accounts.size();
    result.message = 'Found ' + result.totalRecords + ' technology accounts';
    
    return new List<AccountList>{ result };
}
```

**Agent Interaction:**
- User: "Show me technology companies"
- Agent: *Displays beautiful data table with technology accounts*

---

### Example 2: Filter Accounts by Industry

```apex
// Agent asks: "Show me healthcare companies"
// → accountIndustryFilter renders multi-select picker
// → User selects "Healthcare"
// → AccountIndustryFilterAction processes filter
// → Results shown in accountIndustryResults table
```

---

### Example 3: Create New Account

```apex
// Agent asks: "Create a new customer account"
// → accountRecordForm renders with empty fields
// → User fills in: Name, Type, Industry, Address, etc.
// → AccountFormAction.createAccount processes form
// → New account created in Salesforce
// → Success message displayed
```

---

## 📖 Documentation

Each Lightning Type has comprehensive documentation:

| Lightning Type | Documentation Link |
|---|---|
| Account Table | [ACCOUNT_TABLE_LIGHTNING_TYPE.md](./ACCOUNT_TABLE_LIGHTNING_TYPE.md) |
| Account Contact Accordion | [ACCOUNT_CONTACT_ACCORDION.md](./ACCOUNT_CONTACT_ACCORDION.md) |
| Account Industry Filter | [ACCOUNT_INDUSTRY_FILTER.md](./ACCOUNT_INDUSTRY_FILTER.md) |
| Account Lookup | [ACCOUNT_LOOKUP.md](./ACCOUNT_LOOKUP.md) |
| Account Record Form | [ACCOUNT_RECORD_FORM.md](./ACCOUNT_RECORD_FORM.md) |
| General Guide | [CUSTOM_LIGHTNING_TYPES_EXPLAINED.md](./CUSTOM_LIGHTNING_TYPES_EXPLAINED.md) |

Each documentation includes:
- ✅ Detailed feature breakdown
- ✅ Code examples and usage
- ✅ Customization guide
- ✅ Testing instructions
- ✅ Troubleshooting tips
- ✅ API references

---

## ⭐ Key Features

### 🎨 Beautiful UI
- Built with **Salesforce Lightning Design System (SLDS)**
- Responsive design (desktop, tablet, mobile)
- Consistent styling across all components
- Accessible (WCAG 2.1 compliant)

### 🤖 AI-Ready
- Designed for **Agentforce** integration
- Seamless input/output handling
- Natural language interaction support
- Smart data formatting

### ⚡ Performance Optimized
- Efficient data handling
- Lightning Data Service integration
- Lazy loading where applicable
- Minimal API calls

### 🔧 Developer Friendly
- Clean, documented code
- Modular architecture
- Easy to customize
- Follows Salesforce best practices

### 📱 Mobile First
- Responsive layouts
- Touch-friendly interactions
- Salesforce Mobile App compatible
- Progressive enhancement

---

## 📋 Prerequisites

- **Salesforce Org**: API version 64.0 or higher
- **Salesforce CLI**: Latest version
- **VS Code**: With Salesforce Extensions installed (recommended)
- **Node.js**: v18+ (for local development)
- **Agentforce**: Enabled in your org
- **Permissions**: System Administrator or Developer profile

### Enable Agentforce

1. Go to **Setup** > **Agentforce**
2. Enable **Agentforce**
3. Verify **Custom Lightning Types** are available

---

## 📚 Learning Resources

### Official Documentation
- [Custom Lightning Types Guide](https://developer.salesforce.com/docs/einstein/genai/guide/lightning-types-custom.html)
- [Agentforce Documentation](https://developer.salesforce.com/docs/einstein/genai/guide/get-started-agents.html)
- [Lightning Web Components Guide](https://developer.salesforce.com/docs/component-library/documentation/en/lwc)
- [LightningTypeBundle Metadata](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/)

### Component Library
- [Lightning Design System](https://www.lightningdesignsystem.com/)
- [Lightning Component Library](https://developer.salesforce.com/docs/component-library/overview/components)

### Apex Development
- [Apex Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/)
- [Invocable Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotation_InvocableMethod.htm)

---

## 🛠️ Troubleshooting

### Common Issues

**Lightning Types not showing in Agentforce:**
1. Verify API version is 64.0+
2. Check deployment status
3. Ensure Agentforce is enabled
4. Verify user permissions

**Components not rendering:**
1. Check browser console for errors
2. Verify all files deployed successfully
3. Check field-level security
4. Review Apex debug logs

**Data not displaying:**
1. Verify `@AuraEnabled` on Apex properties
2. Check `@JsonAccess` annotation
3. Ensure proper data binding in LWC
4. Review SOQL queries

For detailed troubleshooting, see individual component documentation.

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/AmazingFeature`)
3. **Commit** your changes (`git commit -m 'Add some AmazingFeature'`)
4. **Push** to the branch (`git push origin feature/AmazingFeature`)
5. **Open** a Pull Request

### Contribution Guidelines
- Follow Salesforce coding standards
- Include tests for new features
- Update documentation
- Add comments to complex logic

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- **Salesforce** for the amazing Agentforce platform
- **Lightning Design System** team for beautiful UI components
- **Salesforce Developer Community** for inspiration and support

---

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/SalesforceDiariesBySanket/salesforce-custom-lightning-types/issues)
- **Discussions**: [GitHub Discussions](https://github.com/SalesforceDiariesBySanket/salesforce-custom-lightning-types/discussions)
- **Blog**: [Salesforce Diaries by Sanket](https://salesforcediariesbysanket.com)

---

## ⭐ Show Your Support

If you find this project helpful, please give it a ⭐ on GitHub!

---

## 🗺️ Roadmap

### Future Enhancements
- [ ] Add Contact Lightning Types
- [ ] Add Opportunity Lightning Types
- [ ] Add Case Lightning Types
- [ ] Multi-object relationship displays
- [ ] Advanced filtering with multiple criteria
- [ ] Chart and graph visualizations
- [ ] Export to CSV/Excel functionality
- [ ] Bulk operations support
- [ ] Additional mobile-specific optimizations

---

## 📊 Stats

- **5** Custom Lightning Types
- **15+** Apex Classes
- **7** Lightning Web Components
- **100%** SLDS Compliant
- **Production Ready** ✅

---

**Made with ❤️ by Salesforce Diaries by Sanket**

*Happy Coding! 🚀*
