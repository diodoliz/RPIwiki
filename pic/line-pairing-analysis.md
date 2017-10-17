<!-- TITLE: Line Pairing Analysis -->
<!-- SUBTITLE: A quick summary of Line Pairing Analysis -->

# Line Pairing Analysis
Some usefull queries for LP analysis.

## 1 - Look at the LP rates
See how successfull material numbers are extracted. PICI does not need to capture the Material Number to Pair the line. Description and Price iformation works for pairing most of the time. When you see 

```sql
-- Show line pairing stats by Vendor. This query also shows extraction rates. Low % of LP and High Blank rate is good indication to add some formats to BRWMAT table
Select 
	BRWdocument.SOURCE_ID,
	FIELDNAME,
	COUNT(1) as NumberOfInvoices,
	-- Show Line Pairing Stats (These are usefull to see who well solution pairs PO lines)
	SUM(LP_TOTAL) as LPTotalLines,
	SUM(LP_PAIRED) as LPPairedLines,
	SUM(LP_TOTAL - LP_PAIRED) as LPUnPairedLines,
	CASE WHEN SUM(LP_TOTAL) > 0 THEN 100.00 * SUM(LP_PAIRED) / SUM(LP_TOTAL) ELSE 100 END as LPPairedPercent,
	-- Show How many times Lines are actually extracted
	SUM(TotalLines) as ExtTotalLines,
	SUM(BlankLines) as ExtBlankLines,
	SUM(NonBlankLines) as ExtNonBlankLines,
	CASE WHEN SUM(TotalLines) > 0 THEN 100.00 * SUM(BlankLines) / SUM(TotalLines) ELSE 100 END as ExtBlankLinesPercent

from BRWdocument
join (	Select 
			DOCUMENTNUMBER,
			FIELDNAME,
			COUNT(1) as TotalLines,
			SUM(CASE WHEN CONTENTRTS = '' Then 1 else 0 END) as BlankLines,
			SUM(CASE WHEN CONTENTRTS = '' Then 0 else 1 END) as NonBlankLines
		FROM BRWdistillerFields
		WHERE FIELDNAME in ('LineItems.MaterialNo')
		GROUP BY DOCUMENTNUMBER,FIELDNAME
	  ) as BRWdistillerFieldsStats on BRWdocument.DOCUMENTNUMBER = BRWdistillerFieldsStats.DocumentNumber
Where 
		CURRENT_STATUS = 800
	and DATEEXPORTRTS > '2017-01-01' 
	and CLASSNAMEV <> 'VOID'  
GROUP BY BRWdocument.SOURCE_ID, FIELDNAME
ORDER BY SUM(LP_TOTAL) DESC

```

## 2 - See what lines are paried on the invoices, and what lines are not paired. This query shows exactly which lines where output unpaired.

```
--Identify Paried lines on the invoice
Select 
	BRWdocument.DOCUMENTNUMBER	as DocID,
	PONumber.ROWNUMBER			as InvoiceRow,
	PONumber.CONTENTRTS			as PairedToPO,
	POLineNumber.CONTENTRTS		as PairedToLine,
	LineDesc.CONTENTRTS			as LineDescription,
	LineUnitPrice.CONTENTRTS	as LineUnitPrice,
	LineQuantitiy.CONTENTRTS	as LineQuantitiy,
	LineTotal.CONTENTRTS		as LineTotal
from BRWdocument
left join BRWdistillerFields PONumber on BRWdocument.DOCUMENTNUMBER = PONumber.DOCUMENTNUMBER and PONumber.FIELDNAME = 'LineItems.PO'
left join BRWdistillerFields POLineNumber on BRWdocument.DOCUMENTNUMBER = POLineNumber.DOCUMENTNUMBER and POLineNumber.FIELDNAME = 'LineItems.Line' and PONumber.ROWNUMBER = POLineNumber.ROWNUMBER
left join BRWdistillerFields LineDesc on BRWdocument.DOCUMENTNUMBER = LineDesc.DOCUMENTNUMBER and LineDesc.FIELDNAME = 'LineItems.Description' and PONumber.ROWNUMBER = LineDesc.ROWNUMBER
left join BRWdistillerFields LineUnitPrice on BRWdocument.DOCUMENTNUMBER = LineUnitPrice.DOCUMENTNUMBER and LineUnitPrice.FIELDNAME = 'LineItems.UnitPrice' and PONumber.ROWNUMBER = LineUnitPrice.ROWNUMBER
left join BRWdistillerFields LineQuantitiy on BRWdocument.DOCUMENTNUMBER = LineQuantitiy.DOCUMENTNUMBER and LineQuantitiy.FIELDNAME = 'LineItems.Quantity' and PONumber.ROWNUMBER = LineQuantitiy.ROWNUMBER
left join BRWdistillerFields LineTotal on BRWdocument.DOCUMENTNUMBER = LineTotal.DOCUMENTNUMBER and LineTotal.FIELDNAME = 'LineItems.Total' and PONumber.ROWNUMBER = LineTotal.ROWNUMBER
where BRWdocument.DOCUMENTNUMBER = '<DocID>'
```