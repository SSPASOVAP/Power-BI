---------------------------------------------------------------
    		----------- Clients View -----------
---------------------------------------------------------------

## Clients_Profile 
-- N_Customers:
DEFINE
	MEASURE 'Calcs-Matrix' [N Customers] = 
		COUNT(
			'Dim-Customers'[Customer_Key]
		)
EVALUATE
	SUMMARIZECOLUMNS(
		"@N cust", [N Customers]
	)

-- Age_Groups:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Age],
		"@N Clients", [N Customers]
		)
ORDER BY
	[@N Clients] DESC

----------------------------------------------------------------------
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Age_Range],
		"@N Clients", [N Customers]
		)
ORDER BY
	[@N Clients] DESC

-- Gender:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Gender],
		"@N Clients", [N Customers]
		)
ORDER BY
	[@N Clients] 

-- Work Spheres / Occupation:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Occupation],
		"@N Clients", [N Customers]
		)
ORDER BY
	[@N Clients] DESC 

-- Marital Status:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Marital_Status],
		"@N Clients", [N Customers]
		)

-- N_Kids:
EVALUATE
SUMMARIZECOLUMNS(
	'Dim-Customers'[Number_of_Children],
	"@N Clients", [N Customers]
)

-- Yearly Incomes:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Yearly_Income],
		"@N Clients", [N Customers]
		)
ORDER BY
 [@N Clients] DESC

-- Hous owning:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[House_Owner],
		"@N Clients", [N Customers]
		)

-- N_cars:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Cars_Owned],
		"@N Clients", [N Customers]
		)
ORDER BY
	[Cars_Owned]

## For how long they are our clients:

// DEFINE
// 	VAR MaxDate = MAX('Fact-Sales'[Order_Date])

// EVALUATE
// 	SELECTCOLUMNS(
// 		'Dim-Customers',
// 		'Dim-Customers'[Customer_Name],
// 		'Dim-Customers'[First_Order],
// 		"@Max Date", MaxDate,
// 	    "@Customership_Months", DATEDIFF(
//                 'Dim-Customers'[First_Order], MaxDate, MONTH) -- YEAR
//    		)
        				-- Add column in years--
NEW COLUMN: Customership_Years = DATEDIFF('Dim-Customers'[First_Order], MAX('Fact-Sales'[Order_Date]), YEAR)

        			 -- DAX calculation in months --
EVALUATE
	ADDCOLUMNS(
	    'Dim-Customers',
	    "@Customership_Months", DATEDIFF(
                'Dim-Customers'[First_Order], MAX('Fact-Sales'[Order_Date]), MONTH
                )
	        )
					-- DAX calculation in months vs2 --
EVALUATE
	SELECTCOLUMNS(
		'Dim-Customers',
		'Dim-Customers'[Customer_Name],
	    "@Customership_Months", DATEDIFF(
                'Dim-Customers'[First_Order], MAX('Fact-Sales'[Order_Date]), MONTH

   								)
	        )
	-- Calculation in years decrease distinct values down to 4, which give us better data compression and visualisation.
	-- This is the better option for adding a column, and with DAX we can do additional calcs in months, days etc.
	-- A small minus here is we do not clearly see the differances between using MAX Order_Date & MAX Calendar_Date as the result is in years.
    

        // Clients since:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Customership_Years],
		"@N Clients", [N Customers]
		)
ORDER BY
	[Customership_Years] 

## Additional base measures:

DEFINE
		-- N_products --
	MEASURE 'Calcs-Matrix' [N Items] = COUNT('Fact-Sales'[Order_ID])
		-- N_Orders --
	MEASURE 'Calcs-Matrix' [N Orders] = DISTINCTCOUNT('Fact-Sales'[Order_ID])
		-- $ Total Sales -- 
	MEASURE 'Calcs-Matrix' [Sum of Purch] = SUM('Fact-Sales'[Total_Amount])
	
EVALUATE
	SUMMARIZECOLUMNS(
		"@N Items", [N Items],
		"@N Orders", [N Orders],
		"@ Sum Purch", [Sum of Purch]
	)
