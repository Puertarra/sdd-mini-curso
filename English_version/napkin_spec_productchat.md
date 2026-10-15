# Napkin Spec: ProductChat v0

> This is a **napkin spec**: a person writes it by hand, before
> touching any tool. It is not the formal artifact that Spec Kit produces; it is
> the raw material we are going to hand to it. It forces us to decide what we
> want, and to put in writing what we have not decided yet.

## 1. Goal and business context

ProductChat lets anyone at **Distribuidora Andes** query the company's catalog of
products and services in natural language, without opening the spreadsheet,
without knowing how it is organized and without asking the person who
"knows it by heart".

The real scene is this. A salesperson is on the phone with a customer who asks
whether there are notebooks with 32 GB of RAM. The salesperson opens `product_catalog.xlsx`,
filters by the `category` column and gets fewer results than they should,
because half the catalog is entered as `Laptops` and not as
`Notebooks`. They search for "32 GB" in the `Features` column and find two
machines, but there is a third entered as `32768 MB` and a fourth where someone
wrote just `32`. They also find a Dell that seems to qualify, until they
read the `Notes` column and it says "expandable to 32GB on request". Meanwhile,
the customer is waiting.

The catalog is not clean and **it is not going to be cleaned up before this is built**. That
is the starting point, not a problem someone else will solve:

- The file is `product_catalog.xlsx`, maintained by hand by the sales department.
- The header is not in the first row: there are two rows of loose text above it.
- There are **duplicate SKUs with data that does not match** each other.
- There is **the same product entered under two different SKUs**.
- The category is written in **13 different ways** for 5 real families
  (`Notebooks`, `notebooks`, `NOTEBOOKS`, `Laptops`, `Laptop`, …).
- Attributes have no consistent unit: `32 GB`, `32768 MB`, `RAM 32 GB`, `32`.
- The price is sometimes a number, sometimes text (`949.99`, `$1,299.99`),
  sometimes empty and sometimes says `ask`.
- Stock mixes numbers with `Yes`, `yes`, `sold out` and empty cells.
- There is a free-text `Notes` column where the data that matters sometimes lives.
- There are blank rows, extra spaces and inconsistent spelling.
- There is a **second sheet**, `old_catalog`, with different columns, a different
  header position and different prices for the same SKUs.
- Products and services live together in the same spreadsheet, even though stock
  does not apply to services.

Business success: answer a catalog query correctly in under
ten seconds, **without the user having to know any of the above**.

## 2. Users

- **The salesperson (uses the application).** Works at Distribuidora Andes, serves
  customers by phone and email. Not technical. Today they check the spreadsheet by
  hand. They need answers that are fast and, above all, **reliable**: they would rather get an
  "I'm not sure" than a made-up figure that makes them look bad in front of the customer.
- **The catalog manager (does not use the application, but cares about it).** The person who
  maintains the Excel file. They are not going to rebuild it. It helps them if the chatbot exposes the
  inconsistencies instead of hiding them, because that tells them what to fix first.
- **The end customer (does not use the application).** Gets the answer from the
  salesperson. Never talks to the chatbot in this version.

## 3. User stories

- **US1 — Search for products.** As a salesperson, I want to find products by
  name, category or features written in natural language, so that I do not
  depend on how the data is written in the spreadsheet.
- **US2 — Look up a product's attributes.** As a salesperson, I want to ask for
  the price, the stock or a specific feature of a product, so that I can
  answer the customer on the spot.
- **US3 — Handle missing or insufficient information.** As a salesperson,
  I want the chatbot to tell me clearly when it does not know, so that I do not repeat to the
  customer a piece of data that does not exist.
- **US4 — Deal with an inconsistent catalog.** As a salesperson, I want the
  chatbot to understand that `32768 MB` and `32 GB` are the same and that `Laptops` and
  `Notebooks` are the same family, and to **warn me when the catalog
  contradicts itself**, so that I do not have to pick blindly between two prices.

## 4. Functional requirements

