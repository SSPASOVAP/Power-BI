## Create General Scalar Values 

EVALUATE
ROW(
	"Sum Deliveries", CALCULATE(
		SUM('fact-purchases'[TOTAL]),
			USERELATIONSHIP(
			'fact-purchases'[SHIPPED_DATE],
			'Calendar'[Dates]),
			NOT ISBLANK('fact-purchases'[SHIPPED_DATE]))               				-- Total amount of delivery --
	"Sum Not Deliveried", CALCULATE(SUM('fact-purchases'[TOTAL]),
			'fact-purchases'[SHIPPED_DATE] = BLANK()),                  			-- Total amount of undelivered purchases --                        
	"Sum Purchases", SUM('fact-purchases'[TOTAL]),                      			-- Total amount of purchases --
	"N Orders", DISTINCTCOUNT('fact-purchases'[ORDER_ID]),              			-- Count of orders --
	"N Items", COUNT('fact-purchases'[ORDER_ID]),                       			-- Count of products --
	"Sum Units", SUM('fact-purchases'[QUANTITY]),                       			-- Count of units --
	"Mid Time Delivery", CALCULATE(AVERAGE('fact-purchases'[DELIVERY_TIME]),
		USERELATIONSHIP('Calendar'[Dates], 'fact-purchases'[SHIPPED_DATE]))      	-- Mid time for delivery --
	"N Customers", COUNT('dim-customers'[CUSTOMER_CODE]),               			-- Count of clients --
	"N Countries", DISTINCTCOUNT('dim-locations'[COUNTRY]),             			-- Count of vountries --
	"N Countries", DISTINCTCOUNT('dim-locations'[CITY])                 			-- Count of cities (locations) -- 
	"N Products", COUNT('dim-products'[PRODUCT_NAME]),                  			-- Number of products --
	"N Categories", DISTINCTCOUNT('dim-products'[CATEGORY_NAME])        			-- Number of categories --
	)

                        -- Additional Useful Values --
EVALUATE
ROW(
	"Items Not Delivered", CALCULATE(COUNT('fact-purchases'[ORDER_ID]),
		'fact-purchases'[SHIPPED_DATE] = BLANK()),                              	-- Undelivered products --
	"Orders Not Delivered", CALCULATE(DISTINCTCOUNT('fact-purchases'[ORDER_ID]),
		'fact-purchases'[SHIPPED_DATE] = BLANK()),                              	-- Open (undelivered) orders --
	"Orders Delivered on Time", CALCULATE(DISTINCTCOUNT('fact-purchases'[ORDER_ID]),
		'fact-purchases'[TRACK] = "true"),                                      	-- Orders delivered on time -- 
	"Delayed Orders", CALCULATE(DISTINCTCOUNT('fact-purchases'[ORDER_ID]),          
		'fact-purchases'[TRACK] = "false")                                      	-- Delayed deliveries --
)


## Cross Check per Year

EVALUATE
    SUMMARIZECOLUMNS(
        'Calendar'[Year], 
        "@ Sum Deliveries", CALCULATE(SUM('fact-purchases'[TOTAL]), 'fact-purchases'[SHIPPED_DATE] <> BLANK()),
        "@ Sum Not Delivered", CALCULATE(SUM('fact-purchases'[TOTAL]), 'fact-purchases'[SHIPPED_DATE] = BLANK()),
        "@ Sum Purchases", SUM('fact-purchases'[TOTAL]),         
        "@ N Orders", DISTINCTCOUNT('fact-purchases'[ORDER_ID]), 
        "@ N Items", COUNT('fact-purchases'[ORDER_ID]),          
        "@ Sum Units", SUM('fact-purchases'[QUANTITY]),          
        "@ Mid Time Delivery", CALCULATE( AVERAGE('fact-purchases'[DELIVERY_TIME]),
        						USERELATIONSHIP('Calendar'[Dates], 'fact-purchases'[SHIPPED_DATE]))
            )


