# BizFlow — Universal Product-Business Management Starter

BizFlow is a multi-business starter application for product-based businesses. Each account can own multiple isolated workspaces. This working slice includes:

- Account registration and login with BCrypt password hashing.
- Multiple business workspaces per account, with a workspace switcher.
- Business-scoped products, inventory, contacts, purchases, sales and dashboard metrics.
- Product catalogue with SKU, category, unit, variant, purchase cost, selling price and stock.
- Inventory dashboard and low-stock warnings.
- Purchase entry that increases stock and recalculates weighted-average purchase cost.
- Sales billing that checks stock, creates an invoice and decreases stock.
- Customer and supplier records.
- Recent sales and purchase lists.
- Estimated daily gross profit using the product cost snapshot saved at the time of sale.
- Printable sales invoice.
- Optional GST on sales and purchases. The transaction form asks whether to apply GST and for the rate, calculates tax live, and shows CGST + SGST or IGST on the sale invoice.
- Product barcode field with USB/camera scanning in sales and purchase forms.
- Cash memo image OCR that suggests matching catalogue items, quantities and prices for review before saving.
- An opt-in BharatCode AI business assistant that answers operational questions using a limited summary of the active workspace.
- HTML/CSS/JavaScript frontend served by a Java Spring Boot backend.
- H2 local database by default and optional MySQL support.

## Technology

- Frontend: HTML5, CSS3, vanilla JavaScript
- Backend: Java 21, Spring Boot 3.4.5, Spring Web, Spring Data JPA
- Database: H2 for local development; MySQL 8+ supported
- API: JSON REST endpoints under `/api`

## Barcode and cash memo scanning

Add any product code or barcode in **Products & Stock → Add product**. In a sale or purchase form, use a USB scanner (keyboard-wedge), type the code, or use the camera button. The camera reader uses formats supported by the browser; USB readers and manual entry work with custom/internal codes as well as retail barcodes. UPC-A/EAN-13 leading-zero variants are normalized. A code must be saved on a product or match its SKU.

Use **Read cash memo** to select or photograph a memo. Tesseract.js OCR reads the image in the browser and matches recognized item lines against the catalogue. The app only uses the OCR suggestions to prefill the transaction form; review the party, products, quantities and prices, then save the transaction yourself. The memo image is not saved or uploaded by BizFlow. OCR requires internet access the first time to load Tesseract.js and its language model, and works best with clear English text; always verify the extracted figures.

## Accounts and business workspaces

Open BizFlow and choose **Create an account**. Registration asks for your name, email, password (at least 8 characters), and first shop/firm name. You can add another shop or firm with **＋ Business** and switch between them in the header. Product, contact, purchase and sales records are scoped to the selected business. Each account currently owns its own workspaces; team invitations are not included.

## BharatCode AI business assistant

The home dashboard includes an optional assistant for practical questions such as which low-stock products may need attention. It uses BharatCode's OpenAI-compatible Responses API (`qwen-3.8-27b`, `store:false`). The model call is made by the Java server; the API key is never sent to the browser.

