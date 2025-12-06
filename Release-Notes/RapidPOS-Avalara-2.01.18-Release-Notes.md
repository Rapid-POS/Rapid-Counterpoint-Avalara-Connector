# Rapid POS Avalara Connector v2.1.18 Release Notes

_Release Date: March 11, 2025_

---

## New Functionality

### Populate **Reference Code** in Avalara Transactions
- The **Reference Code** column in Avalara Transactions will now be populated with `Counterpoint <Ticket Number>`.
  - This helps users quickly identify the original source and associated ticket number of the transaction.

### Daily Comparison of Counterpoint and Avalara Transactions
- The connector will now compare the **previous day's** Counterpoint transactions to Avalara transactions created by the connector.
  - This comparison runs once daily at **3:00 am**.
  - If unexpected Counterpoint-based transactions are found in Avalara, users will receive an alert in the Counterpoint Message Center in the following format:

>_The following transaction is in Avalara but not in Counterpoint:_  
>Document Code: <DOC_GUID>  
>Reference Code: Counterpoint <TKT_NO>  
>Customer Code: <CUST_NO>  
>Amount: 99,999.99  
>Tax: 999.99  
>  
>_This could occur when there was an error between saving the transaction in Avalara and saving the ticket in Counterpoint._  
>  
>_To resolve, void this transaction in Avalara._
  
- To remediate, void the duplicate transaction in your Avalara account.
- This functionality applies **only** to transactions created by the connector and does not review or affect transactions from other sources, such as ecommerce platforms.  
