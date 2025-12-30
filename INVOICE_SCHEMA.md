# Invoice Sheet Normalisation

## UNF
INVOICE(InvoiceNo, WorkerName, WorkerAddress, BillingCompany, BillingAddress, InvoiceDate, JobPicture, DateOfWork, JobLocation, Description, JobSheetRef, Quantity, UnitPrice, Amount, Total)

---

## 1NF
INVOICE(**InvoiceNo**, **ItemID**, WorkerName, WorkerAddress, BillingCompany, BillingAddress, InvoiceDate, JobPicture, DateOfWork, JobLocation, Description, JobSheetRef, Quantity, UnitPrice, Amount, Total)

---

## 2NF
INVOICE(**InvoiceNo**, WorkerName, WorkerAddress, BillingCompany, BillingAddress, InvoiceDate, JobPicture, Total)

INVOICE_ITEM(**ItemID**, **InvoiceNo***, DateOfWork, JobLocation, Description, JobSheetRef, Quantity, UnitPrice, Amount)

---

## 3NF
INVOICE(**InvoiceNo**, **BillingCompany***, **WorkerName***, InvoiceDate, JobPicture, Total)

CLIENT(**BillingCompany**, BillingAddress)

WORKER(**WorkerName**, WorkerAddress)

INVOICE_ITEM(**ItemID**, **InvoiceNo***, DateOfWork, JobLocation, Description, JobSheetRef, Quantity, UnitPrice, Amount)

---

# Appendix: Design Decisions & Industry Optimisations

### 1. Deviation from Strict 1NF (Atomicity of Address and Name Fields)
* **Theory:** Strict 1NF requires all columns to be atomic. For example: an address should theoretically be split into AddressLine1, AddressLine2, City, Region, PostCode.
* **Decision:** I have decided to store addresses as condensed fields.
* **Reasoning:** Database normalisation is driven by user access patterns. We only ever access the Client Address to print it as a text block on a PDF. We never need to run analytical queries such as "Select all clients where City = 'Belfast'". Therefore, splitting the address adds frontend complexity (more input fields) and backend complexity (additional logic) with no functional benefit for this specific use case.

### 2. Surrogate Keys vs. Natural Keys
* **Theory:** In the 3NF schema, we are using names (e.g., BillingCompany) as Foreign Keys in the notation.
* **Decision:** In the actual SQL implementation, we will use **Surrogate Keys** (e.g., ClientID as an INTEGER PRIMARY KEY AUTOINCREMENT).
* **Reasoning:** Using "Real World" data is dangerous. If a company changes its name, a Natural Key relationship would break or require complex Cascading Updates across all previous invoices. An Integer ID never changes, making the system robust against real-world renaming.

### 3. Handling of "Worker" Columns
* **Decision:** While the logical schema shows a WORKER table, upon the final design, these details may be stored in a Config File or Environment Variables rather than a queryable database table.
* **Reasoning:** As this is a single-tenant desktop application, the "Worker" does not change per transaction.