To enable it, create an API key from [BharatCode](https://bharatcode.ai/account). Copy `.env.example` to `.env`, paste your key into `BHARATCODE_API_KEY=`, then start the **BizFlow (local data)** profile from VS Code. The launch profile loads `.env` on the server. `.env` is excluded from Git, so the key will not be published with the project. Restart BizFlow after adding or changing the key.

If you start with Maven in a terminal instead of VS Code, load the same environment file first on macOS/Linux:

```bash
set -a
source .env
set +a
mvn spring-boot:run
```

The assistant is opt-in for each question. Before sending, the user must check a disclosure that lists the data shared with BharatCode: the question, product count, inventory value, today's sales and estimated gross profit, and low-stock product names and quantities. Customer names, email addresses, and invoice-level details are excluded; users are told not to include personal or customer details in their question. Responses are advisory and do not change inventory, invoices, or other business records.

## Run locally (easy mode: H2)

Requirements:

1. Java 21 (Java 17+ may work after adjusting `pom.xml`).
2. Apache Maven 3.9+.

From this folder run:

```bash
mvn spring-boot:run
```

Then open:

```text
http://localhost:8080
```

## Build Bharat / BharatCode submission

The root `BharatCode.txt` documents the model API integration for the hackathon. The repository must be published publicly on GitHub and submitted through the official [BharatCode Build League page](https://bharatcode.ai/hackathon); completing those account actions requires the repository URL and a signed-in account.

The local H2 database is stored in `./data/bizflow.mv.db`. The VS Code Java launch profile fixes the working directory to the project root, preventing the runner from opening a second database from another folder. Database files stay out of Git; back up `data/` before moving or deleting the project. Demo products and one supplier/customer are inserted automatically the first time the database is empty.

H2 database console (optional):

```text
http://localhost:8080/h2-console
```

Use JDBC URL `jdbc:h2:file:./data/bizflow`, username `sa`, and an empty password.

## Run with MySQL 8+

1. Start MySQL.
2. Run `database/schema-mysql.sql` in MySQL Workbench or the MySQL CLI. Hibernate can also create/update the tables, but the SQL file documents the intended schema.
3. Set database credentials (recommended) and activate the `mysql` profile.

Linux/macOS:

```bash
export BIZFLOW_DB_URL='jdbc:mysql://localhost:3306/bizflow?useSSL=false&serverTimezone=UTC'
export BIZFLOW_DB_USER='root'
export BIZFLOW_DB_PASSWORD='your_password'
mvn spring-boot:run -Dspring-boot.run.profiles=mysql
```

Windows PowerShell:

```powershell
$env:BIZFLOW_DB_URL='jdbc:mysql://localhost:3306/bizflow?useSSL=false&serverTimezone=UTC'
$env:BIZFLOW_DB_USER='root'
$env:BIZFLOW_DB_PASSWORD='your_password'
mvn spring-boot:run -Dspring-boot.run.profiles=mysql
```

Do not commit real database passwords to source control.

## Main API routes

| Method | Route | Purpose |
|---|---|---|
| GET | `/api/health` | Check the backend |
| GET | `/api/dashboard` | Dashboard totals and alerts |
| GET | `/api/products` | List active products |
| POST | `/api/products` | Create a product |
| PUT | `/api/products/{id}` | Update a product |
| DELETE | `/api/products/{id}` | Deactivate a product |
| GET | `/api/parties?type=SUPPLIER` | List contacts |
| POST | `/api/parties` | Create a customer or supplier |
| POST | `/api/purchases` | Record incoming stock |
| GET | `/api/purchases/recent` | Recent purchases |
| POST | `/api/sales` | Create a sale and invoice |
| GET | `/api/sales/recent` | Recent sales |

### Example: create a product

```json
{
  "sku": "SHOE-001",
  "name": "Everyday Running Shoe",
  "category": "Footwear",
  "unit": "pair",
  "variant": "Size 8 / Black",
  "costPrice": 1200,
  "sellingPrice": 1799,
  "quantity": 12,
  "reorderLevel": 3
}
```

### Example: record a purchase

```json
{
  "supplierId": 1,
  "items": [
    { "productId": 1, "quantity": 5, "price": 1180 }
  ]
}
```

### Example: create a sale

```json
{
  "customerId": null,
  "items": [
    { "productId": 1, "quantity": 1, "price": 1799 }
  ]
}
```

For a sale, the backend verifies available stock before saving. If stock is insufficient, it rejects the complete transaction rather than partially reducing inventory.

## Important limitations of this starter

This is an early MVP foundation, not production-ready ERP software. Accounts currently own their own workspaces; staff invitations, shared business membership and role permissions are not implemented yet. API login uses short-lived in-memory bearer tokens, so a server restart signs everyone out. GST can be entered per transaction; select the rate and supply type that apply to the goods and sale. The app does not determine product-specific tax rates, collect GSTIN details, or produce statutory GST returns. Before real business use, have GST setup and invoices reviewed by an accountant, and add persistent sessions/token revocation, rate limiting, staff roles, audit logs, payment handling, returns/refunds, stock adjustments, invoice numbering rules, backups, automated tests, concurrency controls, and security hardening.

The daily gross profit figure is not net profit: it excludes rent, salaries, overheads and tax.

The first release uses one product variant field (for example, `Size 8 / Black`). A production multi-industry version should model variants, batch/expiry tracking, serial numbers, warehouses and units of measure as structured records rather than free text. OCR matching is a convenience, not accounting verification; uncertain memo lines must be entered or corrected manually.

Multi-branch inventory, manufacturing and e-commerce integrations are not included in this starter.
