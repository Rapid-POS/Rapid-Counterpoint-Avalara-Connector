# Rapid POS Avalara Connector v2.3.00 Release Notes  
**Release Date:** May 5th, 2026  

---

## New Functionality  

### View for Items with Inactive Avalara Tax Codes  
A new view has been added to display items that are currently assigned inactive Avalara Tax Codes.  

This allows users to quickly identify items with invalid tax code mappings and update them accordingly to ensure accurate tax calculation and reporting.  

---

### Void Avalara Transactions When Counterpoint Ticket is Voided  
The connector will now automatically void the corresponding Avalara transaction when a ticket is voided in Counterpoint.  

This ensures that Avalara remains in sync with Counterpoint and that previously reported tax is properly reversed without requiring manual intervention.  

---

### Ship-To Based Avalara Reporting Configuration  
A new Ship-To option has been added to the **Use Avalara For** setting in Store Configuration.  

When enabled, the connector will only report transactions to Avalara that include a Ship-To address.  

This provides greater control over which transactions are reported and supports scenarios where only shipped transactions should be processed through Avalara.  

---

## Bug Fixes and Performance Enhancements  

### Filter Active Tax Codes in Avalara Fields  
Updated the Customer and Item Avalara Tax Code fields to display only active tax codes.  

This prevents users from selecting inactive tax codes and improves data accuracy and consistency in tax reporting.  

---