## Defining Measures

DEFINE
    MEASURE 'Purch-matrix' [Sum Deliveries $] = CALCULATE(
			SUM('fact-purchases'[TOTAL]),
			USERELATIONSHIP('fact-purchases'[SHIPPED_DATE],
							'Calendar'[Dates]),
			NOT ISBLANK('fact-purchases'[SHIPPED_DATE]))
    MEASURE 'Purch-matrix' [Sum Not Delivered $] = CALCULATE(SUM('fact-purchases'[TOTAL]),
	        'fact-purchases'[SHIPPED_DATE] = BLANK())
	MEASURE 'Purch-Matrix' [Sum Purchases $] = SUM('fact-purchases'[TOTAL])
	MEASURE 'Purch-Matrix' [N Orders] = DISTINCTCOUNT('fact-purchases'[ORDER_ID])
	MEASURE 'Purch-Matrix' [N Items] = COUNT('fact-purchases'[ORDER_ID])
	MEASURE 'Purch-Matrix' [N Units] = SUM('fact-purchases'[QUANTITY])
	MEASURE 'Purch-Matrix' [Mid Days Delivery] = CALCULATE( AVERAGE('fact-purchases'[DELIVERY_TIME]),
        						USERELATIONSHIP('Calendar'[Dates], 'fact-purchases'[SHIPPED_DATE]))
    MEASURE 'Purch-Matrix' [Avg Order Amount $] = DIVIDE([Sum Purchases $],[N Orders])
        
EVALUATE
    SUMMARIZECOLUMNS(
    	'Calendar'[Year],
        "@ Sum Deliveries", [Sum Deliveries $],
        "@ Sum Not Delivered", [Sum Not Delivered $],
        "@ Sum Purchases", [Sum Purchases $],         
        "@ N Orders", [N Orders], 
        "@ N Items", [N Items],          
        "@ Sum Units", [N Units] ,          
        "@ Mid Time Delivery", [Mid Days Delivery],
        "@ Mid Order Amaunt", [Avg Order Amount $],
		"@ n countries", 
        	VAR countries =
        		SUMMARIZE(
        			'fact-purchases',
        			'dim-locations'[COUNTRY])
        				RETURN
        					COUNTROWS(
        					countries),
        "@ n categories", 
        	VAR categories =
        		SUMMARIZE(
        			'fact-purchases',
        			'dim-products'[CATEGORY_NAME])
        				RETURN
        					COUNTROWS(
        					categories)
        )


                                    -- Additional Useful Measures --

DEFINE
	MEASURE 'Purch-Matrix' [Items Not Delivered] = 
			CALCULATE(COUNT('fact-purchases'[ORDER_ID]),'fact-purchases'[SHIPPED_DATE] = BLANK())
	MEASURE 'Purch-Matrix' [Orders Not Delivered] = 
			CALCULATE(DISTINCTCOUNT('fact-purchases'[ORDER_ID]),'fact-purchases'[SHIPPED_DATE] = BLANK())
	MEASURE 'Purch-Matrix' [Orders Delivered on Time] = 
			CALCULATE(DISTINCTCOUNT('fact-purchases'[ORDER_ID]),'fact-purchases'[TRACK] = "true")
	MEASURE 'Purch-Matrix' [Delayed Orders] = 
			CALCULATE(DISTINCTCOUNT('fact-purchases'[ORDER_ID]),'fact-purchases'[TRACK] = "false")
	MEASURE 'Purch-Matrix' [N Cities ] = DISTINCTCOUNT('dim-locations'[CITY])

EVALUATE
	SUMMARIZECOLUMNS(
		'dim-locations'[COUNTRY],
		"@ Items not Delieverd",[Items Not Delivered],
		"@ Orders not Completed", [Orders Not Delivered],
		"@ Deliveries on time", [Orders Delivered on Time],
		"@ Delayed Deliveries", [Delayed Orders],
		"@ N Cities", [N Cities],
		)