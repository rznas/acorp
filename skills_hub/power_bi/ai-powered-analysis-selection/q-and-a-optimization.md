# Q&A Optimization Best Practices

How to prepare a Power BI semantic model so the Q&A visual (and Smart narrative, which shares the same natural-language engine) interprets questions accurately. These are standard model-quality optimizations — they help every report, not only Q&A.

## Contents
- Optimization workflow (checklist)
- Model structure fixes
- Naming, types, and column settings
- Normalization patterns
- Synonyms
- Ambiguity handling
- Platform note (Q&A direction)

## Optimization workflow (checklist)

Copy and work top to bottom; each item is a concrete model change:

```
Q&A Readiness:
- [ ] 1. Add missing relationships between all related tables
- [ ] 2. Rename tables/columns to reflect their actual content
- [ ] 3. Fix data types (dates/numbers imported as strings)
- [ ] 4. Set Year/Month/Day/ID/Age columns to Don't summarize (or Average)
- [ ] 5. Assign Data Category to date and geography columns
- [ ] 6. Set Sort By Column where display order != alphabetical
- [ ] 7. Normalize: one model object per real-world "thing"
- [ ] 8. Add synonyms for common business vocabulary
- [ ] 9. Use Q&A Tooling to teach terms and review user questions
```

## Model structure fixes

**Add missing relationships.** Relationships are the cornerstone. Without a relationship between, say, *Orders* and *Customers*, Q&A cannot answer "total sales for Seattle customers." Define relationships before anything else.

## Naming, types, and column settings

**Rename tables and columns to their content.** Q&A assumes names reflect content. A *CustomerSummary* table forces "list the customer summaries in Chicago" instead of "list the customers." Rename *StoreInfo* → *Store*, *Product List* → *Products*. Avoid two tables (e.g., *Headcount* and *Employees*) whose names don't clearly distinguish what each row means.

**Fix incorrect data types.** Date and number columns imported as strings are not interpreted as dates/numbers. Set the correct data type in the model.

**Year and identifier summarization.** Power BI sums numeric columns by default, so "total sales by year" can return a grand total of years alongside sales. Set **Summarization = Don't summarize** on Year, Month, Day, and ID columns. Columns like Age can use **Don't summarize** or **Average**.

**Data Category for date and geography columns.** Data Category adds meaning beyond data type. Q&A uses it (a) to choose visuals — date → line-chart axis, geography → map — and (b) for language biases ("when" → a date column; "in Brown" → likely a city, not a hair color). Mark Zip as a postal code, mark City/Country columns accordingly.

**Sort By Column.** When a column's natural order isn't alphabetical, set its Sort By Column so "sort customers by hat size" orders XS, S, M, L, XL by the underlying size number rather than alphabetically.

## Normalization patterns

General rule: **each unique "thing" the user talks about = exactly one model object** (table or column). Simple transforms can be done with model calculations; richer reshaping in Power Query / Query Editor.

- **Create tables for multi-column entities.** Move Contact Name/Title/Phone out of *Companies* into a *Contacts* table linked back, so contacts can be queried independently.
- **Pivot to eliminate property bags.** A *CustomerDemographics* table with Property/Value rows blocks questions like "average age of customers in Chicago." Pivot to one column per property (Age, Hat Size, City).
- **Union to eliminate partitioning.** Combine *Sales2000-2010* + *Sales2011-2020* into one table so "total sales by year" works. Likewise union pivoted columns (City1/City2/City3) into a single column table so "count books by city" works.
- **Split formatted columns.** A *Full Address* field should also be split into Address, City, CountryRegion. Split full-name columns into First Name and Last Name for partial-name questions.
- **Create tables for multi-value columns.** A Composer column holding several names per song should split into separate rows in a *Composers* table.
- **Denormalize to eliminate inactive relationships** (the one exception to "normalize"). If *Flights* relates to *Cities* via both SourceCityID and DestinationCityID, one relationship is inactive and Q&A only uses active ones. Denormalize the city-name columns into *Flights* so "source city Seattle and destination city San Francisco" works.

## Synonyms

This step is Q&A-specific. Users ask with whatever vocabulary comes to mind (total sales, net sales, total net sales). In Model view, select a field/table and add **Synonyms** in the Properties pane. More sensible synonyms = better experience, because users aren't picking from a predefined list.

## Ambiguity handling

Adding the same synonym to multiple objects introduces ambiguity. Q&A uses context where it can, but "count the customers" with three "customer" objects may answer the wrong one. Keep the **primary synonym unique** so the restatement (e.g., "show the number of archived customer records") alerts the user and hints they should reword.

## Platform note (Q&A direction)

<details>
<summary>Q&A experience succession</summary>

Microsoft has announced that the standalone Q&A experiences are being retired (end of 2026) in favor of **Copilot for Power BI** for natural-language querying. The model-quality practices in this file (relationships, clear naming, correct types, normalization, synonyms) apply equally to Copilot and to any natural-language feature, so this preparation work remains valuable regardless of the front-end. Use Copilot for new natural-language deployments where available.

</details>
