# Build Guide: Daily B2C Order Batching in a Business Central Sandbox

How to build the pattern from the [interactive demonstration](../demo/index.html) against real systems, using free trial and development accounts.

## Goal

Run the pattern against real systems: 1 day of Shopify orders becomes 1 Business Central sales order per store, posted by a service identity, safe to rerun.

Target time: 2 to 3 working sessions.

## What you need

| Item | Where to get it | Cost |
|---|---|---|
| Business Central trial with a sandbox environment | Microsoft Dynamics 365 Business Central free trial (sign up with a work account in a tenant you control) | Free trial |
| Shopify development store | Shopify Partners account, then create a development store | Free |
| Power Automate with premium connectors | Power Automate trial or a developer environment in the same tenant | Free trial |
| Microsoft Entra app registration | Entra admin center in the same tenant | Free |

Use a tenant you control for testing. Never build a demonstration inside a client's production tenant.

## Step 1: Prepare Business Central (session 1)

1. In the sandbox, open the CRONUS demo company.
2. Create 1 customer per store, for example 4 customers, one per store: C-SHOP-A, C-SHOP-B, C-SHOP-C, C-SHOP-D.
3. Note 6 existing item numbers to map to Shopify SKUs (or create items 1001 to 1006 to match the demo).
4. Pick a G/L account for shipping income and note its number.

## Step 2: Create the service identity (session 1)

1. In Microsoft Entra, register an app named svc-integration-demo.
2. Add the API permission Dynamics 365 Business Central, application permission API.ReadWrite.All, and grant admin consent.
3. Create a client secret and store it securely (in production this lives in Azure Key Vault).
4. In Business Central, open the Microsoft Entra Applications page, add the app's client ID, set it to Enabled, and assign only the permission sets needed to create sales orders.
5. Confirm you can get a token with the client credentials flow and call:
   `GET https://api.businesscentral.dynamics.com/v2.0/{tenantId}/{environmentName}/api/v2.0/companies`

## Step 3: Prepare Shopify (session 1)

1. In the development store, create products with SKUs HB-1001 to HB-1006.
2. Create 10 to 20 test orders across a single date.
3. Create a custom app for the store with read access to orders (through the Shopify admin or the Shopify Dev Dashboard, whichever your account offers), and note the Admin API access token.
4. Optional: create 2 more development stores to show multi-store behavior, or simulate stores with order tags.

## Step 4: Build the flow (session 2)

Build in a solution, so it can move between environments and into source control.

1. **Trigger:** Recurrence, daily at 02:00. Compute batchDate as yesterday (yyyy-MM-dd).
2. **Store list:** an array variable with each store's code, customer number and Shopify token reference.
3. **Apply to each store:**
   1. **Read orders:** HTTP call to the Shopify Admin API for paid orders created on batchDate.
   2. **Map and validate:** for each line, look up the SKU in a mapping (SKU to Business Central item number). Unmapped lines go to an exceptions array.
   3. **Sum by item:** build salesOrderLines with lineType Item, lineObjectNumber, quantity and unitPrice. Add 1 Account line for the shipping total.
   4. **Rerun check:** HTTP GET to Business Central salesOrders filtered on externalDocumentNumber eq 'STORE-yyyyMMdd'. If a record exists, log "skipped" and continue.
   5. **Create the order:** HTTP POST to `.../companies({companyId})/salesOrders` with customerNumber, externalDocumentNumber, orderDate and salesOrderLines in one body. Authenticate with the app registration (client credentials).
   6. **Log:** append the store, result, order number, totals and exceptions to a run log (a SharePoint list or Dataverse table works for the demo).
4. **Error handling:** wrap each store in a Scope with a "run after failed" branch that logs the failure, so one store's error never stops the others.

## Step 5: Show the validation (session 2 or 3)

1. Build a 1-page Power BI report on the run log: orders read, Shopify total, Business Central total, held amount, status per channel.
2. Add an exceptions table.

## Step 6: Demonstrate it (session 3)

Show these 4 moments in order, in under 15 minutes:

1. Run the flow once: 4 orders created, 1 per store.
2. Run it again for the same date: nothing posted, 4 skips in the log.
3. Add an order with an unmapped SKU and run a new date: the order posts clean and the exception appears.
4. Open the connection: it runs as the app registration, not as you.

## Note

This guide builds a demonstration in sandbox and development accounts. A production deployment adds environment strategy, source control pipelines, monitoring and the organization's own change approval.
