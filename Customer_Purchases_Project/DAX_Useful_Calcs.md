## Additional Useful Calcs 
	--- Total Amaunt per Order ---
EVALUATE                       
        SUMMARIZE(
                'fact-purchases',
                'fact-purchases'[ORDER_ID],
                "@Order Amaunt", SUM('fact-purchases'[TOTAL])
                )
```

	--- Delivered itesm according to packaging ---
EVALUATE                      
        SUMMARIZE(
        		'dim-products',
                'dim-products'[PACKS],
                "@Items per packs", [N Items]
                )
```

	--- % Purchasess$ per PACKS ---
DEFINE				
	VAR Grand_Total = [Sum Purchases $]
	
EVALUATE	                       
	SUMMARIZECOLUMNS(
	'dim-products'[PACKS],
		"@ Sum Purch", [Sum Purchases $],
		"@ G Total", Grand_Total,
		"@ % Per Pack", ROUND(DIVIDE([Sum Purchases $], Grand_Total) * 100, 2)) --- Round is used only for DAX View
```

	--- Purchased unites by packs ---
EVALUATE                
	SUMMARIZECOLUMNS(
		'dim-products'[PACKS],
		"@ Units per Pack" ,[N Units]
		)
    
    				
	--- Products RANKS Calculation ---
DEFINE
MEASURE 'Purch-Matrix' [Prod Purch Rank $] =
        RANKX(
            ALL('dim-products'[PRODUCT_NAME]),
            [Sum Purchases $],,
            DESC, DENSE
            )
				
MEASURE 'Purch-Matrix' [Prod Purch Times Rank] = 
		RANKX
			(
			ALL('dim-products'[PRODUCT_NAME]),
			[N Orders],,
			DESC, DENSE
			)
			
MEASURE 'Purch-Matrix' [Prod Qnt Rank] = 
        RANKX(
            ALL('dim-products'[PRODUCT_NAME]),
            [N Units],,
            DESC, DENSE
            )

EVALUATE
	SUMMARIZECOLUMNS(
		'dim-products'[PRODUCT_NAME],
		"@ Purch", [Sum Purchases $],
		"@ Orders", [N Orders], -- in how many orders a single product was included, each product is reflected once per order =>
		"@ N Items", [N Items], --> N Items = N Orders
		"@ N Units", [N Units], 
		"@ Product Purch Rank $", [Prod Purch Rank $],
		"@ Product Purch T Rank", [Prod Purch Times Rank],
		"@ Prod Qnt Rank", [Prod Qnt Rank]
	    )
	
ORDER BY
	[PRODUCT_NAME]
```

	--- Packs RANKS Calculation ---
DEFINE
MEASURE 'Purch-Matrix' [Pack Purch Rank $] =
       RANKX(
           ALL('dim-products'[PACKS]),
           [Sum Purchases $],,
           DESC, DENSE
           )
				
MEASURE 'Purch-Matrix' [Pack Purch Times Rank] = 
		RANKX
			(
			ALL('dim-products'[PACKS]),
			[N Orders],,
			DESC, DENSE
			)

MEASURE 'Purch-Matrix' [Pack Items Rank] = 
	IF ( 
	ISINSCOPE('dim-products'[PACKS]),
		RANKX
			(
			ALL('dim-products'[PACKS]),
			[N Items],,
			DESC, DENSE),
		"Pack must be in scope"  -- rest od the measures are supported by integral expressions
		)
			
MEASURE 'Purch-Matrix' [Pack Qnt Rank] = 
       RANKX(
           ALL('dim-products'[PACKS]),
           [N Units],,
           DESC, DENSE
           )

EVALUATE
	SUMMARIZECOLUMNS(
		'dim-products'[PACKS],
		"@ Purch", [Sum Purchases $],
		"@ Orders", [N Orders], -- How many orders have products delivered in the respective packaging (could be more than 1 per order)
		"@ N items", [N Items], -- How many total products were delivered in the respective package => N Items <> N Orders
		"@ N Units", [N Units],
		"@ Purch Rank", [Pack Purch Rank $],
		"@ N Ords Rank", [Pack Orders Rank],
		"@ Items Ranks", [Pack Items Rank],
		"@ Qnt Rank", [Pack Qnt Rank]
		)
