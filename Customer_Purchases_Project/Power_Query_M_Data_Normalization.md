## Customer-Purchases

let
    Source = Csv.Document(File.Contents(#"PS DataPath" & "\customers-purchases-2014-2016.csv"),
    [Delimiter=";", Columns=10, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Promoted Headers", "CONTACT", 
        Splitter.SplitTextByEachDelimiter({"-"}, QuoteStyle.Csv, false), {"CUSTOMER_CODE", "CONTACT.2"}),
    #"Split Column by Delimiter1" = Table.SplitColumn(#"Split Column by Delimiter", "CONTACT.2", 
        Splitter.SplitTextByEachDelimiter({","}, QuoteStyle.Csv, true), {"COMPANY_NAME", "CONTACT_NAME"}),
    #"Split Column by Delimiter2" = Table.SplitColumn(#"Split Column by Delimiter1", "CUSTOMER_LOCATION", 
        Splitter.SplitTextByEachDelimiter({","}, QuoteStyle.Csv, true), {"COUNTRY", "CITY"}),
    #"Split Column by Delimiter3" = Table.SplitColumn(#"Split Column by Delimiter2", "ORDER_NO_DATE", 
        Splitter.SplitTextByEachDelimiter({"/"}, QuoteStyle.Csv, true), {"ORDER_ID", "ORDER_DATE"}),
    #"Split Column by Delimiter4" = Table.SplitColumn(#"Split Column by Delimiter3", "PRODUCT_UNITS", 
        Splitter.SplitTextByDelimiter("(", QuoteStyle.Csv), {"PRODUCT_NAME", "PRODUCT_UNITS.2"}),
    #"Split Column by Delimiter5" = Table.SplitColumn(#"Split Column by Delimiter4", "PRODUCT_UNITS.2", 
        Splitter.SplitTextByDelimiter(")", QuoteStyle.Csv), {"QNT_PER_UNIT", "PRODUCT_UNITS.2.2"}),
    #"Split Column by Delimiter6" = Table.SplitColumn(#"Split Column by Delimiter5", "CONTACT_NAME", 
        Splitter.SplitTextByDelimiter(" ", QuoteStyle.Csv), 
    {"CONTACT_NAME.1", "FIRST_NAME", "MID_NAME", "L_NAME"}),
    #"Replaced Value" = Table.ReplaceValue(#"Split Column by Delimiter6","","NULL",Replacer.ReplaceValue,{"L_NAME"}),
    #"Added LastName" = Table.AddColumn(#"Replaced Value", "LAST_NAME", each if [L_NAME] = null then [MID_NAME] else if [L_NAME] = "NULL" 
    then [MID_NAME] else [L_NAME]),
    #"Removed Columns" = Table.RemoveColumns(#"Added LastName",{"CONTACT_NAME.1", "MID_NAME", "L_NAME", "PRODUCT_UNITS.2.2"}),
    #"Changed Type" = Table.TransformColumnTypes(#"Removed Columns",
    {{"LAST_NAME", type text}, {"ORDER_ID", Int64.Type}, {"ORDER_DATE", type date}, {"REQUIRED_DATE", type date},
    {"SHIPPED_DATE", type date}, {"QUANTITY", Int64.Type}, {"UNIT_PRICE", Currency.Type}, {"DISCOUNT", Percentage.Type}},
    "en-US"),
    #"Added Total" = Table.AddColumn(#"Changed Type", "TOTAL", 
        each if [UNIT_PRICE] = null then null 
        else [UNIT_PRICE] * [QUANTITY]),
    #"Added TotalDisc" = Table.AddColumn(#"Added Total", "TOTAL_DISCOUNT", 
        each if [DISCOUNT] = null then null 
        else [TOTAL] * (1- [DISCOUNT])),
    #"Added Delivery" = Table.AddColumn(#"Added TotalDisc", "DELIVERY_TIME", each [SHIPPED_DATE] - [ORDER_DATE]),
    #"Added Track" = Table.AddColumn(#"Added Delivery", "TRACK", each if [SHIPPED_DATE] = null then "not shipped" else if [SHIPPED_DATE] <= [REQUIRED_DATE] then true else false),
    #"Added Custom" = Table.AddColumn(#"Added Track", "CONTACT_NAME", each Text.Range([FIRST_NAME], 0,1) & ". " & [LAST_NAME]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Custom",{
        {"TOTAL", Currency.Type}, 
        {"TOTAL_DISCOUNT", Currency.Type}, 
        {"DELIVERY_TIME", type duration}, 
        {"TRACK", type text}},
        "en-US"),
    #"Changed Type3" = Table.TransformColumnTypes(#"Changed Type1",{{"CONTACT_NAME", type text}}),
    #"Calculated Total Days" = Table.TransformColumns(#"Changed Type3",{{"DELIVERY_TIME", Duration.TotalDays, type number}}),
    #"Reordered Columns" = Table.ReorderColumns(#"Calculated Total Days",{
        "CUSTOMER_CODE", "COMPANY_NAME", "FIRST_NAME", "LAST_NAME", "COUNTRY", "CITY", 
        "ORDER_ID", "ORDER_DATE", "REQUIRED_DATE", "SHIPPED_DATE", "CATEGORY_NAME", "PRODUCT_NAME", 
        "QNT_PER_UNIT", "QUANTITY", "UNIT_PRICE", "DISCOUNT", "TOTAL", "TOTAL_DISCOUNT", "DELIVERY_TIME", "TRACK"})
