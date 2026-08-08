Analyze the given document and write the extracted data to a JSON file.

Save the JSON file in the same directory as the source document.
Use the same base filename as the source document, but replace the original
extension with ".json".

Example:
"invoice-123.pdf" becomes "invoice-123.json"

Use exactly the following JSON structure:

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
Replace all example values with values extracted from the document.

Requirements:

- Preserve all identifiable product lines.
- Check the complete document explicitly for VAT or tax information.
- Use dates in YYYY-MM-DD format.
- Use a period as the decimal separator.
- Format all monetary amounts with exactly two decimal places.
- Do not include currency symbols in amount fields.
- Confidence values must range from 0.00 to 1.00.
- Use a separate confidence value for every item line.
- Do not invent missing information.
- Write valid, directly parseable JSON.
- Do not include Markdown, comments, code fences, or explanatory text in the JSON file.
- After writing the file, read it back and validate that it contains valid JSON.
- If validation fails, correct the file before finishing.

Do not copy placeholder values from the example.

Estimate each confidence value independently based on the actual document.

Confidence guidance:
- 0.99: clearly visible and unambiguous
- 0.90-0.98: highly likely, minor uncertainty
- 0.70-0.89: readable but somewhat uncertain
- 0.40-0.69: weak or partially obscured
- below 0.40: highly uncertain

Do not assign the same confidence score to every field or every line unless they genuinely have the same level of certainty.

After successfully writing and validating the file, respond only with:
Saved: <full path to JSON file>