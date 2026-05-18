# n8n-nodes-erpnext-stock

[![Status](https://img.shields.io/badge/status-initial%20build%20passing-171717)](#current-status)
[![Runtime audit](https://img.shields.io/badge/runtime%20audit-0%20vulnerabilities-2E7D5F)](#development)
[![Node checks](https://img.shields.io/badge/n8n%20node%20lint%20%2B%20build-passing-2E7D5F)](#development)
[![Package](https://img.shields.io/badge/package-0.1.0-2490EF)](./package.json)

Community n8n node package for ERPNext/Frappe Stock v15-v16.

This package is part of the `n8n2erpnext` ecosystem. It focuses on inventory movement, warehouse balances, batch/serial traceability, and read-only ledger verification.

## Current Status

Initial package scaffold and local verification are complete. Live ERPNext/n8n workflow testing is still pending.

Current verification for `0.1.0`:

```text
npm run lint            passed
npm audit --omit=dev    found 0 vulnerabilities
npm run build           passed
npx @n8n/node-cli lint  passed
npx @n8n/node-cli build passed
```

Stock live testing must be stricter than Buying and Selling because submitted Stock documents affect inventory balances and Stock Ledger Entry rows.

## Supported Resources

- Item
- Item Group
- Warehouse
- Stock Entry
- Stock Reconciliation
- Delivery Note
- Batch
- Serial No
- Bin
- Stock Ledger Entry
- UOM
- UOM Conversion Detail
- Price List
- Item Price
- Material Request
- Custom DocType
- Frappe Method

`Bin` and `Stock Ledger Entry` are intentionally read-only verification resources. They expose only `Get` and `Get Many`.

## Operations

For writable Stock documents:

- Create
- Get
- Get Many
- Update
- Delete
- Submit
- Cancel

For read-only verification documents:

- Get
- Get Many

For Frappe methods:

- Run Method

## API Versions

The node supports both ERPNext/Frappe document API styles:

- `v1`: `/api/resource/:doctype`
- `v2`: `/api/v2/document/:doctype`

Submit and cancel use the shared `n8n2erpnext` helper rule:

- Submit fetches the latest document and sends `{ doc }` to `frappe.client.submit`.
- Cancel fetches the latest document and sends `{ doctype, name }` to `frappe.client.cancel`.

## Credentials

Create an API key and secret in ERPNext/Frappe, then configure:

- Site URL: `https://erp.example.com`
- Site Host Header, optional: `erp.example.com`
- API Key
- API Secret
- Ignore SSL Issues, optional

For the current VPS/LXD test setup:

```text
Site URL: http://10.192.135.2:8001
Site Host Header: erp.thaiduy.digital
```

This is infrastructure routing information for the project test environment, not credential material. API keys and API secrets are not included in this README.

## Stock Test Plan

Core live-test flow:

```text
Item Group
  -> parent Warehouse
  -> child Warehouses
  -> Item with Vietnamese text, spaces, and special characters
  -> Stock Entry: Material Receipt
  -> verify Bin and Stock Ledger Entry
  -> Stock Entry: Material Transfer
  -> verify source and target Warehouse balances
  -> Stock Reconciliation
  -> verify adjusted quantity and valuation
```

Mandatory lifecycle coverage:

- Warehouse with parent warehouse.
- Stock Entry submit and cancel.
- Material Receipt increases `Bin.actual_qty`.
- Material Transfer decreases source warehouse and increases target warehouse.
- Stock Reconciliation adjusts quantity and valuation.
- Delivery Note reduces stock.
- Batch item: create batch, receive by batch, issue the correct batch.
- Serial No item: receive serial numbers, issue serial numbers, and verify reuse fails.

Mandatory negative coverage:

- Issue more than available stock.
- Use a wrong warehouse.
- Use a disabled item.
- Submit Stock Entry with missing warehouse.
- Cancel Delivery Note after it has a linked submitted invoice.
- Create duplicate Batch or Serial No.

Database verification should record:

- `Bin.actual_qty`
- `Stock Ledger Entry.actual_qty`
- `Stock Ledger Entry.qty_after_transaction`
- `Stock Entry.docstatus`
- `Delivery Note.docstatus`

Public webhook responses for write-heavy tests must be allowlisted summaries and must not expose API keys, API secrets, Authorization headers, tokens, passwords, or raw upstream credential material.

## Examples

Get current Bin quantity for an Item/Warehouse:

```json
{
  "resource": "bin",
  "operation": "getMany",
  "apiVersion": "v1",
  "fields": "name,item_code,warehouse,actual_qty,projected_qty,valuation_rate,stock_value",
  "filtersJson": {
    "item_code": "N8N-STOCK-ITEM-001",
    "warehouse": "N8N Target Warehouse - TD"
  },
  "returnAll": false,
  "limit": 20,
  "orderBy": "modified desc"
}
```

Get Stock Ledger Entries for a Stock Entry voucher:

```json
{
  "resource": "stockLedgerEntry",
  "operation": "getMany",
  "apiVersion": "v1",
  "fields": "name,item_code,warehouse,voucher_type,voucher_no,actual_qty,qty_after_transaction,valuation_rate,stock_value",
  "filtersJson": {
    "voucher_type": "Stock Entry",
    "voucher_no": "MAT-STE-2026-00001"
  },
  "returnAll": false,
  "limit": 20,
  "orderBy": "posting_date desc, posting_time desc"
}
```

Create a Material Receipt Stock Entry:

```json
{
  "resource": "stockEntry",
  "operation": "create",
  "apiVersion": "v2",
  "dataJson": {
    "stock_entry_type": "Material Receipt",
    "company": "Your Company",
    "items": [
      {
        "item_code": "N8N-STOCK-ITEM-001",
        "t_warehouse": "N8N Target Warehouse - TD",
        "qty": 10,
        "basic_rate": 25
      }
    ]
  }
}
```

Then submit it with:

```json
{
  "resource": "stockEntry",
  "operation": "submit",
  "apiVersion": "v2",
  "documentName": "MAT-STE-2026-00001"
}
```

## Node Identity

All `n8n2erpnext` module nodes use the same ERPNext-style logo shape. Each module changes only the main background color.

| Module | Color | Hex | Reason |
| --- | --- | --- | --- |
| Core | ERPNext blue | `#2490EF` | Foundation package, closest to the ERPNext brand color. |
| HRMS | People green | `#2E7D5F` | Human operations, employees, attendance, leave, payroll. |
| Accounting | Finance orange-red | `#D94A2B` | Ledger, journals, invoices, financial control. |
| Buying | Procurement amber | `#C47F00` | Purchase flow, suppliers, RFQs, purchase orders, receipts, spend. |
| Selling | Commerce teal | `#00A6A6` | Customer-facing pipeline, quotations, sales orders, revenue. |
| Stock | Frappe black | `#171717` | Warehouses, items, inventory movement; aligned with Frappe black. |

## Development

```bash
npm install
npm run lint
npm audit --omit=dev
npm run build
npx @n8n/node-cli lint
npx @n8n/node-cli build
npm pack --dry-run
```

Expected runtime dependency policy:

- `n8n-workflow` stays in `devDependencies` for local TypeScript/lint/build tooling.
- `n8n-workflow` is declared in `peerDependencies` so the host n8n instance provides it at runtime.
- `form-data` is pinned through `overrides` to avoid vulnerable transitive versions.

## Release Checklist

Before tagging a release:

```bash
npm run lint
npm audit --omit=dev
npm run build
npx @n8n/node-cli lint
npx @n8n/node-cli build
npm pack --dry-run
```

For first publish, follow the ecosystem trusted-publisher/bootstrap pattern if npm returns `E404 Not Found` during the provenance publish:

1. Bootstrap publish `0.1.0` manually if required.
2. Bump to `0.1.1`.
3. Tag `v0.1.1`.
4. Let GitHub Actions publish the provenance-backed recovery version.
