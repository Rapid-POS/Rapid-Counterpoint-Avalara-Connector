# Rapid POS Avalara Connector version 2.02.00 Release Notes

_Release Date: February 18th, 2026_

---

## New Functionality

### Marketplace Location Code Support

-  Many online marketplaces, such as eBay and GunBroker, collect and remit sales tax on behalf of the seller. Because tax is handled directly by the marketplace, these transactions do not require real-time tax calculation through Avalara. However, the gross sales amounts from these marketplace transactions still contribute toward economic nexus thresholds. For this reason, completed marketplace sales must often be recorded in Avalara — without calculating tax — so that Avalara can include them in nexus determination and reporting.

- v2.02.00 now supports this requirement through **Marketplace Location Codes**

