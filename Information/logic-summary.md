# Detailed Workflow Summary

This file summarizes the document-processing workflow in `app/src`, with emphasis on:

- how uploaded files are processed
- which functions update values
- how the parsed data becomes HTML
- how the HTML is turned into a PDF through the browser print flow

## 1. Entry Point: Upload Or Paste Input

### File
- [app/src/components/DatasheetGeneratorPanel.tsx](/mnt/d/Rohan-App/app/src/components/DatasheetGeneratorPanel.tsx)

### Main functions
- `handleFile(files: FileList | null)`
- `handleParse()`
- `handleClear()`
- `updateHeader(field, value)`
- `updateSectionValue(itemIdx, secIdx, rowIdx, newValue)`
- `selectDrawingForItem(lineItemIdx, drawingId)`
- `loadAvailableDrawings(lineItemIdx)`

### What they do
- `handleFile()` reads the uploaded file, extracts text, and starts the parsing flow.
- `handleParse()` parses pasted text instead of an uploaded file.
- `handleClear()` resets the current document state.
- `updateHeader()` updates a field in the SO header and regenerates the HTML.
- `updateSectionValue()` updates a single parsed section row and regenerates the HTML.
- `selectDrawingForItem()` assigns a chosen GA drawing to a specific line item and regenerates the HTML.
- `loadAvailableDrawings()` loads GA drawing candidates for the currently edited instrument.

### Value updates
- `setSoText(text)` stores extracted or pasted text.
- `setDs(datasheet)` stores the parsed structured SO document.
- `setHtml(html)` stores the generated HTML.
- `setView("input" | "edit" | "preview")` controls the workflow state.
- `setParseLog(...)` records parsing stats and errors.
- `setEditingIdx(...)` chooses which line item is being edited.
- `setAvailableDrawings(...)` stores GA drawing options for the selected instrument.
- `setDrawingSelectorOpen(...)` opens or closes the GA drawing picker.

## 2. File Reading And Text Extraction

### File
- [app/src/data/pdfExtractor.ts](/mnt/d/Rohan-App/app/src/data/pdfExtractor.ts)

### Main functions
- `extractTextFromPDF(file: File)`
- `isPDF(file: File)`
- `isTextFile(file: File)`
- `readFileAsText(file: File)`

### What they do
- `extractTextFromPDF()` opens a PDF with `pdfjs-dist`, reads each page, and builds structured text.
- `isPDF()` checks whether the upload is a PDF by MIME type or filename.
- `isTextFile()` checks whether the upload is a text-based file.
- `readFileAsText()` decides how the file should be read:
  - PDF files go through PDF text extraction
  - text files and CSV files are read with `FileReader`

### Important behavior
- If a PDF has no selectable text, the file is treated as scanned and parsing is rejected.
- This workflow is text-driven, not image-driven.
- The current extractor handles PDF, TXT, and CSV directly.

## 3. Parsing Raw Text Into Structured Data

### File
- [app/src/data/api.ts](/mnt/d/Rohan-App/app/src/data/api.ts)
- [app/src/data/dataMapper.ts](/mnt/d/Rohan-App/app/src/data/dataMapper.ts)

### Main functions
- `generateFromSOText(soText: string)`
- `parseSOText(text: string)`
- `updateLineItemGADrawing(datasheet, lineItemIdx, drawingId)`
- `generateFromDatasheet(datasheet)`

### What they do
- `generateFromSOText()` is the main orchestration function for the workflow.
  - It parses the uploaded text.
  - It enriches each line item with a GA drawing if possible.
  - It generates the final HTML string.
- `parseSOText()` converts raw quotation text into a structured `SODatasheet`.
- `updateLineItemGADrawing()` replaces the GA drawing for one line item and regenerates the HTML.
- `generateFromDatasheet()` regenerates HTML from already-parsed structured data.

### Value updates
- `parseSOText()` builds:
  - `header`
  - `lineItems`
- `toLineItem()` inside `dataMapper.ts` maps parsed instrument data into the `LineItem` shape.
- `buildNotes()` inside `dataMapper.ts` collects instrument notes into the header.
- `generateFromSOText()` updates `gaDrawingUrl` and `gaDrawingTotal` on matching line items.

## 4. GA Drawing Matching

### File
- [app/src/data/gaDrawingLookup.ts](/mnt/d/Rohan-App/app/src/data/gaDrawingLookup.ts)
- [app/src/data/gaDrawingStorage.ts](/mnt/d/Rohan-App/app/src/data/gaDrawingStorage.ts)
- [app/src/data/gaDrawingTypes.ts](/mnt/d/Rohan-App/app/src/data/gaDrawingTypes.ts)

### Main functions
- `findGADrawing(instrumentType, size, decodNo)`
- `getDrawingsForFamily(instrumentType)`
- `getDrawingUrlById(drawingId)`
- `getAllDrawings()`
- `getFileDataUrl(id)`
- `getFileBlob(id)`

