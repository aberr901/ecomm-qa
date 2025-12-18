# QA Bug Report


## Bug 1: Not Mobile Friendly

- **Impact:**
  - Many relevant users will frequently access the application from mobile devices or tablets. This makes mobile usability critical for daily operations and productivity.

- **Description:**
  - The application UI does not fully adapt to mobile screens. Several elements and spaces fail to resize dynamically, resulting in horizontal scrolling, overlapping text, and fixed-width components that do not fit smaller screens.

- **Steps to Reproduce:**
  1. Open the application (ecomcentral.app) on a mobile device or tablet (see screenshots below).
  2. Navigate through the main dashboard, logistics, and payments sections.
  3. Observe layout issues, such as:
     - Side navigation and content areas not resizing
     - Table columns overflowing or text being cut off
     - Action buttons and filters not adapting to screen width

- **Expected Result:**
  - All UI elements and spaces should resize and adapt dynamically for mobile screens, with no horizontal scrolling or overlapping content.

- **Actual Result:**
  - Some elements remain fixed-width or overflow, causing:
    - Horizontal scrolling
    - Overlapping or truncated text in tables
    - Action buttons and filters not fully visible


- **Evidence (Screenshots):**
  - ![Warehouse Shipments - Mobile](https://camdentrading279-my.sharepoint.com/:i:/g/personal/abraham_vetdist_com/IQD08nypHkvOSZLjGUoe5h1hATNigWSozXYdiWVQzJX3w4Q?e=dNMyb9)
  - ![Logistics Shipments - Mobile](https://camdentrading279-my.sharepoint.com/:i:/g/personal/abraham_vetdist_com/IQAfGKqwkHHlSKm49yl7DELuAdABLi73MS3rPlYotI8761A?e=WqqG54)
  - ![Payments - Mobile](https://camdentrading279-my.sharepoint.com/:i:/g/personal/abraham_vetdist_com/IQAkhDuwu_mNQYc0Kv1ie7l-ASgVBE_-Z9hiQfcEMV3bgtg?e=Npim8O)

  _Screenshots show navigation, tables, and action buttons not resizing or overflowing on mobile/tablet view._

---

## Bug 2: Bulk Import - UPC Data Imported with Scientific Notation

- **Impact:**
  - Critical data integrity issue. UPC codes are being corrupted during the paste operation from Excel, leading to incorrect product identification and duplicate entries in the system.

- **Description:**
  - When pasting product data from Excel into the bulk import feature, UPC values are being converted to scientific notation (e.g., "7.29001E+12" instead of "729001000000"). This formatting issue causes the UPCs to be stored incorrectly and creates duplicate entries with the same malformed UPC value.

- **Steps to Reproduce:**
  1. Prepare Excel data with UPC codes as regular numbers
  2. Copy rows from Excel including headers
  3. Paste into the bulk import "Paste Excel Data" field
  4. Observe that UPC values display with E notation (e.g., "7.29001E+12", "7.87269E+11", "8.71913E+12")
  5. Continue through the import process
  6. Check imported products - UPCs are saved with E notation

- **Expected Result:**
  - UPC values should be extracted from the clipboard as plain text strings, preserving the full numeric value without scientific notation.

- **Actual Result:**
  - UPCs are converted to scientific notation during paste, corrupting the data and creating duplicate malformed entries.

- **Root Cause:**
  - The paste handler is likely reading formatted values from Excel's HTML/rich text clipboard format instead of plain text values. Excel stores large numbers in scientific notation in its internal format.

- **Suggested Fix:**
  - Modify clipboard extraction to use plain text format or parse HTML clipboard data while preserving numeric precision
  - Add pre-validation to detect and flag scientific notation in UPC fields before allowing import to proceed

- **Evidence (Screenshots):**
  - ![Paste Data with E Notation](images/image%20(5).png) - Shows UPC values pasted with scientific notation
  - ![Preview with E Notation](images/image%20(6).png) - Preview table shows UPC column with E notation values

---

## Bug 3: Bulk Import - No Validation for Duplicates or Invalid Data Types

- **Impact:**
  - High severity. Allows corrupted and duplicate data to enter the system without any warnings or prevention, leading to data integrity issues and operational problems.

- **Description:**
  - The bulk import process lacks validation for duplicate UPCs and invalid data types. Users can import products with:
    - Duplicate UPC values (including malformed E notation UPCs)
    - Invalid data formats
    - No warning or error messages during the process
  - The system allows the entire import to complete despite data quality issues.

- **Steps to Reproduce:**
  1. Paste Excel data containing duplicate UPC values (intentional or caused by E notation bug)
  2. Click "Parse Data" button
  3. Review preview - no warnings about duplicates
  4. Click "Match Products" button
  5. System processes without validation errors
  6. Products are saved with duplicate UPCs

- **Expected Result:**
  - System should validate data before import and:
    - Detect duplicate UPCs within the import batch
    - Check for existing UPCs in the database
    - Validate data types and formats
    - Show clear error messages with specific rows/items that have issues
    - Prevent import from proceeding until issues are resolved

- **Actual Result:**
  - No validation occurs during bulk import
  - Duplicate and malformed data is saved to database
  - Users only discover issues after import is complete
  - Individual edit forms detect duplicates, but bulk import does not

- **Evidence (Screenshots):**
  - ![Products with Duplicate UPCs](images/image%20(2).png) - Shows "dummy prod1" and "dummy prod3" both with UPC "7.29001E+12" (highlighted in red circles)
  - ![Duplicate Detection on Individual Edit](images/image%20(3).png) - Error message "Failed to update product: UPCs already exist in this entity: 7.29001E+12" when trying to edit individual product

---

## Bug 4: Search Bar - No Feedback, Slow Processing, and Incomplete Results

- **Impact:**
  - Poor user experience. Users cannot effectively find products due to slow search, lack of feedback, and incomplete/inconsistent results.

- **Description:**
  - The search functionality has multiple issues:
    - **No visual feedback**: No loading spinner or indication that search is processing
    - **Slow performance**: Takes several minutes to return results (with only 66 products)
    - **Auto-search while typing**: Starts processing while user is still typing, no search button to trigger intentionally
    - **Incomplete results**: Not all products matching the search term appear in results
    - **No immediate refresh**: Newly imported products don't appear in search results until manual page refresh

- **Steps to Reproduce:**
  1. After completing bulk import, attempt to search for newly imported products
  2. Type search term (e.g., "prod") in the search bar
  3. Observe:
     - Search begins automatically while typing
     - No loading indicator appears
     - Results take several minutes to appear
     - Not all matching products are returned
  4. Only after manual page refresh do all products appear

- **Expected Result:**
  - Search should have:
    - A search button to trigger search intentionally (or debounced auto-search after user stops typing)
    - Visual loading indicator while processing
    - Fast response time (< 2 seconds for 66 products)
    - Complete and accurate results matching search criteria
    - Automatic refresh of product list after import completion

- **Actual Result:**
  - Search processes while typing with no feedback
  - Takes minutes to complete with small dataset
  - Missing products from results even when they match search term
  - Requires manual refresh to see newly imported products

- **Performance Concern:**
  - With only 66 products, search takes minutes. In production with thousands of products, this will be unusable.

- **Evidence (Screenshots):**
  - ![Search Results Missing Products](images/image%20(1).png) - Search for "prod" showing results, but incomplete

---

## Bug 5: Database Allows Duplicate UPCs Despite Individual Edit Validation

- **Impact:**
  - Data integrity issue. Inconsistent validation between bulk import and individual edit operations creates confusion and allows corrupted data to persist.

- **Description:**
  - Database-level constraints for duplicate UPCs are not enforced. While individual product edit forms detect and prevent duplicate UPCs, the bulk import bypasses this validation. This suggests validation is only implemented at the application form level for individual edits, not at the database schema level.

- **Steps to Reproduce:**
  1. Import products via bulk import with duplicate UPCs (due to E notation bug)
  2. Products save successfully despite duplicates
  3. Attempt to edit one of the duplicate products individually
  4. System shows error: "Failed to update product: UPCs already exist in this entity: 7.29001E+12"
  5. Cannot save the edit due to duplicate detection

- **Expected Result:**
  - Database should have UNIQUE constraints on UPC field
  - Any attempt to insert/update duplicate UPCs should fail at database level
  - Consistent validation across all data entry methods (bulk import, individual forms, API calls)

- **Actual Result:**
  - Bulk import can create duplicate UPCs
  - Individual edit form detects duplicates and prevents save
  - Inconsistent validation creates data integrity issues

- **Suggested Fix:**
  - Add UNIQUE constraint on UPC field at database level
  - Add proper error handling for constraint violations in bulk import
  - Ensure all data entry paths respect database constraints

- **Evidence (Screenshots):**
  - ![Products with Duplicate UPCs](images/image%20(2).png) - Multiple products with same UPC in database
  - ![Duplicate Detection on Edit](images/image%20(3).png) - Error when trying to edit: "Failed to update product: UPCs already exist in this entity: 7.29001E+12"
  - ![ASIN Column Showing Duplicates](images/image%20(4).png) - ASIN column view showing multiple instances of "7.29001E+12"

---

## Bug 6: Bulk Import - Cannot Set Brand or Category

- **Impact:**
  - Workflow inefficiency. Users must manually edit each product individually after bulk import to set essential data points, defeating the purpose of bulk import.

- **Description:**
  - The bulk import feature does not include fields for Brand or Category. These are important classification data points that must be set manually one-by-one after import through individual product editing.

- **Steps to Reproduce:**
  1. Use bulk import "Paste Excel Data" feature
  2. Observe available fields: ASIN, EAN, SKU, Name, Fulfillment, Market
  3. Notice Brand and Category fields are not available
  4. After import, must open each product individually to set Brand and Category

- **Expected Result:**
  - Bulk import should include fields for:
    - Brand (dropdown or text field)
    - Category (dropdown or text field)
  - Users should be able to set these values during the import process, either:
    - As columns in the Excel paste data
    - As bulk settings that apply to all imported products in the batch

- **Actual Result:**
  - Brand and Category must be set individually after import
  - Significantly increases time and effort for bulk product setup

- **Evidence (Screenshots):**
  - ![Import Excel Dialog](images/image%20(5).png) - Shows paste area with limited fields
  - ![Preview Table](images/image%20(6).png) - Preview columns don't include Brand or Category

---

## Bug 7: No Data Refresh After Bulk Upload Completion

- **Impact:**
  - User confusion and workflow disruption. Users cannot immediately verify their imported products without manually refreshing the page.

- **Description:**
  - After completing a bulk import, the product list does not automatically refresh to show the newly imported products. Users must manually refresh the page or navigate away and back to see their imported data.

- **Steps to Reproduce:**
  1. Complete bulk import process
  2. Import success message appears
  3. Return to products list
  4. Search for newly imported products
  5. Products do not appear in search results
  6. Manually refresh page
  7. Products now appear

- **Expected Result:**
  - After successful bulk import:
    - Product list should automatically refresh
    - Newly imported products should immediately appear in the list
    - User should be able to search and find new products without manual refresh

- **Actual Result:**
  - No automatic refresh occurs
  - Newly imported products are invisible until manual page refresh
  - Causes confusion about whether import succeeded

- **Suggested Fix:**
  - Trigger automatic data refresh/reload after import completion
  - Update search index immediately after import
  - Show confirmation with count of successfully imported products

---

_Additional bugs can be added as discovered during QA testing._
