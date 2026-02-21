# My Team Modeling Best Practices

## General rules

- All measures, tables and columns must have a business-friendly description. Incorporate business verbiage of the COMPANY in the descriptions. (See [COMPANY verbiage](#company-verbiage))
- Data Source location (e.g. folder) should be configured as a semantic model parameter
- Make sure model includes an `About` table that describes the Author and version of the model. See [About](#about-table) for details of how to create the table if not exists.
- All Measures must be UPPERCASE
- All Measures must have a format string

## Naming Conventions

- Tables: business-friendly names, don't use terms as 'Fact' or 'Dim', use plural names for fact tables and singular names for dimension tables (e.g. `Sales`, `Product`, `Customer`). 
- Columns: Readable names with spaces (e.g. `Order Date`, `Product`, `Unit Price`)
- Columns: For dimension name prefer a column with the same name of the dimension. `Product` instead of `Product Name`
- Measures: clear naming patterns (`Total Sales`, `Total Quantity`, `# Customers`, `# Products`)
- Measures variations (e.g. time intelligence) should follow a consistent naming convention:
  - [measure name] for base measure
  - [measure name (ly)] for last year value of the base measure
  - [measure name (ytd)] for ytd value for the base measure
- Object names must not contain tabs, line breaks, or other control characters
- Object names must not start or end with a space
- **Critical**: Always use exact case-sensitive names when referencing objects
- **Verify names with List operations** before any update/rename operations

## Company verbiage

- sells products from a series of brands across multiple countries.
- operates physical retail stores and an online platform to reach global customers.
- offers a wide range of products including clothing, home goods, and electronics.
- serves millions of customers annually through both digital and in-store experiences.
- uses data and technology to personalize the shopping experience.
- partners with manufacturers and suppliers to ensure product quality and availability.
- invests in sustainable practices across its supply chain and packaging.
- has a global workforce and local teams to support regional markets.
- adapts its product offerings to meet the cultural and seasonal needs of each market.
- runs marketing campaigns tailored to specific audiences across different countries.
- manages a loyalty program to reward repeat customers and drive retention.
- constantly evaluates trends to introduce new brands and product lines.
- integrates inventory and logistics systems for efficient order fulfillment.
- participates in corporate social responsibility initiatives around the world.

## About table

- Create the table with columns:
  - Key: Text
  - Value: Text
  - Order: Number
- Use the following PowerQuery code as partition:
    ```powerquery
    let
        Source = #table({ "Key", "Value" },{
            { "Developed by", "Microsoft" },
            { "Version", "1.0" },
            { "Description", "Sales.pbip" },
            { "Last Refresh", DateTime.ToText(DateTime.LocalNow(), "yyyy-MM-dd HH:mm:ss") }
        }),
        #"Added Index" = Table.AddIndexColumn(Source, "Order", 1, 1),
        #"Changed Type" = Table.TransformColumnTypes(#"Added Index",{{"Key", type text},  {"Value", type text},{"Order", Int64.Type}}),
        #"Reordered Columns" = Table.ReorderColumns(#"Changed Type",{"Key", "Value", "Order"})
    in
        #"Reordered Columns"
    ```