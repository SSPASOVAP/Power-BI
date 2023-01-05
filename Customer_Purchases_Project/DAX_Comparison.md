## Compare all purchases per month for the period of 2014 - 2016:
                        --------------- Basic Calc -------------------
EVALUATE
	SUMMARIZE(
		'fact-purchases',
		'Calendar'[Month],
		'Calendar'[Year],
		"@Purches", DISTINCTCOUNT('fact-purchases'[ORDER_ID])
		)
ORDER BY
	[Month]
,	[Year]



                        -------------- Measure LYM Calcs -----------------
						--- Same calculated for $ Amaunts, Items & Units ---
DEFINE
	MEASURE 'Purch-Matrix'[Orders LYM] = 
		CALCULATE(
				[N Orders],
                CALCULATETABLE(
                                -- table --	
                SAMEPERIODLASTYEAR('Calendar'[Dates]),
                                -- filter --
                'Calendar'[Purch Dates] = TRUE 
                        ) -- calc tab
	        ) -- calc
	MEASURE 'Purch-Matrix'[Orders MoM] = 
		IF ( 
                NOT ISBLANK([N Orders])
                        && 
                NOT ISBLANK([Orders LYM]),
                [N Orders] - [Orders LYM]
                )
	MEASURE 'Purch-Matrix' [Orders MoM %] = 
	 	DIVIDE([Orders MoM],[Orders LYM])

EVALUATE
	SUMMARIZECOLUMNS(
		'Calendar'[Year],
		'Calendar'[Month],
		"@ Orders",[N Orders],
		"@ Orders LYM",[Orders LYM],
		"@ Orders MoM", [Orders MoM],
		"@ Orders MoM %", [Orders MoM %]
		)
		
ORDER BY
	[Year],
	[Month]


## For each year identify the days of the week with the most items purchased no matter what quantities are purchased:
		------ Step 1 -----
DEFINE
	VAR dayofweek_items = SUMMARIZECOLUMNS(
	'Calendar'[Year],
	'Calendar'[Day of Week],
	"@ items per dayofweek", [N Items],
	"@ dayofweek items rank", RANKX(
						ALL('Calendar'[Day of Week]),
						[N Items],,
						DESC, DENSE)
						)
EVALUATE
FILTER(
	dayofweek_items,
	[Year] <> BLANK()
		&&
	[@ dayofweek items rank] = 1
	)

		------ Step 2 -----

DEFINE
	MEASURE'Purch-Matrix' [Items DayOfWeek Rank] =
						RANKX(
						ALL('Calendar'[Day of Week]),
						[N Items],,
						DESC, DENSE)
						
EVALUATE
	SUMMARIZECOLUMNS(
	'Calendar'[Year],
	'Calendar'[Day of Week],
	"@ items per dayofweek",[N Items],
	"@ Rank", [Items DayOfWeek Rank]
	)


					------------- Same calculation including the name of the day --------------
DEFINE  
	VAR dayofweek_items = SUMMARIZECOLUMNS(
		'Calendar'[Year],
		'Calendar'[Day Name],
		"@ items per dayofweek", [N Items],
		"@ dayofweek items rank", RANKX(
							ALL('Calendar'[Day Name]),
							[N Items],,
							DESC, DENSE)
						)
EVALUATE
FILTER(
	dayofweek_items,
	[Year] <> BLANK()
		&&
	[@ dayofweek items rank] = 1
	)

			
## For 2015, compare the purchased products by month and determine the difference current previous month
and the percentage change in quantities:

DEFINE
	MEASURE 'Purch-Matrix' [Qnt of Units LM] = 
		CALCULATE(
			-- exp
			[N Units], 						
			-- calc filter
			CALCULATETABLE(  
				-- calcT tab
				-- PARALLELPERIOD works for a single year, but DATEADD must be used for comparing the entire period [2014-2016]
				PARALLELPERIOD('Calendar'[Dates], -1, MONTH), 
				-- filter
				'Calendar'[Purch Dates] = TRUE
			) -- calc tab
		) -- calc
		
	MEASURE 'Purch-Matrix' [Qnt of Units MoM] = 
		IF(
			NOT ISBLANK([N Units])
				&& 
			NOT ISBLANK([Qnt of Units LM]),
			[N Units] - [Qnt of Units LM]
		) -- if

	MEASURE 'Purch-Matrix' [Qnt of Units MoM %] =
		 DIVIDE(
			[Qnt of Units MoM],[Qnt of Units LM]
			)

EVALUATE
	SUMMARIZECOLUMNS(
		'Calendar'[Year],
		'Calendar'[Month],
	FILTER('Calendar',
		'Calendar'[Year] = 2015),
		"@ Qnt per month",[N Units],
		"@ Units Qnt LM", [Qnt of Units LM],
		"@ Units Qnt MoM", [Qnt of Units MoM],
		"@ Units Qnt MoM %", [Qnt of Units MoM %]
	) -- summ
	
ORDER BY
	[Year],
	[Month]


## Compare the average number of days for delivery of an order with the same period the previous year:

DEFINE
	MEASURE 'Purch-Matrix'[MidTime Del LYM] = 
		TRUNC(CALCULATE(
				-- exp
		[Mid Days Delivery],
				--filter
			CALCULATETABLE(
				--tab
			SAMEPERIODLASTYEAR('Calendar'[Dates]),
				--filter
			'Calendar'[Shipping Dates] = TRUE
			),--calcT
		USERELATIONSHIP('Calendar'[Dates],'fact-purchases'[SHIPPED_DATE])
			) --calc
		)--trunc
	MEASURE 'Purch-Matrix'[MidTime Del MvsM] = 
		IF(
			NOT ISBLANK([Mid Days Delivery])
				&&
			NOT ISBLANK([MidTime Del LYM]),
			[Mid Days Delivery] - [MidTime Del LYM]
			)
	MEASURE 'Purch-Matrix'[MidTime Del MvsM %] = 
			DIVIDE([MidTime Del MvsM], [MidTime Del LYM]) 

EVALUATE
	CALCULATETABLE(
		SUMMARIZECOLUMNS(
			'Calendar'[Year],
			'Calendar'[Month],
			"@ Mid Time Del", [Mid Days Delivery],
			"@ MidTime Del LYM", [MidTime Del LYM],
			"@ MidTime Del M-M", [MidTime Del MvsM],
			"@ MidTime Del M-M %", [MidTime Del MvsM %]
		), --summ 
		USERELATIONSHIP(
			'fact-purchases'[SHIPPED_DATE],
			'Calendar'[Dates]
		)
	)
ORDER BY
	[Year]
	,[Month]
