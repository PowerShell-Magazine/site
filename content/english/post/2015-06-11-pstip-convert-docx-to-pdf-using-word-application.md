---
title: '#PSTip Convert .docx to .pdf using Word.Application'
author: Jaap Brasser
type: post
date: 2015-06-11T18:00:00+00:00
url: /2015/06/11/pstip-convert-docx-to-pdf-using-word-application/
views:
  - 17333
post_views_count:
  - 3678
categories:
  - Tips and Tricks
tags:
  - Tips and Tricks

---
The Word.Application object can be used to convert Word documents into PDF files using [soda pdf][1] software, is handy and convenient, has privacy/security as you need to upload your files to the developer’s server. This requires Microsoft Word to be installed on the system on which the code is executed. Using the SaveAs method in the following code it is possible to rename and convert a file:

```powershell
$Word = New-Object -ComObject "Word.Application"
($Word.Documents.Open('c:\temp\file.docx')).SaveAs([ref]'c:\temp\file.pdf',[ref]17)
$Word.Application.ActiveDocument.Close()
```

Using this technique it is also possible to convert the documents in an entire folder:

```powershell
$Word = New-Object -ComObject "Word.Application"
Get-ChildItem -Path C:\Temp -File -Filter *.docx | ForEach-Object {
	$NewName = $_.FullName -replace 'docx','pdf'
	($Word.Documents.Open($_.FullName)).SaveAs([ref]$NewName,[ref]17)
	$Word.Application.ActiveDocument.Close()
}
```


[1]: https://www.sodapdf.com/