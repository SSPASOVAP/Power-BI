-------------------------------------------------------
-- Fact-Sales

let
    Source = Excel.Workbook(File.Contents(DataPath & "\AdventureWorks-ms-excel.xlsx"), null, true),
    Sales_Table = Source{[Item="Sales",Kind="Table"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(Sales_Table,{
        {"ProductKey", Int64.Type}, {"OrderDate", type date}, {"CustomerKey", Int64.Type}, 
        {"SalesTerritoryKey", Int64.Type}, {"SalesOrderNumber", type text}, {"SalesOrderLineNumber", Int64.Type}, 
        {"OrderQuantity", Int64.Type}, {"UnitPrice", Currency.Type}, {"SalesAmount", Currency.Type}
        }),
    #"Rounded Off" = Table.TransformColumns(#"Changed Type",{
        {"UnitPrice", each Number.Round(_, 2), Currency.Type},
        {"SalesAmount", each Number.Round(_, 2), Currency.Type}
        }),
    #"Removed Other Columns" = Table.SelectColumns(#"Rounded Off",
        {"ProductKey", "OrderDate", "CustomerKey", "SalesTerritoryKey", "SalesOrderNumber", 
        "SalesOrderLineNumber", "OrderQuantity", "UnitPrice", "SalesAmount"}),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Other Columns",{
        {"SalesTerritoryKey", "Territory_Key"}, {"SalesOrderNumber", "Order_ID"}, {"SalesOrderLineNumber", "Order_LineItem"}, 
        {"OrderQuantity", "Quantity"}, {"SalesAmount", "Total_Amount"}, {"ProductKey", "Product_Key"}, 
        {"OrderDate", "Order_Date"}, {"CustomerKey", "Customer_Key"}, {"UnitPrice", "Unit_Price"}
        }),
    #"Reordered Columns" = Table.ReorderColumns(#"Renamed Columns",
        {"Territory_Key", "Customer_Key", "Order_Date", "Order_ID", "Order_LineItem", 
        "Product_Key", "Quantity", "Unit_Price", "Total_Amount"})
in
    #"Reordered Columns"

-------------------------------------------------------
-- Dim-Customers

