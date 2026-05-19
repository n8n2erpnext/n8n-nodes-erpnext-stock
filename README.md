# n8n-nodes-erpnext-stock

[![Live tested](https://img.shields.io/badge/live--tested-ERPNext%20v16%20%2F%20n8n%20self--hosted%20%2F%20LXD%20%2F%20API%20v2-171717)](#live-tested-status)
[![Runtime audit](https://img.shields.io/badge/runtime%20audit-0%20vulnerabilities-2E7D5F)](#development)
[![Node checks](https://img.shields.io/badge/n8n%20node%20lint%20%2B%20build-passing-2E7D5F)](#development)
[![Package](https://img.shields.io/badge/package-0.1.0-2490EF)](./package.json)

Community n8n node package for ERPNext/Frappe Stock v15-v16.

This package is part of the `n8n2erpnext` ecosystem. It focuses on real inventory movement, warehouse balances, batch/serial traceability, Bin verification, and Stock Ledger Entry verification.

## Connected Ecosystem Coverage

Stock is the operational center of the `n8n2erpnext` ERP ecosystem. The live workflow suite proves that Buying, Stock, Selling, and Accounting work together as one business system, not as separate community nodes.

Two end-to-end enterprise workflows are included and live-tested:

- Standard Product Lifecycle: Supplier -> Purchase Receipt -> inventory increase -> Purchase Invoice -> Customer Sale -> Sales Invoice `update_stock = 1` -> inventory decrease -> final Bin/SLE/accounting verification.
- Exception / After-Sales Lifecycle: sale -> return credit note -> warranty warehouse -> defective warehouse -> repair/virtual workshop -> disposal -> final Bin/SLE/accounting verification.

The suite also verifies cross-module document protection:

- Linked Purchase Invoice blocks unsafe Purchase Receipt cancellation.
- Linked Sales Invoice blocks unsafe Delivery Note cancellation.
- Non-stock fee/rebate invoices do not create Stock Ledger Entry rows.

This positions the project as ERPNext operational inventory workflow validation focused on Retail, Distribution, FMCG/Fresh basics, and Manufacturing basics.

## Live-Tested Status

This package has been live-tested end to end on the project ERPNext/Frappe test environment:

| Area | Status |
| --- | --- |
| ERPNext/Frappe target | Live-tested on ERPNext v16/Frappe v16 behavior |
| n8n runtime | Live-tested on self-hosted n8n `2.20.7-exp.0` |
| Infrastructure | Live-tested through LXD ERPNext container at `http://10.192.135.2:8001` with host header `erp.thaiduy.digital` |
| API coverage | Live-tested with Frappe API v1 read workflows and API v2 document workflows |
| Ecosystem coverage | Buying -> Stock -> Selling -> Accounting end-to-end workflows |
| Core stock lifecycle | Material Receipt, Material Transfer, Stock Reconciliation, Delivery Note, Bin, and Stock Ledger Entry |
| Retail and after-sales coverage | Sale, return credit note, warranty, defective stock, repair/virtual workshop, disposal, exchange, and return fee |
| Manufacturing coverage | BOM, Work Order, WIP transfer, Manufacture Stock Entry, Finished Goods, and finished-goods sale |
| FMCG/Fresh coverage | Batch expiry, batch sale, spoilage/damage, vendor claim warehouse, claim closure, and non-stock fee/rebate |
| Negative coverage | Over-issue, wrong warehouse, disabled item, missing warehouse, duplicate Batch/Serial, linked-document cancel locks |
| Security response policy | Public webhook responses were allowlisted summaries; `securityFindings: []` |
| Cleanup | Temporary workflows were deactivated and verified as `404 Active version not found` |

The live verification used traceable demo records in the ERPNext LXD test instance. The README intentionally includes test infrastructure routing values and document IDs, but no API keys, API secrets, Authorization headers, database passwords, npm tokens, or credential material.

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

## Webhook From n8n to ERPNext Stock

Use this pattern when you want an HTTP endpoint in n8n that reads or writes Stock data in ERPNext.

```text
Client / Browser / App / BI Tool
  -> n8n webhook URL
  -> ERPNext Stock node
  -> GET/POST /api/resource or /api/v2/document
  -> ERPNext Stock DocType
  -> JSON response or safe summary
```

### 1. Configure the ERPNext Credential

In n8n, create or edit an `ERPNext API` credential:

- Site URL: `http://erpnext.internal:8001`
- Site Host Header: `erp.example.com`
- API Key: your ERPNext API key
- API Secret: your ERPNext API secret
- Ignore SSL Issues: `false`

For the current VPS/LXD production setup:

```text
Site URL: http://10.192.135.2:8001
Site Host Header: erp.thaiduy.digital
```

### 2. Create a Read Workflow

Create a workflow with these nodes:

```text
GET Webhook -> ERPNext Stock
```

Webhook node:

- HTTP Method: `GET`
- Path: `erpnext-stock-get-warehouses`
- Respond: `When Last Node Finishes`
- Response Data: `All Entries`

ERPNext Stock node:

- Credential: your `ERPNext API` credential
- Resource: `Warehouse`
- Operation: `Get Many`
- API Version: `v1`
- Fields: `name,warehouse_name,parent_warehouse,company,is_group,disabled`
- Filters JSON: `[]`
- Return All: `false`
- Limit: `20`
- Order By: `modified desc`

### 3. Activate and Test

Activate the workflow, then call:

```bash
curl -i https://n8n.example.com/webhook/erpnext-stock-get-warehouses
```

On the local VPS, you can test without going through the public proxy:

```bash
curl -i http://127.0.0.1:5678/webhook/erpnext-stock-get-warehouses
```

The working workflow artifact is included in this repository:

```text
n8n-webhook-erpnext-stock-get-warehouses.workflow.json
```

## Webhook From ERPNext v16 to n8n

Use this pattern when ERPNext should call n8n automatically after a Stock document is created, submitted, cancelled, or updated. For example, ERPNext can call a n8n workflow whenever a `Stock Entry`, `Delivery Note`, `Batch`, or `Serial No` changes.

```text
ERPNext Doc Event
  -> Frappe Webhook
  -> POST n8n webhook URL
  -> n8n workflow
  -> validation, notification, sync, audit, replenishment, or downstream automation
```

### 1. Create The n8n Webhook Receiver

Create a workflow in n8n with a Webhook trigger:

```text
Webhook -> your processing nodes
```

Webhook node:

- HTTP Method: `POST`
- Path: `erpnext-stock-event`
- Authentication: `None` for a private/internal test, or `Header Auth` for production
- Respond: `Immediately` or `When Last Node Finishes`

The production webhook URL will look like:

```text
https://n8n.example.com/webhook/erpnext-stock-event
```

On this VPS, if ERPNext and n8n are on the same host/network, you can also use an internal n8n URL from ERPNext.

### 2. Add The Webhook In ERPNext/Frappe v16

In ERPNext/Frappe Desk:

1. Open the global search bar.
2. Search for `Webhook`.
3. Open `Webhook` from the Integrations area.
4. Click `New`.

Configure the Webhook:

- Enabled: checked
- Webhook Doctype: `Stock Entry`, `Delivery Note`, `Batch`, `Serial No`, or another Stock DocType
- Doc Event: `on_submit`, `on_cancel`, `after_insert`, or `on_update` depending on the workflow
- Request URL: your n8n production webhook URL
- Request Method: `POST`
- Request Structure: `JSON`
- Webhook JSON: use an allowlisted body like the example below

Example JSON body:

```json
{
  "event": "stock_entry_submitted",
  "doctype": "{{ doc.doctype }}",
  "name": "{{ doc.name }}",
  "company": "{{ doc.company }}",
  "stock_entry_type": "{{ doc.stock_entry_type }}",
  "posting_date": "{{ doc.posting_date }}",
  "from_warehouse": "{{ doc.from_warehouse }}",
  "to_warehouse": "{{ doc.to_warehouse }}",
  "docstatus": "{{ doc.docstatus }}",
  "modified": "{{ doc.modified }}"
}
```

For production, add a shared secret header and validate it in n8n:

```text
X-ERPNext-Webhook-Secret: your-long-random-secret
```

If you use Frappe's Webhook Secret field, Frappe adds an `X-Frappe-Webhook-Signature` header generated from the payload and secret. You can verify this signature in n8n with a Code node if needed.

Official Frappe reference:

- [Frappe Webhooks](https://docs.frappe.io/framework/user/en/guides/integration/webhooks)

## Tested Workflow Artifacts

The repository includes workflow artifacts used during live ERPNext LXD testing.

Read-only artifact:

```text
n8n-webhook-erpnext-stock-get-warehouses.workflow.json
```

Core Stock lifecycle artifacts:

```text
n8n-webhook-erpnext-stock-v2-inventory-lifecycle-test.workflow.json
n8n-webhook-erpnext-stock-v2-reconciliation-cancel-test.workflow.json
n8n-webhook-erpnext-stock-v2-delivery-note-test.workflow.json
n8n-webhook-erpnext-stock-v2-batch-test.workflow.json
n8n-webhook-erpnext-stock-v2-serial-test.workflow.json
n8n-webhook-erpnext-stock-v2-negative-cases-test.workflow.json
```

Cross-module and operational artifacts:

```text
n8n-webhook-erpnext-stock-v2-linked-invoice-dn-cancel-test.workflow.json
n8n-webhook-erpnext-stock-v2-buying-receipt-invoice-lock-test.workflow.json
n8n-webhook-erpnext-stock-v2-repack-manufacturing-test.workflow.json
n8n-webhook-erpnext-stock-v2-retail-return-warranty-test.workflow.json
n8n-webhook-erpnext-stock-v2-retail-exchange-fee-test.workflow.json
n8n-webhook-erpnext-stock-v2-manufacturing-basic-test.workflow.json
n8n-webhook-erpnext-stock-v2-fmcg-fresh-basic-test.workflow.json
```

Enterprise E2E artifacts:

```text
n8n-webhook-erpnext-stock-v2-e2e-standard-product-lifecycle-test.workflow.json
n8n-webhook-erpnext-stock-v2-e2e-exception-return-warranty-lifecycle-test.workflow.json
```

Stock write-test workflows create real demo/test documents in the ERPNext LXD test instance. They should be activated only during testing and deactivated after verification unless a trusted operator intentionally keeps them active.

## Enterprise E2E And Stock Lifecycle Tests

Stock write tests are intentionally shipped as inactive workflow artifacts. Import and review them before activation because they create submitted inventory, selling, buying, and accounting documents.

Standard Product Lifecycle artifact:

```text
n8n-webhook-erpnext-stock-v2-e2e-standard-product-lifecycle-test.workflow.json
```

Standard lifecycle shape:

```text
POST Webhook
-> Create Supplier, Customer, Item Group, Warehouse, stock Item
-> Create and submit Purchase Receipt
-> Verify Bin and Stock Ledger Entry increase
-> Create and submit linked Purchase Invoice
-> Verify Purchase Receipt cancel is blocked by the linked invoice
-> Create and submit Sales Invoice with update_stock = 1
-> Verify Bin decrease and Stock Ledger Entry negative movement
-> Return safe allowlisted summary
```

Verified result:

```text
Run ID: N8N-STOCK-E2E-STD-1779149074499
Status: passed
Purchase Receipt: MAT-PRE-2026-00005
Purchase Invoice: ACC-PINV-2026-00004
Sales Invoice: ACC-SINV-2026-00028
Purchased quantity: 5
Sold quantity: 2
Final Bin actual_qty: 3
Security findings: []
```

Exception / After-Sales Lifecycle artifact:

```text
n8n-webhook-erpnext-stock-v2-e2e-exception-return-warranty-lifecycle-test.workflow.json
```

Exception lifecycle shape:

```text
POST Webhook
-> Create Customer, Item Group, sellable/warranty/defective/repair Warehouses, stock Item
-> Create and submit Material Receipt
-> Create and submit Sales Invoice with update_stock = 1
-> Create and submit return Credit Note into warranty warehouse
-> Move stock warranty -> defective -> repair/virtual workshop
-> Material Issue disposal from repair/virtual workshop
-> Verify final Bin and Stock Ledger Entry rows
-> Return safe allowlisted summary
```

Verified result:

```text
Run ID: N8N-STOCK-E2E-EXC-1779149074522
Status: passed
Sales Invoice: ACC-SINV-2026-00026
Credit Note: ACC-SINV-2026-00027
Receipt Stock Entry: MAT-STE-2026-00062
Disposal Stock Entry: MAT-STE-2026-00065
Sellable final actual_qty: 3
Warranty final actual_qty: 0
Defective final actual_qty: 0
Repair final actual_qty: 0
Security findings: []
```

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
- `Purchase Receipt.docstatus`
- `Purchase Invoice.docstatus`
- `Work Order.status`
- `Work Order.produced_qty`

## Security Baseline

Stock data includes item master data, warehouse balances, batch and serial traceability, delivery documents, manufacturing movement, inventory valuation, and cross-module document links. Treat every workflow as sensitive by default.

Recommended baseline:

- Use a dedicated ERPNext API user for n8n integrations.
- Avoid using a full Administrator API key in production.
- Scope ERPNext roles to the exact DocTypes and actions needed by the workflow.
- Prefer `Get Many` with explicit `Fields` over `Get` when exposing webhook responses, because `Get` can return full documents including comments, owners, child tables, and operational metadata.
- Keep n8n webhook URLs private unless they are meant to be public.
- Add authentication to public n8n webhooks, such as header auth, reverse proxy auth, VPN, IP allowlisting, or a shared secret.
- Do not log API keys, API secrets, Authorization headers, raw upstream error bodies, or full inventory documents into external systems unless there is a clear retention policy.
- Use HTTPS for public traffic.
- If n8n and ERPNext are on the same VPS or private network, prefer the internal ERPNext URL plus `Site Host Header`.
- Rotate API keys after testing, after staff changes, and after any suspected exposure.
- Review n8n execution data retention. Disable or reduce saved execution data for workflows that process stock movement, batch/serial data, delivery notes, manufacturing entries, or cross-module invoice links.
- Deactivate temporary write-test workflows after verification.

## Security Notice

This package pins transitive `form-data` resolution with an npm `overrides` entry so `npm audit --omit=dev` reports no known vulnerabilities at release time. Keep this override in place until upstream dependencies resolve to a safe version without assistance.

In this package's tested deployment model, security risk is also reduced by:

- Internal network access between n8n and ERPNext.
- Reverse proxy or VPN controls for public endpoints.
- Dedicated ERPNext API credentials with scoped roles.
- Explicit field selection for public webhook responses.
- Safe summary nodes for write-heavy lifecycle tests.
- Avoiding public exposure of generic Custom DocType and Frappe Method workflows.

Do not treat this mitigation as a permanent substitute for dependency maintenance. Re-run `npm audit --omit=dev` before publishing a new package version and upgrade compatible n8n dependencies when the upstream dependency chain allows it without breaking n8n node compatibility.

## Deployment Checklist For SME And Mid-Market Teams

Before going live:

- Confirm ERPNext/Frappe version and choose API `v1` or `v2`.
- Create a dedicated ERPNext integration user.
- Assign only the required Stock roles and permissions.
- Configure n8n credentials with the ERPNext internal URL when available.
- Set `Site Host Header` if ERPNext is served by a named Frappe site.
- Build and install the packed node package into the n8n custom nodes environment.
- Test `Get Many` for each required resource with limited fields.
- Test write operations in a staging or dedicated ERPNext test site before production.
- Review n8n execution data retention and error logging.
- Protect public webhooks with authentication or network controls.
- Keep workflow JSON exports out of public repositories if they contain real URLs, headers, filters, or business logic.
- Validate company, warehouse, item group, item flags, and account names before importing workflow artifacts across companies.

Suggested production approach:

- Start with read-only warehouse, Bin, and Stock Ledger Entry reporting workflows.
- Add stock movement workflows only after role permissions, approval paths, and audit logs are reviewed.
- Add cross-module workflows only after Buying, Selling, and Accounting document locks are understood.
- Keep Custom DocType and Frappe Method workflows limited to trusted internal operators.
- Document each production workflow owner, purpose, data fields, stock impact, ledger impact, and rollback path.

## Troubleshooting

Common checks:

- `401` or `403`: verify API key, API secret, user roles, and DocType permissions in ERPNext.
- TLS `EPROTO` or `tlsv1 alert internal error`: use the internal ERPNext HTTP URL from n8n when the public domain is protected by a reverse proxy or VPN layer.
- Empty `[]` response: the node is working, but filters may not match any records.
- Frappe site not found or wrong site: set `Site Host Header` to the public ERPNext site name.
- n8n webhook does not run: activate the workflow and use the production `/webhook/` URL, not `/webhook-test/`.
- Unexpected sensitive fields in output: switch from `Get` to `Get Many` and set an explicit `Fields` list.
- Stock Entry submit fails: verify item flags, source/target warehouse, quantity, valuation rate, posting date, and company.
- Batch or Serial No validation fails: verify the item has batch/serial tracking enabled and use ERPNext v16 Serial and Batch Bundle behavior where required.
- Delivery Note or Purchase Receipt cancel fails: check whether a linked Sales Invoice or Purchase Invoice has already been submitted.
- Work Order or Manufacture submit fails: verify BOM, WIP warehouse, Finished Goods warehouse, and `fg_completed_qty`.
- Stock Ledger Entry does not appear: confirm the source document is submitted; drafts should not create Stock Ledger rows.
- Bin quantity looks wrong: check warehouse filters, cancelled SLE rows, and whether the item is batch/serial controlled.

## Development

```bash
npm install
npm run build
npm run lint
npm audit --omit=dev
npx @n8n/node-cli lint
npx @n8n/node-cli build
npm pack --dry-run
```

For local n8n testing, link this package into your n8n custom nodes directory or install it from a packed tarball.

Expected runtime dependency policy:

- `n8n-workflow` stays in `devDependencies` for local TypeScript/lint/build tooling.
- `n8n-workflow` is declared in `peerDependencies` so the host n8n instance provides it at runtime.
- `form-data` is pinned through `overrides` to avoid vulnerable transitive versions.

Useful n8n references:

- [n8n community nodes installation](https://docs.n8n.io/integrations/community-nodes/installation/)
- [Install community nodes from the n8n GUI](https://docs.n8n.io/integrations/community-nodes/installation/gui-install/)
- [Manual community node installation](https://docs.n8n.io/integrations/community-nodes/installation/manual-install/)
- [Using community nodes](https://docs.n8n.io/integrations/community-nodes/usage/)
- [Creating n8n nodes](https://docs.n8n.io/integrations/creating-nodes/)
- [Using the n8n-node tool](https://docs.n8n.io/integrations/creating-nodes/build/n8n-node/)
- [n8n node linter](https://docs.n8n.io/integrations/creating-nodes/test/node-linter/)
- [Submit community nodes](https://docs.n8n.io/integrations/creating-nodes/deploy/submit-community-nodes/)

## Scope And Roadmap

This package is intentionally Stock-focused. Other ERPNext modules should live in separate packages so each module can evolve independently:

- `n8n-nodes-erpnext-hrms`
- `n8n-nodes-erpnext-accounting`
- `n8n-nodes-erpnext-buying`
- `n8n-nodes-erpnext-selling`

Recommended positioning:

```text
ERPNext operational inventory workflow validation focused on Retail, Distribution, FMCG/Fresh basics, and Manufacturing basics.
```

Recommended next hardening tasks before wider public adoption:

- Add automated unit tests for request construction and API v1/v2 endpoint behavior.
- Add credential redaction checks around error messages.
- Add sample workflows that use limited fields by default.
- Split operational documentation into `TESTING.md`, `WORKFLOWS.md`, and focused docs under `docs/`.
- Add a production security checklist to release notes for every package version.
- Add focused workflows for more advanced manufacturing, subcontracting, and multi-company inter-company stock movement if the ERPNext site is configured for those scenarios.

## Official References

Frappe / ERPNext:

- [ERPNext introduction](https://docs.frappe.io/erpnext)
- [ERPNext Stock](https://docs.frappe.io/erpnext/user/manual/en/stock)
- [Item](https://docs.frappe.io/erpnext/user/manual/en/item)
- [Warehouse](https://docs.frappe.io/erpnext/user/manual/en/warehouse)
- [Stock Entry](https://docs.frappe.io/erpnext/user/manual/en/stock-entry)
- [Stock Reconciliation](https://docs.frappe.io/erpnext/user/manual/en/stock-reconciliation)
- [Delivery Note](https://docs.frappe.io/erpnext/user/manual/en/delivery-note)
- [Batch](https://docs.frappe.io/erpnext/user/manual/en/batch)
- [Serial No](https://docs.frappe.io/erpnext/user/manual/en/serial-no)
- [Frappe REST API](https://docs.frappe.io/framework/user/en/api/rest)
- [Frappe token based authentication](https://docs.frappe.io/framework/v15/user/en/guides/integration/rest_api/token_based_authentication)
- [Generate Frappe API key and secret](https://docs.frappe.io/framework/v15/user/en/guides/integration/how_to_setup_token_based_auth)
- [Frappe Webhooks](https://docs.frappe.io/framework/user/en/guides/integration/webhooks)

n8n:

- [n8n integrations and nodes overview](https://docs.n8n.io/integrations/)
- [n8n community nodes installation](https://docs.n8n.io/integrations/community-nodes/installation/)
- [Manual community node installation](https://docs.n8n.io/integrations/community-nodes/installation/manual-install/)
- [Using community nodes](https://docs.n8n.io/integrations/community-nodes/usage/)
- [Creating n8n nodes](https://docs.n8n.io/integrations/creating-nodes/)
- [Submit community nodes](https://docs.n8n.io/integrations/creating-nodes/deploy/submit-community-nodes/)

## License

MIT

## Acknowledgement

Part of the `n8n2erpnext` ecosystem for connecting n8n automation with ERPNext/Frappe business operations.