- **FR1.** The system must read the catalog from `product_catalog.xlsx`,
  locating the header row even if it is not the first row of the file.
- **FR2.** The system must ignore empty rows and trim extra spaces in
  text values.
- **FR3.** The system must normalize the category, so that variants in
  capitalization, spelling and declared synonyms are treated as the same family.
- **FR4.** The system must normalize numeric attributes with units, so
  that `32 GB`, `32768 MB`, `RAM 32 GB` and `32` are interpreted as 32 GB.
- **FR5.** The system must normalize the price to a numeric value in US
  dollars, and mark empty or text cases (`ask`, `per quote`) as
  **price not available**.
- **FR6.** The system must normalize stock to three states: **available**,
  **out of stock** and **stock not reported**.
- **FR7.** The system must detect duplicate SKUs and, when the duplicate
  records do not match on a queried attribute, **report the conflict
  instead of silently picking a value**.
- **FR8.** The system must allow natural language queries by name,
  category and features.
- **FR9.** The system must be able to return several results when the query
  allows for them.
- **FR10.** The system must distinguish a **declared** attribute from a
  **conditional or potential** attribute mentioned in free text, and must not count the second
  as if it were the first.
- **FR11.** When there is not enough information to answer, the system
  must say so explicitly.
- **FR12.** The system must state, alongside each answer, the SKU the
  data comes from, so that the salesperson can verify it in the spreadsheet.

## 5. Business rules

- **BR1.** The `Catalog` sheet is the source of truth. The `old_catalog` sheet
  is historical and **is not used** in this version.
- **BR2.** Equivalent families: `Notebooks` ≡ `Laptops` ≡ `Laptop`
  (with any combination of capitalization and spelling).
- **BR3.** Memory unit conversion: 1024 MB = 1 GB. A number without a
  unit in a memory field is interpreted in GB.
- **BR4.** An attribute mentioned in `Notes` as a future condition ("expandable to",
  "on request", "made to order") **does not count as an attribute of the product**.
- **BR5.** When a duplicate SKU has conflicting values, the chatbot **does not choose**:
  it reports both values and states that the catalog is inconsistent on that point.
- **BR6.** The chatbot does not invent products, prices or attributes. Every piece of data
  it delivers must be traceable to a row of the file.

### Reference example (must match exactly)

Query: **"Which notebooks have 32 GB of RAM?"**

| How it is solved | Result | Verdict |
|---|---|---|
| Search for the literal `32 GB` in `Features` | NB-1005, NB-1012 | **Wrong**: 2 false negatives |
| Search for the text `32` in all fields | NB-1003, NB-1005, NB-1007, NB-1009, NB-1012 | **Wrong**: 1 false positive |
| Normalize memory to GB and apply BR3 and BR4 | NB-1005, NB-1007, NB-1009, NB-1012 | **Correct: 4 models** |

Why each one is in or out:

| SKU | Data in the spreadsheet | Interpretation | Qualifies? |
|---|---|---|---|
| NB-1005 | `32 GB RAM, SSD 1TB` | 32 GB declared | Yes |
| NB-1007 | `RAM 32768 MB, SSD 512 GB` | 32768 MB = 32 GB (BR3) | Yes |
| NB-1009 | `32` | no unit, read as GB (BR3) | Yes |
| NB-1012 | `RAM 32 GB, SSD 1 TB, i7` | 32 GB declared | Yes |
| NB-1003 | `16GB RAM DDR4` + Notes `expandable to 32GB` | conditional (BR4) | No |

Query: **"What is the price of the ThinkBook 14?"**

The SKU `NB-1001` appears **twice in the `Catalog` sheet**, with prices
899.99 and 949.99, and a third time in `old_catalog` with 879.99. Under BR1
the historical sheet is discarded; under BR5 the chatbot must answer that there are two
prices recorded for that SKU and that the catalog is inconsistent. **It must not
pick one.** Which of the two wins is a business decision that has not been
made yet: see OQ1.

