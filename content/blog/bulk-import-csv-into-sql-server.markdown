---
title: "Bulk Import CSV Files Into SQL Server Using SQLBulkCopy and CSVHelper"
date: 2019-07-30
categories:
- Programming
comments: true
---

My recent project got me back into some long lost technologies, including Excel sheets, vb scripts, Silverlight, bash scripts, and whatnot. Amongst one of the things was a bash script that imported data from different CSV files to a data store. There were 40-50 different CSV schemas mapped to their corresponding tables to import. I had to repoint these scripts to write to a SQL Server.

### Challenges with BCP Utility

The [bcp Utility](https://docs.microsoft.com/en-us/sql/tools/bcp-utility?view=sql-server-2017) is one of the best possible options to use from a command line to bulk import data from CSV files. However, the bcp utility requires the fields (columns) in the CSV data file to match the order of the columns in the SQL table. The columns counts must also match. Both were not the case in my scenario. The way that you can work around this using bcp is by providing a [Format File](https://docs.microsoft.com/en-us/sql/relational-databases/import-export/use-a-format-file-to-map-table-columns-to-data-file-fields-sql-server?view=sql-server-2017). 

> - CSV file columns must match order and count of the SQL table
- Requires a Format File otherwise 
    - Static generation is not extensible
    - Dynamic generation calls for a better programming language 

Format File has an XML and a Non-XML variant, that can be pre-generated or dynamically generated using code. I did not want to pre-generate the format file because I do not own the generation of the CSV files. There could be more files, new columns, and order of columns can change. Dynamic generation involved a lot more work and bash scripts didn't feel like the appropriate choice. These made me to rewrite that code.

### SqlBulkCopy

CSharp being my natural choice for programming language and having excellent support for [SQLBulkCopy](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlbulkcopy?view=netframework-4.8), I decided to rewrite the existing bash script. SQLBulkCopy is the equivalent of bcp utility and allows to write managed code solutions for similar functionality. SQLBulkCopy can only write data to SQL Server. However, any data source can be used as long as it can be loaded into a DataTable instance.

The [**ColumnMappings**](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlbulkcopy.columnmappings?view=netframework-4.8#System_Data_SqlClient_SqlBulkCopy_ColumnMappings) property defines the relationship between columns in the data source and the SQL table. It can be dynamically added reading the headers of the CSV data file and adding them. In my case, the names of the column were the same as that of the table. My first solution involved reading the CSV file and splitting data using comma (",").

``` csharp
var lines = File.ReadAllLines(file);
if (lines.Count() == 0) 
    return;

var tableName = GetTableName(file);
var columns = lines[0].Split(',').ToList();
var table = new DataTable();
sqlBulk.ColumnMappings.Clear();

foreach (var c in columns)
{
    table.Columns.Add(c);
    sqlBulk.ColumnMappings.Add(c, c); 
}

for (int i = 1; i < lines.Count() - 1; i++)
{
    var line = lines[i];
    // Explicitly mark empty values as null for SQL import to work
    var row = line.Split(',')
        .Select(a => string.IsNullOrEmpty(a) ? null : a).ToArray();
    table.Rows.Add(row);
}

sqlBulk.DestinationTableName = tableName;
sqlBulk.WriteToServer(table);
```

CSV file may have empty columns where the associated column in SQL table is NULLABLE.  SQLBulkCopy throws errors like below, depending on the column type and if the value is an empty string.
*<span style='color:red'>The given value of type String from the data source cannot be converted to type \<TYPENAME> of the specified target column</span>*

Explicitly marking the empty values as null (as in the code above) solves the problem.

### CSV File Gotchas

The above code worked all file until there were some CSV files which had comma as a valid value for few of the columns, as shown below.

``` csv
Id,Name,Address,Qty
1,Rahul,"Castlereagh St, Sydney NSW 2000",10
```

Splitting on comma no longer works as expected, as it also splits the address into two-part. Using a NuGet package for reading the CSV data made more sense and decided to switch to [CSVHelper](https://github.com/JoshClose/CsvHelper). Even though CSVHelper is intended to work with strong types, it does work with generic types  - thanks to dynamics. Generating types for each CSV format is equivalent to generating a format file required for the bcp utility. 

``` csharp
List<dynamic> rows;
List<string> columns;
using (var reader = new StreamReader(file))
using (var csv = new CsvReader(reader))
{
    rows = csv.GetRecords<dynamic>().ToList();
    columns = csv.Context.HeaderRecord.ToList();
}

if (rows.Count == 0)
    return;

var table = new DataTable();
sqlBulk.ColumnMappings.Clear();

foreach (var c in columns)
{
    table.Columns.Add(c);
    sqlBulk.ColumnMappings.Add(c, c);
}

foreach (IDictionary<string, object> row in rows)
{
    var rowValues = row.Values
        .Select(a => string.IsNullOrEmpty(a.ToString()) ? null : a)
        .ToArray();
    table.Rows.Add(rowValues);
}

sqlBulk.DestinationTableName = tableName;
sqlBulk.WriteToServer(table);
```

With the CSVHelper added in I am now successfully able to import the different CSV file formats into their corresponding tables in SQL Server. Do add in the required error handling around the above functionality. Also, take note of the [transaction behavior](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/sql/transaction-and-bulk-copy-operations) of SQLBulkCopy. By default, each operation is performed as an isolated operation. On failure, no records are written to the database. A managed transaction can be passed in to manage multiple database operations. 

Hope this helps!