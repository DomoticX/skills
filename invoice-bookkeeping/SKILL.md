---
name: invoice-bookkeeping
description: Extract bookkeeping data from invoices, receipts, bills, and purchase documents into a strict JSON file saved next to the source document. Use this skill when the user asks to analyze, extract, process, convert, or bookkeep an invoice or receipt.
---

# Invoice Bookkeeping Extraction

Analyze the specified invoice, receipt, bill, or purchase document and extract its bookkeeping data.

## Output file

Write exactly one JSON file.

Save the JSON file in the same directory as the source document.

Use the same base filename as the source document and replace only the original extension with `.json`.

Example:

`invoice-123.pdf` → `invoice-123.json`

Do not overwrite or modify the source document.

## PDF document reading

For PDF documents, use the MCP PDF Reader tools whenever they are available.

Preferred tool workflow:

1. Use `read_pdf` / `Read Pdf` from the PDF Reader MCP extension.
2. This tool uses MuPDF and automatically attempts native text extraction.
3. If native text is not usable, allow the PDF Reader MCP to render the document to images.
4. Inspect returned page images using the available image-reading/vision capability.
5. Use `pdf_info` / `Pdf Info` only when document metadata or PDF information is actually needed.

Do NOT use the generic `Pdf Tool` with `extract_text` when the PDF Reader MCP is available.

Do NOT manually parse PDF binary data.

Do NOT dump raw PDF objects, encoded streams, compressed content, or full binary representations into the conversation context.

Do NOT install additional PDF or OCR software when the PDF Reader MCP is available.

If the PDF Reader MCP is not currently enabled but is available as an extension, enable it before processing the PDF.

## JSON structure

Use exactly this structure and these field names:

{
  "ShopName": "",
  "Date": "",
  "Reference": "",
  "TotalAmount": "",
  "VATAmount": "",
  "Confidence": "",
  "Lines": [
    {
      "ItemDescription": "",
      "ItemQuantity": "",
      "ItemPriceSKU": "",
      "ItemPriceTotalAmount": "",
      "ItemConfidence": ""
    }
  ]
}

Do not add, remove, rename, or reorder fields unless the user explicitly requests a different schema.

Replace all placeholder values with values extracted from the actual document.

## Extraction rules

- Preserve all identifiable product or service lines.
- Inspect the complete document explicitly for VAT, BTW, tax, sales tax, tax rates, tax totals, inclusive/exclusive tax statements, or equivalent tax information.
- Do not infer VAT merely from the total amount.
- Do not invent missing information.
- Do not calculate a missing value unless it can be unambiguously derived from values explicitly present in the document.
- If a field cannot be determined reliably from the document, use an empty string `""`.
- Preserve meaningful descriptions as they appear in the document, while removing obvious layout artifacts.
- Do not copy placeholder values from the JSON example.

## Formatting rules

### Dates

Use:

`YYYY-MM-DD`

Example:

`2026-07-01`

### Monetary amounts

- Use a period as the decimal separator.
- Use exactly two decimal places.
- Do not include currency symbols.
- Do not include thousands separators.

Examples:

`12.10`

`1234.50`

### Quantities

Preserve the quantity shown in the document.

Do not invent a quantity when none is identifiable.

### References

Use the invoice number, receipt number, transaction reference, order reference, or equivalent primary document reference when identifiable.

Do not substitute unrelated numbers such as postal codes, customer numbers, telephone numbers, or bank account numbers.

## VAT rules

VATAmount must represent the identifiable VAT/tax amount for the complete document.

Explicitly inspect the document for terms or values such as:

- VAT
- BTW
- Tax
- Sales tax
- Incl.
- Excl.
- VAT amount
- BTW bedrag
- tax amount
- tax rate
- 21%
- 9%
- 0%

If VAT/tax information is present, extract the actual identifiable VAT/tax amount.

If a tax rate is shown but the VAT amount itself is not stated, only calculate VATAmount when the calculation is unambiguous from explicitly stated taxable amounts and tax treatment.

Otherwise use:

`""`

Never use `0.00` merely to represent unknown or missing VAT.

Use `0.00` only when the document explicitly establishes that the applicable VAT/tax amount is zero.

## Confidence scoring

Confidence values must range from `0.00` through `1.00`.

Estimate confidence independently from the actual evidence in the document.

Guidance:

- `0.99` — clearly visible/readable and unambiguous
- `0.90`–`0.98` — highly likely with only minor uncertainty
- `0.70`–`0.89` — readable but somewhat uncertain
- `0.40`–`0.69` — weak, incomplete, or partially obscured
- below `0.40` — highly uncertain

Do not automatically assign `0.95`, `0.99`, or any other fixed value.

Do not assign identical confidence scores to every line unless the evidence genuinely supports the same confidence for each line.

### ItemConfidence

Estimate `ItemConfidence` independently for every item line.

Base it on the certainty of the extracted description, quantity, unit price, and line total for that specific line.

### Confidence

`Confidence` represents the overall confidence in the extracted document-level result.

Consider at least:

- ShopName
- Date
- Reference
- TotalAmount
- VATAmount
- completeness of Lines

Do not calculate the overall Confidence as a simple arithmetic average unless there is a specific reason to do so.

## Line fields

For each identifiable item or service line:

### ItemDescription

Use the identifiable product or service description.

### ItemQuantity

Use the explicitly identifiable quantity.

If no quantity can be identified, use:

`""`

### ItemPriceSKU

Use the identifiable unit price.

If a separate unit price is not identifiable, use:

`""`

### ItemPriceTotalAmount

Use the identifiable total amount for that specific line.

If no separate line total is identifiable, use:

`""`

### ItemConfidence

Estimate confidence for that line independently.

## Validation

Before finishing:

1. Write the JSON file.
2. Read the saved JSON file back from disk.
3. Parse it as JSON.
4. Confirm that it contains exactly the required top-level fields.
5. Confirm that `Lines` is an array.
6. Confirm that every line contains exactly the required line fields.
7. Confirm all known monetary values use exactly two decimal places.
8. Confirm the date uses `YYYY-MM-DD` when a date was identified.
9. Confirm confidence values are between `0.00` and `1.00`.
10. Confirm no Markdown, comments, code fences, or explanatory prose were written into the JSON file.

If validation fails, correct the JSON file and validate it again before finishing.

## Final response

After the JSON file has been successfully written and validated, respond only with:

`Saved: <full path to JSON file>`

Do not include any other explanation.