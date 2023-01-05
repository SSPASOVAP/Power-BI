
---------------------------------------------------------------
	--------- Some Useful Measures & Calculations ---------
---------------------------------------------------------------

## Additional Calculations

				// Clients by GEO, region and/or country:
EVALUATE
	SUMMARIZE(
		'Fact-Sales',
//		'Dim-Locations'[Geo_Group],
//		'Dim-Locations'[Region],
		'Dim-Locations'[Country],
		"@N clients", [N Customers]
	)
			// $ Purchases by months and total with sum accumulation for each Q:
	//1. 
DEFINE
	MEASURE 'Calendar' [Dates_Purch] = 
        VAR max_purch_date = MAX('Fact-Sales'[Order_Date])
        VAR min_calendar_date = MIN('Calendar'[Date])
        VAR purch_dates =  min_calendar_date <= max_purch_date
        RETURN
        	purch_dates
	//2.
EVALUATE
	SUMMARIZECOLUMNS(
		'Calendar'[Year],
		'Calendar'[Quarter],
		'Calendar'[Month],
		"@Sum$", [Sum of Purch],
		"@RT Qs",
			IF(
				[Dates_Purch],
					CALCULATE(
						[Sum of Purch],
						CALCULATETABLE(
						DATESQTD('Calendar'[Date])
					) --caltab
			)-- calc
		), --if
		"@RT Ys", 
			IF(
				[Dates_Purch],
					CALCULATE(
					[Sum of Purch],
					CALCULATETABLE(
					DATESYTD('Calendar'[Date])
				) --caltab
			)-- calc
		) --if
	) --summ

ORDER BY
	[Year]
,	[Month]

	// Purchases view per country and GEO group:
EVALUATE
SUMMARIZECOLUMNS(
	'Dim-Locations'[Country],
	"@Sum", [Sum of Purch],
	"@Sum Orders", [N Orders]
	)
ORDER BY
	[@Sum] DESC

-----------------------------------------------------
EVALUATE
SUMMARIZECOLUMNS(
	'Dim-Locations'[Geo_Group],
	"@Sum", [Sum of Purch],
	"@Sum Orders", [N Orders]
	)
ORDER BY
	[@Sum] DESC

		// AVG Client Yearly Incoms by occupation:
EVALUATE
SUMMARIZECOLUMNS(
	'Dim-Customers'[Occupation],
	"Sum Y_Incomes", SUM('Dim-Customers'[Yearly_Income]),
	"Avg Y_Incomes", AVERAGE('Dim-Customers'[Yearly_Income])
	)

		// What accupation are the customers with the highest value purchases?:
-- Professional & Skilled Manual represent more then 50% from all purchases $
DEFINE
	VAR TotalPurch = [Sum of Purch]
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Occupation],
		"@Sum$", [Sum of Purch],
		"@G-total", TotalPurch,
		"@Indestry Purch%", 
				ROUND(DIVIDE([Sum of Purch],TotalPurch) * 100, 2)		
	)
ORDER BY
	[@Indestry Purch%] DESC


		//From the customers with what income is formed "x" % of the total purchases?:
-- More then 50% from all purchases $ are formed by clients with yearly incomes between 30 и 70K.

DEFINE
 	VAR TotalPurch = [Sum of Purch]
 EVALUATE
 	SUMMARIZECOLUMNS(
 	'Dim-Customers'[Yearly_Income],
 	"@Sum$", [Sum of Purch],
 	"@G-total", TotalPurch,
 	"@Indestry Purch%", 
 				ROUND(DIVIDE([Sum of Purch],TotalPurch) * 100, 2)	
 		)
 ORDER BY
 	[@Indestry Purch%] DESC


-------------- Additional solutions  with adding a running total column: 

DEFINE      
    VAR total_purchases = [Sum of Purch]
EVALUATE
    ADDCOLUMNS(
        VALUES('Dim-Customers'[Yearly_Income]),
        "@Purchases", [Sum of Purch],
        "@RT", 
            VAR income = 'Dim-Customers'[Yearly_Income]
            RETURN
            CALCULATE(
                [Sum of Purch],
                CALCULATETABLE(
                    'Dim-Customers',
                    'Dim-Customers'[Yearly_Income] <= income
                )
            ),
        "@RT %",
            VAR income = 'Dim-Customers'[Yearly_Income]
            RETURN
            CALCULATE(
                [Sum of Purch],
                CALCULATETABLE(
                    'Dim-Customers',
                    'Dim-Customers'[Yearly_Income] <= income
                )
            ) / total_purchases

    )
ORDER BY
    [Yearly_Income]
```


//Create a measure:
DEFINE
    MEASURE 'Calcs-Matrix'[RT Purchases By Yearly Income] = 
             CALCULATE(
                    [Sum of Purch],
                    FILTER(
                        ALL('Dim-Customers'),
                        COUNTROWS(
                            FILTER(
                                'Dim-Customers',
                                EARLIER('Dim-Customers'[Yearly_Income]) <= 'Dim-Customers'[Yearly_Income]
                            )
                        )
                    )
                )
    VAR total_purchases = [Sum of Purch]
EVALUATE
    ADDCOLUMNS(
        VALUES('Dim-Customers'[Yearly_Income]),
        "@Purchases", [Sum of Purch],
        "@RT", [RT Purchases By Yearly Income],
        "@RT %",[RT Purchases By Yearly Income] / total_purchases

    )
ORDER BY
    [Yearly_Income]
```


// How many of the customers have respectively above/below the average yearly income for the sphere they are accouped?:
DEFINE
    VAR avg_income = AVERAGE('Dim-Customers'[Yearly_Income]) 
EVALUATE
    SUMMARIZECOLUMNS(
        'Dim-Customers'[Occupation],
        "N Customers < Avg Income", COUNTX( 'Dim-Customers', IF('Dim-Customers'[Yearly_Income] < avg_income,1) ),
        "N Customers >= Avg Income", COUNTX( 'Dim-Customers', IF('Dim-Customers'[Yearly_Income] >= avg_income,1) )
    )
```
---------------------------------

EVALUATE
SUMMARIZECOLUMNS(
	'Dim-Customers'[Occupation],
	"@Sum Y_Incomes", SUM('Dim-Customers'[Yearly_Income]),
	"@N Cust", [N Customers],
	"@Avg Y_Incomes", SUM('Dim-Customers'[Yearly_Income]) / [N Customers],
	"@N Customers < Avg Income", 			
			COUNTX( 
				'Dim-Customers', IF(
				'Dim-Customers'[Occupation] = 'Dim-Customers'[Occupation] &&
				'Dim-Customers'[Yearly_Income] < SUM('Dim-Customers'[Yearly_Income]) / [N Customers],1
								) -- If
			) -- count
) --summ