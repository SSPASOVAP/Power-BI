## Break down products to few columns for making work with the data easier

let
    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("bZZNruIwDICvErHmPaWlYXoXxKJ0OhVDBxAtEnqnn8RxHNvJjnxx/W+H02nXWHN5fKbVfEzrfw7zanb7QL/oHO935/1p13aeN615/Hi6bcsE0jUapH85a8bRPKcXclDtZfEiwSDrPw/cjAvXXKMkfXDO/FvMONyzrGAgefTUWWtms11RUKMgF87+Q3QQT9xBCBLV69g1Jnn77UwprigF5JxV2qs4yHe9vziGjP8dXiCqSZA6HFPxYlj+/L5fNyxpdrMtDFcxOdpzw5qAVugfc5upozQB9+hLZrkCyU9hVxPyLnaicI8jlBOJ8edqYkQfZKsag0oX8t8631VkWiOQs+Z5m9dvMm3RdKBJxGL2ZzaRmjEnPSOdEuVqdMDmZVjXKVdEUapKHA/KUcmynw3g53UaJ+appBh42jS9EIcyIsj5P/jvr3825kMQjKkKF19yLcX4P6ZjqiEVBCgdHVij2ndkPFDtqu8M5WsmPBUtxXwRjccQag4uQXd/mFKOqJsO8PHr8b7/zv0kIemMxcmdJRmtuJlvuJn3XPY2wn2J0NgNzryBI2H7xJYt2UpnmkIN1+L0reO3sMDjLqEkaiR2W+GLmo+uTKBieTKSn1FMopxJZVWxNJUwyKpnLe9ZEGArFSyld0wTMN5XeqEvQ+nECNanwKeA/hrQxMT8VSbAUTh+UNfhvaJ1Z6FpEbHd5UvGxzAekwOkMzU8RaNY9DQMRg+LNv0bKBBEbUVHgz7F2DCY5aIqSCi3DQwuVUQRDFN3+lENjKxEBjkRTVspa1uWNfZZfJ3EAyiYmg35ogqm3lTxV4sj0IjTR3HQmZZU3AHsDarR/D8TPBHOcQRabzKzcMRa1968Cjyf/wM=", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [QNT_PER_UNIT = _t, CANONICAL_FORM = _t]),
    #"Trimmed Text" = Table.TransformColumns(Source,{{"CANONICAL_FORM", Text.Trim, type text}}),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Trimmed Text", "CANONICAL_FORM", Splitter.SplitTextByEachDelimiter(
        {"-"}, QuoteStyle.Csv, false), {"PACK_QNT", "CANONICAL_FORM.2"}),
    #"Trimmed Text1" = Table.TransformColumns(#"Split Column by Delimiter",{{"CANONICAL_FORM.2", Text.Trim, type text}}),
    #"Split Column by Delimiter1" = Table.SplitColumn(#"Trimmed Text1", "CANONICAL_FORM.2", Splitter.SplitTextByEachDelimiter(
        {" "}, QuoteStyle.Csv, true), {"MEASURE", "PACKS"}),
    #"Changed Type" = Table.TransformColumnTypes(#"Split Column by Delimiter1",{{"PACK_QNT", Int64.Type}}),
    #"Added Index" = Table.AddIndexColumn(#"Changed Type", "PRODUCT_PACK_ID", 1, 1, Int64.Type),
    #"Reordered Columns" = Table.ReorderColumns(#"Added Index",{"PRODUCT_PACK_ID", "QNT_PER_UNIT", "PACK_QNT", "MEASURE", "PACKS"})
in
    #"Reordered Columns"
