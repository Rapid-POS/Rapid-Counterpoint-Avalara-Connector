# Rapid POS Avalara Connector - Version X.X.X
Updated 11/14/2025

---

## Overview

Avalara is a cloud-based solution designed to simplify tax compliance by automating transaction tax calculations and filing. It uses real-time tax data from over 13,000 U.S. sales and use tax jurisdictions (as of 2025), ensuring your taxes are calculated based on the most current rules.

---

## Minimum System Requirements:
- Minimum Counterpoint version: **8.5.6.2**
- Minimum SQL Server version: **2016**

If you would like the Avalara connector but your system does not meet these minimum requirements, please consult your Care Team Lead (vCIO) for an upgrade quote.

---

## Table of Contents

## Table of Contents

- [Minimum System Requirements](#minimum-system-requirements)
- [SECTION 1: How the Avalara Connector Works (API Overview)](#section-1-how-the-avalara-connector-works-api-overview)
- [SECTION 2: How Avalara Determines Tax (Core Inputs and Updates)](#section-2-how-avalara-determines-tax-core-inputs-and-updates)
- [SECTION 3: Ship-To Address (Document Header-Level Information)](#section-3-ship-to-address-document-header-level-information)
- [SECTION 4: Avalara Tax Codes (Line Item and Miscellaneous Charge Classification)](#section-4-avalara-tax-codes-line-item-and-miscellaneous-charge-classification)
- [SECTION 5: Avalara Entity Use Codes (Customer Classification)](#section-5-avalara-entity-use-codes-customer-classification)
- [SECTION 6: Avalara Connector Configuration](#section-6-avalara-connector-configuration)
  - [Confirm Avalara Tax Authority and Tax Code](#confirm-that-the-avalara-tax-authority-and-avalara-tax-code-are-properly-defined-in-counterpoint)
  - [Configure Custom Store Settings](#configure-custom-store-settings-for-the-avalara-connector)
  - [Enable Avalara Tax Calculation for a Store](#enable-avalara-tax-calculation-for-a-store)
  - [Enable Avalara for Additional Stores](#enable-avalara-tax-calculation-for-additional-stores)
- [SECTION 7: Viewing Transactions in an Avalara Account](#section-7-viewing-transactions-in-an-avalara-account)
- [SECTION 8: Ecommerce Orders and Avalara Tax Calculation](#section-8-ecommerce-orders-and-avalara-tax-calculation)
- [SECTION 9: Marketplace Location Code Support (eBay, GunBroker, and Other Marketplaces)](#section-9-marketplace-location-code-support-ebay-gunbroker-and-other-marketplaces)
- [Conclusion](#conclusion)

---

## SECTION 1: How the Avalara Connector Works (API Overview)

Rapid’s Counterpoint-to-Avalara Connector communicates directly with Avalara’s AvaTax API to provide accurate, real-time sales tax calculation. Each transaction follows a defined sequence of API interactions that ensure taxes are evaluated according to current jurisdictional rules.

### 1. Transaction Initialization  
When a ticket or order is created in Counterpoint, the connector prepares the transaction by gathering header information, item lines, miscellaneous charges, and customer details. This data is transmitted to Avalara through an API call.

### 2. Real-Time Tax Calculation  
Avalara evaluates the submitted data using its internal tax rules, jurisdiction mappings, product classifications, and exemption logic. Avalara then returns the calculated tax amount.

### 3. Document Updates  
If the Counterpoint document changes (items added, removed, or modified), the connector resubmits the updated data to Avalara. Avalara recalculates the tax and returns an updated tax amount.

### 4. Transaction Commit (Completed Ticket)  
When a ticket is completed in Counterpoint, the final tax data is reported back to Avalara. Avalara stores the final transaction in its reporting system.  

---

## SECTION 2: How Avalara Determines Tax (Core Inputs and Updates)

Avalara’s calculation engine evaluates each transaction using three essential inputs:

1. **Destination Address** (Ship-To Address, or Store Address when no Ship-To is provided)  
2. **Avalara Tax Code** (assigned to each item or miscellaneous charge)  
3. **Avalara Entity Use Code** (assigned to the customer to represent exemption status)

These inputs determine the tax outcome for every transaction. Avalara considers **where** the product ships, **what** is being sold, and **who** is purchasing the product. Together, these elements form the foundation of Avalara’s calculation logic and ensure jurisdictionally accurate tax results.

### Nightly Updates from Avalara

Counterpoint receives a **nightly refresh** of the available:

- **Item Avalara Tax Codes**  
- **Customer Avalara Entity Use Codes**

This process updates the lists of codes published by Avalara, ensuring that the most current product classifications and exemption types are available for selection in Counterpoint.

**Important:**  
- The nightly update refreshes only the **lists of available codes**.  
- It does **not** modify, remove, or replace any existing Item Tax Code or Entity Use Code assignments already stored in Counterpoint. All previously assigned codes remain unchanged unless manually updated by the user.

---

## SECTION 3: Ship-To Address (Document Header-Level Information)

Avalara relies on precise geographical information to determine which state, county, city, and special jurisdiction taxes apply. A full destination address should include:

- Street Address  
- City  
- State  
- ZIP Code  

### Why the Address Matters  
ZIP Code alone is not sufficient for accurate tax calculation. Many ZIP Codes span multiple tax jurisdictions. Avalara uses geolocation to map the full address to the correct jurisdiction.

### Address Fallback  
If no ship-to address is provided:

- The **store address** is used as the destination  
- Taxes are calculated based on the store’s physical location  

---

## SECTION 4: Avalara Tax Codes (Line Item and Miscellaneous Charge Classification)

Avalara does not rely on simple taxable or exempt flags for items. Instead, Avalara determines taxability in part based on the type of item (or miscellaneous charge). The **Avalara Tax Code** identifies the item type so that Avalara can calculate taxes accurately.

### Assigning Tax Codes in Counterpoint  
- **Items:** Assigned on the item record’s Custom tab  
- **Miscellaneous Charges:** Assigned on the Store record’s Custom tab  

### Default Behavior  

If no Avalara Tax Code is assigned to an **item**:
- Avalara applies the default code **`P0000000`** for **tangible personal property (TPP)**  
- Standard jurisdictional tax rules apply  

If no Avalara Tax Code is assigned to a **miscellaneous charge**:
- No information for that MISC charge is transmitted to Avalara
  
### **Item Tax Code Example 1**

- **Avalara Tax Code:** `PC040110` – Clothing and Related Products (Business-to-Customer) – Boots  
- **Location:** Rhode Island  
- **Item Price:** $125  

**Outcome in Rhode Island:**  
- The item is **tax exempt** at this price point.  
- If the price exceeds **$250**, the item becomes **taxable** under Rhode Island rules.

**Outcome in South Carolina:**  
- The item is **generally taxable** for most of the year.  
- It becomes **tax exempt** during the **back-to-school tax holiday** held the first weekend of August.

### **Item Tax Code Example 2**

- **Avalara Tax Code:** `PF050000` – Food and Food Ingredients (Non-Prepared Foods) – Sold by Qualified Food Retailer  
- **Location:** Pennsylvania  

**Outcome in Pennsylvania:**  
- The item is **generally not taxed**.

**Outcome in Missouri:**  
- The item is **taxable**, typically at a **reduced rate** compared to general merchandise.

### Help Selecting Avalara Tax Codes  

Selecting the correct Avalara Tax Code is essential because it works alongside the destination address and the customer's Entity Use Code to determine the final tax result. These three inputs form the foundation of Avalara’s calculation logic, and accurate tax codes result in a reliable and compliant tax calculation.  

For detailed guidance on selecting the appropriate Avalara Tax Codes, refer to:  
https://knowledge.avalara.com/bundle/dqa1657870670369_dqa1657870670369/page/Avalara_tax_codes.html

---

## SECTION 5: Avalara Entity Use Codes (Customer Classification)
  
Avalara does not rely solely on simple taxable or exempt flags for customers. Instead, Avalara determines taxability in part based on the customer’s tax-exempt reason. The **Avalara Entity Use Code** identifies the customer's tax-exempt reason so that Avalara can calculate taxes accurately.

### Assigning Entity Use Codes in Counterpoint  
Entity Use Codes are assigned on:

- The **Customer record’s Custom tab**

### Behavior When No Entity Use Code Is Assigned  
If the field is left blank:

- The transaction is submitted without exemption information  
- Avalara does not apply customer-based exemptions 

### **Customer Entity Use Code Example 1**

- **Entity Use Code:** `Charitable / Exempt Organization`  
- **Location:** North Carolina  

**Outcome in North Carolina:**  
- Sales to charitable organizations are **not exempt**; the transaction is **taxable**.

**Outcome in Alabama:**  
- Sales to charitable organizations **are exempt**; the transaction is **not taxed**.

### **Customer Entity Use Code Example 2**

- **Entity Use Code:** `Agriculture` (agricultural exemption certificate)  
- **Location:** California  
- **Item Tax Code:** `PA020100` – Agricultural, Commercial Use – Machinery and Equipment  

**Outcome in California:**  
- A **partial tax** is applied due to a **5% reduction** for agricultural customers purchasing qualifying machinery.

**Outcome in Michigan:**  
- The transaction is **fully exempt**.

**Outcome in Hawaii:**  
- The transaction is **fully taxable**.

### Help Selecting Avalara Entity Use Codes

Selecting the correct Avalara Entity Use Code is essential because it works in tandem with the destination address and the item’s Avalara Tax Code to determine the final tax outcome. This code identifies the customer’s exemption status and ensures that Avalara applies the appropriate jurisdictional rules and exemption criteria for each transaction.

For detailed guidance on selecting the appropriate Avalara Entity Use Code, refer to:  
https://knowledge.avalara.com/bundle/dqa1657870670369_dqa1657870670369/page/Exempt_reason_matrix_for_the_U.S._and_Canada_entity_use_code_list.html

---

## SECTION 6: Avalara Connector Configuration

Accurate sales tax calculation requires that both the Avalara Tax Authority and Avalara Tax Code are properly defined in Counterpoint, and that the Avalara configuration is completed correctly within Store Setup. The steps below ensure that tax calculation is performed by the Avalara Connector rather than by Counterpoint’s standard tax functionality.

**Configuration Steps**
1. Confirm That the Avalara Tax Authority and Avalara Tax Code Are Properly Defined in Counterpoint  
2. Configure Custom Store Settings for the Avalara Connector  
  - Fallback Tax Code Setup  
  - Define the Value for “Use Avalara For”  
  - Define the Avalara Tax Code for Each Miscellaneous Charge  
3. Enable Avalara Tax Calculation for a Store  
4. Repeat for additional stores as needed  

### Confirm That the Avalara Tax Authority and Avalara Tax Code Are Properly Defined in Counterpoint

- Tax Authority Setup for Avalara
  - To review the configuration for the Avalara Tax Authority, navigate to: **Setup > System > Tax Authorities**

[IMAGE PLACEHOLDER]

- Tax Code Setup for Avalara
  - To review the configuration for the Avalara Tax Code, navigate to: **Setup > System > Tax Codes**

[IMAGE PLACEHOLDER]

The Avalara Tax Code will later be assigned to each store that should rely on Avalara for tax calculation. A **Fallback Tax Code** must also be configured to support scenarios in which the Avalara Connector is unable to communicate with Avalara, such as during a temporary outage.

### Configure Custom Store Settings for the Avalara Connector

It is recommended to first configure the store’s **Custom** tab. These settings can be entered at any time. The final configuration on the **Main** tab determines when the Avalara Connector becomes active.

Navigate to: **Setup > Point of Sale > Stores > Custom Tab**

The following fields must be configured:

1. Fallback Tax Code  
2. Value for **“Use Avalara For”**  
3. Avalara Tax Code for each miscellaneous charge  

#### 1. Fallback Tax Code Setup

Select a Fallback Tax Code to be used when a connection to Avalara cannot be established.

If no tax codes have been defined yet,
- Navigate to: **Setup > System > Tax Codes**
- Create the required tax code
- Return to:**Setup > Point of Sale > Stores > Custom Tab**
- and assign it to the Fallback Tax Code field

#### 2. Define the Value for “Use Avalara For”

Valid options include:

- **All Transactions** (Recommended)  
  - Ensures that all documents have taxes calculated using Avalara Tax Codes and the customer’s Entity Use Code.  
  - Provides consistent calculation even when store-based rules would otherwise apply.  
  - Transmits all transactions to Avalara, supporting accurate reporting and tax remittance.

- **Out-of-State Transactions**  
  - Counterpoint evaluates the ship-to state and compares it to the store’s state.  
  - **If the ship-to state matches the store’s state:**  
    - The Fallback Tax Code is used.  
    - The transaction is not reported to Avalara.  
  - **If the ship-to state is blank:**  
    - The Fallback Tax Code is used.  
    - The transaction is not reported to Avalara.  
  - **If the ship-to state does not match the store’s state** and the specified tax code is an Avalara Tax Code:  
    - The transaction is transmitted to Avalara.  
    - Avalara calculates and applies the tax and reports the transaction.

- **Ship-To Transactions**  
  - Not currently a valid option.  
  - Planned for future development.  
  - Once implemented, the connector will retrieve tax from Avalara only when the document contains a ship-to address.

#### 3. Define the Avalara Tax Code for Each Miscellaneous Charge

- Assign an Avalara Tax Code to each miscellaneous charge used by the store.  
- This ensures that each charge is sent to Avalara as a document line and included in the tax calculation.

### Enable Avalara Tax Calculation for a Store

Navigate to: **Setup > Point of Sale > Stores > Main Tab**

Within the **Main** tab:
- Set **Use tax code from** to **`Store`**  
- Set **Store tax code** to **`AVALARA`**

These settings ensure that all transactions from the store are sent to Avalara for real-time tax determination.

**Important:**  
The `AVALARA` Tax Code must not be entered until the store is ready for Avalara to begin calculating tax.

Until `AVALARA` is assigned as the Store Tax Code, Counterpoint will continue to calculate tax using its internal tax rules, which may not reflect the most current jurisdictional rates or tax structures.

### Enable Avalara Tax Calculation for Additional Stores

Repeat the steps above for each Counterpoint store that will use the Avalara Connector for tax calculation.

---

## SECTION 7: Viewing Transactions in an Avalara Account

The connector may request tax calculations from Avalara for tickets, orders, or layaways. However, only the **final sale transaction** (the completed ticket) is recorded in the Avalara account.

Deposit tickets are not considered sales. A deposit represents a down payment for a portion of the order total, up to 100 percent. Because a deposit is not a completed sale, deposit tickets are not recorded in Avalara. Only the final completed sale ticket is submitted.

---

## SECTION 8: Ecommerce Orders and Avalara Tax Calculation

Rapid’s Avalara Connector calculates tax for manually entered tickets and orders in Counterpoint. However, it does not calculate tax for orders imported through Rapid’s ecommerce connectors. Each ecommerce platform uses its own tax engine to determine sales tax at checkout, and that tax amount should remain unchanged when the order arrives in Counterpoint.

### How Imported Ecommerce Orders Are Handled in Counterpoint

When an order is imported into Counterpoint from an ecommerce platform:

- The order is automatically assigned to a designated **ecommerce store** in Counterpoint.  
- The tax calculated by the ecommerce platform is preserved exactly as provided.  
- The Avalara Connector does not attempt to recalculate or modify this tax.

This ensures that the tax determined by the website remains consistent on the order.

### Preventing Avalara Tax Calculation on Ecommerce Orders

To ensure that Avalara does not calculate tax for ecommerce transactions:

- **Do not assign an Avalara Tax Code to the ecommerce store.**  
- This prevents the Avalara Connector from attempting to process ecommerce-originated transactions.  
- As a result, ecommerce orders remain excluded from Avalara’s tax calculation and are not sent to Avalara for reporting.

This configuration keeps ecommerce tax logic separate from Counterpoint, avoiding conflicting tax results.

### Example Scenario

A customer places an order on a WooCommerce website. WooCommerce uses its own tax engine to calculate tax at checkout, then stores that tax amount with the order.

When the order is imported into Counterpoint:

1. The order is assigned automatically to the designated ecommerce store.  
2. The tax amount calculated by WooCommerce is preserved.  
3. The ecommerce store is not configured with an Avalara Tax Code, so the Avalara Connector does not calculate or update tax.  
4. The transaction is recorded in Counterpoint exactly as it was calculated on WooCommerce.  
5. The transaction is **not** reported to Avalara.  

This prevents duplicate or conflicting tax calculations and keeps ecommerce activity separate from in-store transactions.

### Reporting Ecommerce Transactions to Avalara

There are two situations where ecommerce transactions **can** be reported to Avalara:

#### **1. Using an Avalara Plugin on the Ecommerce Platform**
If the ecommerce platform supports an Avalara plugin:

- The website can send completed ecommerce transactions directly to Avalara.  
- Avalara calculates tax at checkout.  
- Avalara records the transaction for nexus purposes.  
- Counterpoint simply imports the order without recalculating tax.

This is the recommended method if Avalara reporting is required. Work with your web developer and Avalara account representative to learn more about this setup.

#### **2. Treating the Ecommerce Platform as a Marketplace**
If the ecommerce platform calculates and remits tax on the retailer’s behalf and does not report this to Avalara directly:

- Ecommerce activity may be treated as marketplace activity.  
- A **Marketplace Location Code** can be used to send completed ecommerce sales to Avalara for **record-only** reporting (without tax calculation).  

This ensures that ecommerce sales contribute to economic nexus tracking even if the ecommerce platform handles tax remittance.

### Summary

- Ecommerce platforms calculate tax independently of Avalara.  
- To prevent conflicts, ecommerce stores in Counterpoint must **not** be assigned an Avalara Tax Code.  
- Ecommerce transactions are not reported to Avalara unless the website uses an Avalara plugin or a Marketplace Location Code is configured for record-only reporting.  
- This approach preserves tax accuracy and keeps ecommerce, in-store, and marketplace transactions clearly separated within Avalara.

---

## SECTION 9: Marketplace Location Code Support (eBay, GunBroker, and Other Marketplaces)

Many online marketplaces, such as **eBay** and **GunBroker**, collect and remit sales tax on behalf of the seller. Because tax is handled directly by the marketplace, these transactions do not require real-time tax calculation through Avalara. However, the **gross sales amounts** from these marketplace transactions still contribute toward **economic nexus thresholds**. For this reason, completed marketplace sales must often be recorded in Avalara — **without** calculating tax — so that Avalara can include them in nexus determination and reporting.

The Avalara Connector supports this requirement through **Marketplace Location Codes**.

### How Marketplace Transactions Are Identified in Counterpoint

Each marketplace should have its own **marketplace-specific tax code** defined in Counterpoint, such as `EBAY` or `GUNBROKER`. These tax codes do not perform tax calculation. Instead, they act as identifiers that allow the Avalara Connector to determine:

- which transactions originated from a marketplace, and  
- which **Marketplace Location Code** should be applied during transmission to Avalara.

The marketplace tax code must be assigned correctly to all corresponding transactions in Counterpoint to ensure accurate reporting in Avalara.

### What a Marketplace Location Code Is

A **Marketplace Location Code** is an identifier created within Avalara that represents a specific marketplace (such as eBay or GunBroker). When a completed Counterpoint transaction is sent to Avalara with this code:

- The transaction is recorded for **gross sales tracking**  
- Avalara does **not** calculate tax  
- The sale is included in **economic nexus** evaluation  
- The transaction is grouped and handled separately from in-store and ecommerce sales  

This allows marketplace activity to be reflected in Avalara reporting without affecting the seller’s tax liability.

### Configuring the Marketplace Location Code in Counterpoint

Marketplace configuration is completed within the Tax Code setup. To configure the **Avalara Marketplace Location Code** and **Tax Override for Nexus Calculation**, navigate to:

**Setup > System > Tax Codes > Custom Tab**

For each tax code associated with a marketplace (e.g., `EBAY`, `GUNBROKER`), review the following fields:

- **Avalara Marketplace Location Code**  
  Specifies which Marketplace Location Code from your Avalara account should be attached to transactions recorded with this tax code. This is what allows Avalara to recognize the transaction as marketplace-originated.

- **Use Document for Nexus Calculation**  
  Overrides tax calculation and sets the tax amount to **0**, ensuring that Avalara records the transaction **for nexus reporting only**, without attempting to calculate or adjust tax.

These settings allow Counterpoint to send completed marketplace transactions to Avalara for **record-only transmission**, without triggering tax calculation.

### Summary

Marketplace Location Code functionality ensures that marketplace sales are properly included in Avalara’s reporting and nexus analysis, even though tax is remitted by the marketplace itself.

This functionality enables Avalara to:

- Record marketplace sales without calculating tax  
- Keep marketplace sales distinct from in-store and ecommerce activity  
- Ensure all sales channels contribute to economic nexus thresholds  
- Provide accurate multi-channel sales reporting within a single Avalara account  

This support is especially important when using marketplaces such as eBay and GunBroker, where the marketplace — not the seller — is responsible for tax calculation and remittance.

## Conclusion  

The Rapid Avalara Connector ensures accurate and compliant sales tax calculation across Counterpoint by integrating Avalara’s real-time tax engine with your in-store, ecommerce, and marketplace workflows. With proper configuration of Tax Codes, Entity Use Codes, Marketplace Location Codes, and store settings, each transaction is processed according to the correct jurisdictional rules and reporting requirements.

As your business expands and tax obligations evolve, Avalara provides the flexibility to support additional states, channels, and volume. The Rapid Avalara Connector keeps your Counterpoint environment aligned with these changes to maintain consistent and dependable tax results.

For assistance with configuration changes or troubleshooting, contact Rapid Support. 