### What they do
- `findGADrawing()` searches for the best GA drawing for a line item.
- It uses product-family detection, size matching, and de-codification matching.
- `getDrawingsForFamily()` returns all drawings that belong to the same product family.
- `getDrawingUrlById()` resolves a drawing record into a usable preview URL.
- `getAllDrawings()` and `getFileDataUrl()` are storage-level helpers.

### Value updates
- `findGADrawing()` provides the values that later get written into each line item:
  - `gaDrawingUrl`
  - `gaDrawingTotal`
- `selectDrawingForItem()` in `DatasheetGeneratorPanel.tsx` uses `getDrawingUrlById()` to overwrite the selected line item's drawing URL.

## 5. HTML Generation

### File
- [app/src/data/datasheetTemplate.ts](/mnt/d/Rohan-App/app/src/data/datasheetTemplate.ts)

### Main functions
- `generateDatasheetTemplate()`
- `renderCompactSpecs(item)`
- `renderConnectionTable(item)`
- `renderInstrumentPage(item, seqNo, total)`
- `coverPage(header)`
- `indexPage(header, items)`

### What they do
- `generateDatasheetTemplate()` converts the parsed document into a full HTML datasheet.
- `renderCompactSpecs()` flattens parsed section data into a compact technical table.
- `renderConnectionTable()` adds connection-dimension tables when the line item has process connection data.
- `renderInstrumentPage()` builds the per-instrument page layout.
- `coverPage()` and `indexPage()` build the opening sections of the datasheet.

### Value updates
- These functions do not mutate the parsed model directly.
- They consume:
  - `header`
  - `lineItems`
  - `sections`
  - `processConnection`
  - `gaDrawingUrl`
- Their output is the final HTML string stored in component state.

## 6. Preview And PDF Output

### File
- [app/src/data/api.ts](/mnt/d/Rohan-App/app/src/data/api.ts)
- [app/src/components/DocumentViewer.tsx](/mnt/d/Rohan-App/app/src/components/DocumentViewer.tsx)
- [app/src/components/DatasheetGeneratorPanel.tsx](/mnt/d/Rohan-App/app/src/components/DatasheetGeneratorPanel.tsx)

### Main functions
- `openPrintWindow(html)`
- `downloadHtml(html, soNo)`
- `handlePrint()` in `DocumentViewer.tsx`
- `handleDownload()` in `DocumentViewer.tsx`

### What they do
- `openPrintWindow()` opens a new browser window, writes the HTML into it, and calls `print()`.
- `downloadHtml()` saves the generated HTML as a `.html` file.
- `handlePrint()` prints the current iframe content.
- `handleDownload()` saves the current iframe content as an HTML file.

### How HTML becomes PDF
- The app does not generate PDF bytes directly.
- Instead:
  1. HTML is rendered in the browser.
  2. The browser print dialog opens.
  3. The user chooses Save as PDF.

This means PDF output is produced by the browser print engine, not by a backend PDF library.

## 7. Where Values Change During The Workflow

### File
- [app/src/components/DatasheetGeneratorPanel.tsx](/mnt/d/Rohan-App/app/src/components/DatasheetGeneratorPanel.tsx)

### Functions that update values
- `handleFile()` updates extracted text, parsed data, HTML, and parse logs.
- `handleParse()` updates parsed data, HTML, and parse logs from pasted text.
- `updateHeader()` updates a field in `ds.header` and regenerates HTML.
- `updateSectionValue()` updates a cell inside `ds.lineItems[*].sections[*].rows[*]` and regenerates HTML.
- `selectDrawingForItem()` updates one line item's `gaDrawingUrl` and regenerates HTML.
- `handleClear()` resets all document-related state.

### State variables affected
- `soText`
- `ds`
- `html`
- `view`
- `parseLog`
- `editingIdx`
- `availableDrawings`
- `drawingSelectorOpen`

## 8. End-To-End Flow

1. The user uploads a file or pastes text.
2. `readFileAsText()` extracts raw text from the file.
3. `generateFromSOText()` parses the text and enriches it with GA drawing matches.
4. `parseSOText()` converts the raw text into a structured `SODatasheet`.
5. `generateDatasheetTemplate()` turns the structure into HTML.
6. The HTML is previewed in an iframe.
7. `openPrintWindow()` or `handlePrint()` sends that HTML into the browser print flow.
8. The browser saves the result as PDF.

## 9. Practical Notes

- The workflow is entirely client-side.
- The file picker accepts `.doc` and `.docx`, but the current text extraction path does not directly parse those formats.
- Scanned PDFs are not supported by the current extraction logic because the parser requires selectable text.
- GA drawing selection is optional, but when available it is embedded into the generated HTML automatically.