## 6. Acceptance criteria (verifiable without reading code)

- **AC1.** Given "which notebooks have 32 GB of RAM?", the chatbot returns
  exactly the 4 models in the table above; it does not include NB-1003.
- **AC2.** Given "which notebooks are there?", the result includes the machines entered
  as `Laptops` and `Laptop`, not only those entered as `Notebooks`.
- **AC3.** Given "what is the price of the ThinkBook 14?", the chatbot reports that
  there are two conflicting prices for NB-1001 and does not give a single one as if it were
  the correct one.
- **AC4.** Given a query about the price of `MON-3002` (Monitor LG UltraWide),
  the chatbot answers that the price is not available and must be requested,
  instead of leaving out the product or inventing a value.
- **AC5.** Given a query about the stock of `NB-1007`, whose value is `Yes`, the
  chatbot answers "available" and not a number.
- **AC6.** Given a query about a product that does not exist in the catalog, the
  chatbot says it found no information.
- **AC7.** Every piece of data delivered comes with its source SKU.
- **AC8.** A non-technical person can run the queries above and
  verify each answer by opening `product_catalog.xlsx`.

## 7. Edge cases

- **EC1.** Completely empty row in the middle of the data: it is ignored.
- **EC2.** Product without features (`MON-3003`): it can be found by
  name and category, and when a feature is requested the answer is that it is not
  recorded.
- **EC3.** Product without a price (`NB-1009`): it appears in the results, with the
  price marked as not available.
- **EC4.** Very similar names (`NB-1001` "Notebook Lenovo ThinkBook 14" and
  `NB-1015` "notebook lenovo thinkbook 14 gen 4"): both are returned and
  told apart by SKU.
- **EC5.** Same product under two SKUs (`NB-1002` and `NB-2002`; `MON-3001` and
  `MON-3004`): both are returned; the chatbot does not merge them on its own.
- **EC6.** Services (`SRV-*`): they have no stock; asking about their stock must not
  produce "out of stock" but "not applicable".
- **EC7.** The spreadsheet is open or the file cannot be read: the chatbot
  reports it and does not answer with old data.

## 8. Out of scope (v0)

- Modifying, creating or deleting products from the chatbot.
- **Fixing the Excel file.** The chatbot reads; it does not write.
- Buying, issuing formal quotes or managing orders.
- Recommending products based on criteria outside the catalog.
- Answering questions that are not about the catalog.
- Using the `old_catalog` sheet.
- Connecting to a SQL database. In this version the source is the Excel file, and
  only the Excel file.
- User accounts, permissions and multi-user support.
- Any functionality not explicitly described here.

## 9. Open questions (for the Clarify session)

Things I know I have not decided. I am not hiding them: I am writing them down to resolve them in
the next step.

- **OQ1. Conflicting duplicates.** BR5 says "report both". Is that the
  final behavior, or does the business prefer a tie-break rule (the highest
  price, the most recent according to `Notes`, the last row in the file)?
- **OQ2. Does "have" mean catalog or stock?** Of the 4 notebooks with 32 GB,
  only NB-1005 and NB-1012 have numeric stock greater than zero. Is the
  salesperson's query asking about what exists in the catalog or about what can be sold
  today?
- **OQ3. Category synonyms.** BR2 declares the notebook/laptop equivalence
  by hand. Do we keep a fixed list of synonyms, or should the system infer them?
  If it is a list, who maintains it?
- **OQ4. Scope of free text.** BR4 discards what is conditional. Should the chatbot
  still **mention** that the Dell is expandable to 32 GB, as additional
  information, or leave it out?
- **OQ5. Data freshness.** Is the Excel file read on every query, or loaded once
  at startup? If the catalog manager edits the spreadsheet at noon, what does the
  salesperson see at 12:01?
- **OQ6. Confidence threshold.** When should the chatbot say "I'm not sure"
  instead of answering? Is there a case where it answers with caveats?
- **OQ7. Products and services.** Are they queried together, or should the user be able to
  ask for "services only"?
