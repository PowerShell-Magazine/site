---
title: Natively Query CSV Files using SQL Syntax in PowerShell
author: Chrissy LeMaire
type: post
date: 2015-05-12T16:00:15+00:00
url: /2015/05/12/natively-query-csv-files-using-sql-syntax-in-powershell/
views:
  - 28493
post_views_count:
  - 7885
categories:
  - How To
  - SQL
tags:
  - How To
  - SQL

---
**Note:** The previous version of this article relied on the 32-bit JET engine. It has been updated to use ACE, which works in both 64-bit and 32-bit environments.

Using Microsoft&#8217;s ACE provider, PowerShell can natively query CSV files using SQL syntax. This technique is particularly useful when processing large CSV files that could exhaust your workstation&#8217;s memory if processed using Import-Csv.  For those of you already familiar with SQL syntax, it can also simplify queries. Consider the following pure PowerShell code vs. a SQL statement which produce the same results:

```sql
Import-Csv "$env:TEMP\top-tracks.csv" | Group-Object Artist | 
Select @{Name="Playcount";Expression={($_.group | 
Measure-Object -sum Playcount).sum}}, name | 
Where-Object { $_.Playcount -gt 5 -and $_.name -ne 'Fall Out Boy'} | 
Sort-Object Playcount -Descending | Select -First 20
$sql = "SELECT TOP 20 SUM(playcount) AS playcount, artist from [$tablename] WHERE artist <> 'Fall Out Boy' GROUP BY artist HAVING SUM(playcount) > 4 ORDER BY SUM(playcount) DESC"
```


Querying CSV files using SQL can be accomplished using either ODBC (OdbcConnection) or OLEDB (OleDbconnection). Below, we will explore querying CSV files with OLEDB.

![](/images/sql1.png)

Figure 1. Querying a CSV file using a [formalized version][1] of the scripting technique described in this article.

### Getting Started

To query a CSV file using a SQL statement, we&#8217;ll follow these basic steps:

  1. Build the connection string
  2. Create and open a new OleDbconnection object
  3. Create a new OleDBCommand, and associate it with the OleDbconnection object
  4. Execute the SQL statement
  5. Load the results into a data table
  6. Clean up

### Building the Connection String

In order to provide a working example, my top tracks from Last.FM in CSV format will be downloaded and saved to a temp directory. This CSV file, which contains over 1,300 rows, has four columns: Artist, Title, Rank, and Playcount,

```powershell
$csv = "$env:TEMP\top-tracks.csv"
Invoke-WebRequest http://git.io/vvzxA | Set-Content -Path $csv
$firstRowColumnNames = "Yes"
$delimiter = ","
```