let
    Source = Excel.Workbook(File.Contents(DataPath & "\AdventureWorks-ms-excel.xlsx"), null, true),
    Customers_Table = Source{[Item="Customers",Kind="Table"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(Customers_Table,{
        {"CustomerKey", Int64.Type}, {"GeographyKey", Int64.Type}, {"Name", type text}, {"BirthDate", type date}, 
        {"MaritalStatus", type text}, {"Gender", type text}, {"YearlyIncome", Currency.Type}, {"Occupation", type text}, {"NumberCarsOwned", Int64.Type}, {"DateFirstPurchase", type date}, {"AddressLine1", type text}, {"Phone", type text}, {"HouseOwnerFlag", type text}, {"NumberChildrenAtHome", Int64.Type}
        }),
    #"Replaced 0-No" = Table.ReplaceValue(#"Changed Type","0","No",Replacer.ReplaceText,{"HouseOwnerFlag"}),
    #"Replaced 1-Yes" = Table.ReplaceValue(#"Replaced 0-No","1","Yes",Replacer.ReplaceText,{"HouseOwnerFlag"}),
    #"Removed Columns" = Table.RemoveColumns(#"Replaced 1-Yes",{"GeographyKey", "AddressLine2"}),
    #"Reordered Columns" = Table.ReorderColumns(#"Removed Columns",
        {"CustomerKey", "Name", "AddressLine1", "Phone", "BirthDate", "Gender", 
        "MaritalStatus", "YearlyIncome", "HouseOwnerFlag", "NumberCarsOwned", 
        "Occupation", "DateFirstPurchase"}),
    #"Renamed Columns" = Table.RenameColumns(#"Reordered Columns",{
        {"AddressLine1", "Address"}, {"HouseOwnerFlag", "House_Owner"}, {"NumberCarsOwned", "Cars_Owned"}, 
        {"DateFirstPurchase", "First_Order"}, {"CustomerKey", "Customer_Key"}, {"BirthDate", "Birth_Date"}, 
        {"MaritalStatus", "Marital_Status"}, {"YearlyIncome", "Yearly_Income"}, 
        {"NumberChildrenAtHome", "Number_of_Children"}
        }),
    #"Trimmed Text" = Table.TransformColumns(#"Renamed Columns",{{"Name", Text.Trim, type text}}),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Trimmed Text", "Name", Splitter.SplitTextByDelimiter(" ", QuoteStyle.Csv), 
        {"Name.1", "Name.2"}),
    #"Invoked Custom Function" = Table.AddColumn(#"Split Column by Delimiter", "Customer_Name", each NameFunc([Name.1], [Name.2], null)),
    #"Removed Columns1" = Table.RemoveColumns(#"Invoked Custom Function",{"Name.1", "Name.2"}),
    #"Changed Type3" = Table.TransformColumnTypes(#"Removed Columns1",{{"Customer_Name", type text}}),
    #"Added Age" = Table.AddColumn(#"Changed Type3", "Age", each [First_Order] - [Birth_Date]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Age",{{"Age", type duration}}),
    #"Calculated Total Years" = Table.TransformColumns(#"Changed Type1",{{"Age", each Duration.TotalDays(_) / 365, type number}}),
    #"Rounded Down" = Table.TransformColumns(#"Calculated Total Years",{{"Age", Number.RoundDown, Int64.Type}}),
    #"Added AgeRange" = Table.AddColumn(#"Rounded Down", "Age_Range", 
    each if
        [Age] >= 20 and [Age] <= 30 then "20-30"
        else if [Age] > 30 and [Age] <= 40 then "31-40"
        else if [Age] > 40 and [Age] <= 50 then "41-50"
        else if [Age] > 50 and [Age] <= 60 then "51-60"
        else if [Age] > 60 and [Age] <= 70 then "61-70"
        else "70+"),
    #"Changed Type2" = Table.TransformColumnTypes(#"Added AgeRange",{{"Age_Range", type text}}),
    #"Reordered Columns1" = Table.ReorderColumns(#"Changed Type2",{"Customer_Key", "Customer_Name", "Address", "Phone", "Birth_Date", "Gender", "Marital_Status", "Yearly_Income", "House_Owner", "Cars_Owned", "Occupation", "First_Order", "Age", "Age_Range"})
in
    #"Reordered Columns1"

-------------------------------------------------------
-- Dim-Products

let
    Source = Excel.Workbook(File.Contents(DataPath & "\AdventureWorks-ms-excel.xlsx"), null, true),
    Products_Table = Source{[Item="Products",Kind="Table"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(Products_Table,{
        {"ProductKey", Int64.Type}, {"ProductName", type text}, 
        {"StandardCost", Currency.Type}, {"Color", type text}, {"Class", type text}, {"ModelName", type text}, 
        {"Description", type text}, {"StartDate", type date}, {"EndDate", type date}, {"SubCategory", type text}, 
        {"Category", type text}, {"SizeRange", type text}, {"ProductLine", type text}, 
        {"SafetyStockLevel", Int64.Type}, {"DaysToManufacture", Int64.Type}, {"Size", type text}
        }),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type",
        {"StandardCost", "SafetyStockLevel", "Weight", "DaysToManufacture", "Description", 
        "StartDate", "EndDate", "Status", "ProductName", "ListPrice", "DealerPrice","ProductSubcategoryKey"}),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns",{
        {"ModelName", "Product_Name"}, {"ProductKey", "Product_Key"}, {"SizeRange", "Size_Range"}, {"ProductLine", "Product_Line"}, {"SubCategory", "SubCategory_Name"}, {"Category", "Category_Name"}
        }),
    #"Reordered Columns" = Table.ReorderColumns(#"Renamed Columns",{"Category_Name", "SubCategory_Name", 
        "Product_Key", "Product_Name", "Size", "Color", "Size_Range", "Product_Line", "Class"}),
    #"Replaced null-OneSize" = Table.ReplaceValue(#"Reordered Columns",null,"One-Size",Replacer.ReplaceValue,{"Size"}),
    #"Replaced NA-Custom" = Table.ReplaceValue(#"Replaced null-OneSize","NA","Custom",Replacer.ReplaceText,{"Color"}),
    #"Replaced PL null-Other" = Table.ReplaceValue(#"Replaced NA-Custom",null,"Other",Replacer.ReplaceValue,{"Product_Line"}),
    #"Replaced C null-Other" = Table.ReplaceValue(#"Replaced PL null-Other",null,"Other",Replacer.ReplaceValue,{"Class"}),
    #"Replaced NA-OneSize" = Table.ReplaceValue(#"Replaced C null-Other","NA","One-Size",Replacer.ReplaceText,{"Size_Range"})
