# Rapid POS Avalara Connector v2.1.19 Release Notes

_Release Date: April 9, 2025_

---

## Bug Fixes and Performance Enhancements

### Update to **`USER_SP_AVALARA_TAX_CALC`** Stored Procedure

- The stored procedure has been updated to improve ticket handling where all lines on the ticket are unshipped and to allow for backward compatability with older versions of the connector.

- **Do not process non-deposit tickets where all lines are unshipped.**  
  - This update prevents the procedure from encountering an error when all ticket lines are unshipped.
  - This is necessary because when a user checks **“Final Ticket Balance”** and payment is collected only after the lines are shipped, Counterpoint flags what would normally be a **deposit ticket** (`DEP_ONLY_TKT = Y`) as **not a deposit ticket** (`DEP_ONLY_TKT = N`).

- **Do not process transactions that meet client-defined exception rules in the `EXCLUDE_SQL` configuration table.**  
  - Ensures backward compatibility for clients migrating from older connector versions that rely on exclusions — such as excluding all tickets from an ecommerce store.  