in
    #"Reordered Columns"


## Fact-Purchases

let
    Source = #"customers-purchases-2014-2016",
    #"Merged Customers" = Table.NestedJoin(Source, {"CUSTOMER_CODE"}, #"dim-customers", {"CUSTOMER_CODE"}, "dim-customers", JoinKind.Inner),
    #"Merged Products" = Table.NestedJoin(#"Merged Customers", {"PRODUCT_NAME"}, #"dim-products", {"PRODUCT_NAME"}, "dim-products", JoinKind.LeftOuter),
    #"Merged Locations" = Table.NestedJoin(#"Merged Products", {"COUNTRY", "CITY"}, #"dim-locations", {"COUNTRY", "CITY"}, "dim-locations", JoinKind.Inner),
    #"Expanded dim-customers" = Table.ExpandTableColumn(#"Merged Locations", "dim-customers", {"CUSTOMER_ID"}, {"CUSTOMER_ID"}),
    #"Expanded dim-products" = Table.ExpandTableColumn(#"Expanded dim-customers", "dim-products", {"PRODUCT_ID"}, {"PRODUCT_ID"}),
    #"Expanded dim-locations" = Table.ExpandTableColumn(#"Expanded dim-products", "dim-locations", {"LOCATION_ID"}, {"LOCATION_ID"}),
    #"Removed Columns" = Table.RemoveColumns(#"Expanded dim-locations",
        {"COMPANY_NAME", "FIRST_NAME", "LAST_NAME", "PRODUCT_NAME", "QNT_PER_UNIT", "CATEGORY_NAME", "CUSTOMER_CODE","COUNTRY", "CITY", "REQUIRED_DATE", "CONTACT_NAME"}),
    #"Reordered Columns" = Table.ReorderColumns(#"Removed Columns",{"CUSTOMER_ID", "LOCATION_ID", "ORDER_ID", "PRODUCT_ID", "ORDER_DATE", "SHIPPED_DATE", "QUANTITY", "UNIT_PRICE", "DISCOUNT", "TOTAL", "TOTAL_DISCOUNT", "DELIVERY_TIME", "TRACK"}),
    #"Sorted Rows" = Table.Sort(#"Reordered Columns",{{"ORDER_ID", Order.Ascending}}),
    #"Filtered Rows" = Table.SelectRows(#"Sorted Rows", each ([ORDER_ID] <> null))
in
    #"Filtered Rows"


## Dim-Customer

let
    Source = #"customers-purchases-2014-2016",
    #"Removed Other Columns" = Table.SelectColumns(Source,{"COMPANY_NAME", "CUSTOMER_CODE", "CONTACT_NAME"}),
    #"Reordered Columns" = Table.ReorderColumns(#"Removed Other Columns",{"CUSTOMER_CODE", "COMPANY_NAME"}),
    #"Removed Duplicates" = Table.Distinct(#"Reordered Columns"),
    #"Added Index" = Table.AddIndexColumn(#"Removed Duplicates", "CUSTOMER_ID", 1, 1, Int64.Type),
    #"Reordered Columns1" = Table.ReorderColumns(#"Added Index",{"CUSTOMER_ID", "CUSTOMER_CODE", "COMPANY_NAME", "CONTACT_NAME"})
in
    #"Reordered Columns1"

## Dim-Products

let
    Source = #"customers-purchases-2014-2016",
    #"Removed Other Columns" = Table.SelectColumns(Source,{"PRODUCT_NAME", "QNT_PER_UNIT", "CATEGORY_NAME"}),
    #"Filtered Rows" = Table.SelectRows(#"Removed Other Columns", each ([PRODUCT_NAME] <> "")),
    #"Removed Duplicates" = Table.Distinct(#"Filtered Rows"),
    #"Sorted Rows" = Table.Sort(#"Removed Duplicates",{{"CATEGORY_NAME", Order.Ascending}, {"PRODUCT_NAME", Order.Ascending}}),
    #"Added Index" = Table.AddIndexColumn(#"Sorted Rows", "PRODUCT_ID", 1, 1, Int64.Type),
    #"Reordered Columns" = Table.ReorderColumns(#"Added Index",{"PRODUCT_ID", "CATEGORY_NAME", "PRODUCT_NAME", "QNT_PER_UNIT"}),
    #"Merged Queries" = Table.NestedJoin(#"Reordered Columns", {"QNT_PER_UNIT"}, #"product-qnt-per-unit", {"QNT_PER_UNIT"}, "product-qnt-per-unit", JoinKind.Inner),
    #"Expanded product-qnt-per-unit" = Table.ExpandTableColumn(#"Merged Queries", "product-qnt-per-unit", {"PACK_QNT", "MEASURE", "PACKS"}, {"PACK_QNT", "MEASURE", "PACKS"})
in
    #"Expanded product-qnt-per-unit"


## Dim-Locations

let
    Source = #"customers-purchases-2014-2016",
    #"Removed Other Columns" = Table.SelectColumns(Source,{"COUNTRY", "CITY"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Other Columns"),
    #"Added Index" = Table.AddIndexColumn(#"Removed Duplicates", "LOCATION_ID", 1, 1, Int64.Type),
    #"Reordered Columns" = Table.ReorderColumns(#"Added Index",{"LOCATION_ID", "COUNTRY", "CITY"})
in
    #"Reordered Columns"