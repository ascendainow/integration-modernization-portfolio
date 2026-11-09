# Flow Logic

Step-by-step logic for the daily order batching flow. Expressions use Power Automate workflow definition language.

## 1. Trigger and batch date

- **Trigger:** Recurrence, daily at 02:00 in the business time zone.
- **batchDate** (Compose):

```
formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')
```

## 2. Channel list

An array variable, 1 entry per channel. Secrets are referenced from Azure Key Vault, never stored in the flow.

```json
[
  { "code": "STOREA", "customerNumber": "C-SHOP-A", "secretName": "shopify-storea-token" },
  { "code": "STOREB", "customerNumber": "C-SHOP-B", "secretName": "shopify-storeb-token" }
]
```

## 3. For each channel (inside a Scope with error handling)

1. **Read orders:** HTTP request to the Shopify Admin API for paid orders processed on `batchDate`.
2. **Map SKUs:** look up each line's SKU in the item mapping. Lines with no match are appended to `exceptions` and excluded from the order.
3. **Sum by item:** group mapped lines by Business Central item number, summing quantity and amount. Unit price = amount / quantity.
4. **Add shipping:** 1 G/L account line for the day's shipping total.
5. **Build the external document number:**

```
concat(items('For_each_channel')?['code'], '-', formatDateTime(variables('batchDate'), 'yyyyMMdd'))
```

6. **Rerun check:** HTTP GET to Business Central:

```
GET {baseUrl}/api/v2.0/companies({companyId})/salesOrders?$filter=externalDocumentNumber eq 'STOREA-20261108'
```

   If the response contains a record, log `skipped` and move to the next channel.

7. **Create the order:** HTTP POST to `{baseUrl}/api/v2.0/companies({companyId})/salesOrders` with the order header and its `salesOrderLines` in 1 request body ([example](../samples/bc-salesorder-payload.json)). Authentication: Microsoft Entra ID OAuth, client credentials, using the organization's app registration.
8. **Log:** write channel, batch date, result, order number, line count, totals and exceptions to the run log.

## 4. Error handling

Each channel runs inside its own Scope. A parallel branch configured to run after **has failed** or **has timed out** logs the failure and continues, so 1 channel's error never stops the others.

## 5. Validation (Power BI)

| Measure | Definition |
|---|---|
| Source total | Sum of Shopify order totals for the channel and day |
| Posted total | Business Central sales order amount for the channel and day |
| Held total | Sum of exception line amounts |
| Status | Matched when Source total = Posted total + Held total; otherwise Mismatch |