ORDER BY
	[PACKS]
```

	--- Prod & Packs Rank Integral Measures ---
		-- In case both are in scope, PACK is the bigger step
DEFINE
	MEASURE 'Purch-Matrix' [Prod and Pack Purch Rank$] =
		SWITCH(
			TRUE,
			ISINSCOPE('dim-products'[PACKS]), [Pack Purch Rank $],
			ISINSCOPE('dim-products'[PRODUCT_NAME]), [Prod Purch Rank $],
			"Product or Pack must be in scope"
			)
			
	MEASURE 'Purch-Matrix' [Prod and Pack Orders Rank] =
		SWITCH(
			TRUE,
			ISINSCOPE('dim-products'[PACKS]), [Pack Orders Rank],
			ISINSCOPE('dim-products'[PRODUCT_NAME]), [Prod Purch Times Rank],
			"Product or Pack must be in scope"
			)

	MEASURE 'Purch-Matrix' [Prod and Pack Qnt Rank] =
		SWITCH(
			TRUE,
			ISINSCOPE('dim-products'[PACKS]), [Pack Qnt Rank],
			ISINSCOPE('dim-products'[PRODUCT_NAME]), [Prod Qnt Rank],
			"Product or Pack must be in scope"
			)


EVALUATE
	SUMMARIZECOLUMNS(
		'dim-products'[PRODUCT_NAME],
		"@ Purch", [Sum Purchases $],
		"@ Orders",[N Orders],
		"@ Qnt", [N Units],
		"@ Purch Rank", [Prod and Pack Purch Rank$],
		"@ Ords Rank", [Prod and Pack Orders Rank],
		"@ Qnt Rank", [Prod and Pack Qnt Rank]
		)
		
ORDER BY
	[@ Purch] DESC
```

	--- N-Days Back View ---
DEFINE
	MEASURE 'Purch-Matrix'[N-days back purch $] = 
		VAR n_days = 5
		VAR max_date = MAX('Calendar'[Dates])
	RETURN
		CALCULATE(
			[Sum Purchases $],
			CALCULATETABLE(
			VALUES('Calendar'[Dates]),
			'Calendar'[Dates] > max_date - n_days
				&&
			'Calendar'[Dates] <= max_date
			) 
		)

EVALUATE
	SUMMARIZECOLUMNS(
	'Calendar'[Year],
	'Calendar'[Month],
	'Calendar'[Day],
	"@ purch", [Sum Purchases $],
	"@ n-days back view", [N-days back purch $]
		
	) 
ORDER BY
	[Year],
	[Month],
	[Day]							
```

	--- Working Days VS Wekends ---
DEFINE
	MEASURE 'Purch-Matrix'[Working Days Purches] = 
			CALCULATE(
				[Sum Purchases $],
				FILTER('Calendar',
				'Calendar'[Day of Week] >= 1 
					&&
				'Calendar'[Day of Week] <= 5)
					) -- calc
	MEASURE 'Purch-Matrix'[Weekend Purches] = 
			VAR sat = 6 
			VAR sun = 0 
			VAR nonworking =
				CALCULATETABLE(
				VALUES('Calendar'[Dates]),
				'Calendar'[Day of Week] IN {sat, sun}
				) --calcTab
			RETURN
				SUMX(
				nonworking,
				[Sum Purchases $]
				)

EVALUATE
	SUMMARIZECOLUMNS(
		'dim-customers'[CUSTOMER_CODE],
		"@ Total Purch", [Sum Purchases $],
		"@ Purch WD", [Working Days Purches],
		"@ Purch Weekend",[Weekend Purches],
		"@ Quick Check", [Working Days Purches] + [Weekend Purches]	
		) 
		
ORDER BY
[CUSTOMER_CODE]