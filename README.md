# n8n-nodes-erpnext-stock

Planned n8n community node package for ERPNext Stock.

Planned resources:

- Item
- Item Group
- Warehouse
- Stock Entry
- Stock Reconciliation
- Delivery Note
- Batch
- Serial No

## Node Identity

All `n8n2erpnext` module nodes use the same ERPNext-style logo shape. Each module changes only the main background color.

Stock uses Frappe black `#171717` because the module represents warehouses, items, batches, serial numbers, and inventory movement, and should visually align with the Frappe brand.

Full module color map:

| Module | Color | Hex |
| --- | --- | --- |
| Core | ERPNext blue | `#2490EF` |
| HRMS | People green | `#2E7D5F` |
| Accounting | Finance orange-red | `#D94A2B` |
| Buying | Procurement amber | `#C47F00` |
| Selling | Commerce teal | `#00A6A6` |
| Stock | Frappe black | `#171717` |

When building this module, copy the HRMS/Accounting SVG structure and change only the main background fill to `#171717`.