in
    #"Replaced NA-OneSize"

-------------------------------------------------------
-- Dim-Locations

let
    Source = Excel.Workbook(File.Contents(DataPath & "\AdventureWorks-ms-excel.xlsx"), null, true),
    Territory_Table = Source{[Item="Territory",Kind="Table"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(Territory_Table,{
        {"Territory Key", Int64.Type}, {"Region", type text}, {"Country", type text}, {"Group", type text}
        }),
    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each ([Territory Key] <> 11)),
    #"Renamed Columns" = Table.RenameColumns(#"Filtered Rows",{{"Territory Key", "Territory_Key"}, {"Group", "Geo_Group"}})
in
    #"Renamed Columns"

-------------------------------------------------------
-- Calendar

let
    Source = Excel.Workbook(File.Contents(DataPath & "\AdventureWorks-ms-excel.xlsx"), null, true),
    Calendar_Table = Source{[Item="Calendar",Kind="Table"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(Calendar_Table,{
        {"Date", type date}, {"ID", Int64.Type}, {"DayName", type text}, {"DayNumberOfMonth", Int64.Type}, 
        {"DayNumberOfYear", Int64.Type}, {"WeekNumberOfYear", Int64.Type}, {"MonthName", type text}, {"MonthNumberOfYear", Int64.Type}, {"CalendarQuarter", Int64.Type}, {"CalendarYear", Int64.Type}, {"DayNumberOfWeek", Int64.Type}
        }),
    #"Removed Other Columns" = Table.SelectColumns(#"Changed Type",
        {"Date", "DayNumberOfWeek", "DayName", "DayNumberOfMonth",  "DayNumberOfYear", "WeekNumberOfYear", "MonthName", "MonthNumberOfYear", "CalendarQuarter", "CalendarYear"}),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Other Columns",{
        {"DayNumberOfMonth", "Day_Month"}, {"DayNumberOfYear", "Day_Year"}, {"WeekNumberOfYear", "Week_Year"}, 
        {"MonthNumberOfYear", "Month"}, {"CalendarQuarter", "Quarter_Name"}, {"CalendarYear", "Year"}, 
        {"DayName", "Day_Name"}, {"MonthName", "Month_Name"}, {"DayNumberOfWeek", "Day_Week"}
        }),
    #"Inserted Quarter" = Table.AddColumn(#"Renamed Columns", "Quarter", each Date.QuarterOfYear([Date]), Int64.Type),
    #"Added Prefix" = Table.TransformColumns(#"Inserted Quarter", {{"Quarter_Name", each "Q-" & Text.From(_, "en-GB"), type text}}),
    #"Replaced Value" = Table.ReplaceValue(#"Added Prefix", each [Day_Week], each Date.DayOfWeek([Date], Day.Sunday),
        Replacer.ReplaceValue,
        {"Day_Week"}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Replaced Value",{{"Day_Week", Int64.Type}}),
    #"Reordered Columns" = Table.ReorderColumns(#"Changed Type1",{"Date", "Year", "Quarter", "Quarter_Name", "Month", "Month_Name", "Day_Year", "Day_Month", "Day_Week", "Day_Name", "Week_Year"})
in
    #"Reordered Columns"
