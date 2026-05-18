# n8n-nodes-erpnext-stock

[![Status](https://img.shields.io/badge/status-live%20workflow%20tested-171717)](#tested-operational-coverage)
[![Runtime audit](https://img.shields.io/badge/runtime%20audit-0%20vulnerabilities-2E7D5F)](#development)
[![Node checks](https://img.shields.io/badge/n8n%20node%20lint%20%2B%20build-passing-2E7D5F)](#development)
[![Package](https://img.shields.io/badge/package-0.1.0-2490EF)](./package.json)

Community n8n node package for ERPNext/Frappe Stock v15-v16.

This package is part of the `n8n2erpnext` ecosystem. It focuses on real inventory movement, warehouse balances, batch/serial traceability, Bin verification, and Stock Ledger Entry verification.

## Tested Operational Coverage

Stock is intentionally tested more deeply than Buying and Selling because submitted Stock documents change real inventory balances and ledger rows.

The current live workflow suite validates core Stock, retail, distribution, FMCG/Fresh basics, and Manufacturing basics:

- Core inventory lifecycle: Item Group, parent/child Warehouse, Item, Material Receipt, Material Transfer, Stock Reconciliation, Delivery Note, Bin checks, and Stock Ledger Entry checks.
- Batch and Serial flows: batch receipt/issue, serial receipt/issue, duplicate protection, and serial reuse failure.
- Negative inventory protection: over-issue, wrong warehouse, disabled item, missing warehouse, duplicate Batch, and duplicate Serial No.
- Cross-module integrity: Buying Purchase Receipt/Purchase Invoice lock and Delivery Note cancel blocked by linked Sales Invoice.
- Retail operations: sale, customer return, warranty warehouse, defective warehouse, repair/virtual workshop movement, disposal, exchange, and non-stock return fee.
- Manufacturing basics: BOM, Work Order, WIP transfer, Manufacture Stock Entry, Finished Goods stock, and Sales Invoice `update_stock = 1`.
- FMCG/Fresh basics: batch expiry, batch sale, spoilage/damage issue, vendor claim warehouse movement, claim closure/return-to-supplier style issue, and non-stock fee/rebate invoice with no Stock Ledger Entry.

The scope is operational validation, not enterprise planning. Advanced MRP, capacity planning, workstation scheduling, demand forecasting, dynamic pricing, chain-wide replenishment, and production analytics are intentionally outside this Stock node validation suite.

## Who This Is For

This package is built for teams that run ERPNext Stock and want controlled inventory automations in n8n.

Typical users:

- ERP administrators who maintain ERPNext/Frappe.
- Warehouse, retail, manufacturing, finance, or operations teams that need reliable stock movement workflows.
- Integration teams that want repeatable n8n automations without writing custom Frappe client code for every inventory process.

The node is intentionally conservative: it exposes standard Stock document operations, supports Frappe API v1 and v2, keeps Bin and Stock Ledger Entry read-only, and allows controlled fallback access to custom DocTypes and whitelisted Frappe methods.

## Architecture At A Glance

Read workflow from left to right:

```text
ERPNext / Frappe Stock  <---- API token ---->  n8n ERPNext Stock node  <---- webhook/API ---->  Client / App / Report
```

Common read pattern:

```text
Client
  -> n8n Webhook
  -> ERPNext Stock node
  -> Frappe REST API
  -> ERPNext Stock DocType
  -> filtered JSON response
```

Common inventory lifecycle pattern:

```text
n8n Webhook / Schedule / App Event
  -> validation / mapping / approval logic
  -> ERPNext Stock node
  -> Item, Warehouse, Stock Entry, Stock Reconciliation, Delivery Note, Batch, Serial No, Bin, or Stock Ledger Entry
  -> safe summary response or downstream system
```

Recommended production network pattern:

```text
Public Client
  -> HTTPS reverse proxy / VPN / allowlist
  -> n8n
  -> private network or internal VPS address
  -> ERPNext / Frappe site
```

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

`Custom DocType` is used when a Stock workflow crosses into another ERPNext module, such as `BOM`, `Work Order`, `Sales Invoice`, or `Purchase Invoice`.

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

When building another module, copy the HRMS/Accounting SVG structure and change only the main background fill to that module color.

## Operations

For writable Stock doctypes:

- Create
- Get
- Get Many
- Update
- Delete
- Submit
- Cancel

For read-only verification doctypes:

- Get
- Get Many

For Frappe methods:

- Run Method

## API Versions

The node supports both ERPNext/Frappe document API styles:

- `v1`: `/api/resource/:doctype`
- `v2`: `/api/v2/document/:doctype`

Use `v1` for broad compatibility. Use `v2` when your ERPNext/Frappe v16 environment is ready for the newer document API behavior.

Submit and cancel use the shared `n8n2erpnext` helper rule:

- Submit fetches the latest document and sends `{ doc }` to `frappe.client.submit`.
- Cancel fetches the latest document and sends `{ doctype, name }` to `frappe.client.cancel`.

Reference:

- [Frappe REST API](https://docs.frappe.io/framework/user/en/api/rest)

## Credentials

Create an API key and secret in ERPNext/Frappe, then configure:

- Site URL: `https://erp.example.com`
- Site Host Header, optional: `erp.example.com`
- API Key
- API Secret
- Ignore SSL Issues, optional

The node authenticates with:

```http
Authorization: token api_key:api_secret
```

Credential fields are marked as password fields where appropriate. Do not expose API keys, API secrets, Authorization headers, tokens, or passwords in webhook responses, logs, README examples, or package artifacts.

### Internal URL With Public Host Header

When n8n and ERPNext run on the same VPS, you can point n8n at the internal ERPNext address and still send the public ERPNext host header:

- Site URL: `http://erpnext.internal:8001`
- Site Host Header: `erp.example.com`

This avoids public reverse-proxy authentication while still letting ERPNext receive the expected site host.

For the current VPS/LXD test setup:

```text
Site URL: http://10.192.135.2:8001
Site Host Header: erp.thaiduy.digital
```

This is infrastructure routing information for the project test environment, not credential material. API keys and API secrets are not included in this README.

For production, create a dedicated ERPNext integration user instead of using a daily admin account. Give that user only the roles required for the workflows it runs.

Official Frappe references:

- [Frappe REST API authentication](https://docs.frappe.io/framework/user/en/api/rest)
- [Frappe token based authentication](https://docs.frappe.io/framework/v15/user/en/guides/integration/rest_api/token_based_authentication)
- [Generate Frappe API key and secret](https://docs.frappe.io/framework/v15/user/en/guides/integration/how_to_setup_token_based_auth)

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
    "warehouse": "N8N Target Warehouse - TDD"
  },
  "returnAll": false,
  "limit": 20,
  "orderBy": "modified desc"
}
```

Get Stock Ledger Entries for a voucher:

```json
{
  "resource": "stockLedgerEntry",
  "operation": "getMany",
  "apiVersion": "v1",
  "fields": "name,item_code,warehouse,voucher_type,voucher_no,actual_qty,qty_after_transaction,serial_and_batch_bundle",
  "filtersJson": {
    "voucher_type": "Stock Entry",
    "voucher_no": "MAT-STE-2026-00058"
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
        "t_warehouse": "N8N Target Warehouse - TDD",
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
  "documentName": "MAT-STE-2026-00058"
}
```

Use `Custom DocType` for manufacturing-adjacent documents:

```json
{
  "resource": "customDocType",
  "customDocType": "Work Order",
  "operation": "get",
  "apiVersion": "v2",
  "documentName": "MFG-WO-2026-00002"
}
```

Run a whitelisted Frappe method:

```json
{
  "resource": "frappeMethod",
  "operation": "runMethod",
  "methodName": "frappe.client.get_value",
  "argumentsJson": {
    "doctype": "Bin",
    "filters": {
      "item_code": "N8N-STOCK-ITEM-001",
      "warehouse": "N8N Target Warehouse - TDD"
    },
    "fieldname": ["name", "actual_qty", "stock_value"]
  }
}
```

## Live Workflow Artifacts

The repository includes importable n8n workflow JSON artifacts used for live validation. They are inactive in the repository by default.

- `n8n-webhook-erpnext-stock-get-warehouses.workflow.json`
- `n8n-webhook-erpnext-stock-v2-inventory-lifecycle-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-reconciliation-cancel-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-delivery-note-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-batch-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-serial-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-negative-cases-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-linked-invoice-dn-cancel-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-buying-receipt-invoice-lock-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-repack-manufacturing-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-retail-return-warranty-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-retail-exchange-fee-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-manufacturing-basic-test.workflow.json`
- `n8n-webhook-erpnext-stock-v2-fmcg-fresh-basic-test.workflow.json`

Public webhook responses for write-heavy tests must be allowlisted summaries and must not expose API keys, API secrets, Authorization headers, tokens, passwords, or raw upstream credential material.

## Verification Fields

Database verification for Stock workflows should record:

- `Bin.actual_qty`
- `Stock Ledger Entry.actual_qty`
- `Stock Ledger Entry.qty_after_transaction`
- `Stock Ledger Entry.serial_and_batch_bundle`
- `Serial and Batch Entry.batch_no`
- `Stock Entry.docstatus`
- `Delivery Note.docstatus`
- `Sales Invoice.docstatus`
- `Sales Invoice.update_stock`
- `Work Order.status`
- `Work Order.produced_qty`

## Documentation Roadmap

The Stock README should stay readable as coverage grows. Future operational detail should move into:

- `TESTING.md`
- `WORKFLOWS.md`
- `docs/retail.md`
- `docs/manufacturing-basic.md`
- `docs/fmcg-fresh.md`
- `docs/coverage-matrix.md`

Recommended positioning:

```text
ERPNext operational inventory workflow validation focused on Retail, Distribution, FMCG/Fresh basics, and Manufacturing basics.
```

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