The default delimiter for an OleDb connection is a comma. Other delimiters include semicolons, tabs (\`t), pipes (|), and fixed widths (FixedLength).

If you&#8217;re using any delimiter other than a comma, the [schema.ini][2] file must be used. Microsoft&#8217;s website suggests that FMT=Delimiter($delimiter) within the connection string works, but that has not been my experience. If you are using an alternative delimiter, use the following code to create a basic schema.ini file.

```powershell
if ($delimiter -ne ",") {
	$filename = Split-Path $csv –leaf
	Set-Content -Path schema.ini -Value "[$filename]"
	Add-Content -Path schema.ini -Value "Format=Delimited($delimiter)"
}
```


If you&#8217;re using fixed width, check out [Mow&#8217;s article on this topic][3] for more information, since a lot more information is required within the schema.ini file.

The connection string for the OleDb connection is a little different from the more familiar SQL Server connection string. It requires a Data Source, which is the directory that the CSV file resides within. Since $csv contains this information already, we&#8217;ll parse it for that information using Split-Path. You must also specify whether the first row contains the headers. This is answered with a “Yes” or “No” by the $firstRowColumnNames variable. Note that if the first line of your CSV file does not contain the column names, OleDbconnection defaults to using F1, F2, F3, and so on for column/field names.

First, we will select the ACE provider. If more than one ACE provider version is returned, we’ll just use the first one. Then, we’ll use the resulting provider name within the connection string.

```powershell
$provider = (New-Object System.Data.OleDb.OleDbEnumerator).GetElements() | Where-Object { $_.SOURCES_NAME -like "Microsoft.ACE.OLEDB.*" } 

if ($provider -is [system.array]) { $provider = $provider[0].SOURCES_NAME } else {  $provider = $provider.SOURCES_NAME }
$connstring = "Provider=$provider;Data Source=$(Split-Path $csv);Extended Properties='text;HDR=$firstRowColumnNames;';"
```

Next, the table name, which is called within the actual SQL statement, is the CSV file&#8217;s name with the period replaced by a hashtag.

```powershell
$tablename = (Split-Path $csv -leaf).Replace(".","#")
```

### Crafting the SQL Statement

First, we&#8217;ll create a mildly complex SQL statement that uses SELECT, SUM, WHERE, GROUP BY, HAVING, and ORDER BY. Note that the query is case-sensitive, so take special care when using the WHERE clause.

```sql
$sql = "SELECT TOP 20 SUM(playcount) AS playcount, artist from [$tablename] WHERE artist <>; 'Fall Out Boy' GROUP BY artist HAVING SUM(playcount) > 4 ORDER BY SUM(playcount) DESC, artist"
```


Upon executing this query, I expect to see the top 20 artists that I&#8217;ve listened to, ranked by the number of times I&#8217;ve played their songs, but I should not see Fall Out Boy in my results, or any artists that I&#8217;ve listened to less than five times.

If you&#8217;re looking to test a simple SQL query, the following statement also works well:

```powershell
$sql = "SELECT top 10 * from table"
```


A good resource for learning more about the supported SQL syntax is [Microsoft Access&#8217; SQL reference page][4], since it’s the same engine. If you&#8217;re wondering if you can DELETE or UPDATE records within the CSV, the answer appears to be no. You can, however, manipulate the resulting data table output, and re-export to CSV, as seen in the section titled “Data subset and UPDATE example.”

### Setting up the Connection and Executing the SQL statement

In order to execute the SQL statement, an OleDbconnection object must be created, opened and associated with a new OleDBCommand object. Once the OleDBCommand&#8217;s CommandText is set to the $sql variable, we will create a DataTable object and fill it with the results.

```powershell
# Setup connection and command
$conn = New-Object System.Data.OleDb.OleDbconnection
$conn.ConnectionString = $connstring
$conn.Open()
$cmd = New-Object System.Data.OleDB.OleDBCommand
$cmd.Connection = $conn
$cmd.CommandText = $sql
# Load into datatable
$dt = New-Object System.Data.DataTable
$dt.Load($cmd.ExecuteReader("CloseConnection"))
# Clean up
$cmd.dispose | Out-Null; $conn.dispose | Out-Null
# Output results
$dt | Format-Table -AutoSize
```


![](/images/sql2.png)

Figure 2. Script output

$cmd.ExecuteReader returns System.Data.Common.DbDataReader, and the easiest way to work with this in PowerShell is to place these results inside a DataTable. The CloseConnection behavior is passed to ensure that the OleDbconnection is closed once the DbDataReader object is closed.

### Bringing it all together

To see the script in action, just copy and paste the code below:

```powershell
$provider = (New-Object System.Data.OleDb.OleDbEnumerator).GetElements() | Where-Object { $_.SOURCES_NAME -like "Microsoft.ACE.OLEDB.*" }
if ($provider -is [system.array]) { $provider = $provider[0].SOURCES_NAME } else {  $provider = $provider.SOURCES_NAME }
$csv = "$env:TEMP\top-tracks.csv"
$connstring = "Provider=$provider;Data Source=$(Split-Path $csv);Extended Properties='text;HDR=$firstRowColumnNames;';"
Invoke-WebRequest http://git.io/vvzxA | Set-Content -Path $csv
$firstRowColumnNames = "Yes"
$delimiter = ","
$tablename = (Split-Path $csv -leaf).Replace(".","#")
$sql = "SELECT TOP 20 SUM(playcount) AS playcount, artist from [$tablename] WHERE artist <> 'Fall Out Boy' GROUP BY artist HAVING SUM(playcount) > 4 ORDER BY SUM(playcount) DESC, artist"
# Setup connection and command
$conn = New-Object System.Data.OleDb.OleDbconnection
$conn.ConnectionString = $connstring
$conn.Open()
$cmd = New-Object System.Data.OleDB.OleDBCommand
$cmd.Connection = $conn
$cmd.CommandText = $sql
# Load into datatable
$dt = New-Object System.Data.DataTable
$dt.Load($cmd.ExecuteReader("CloseConnection"))
# Clean up
$cmd.dispose | Out-Null; $conn.dispose | Out-Null
# Output results
$dt | Format-Table -AutoSize
```

### Data subset and UPDATE example

You can also grab subsets of data and then manipulate the data within the resulting datatable. Here&#8217;s how:

```powershell
$sql = "SELECT artist, title from [$tablename] WHERE artist = 'Bastille' or artist = 'Jeff Beal'"

# Setup connection and command
$conn = New-Object System.Data.OleDb.OleDbconnection
$conn.ConnectionString = $connstring
$conn.Open()
$cmd = New-Object System.Data.OleDB.OleDBCommand
$cmd.Connection = $conn
$cmd.CommandText = $sql

# Load into datatable
$dt = New-Object System.Data.DataTable
$dt.Load($cmd.ExecuteReader("CloseConnection"))

# Clean up
$cmd.dispose | Out-Null; $conn.dispose | Out-Null

# UPDATE example
$rows = $dt.Select("artist = 'Jeff Beal'")
foreach ($row in $rows) {
  $row["artist"] = "House of Cards Soundtrack"
}

$dt | Export-CSV  "$env:TEMP\updated-top-tracks.csv" -NoTypeInformation
&"$env:TEMP\updated-top-tracks.csv"
```

### Other Use Cases

This approach is especially useful when working with larger CSV files that would potentially exhaust a machine&#8217;s memory if Import-Csv were used. I have employed a similar technique to [quickly find duplicates][5] in large CSV files, and once identified over 9,000 duplicates within a million row CSV in only 6.4 seconds.

If your resulting datatable is a reasonable size, you can also use it in conjunction with SqlBulkCopy to efficiently populate SQL Server tables with subsets of CSV data. When uploading larger datasets, I recommend using [a paged approach][6] instead, in order to avoid memory exhaustion.

#### Invoke-CsvSqlcmd.ps1

You can find a formalized version of this technique within a script called [Invoke-CsvSqlcmd.ps1][1], which is available on Microsoft Script Center. Invoke-CSVSQLcmd.ps1 features a seamless shell switch and straightforward delimiter support. The syntax has been simplified, making querying a CSV file as simple as:

```powershell
.\Invoke-CsvSqlcmd.ps1 –csv file.csv –sql "select * from table"
```


Check out [Invoke-CsvSqlcmd.ps1 on Script Center][1] for more information.

### Conclusion

PowerShell can natively query CSV files using SQL syntax, which drastically increases performance when working with large files and complex queries.

[1]: https://gallery.technet.microsoft.com/scriptcenter/Query-CSV-with-SQL-c6c3c7e5
[2]: https://msdn.microsoft.com/en-us/library/ms709353%28v=vs.85%29.aspx
[3]: http://mow001.blogspot.be/2006/07/working-with-fixed-length-delimited.html
[4]: https://msdn.microsoft.com/en-us/library/office/dn123881.aspx
[5]: https://blog.netnerds.net/2015/01/quickly-find-duplicates-from-csv-using-powershell/
[6]: https://gallery.technet.microsoft.com/scriptcenter/Import-Large-CSVs-into-SQL-216223d9