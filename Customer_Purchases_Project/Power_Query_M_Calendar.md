## Generate Calendar

let
    Calendar_Func = (StartDate as date, EndDate as date) as table => 
    let
        start_num = Number.From(StartDate),
        end_num = Number.From(EndDate),
        list_nums = {start_num..end_num},
        convert_to_table = Table.FromList(list_nums, Splitter.SplitByNothing()),
        #"Changed Type" = Table.TransformColumnTypes(convert_to_table,{{"Column1", type date}}),
        #"Renamed Columns" = Table.RenameColumns(#"Changed Type",{{"Column1", "Dates"}})
    in
        #"Renamed Columns"
in
    Calendar_Func


## Formatting Calendar

let
    Source = GenerateCalendar(#date(2014, 1, 1), #date(2016, 12, 31)),
    #"Inserted Year" = Table.AddColumn(Source, "Year", each Date.Year([Dates]), Int64.Type),
    #"Inserted Quarter" = Table.AddColumn(#"Inserted Year", "Quarter", each Date.QuarterOfYear([Dates]), Int64.Type),
    #"Inserted Month Name" = Table.AddColumn(#"Inserted Quarter", "Month Name", each Date.MonthName([Dates]), type text),
    #"Inserted Week of Year" = Table.AddColumn(#"Inserted Month Name", "Week of Year", each Date.WeekOfYear([Dates]), Int64.Type),
    #"Inserted Week of Month" = Table.AddColumn(#"Inserted Week of Year", "Week of Month", each Date.WeekOfMonth([Dates]), Int64.Type),
    #"Inserted Day Name" = Table.AddColumn(#"Inserted Week of Month", "Day Name", each Date.DayOfWeekName([Dates]), type text),
    #"Inserted Month" = Table.AddColumn(#"Inserted Day Name", "Month", each Date.Month([Dates]), Int64.Type),
    #"Inserted Day" = Table.AddColumn(#"Inserted Month", "Day", each Date.Day([Dates]), Int64.Type),
    #"Inserted Day of Week" = Table.AddColumn(#"Inserted Day", "Day of Week", each Date.DayOfWeek([Dates], Day.Sunday), Int64.Type),
    #"Inserted Quarter1" = Table.AddColumn(#"Inserted Day of Week", "Numb Quarter", each Date.QuarterOfYear([Dates]), Int64.Type),
    #"Changed Type" = Table.TransformColumnTypes(#"Inserted Quarter1",{{"Quarter", type text}}),
    #"Added Prefix" = Table.TransformColumns(#"Changed Type", {{"Quarter", each "Q " & _, type text}}),
    #"Added Prefix1" = Table.TransformColumns(#"Added Prefix", {{"Week of Year", each "Y-W " & Text.From(_, "en-GB"), type text}}),
    #"Added Prefix2" = Table.TransformColumns(#"Added Prefix1", {{"Week of Month", each "M-W " & Text.From(_, "en-GB"), type text}}),
    #"Added Custom" = Table.AddColumn(#"Added Prefix2", "Year Q", each Text.From([Year]) & " - " & Text.From([Quarter])),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Custom",{{"Year Q", type text}})
in
    #"Changed Type1"
