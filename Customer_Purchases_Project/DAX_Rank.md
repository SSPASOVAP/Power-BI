## Find top ten clients having the highest amount of purchases:

DEFINE
 	MEASURE 'Purch-Matrix' [Purch Client Rank] = 
 		IF(
	 	    ISINSCOPE(
                'dim-customers'[CUSTOMER_CODE]),
				RANKX(
				ALL('dim-customers'),
				[Sum Purchases $],,
				DESC, DENSE),
				"Customer Code is missing"
	  		)
VAR p_ranks =
  	SUMMARIZECOLUMNS(
  		'dim-customers'[CUSTOMER_CODE],
  		"@ Purch", [Sum Purchases $],
  		"@ Purch Rank", [Purch Client Rank]
  		)
EVALUATE
	FILTER(
  	p_ranks, 
  	[@ Purch Rank] <= 10
  	)
ORDER BY
    [@ Purch Rank]


## Find top five clients with highest count of orders per country:

DEFINE
 	MEASURE 'Purch-Matrix' [Orders Client Rank] = 
 		IF(
	 		ISINSCOPE(
            	'dim-customers'[CUSTOMER_CODE]),
	 			RANKX(
                ALL('dim-customers'),
	  			[N Orders],,
	  			DESC, DENSE),
	  		     "Customer Code is missing"
	  	)
VAR o_ranks = 
FILTER(
  	SUMMARIZECOLUMNS(
                'dim-locations'[COUNTRY],
  		'dim-customers'[CUSTOMER_CODE],
  		"@ Orders", [N Orders],
  		"@ Orders Rank", [Orders Client Rank]
  		), [@ Orders] <> BLANK()
                )
EVALUATE
    FILTER(
		o_ranks,
		[@ Orders Rank] <= 5
	)   
	
ORDER BY
	[COUNTRY],
	[@ Orders] DESC


## Find which are the top three countries having the shortest average time for delivery:

DEFINE
	MEASURE 'Purch-Matrix' [Country MidTime Delivery Rank] = 
		IF(
			ISINSCOPE(
				'dim-locations'[COUNTRY]),
			RANKX(
				ALL('dim-locations'[COUNTRY]),
		 		[Mid Days Delivery],,
	 			ASC, DENSE),
	 			        "Country is missing"
	 	)
VAR country_delivery =
	SUMMARIZECOLUMNS(
		'dim-locations'[COUNTRY],
		"@ Delivery Time", ROUND([Mid Days Delivery],2),
		"@ Delivery Time Rank", [Country MidTime Delivery Rank]
 	)
EVALUATE
	-- option 1 --
	// FILTER(
	// 	country_delivery,
	// 	[@ Delivery Time Rank] <= 3
	// )
	-- option2 --
	TOPN(
		3,
		country_delivery,
		[@ Delivery Time Rank],
		ASC
	)
 ORDER BY
 	[@ Delivery Time Rank] ASC

