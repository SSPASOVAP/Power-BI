## What is the average amount of order?:
    -- Option 1 --
EVALUATE
	ROW("Metric", "Value", "Avg Order Amunt",SUM('fact-purchases'[TOTAL])/DISTINCTCOUNT('fact-purchases'[ORDER_ID]))

    -- Option 2 --
EVALUATE
	SUMMARIZECOLUMNS(
	"@ Avg Order", SUM('fact-purchases'[TOTAL]) / DISTINCTCOUNT('fact-purchases'[ORDER_ID]))

	-- Option 3 --
EVALUATE
	ROW("Avg Order Amount", DIVIDE([Sum Purchases $],[N Orders]))

	-- Option 4 --
DEFINE
	VAR orders =
		SUMMARIZE(
		'fact-purchases',
		'fact-purchases'[ORDER_ID]
		)	
EVALUATE
	SUMMARIZECOLUMNS(
	"@ AVG order amaunt", AVERAGEX(
							orders,
							[Sum Purchases $]
							)
	)


## What is the average time of delivery?:

EVALUATE
	// ROW ("Mid Days Delivery", TRUNC(CALCULATE(AVERAGE('fact-purchases'[DELIVERY_TIME]), -- TRUNC is used only for DAX View
    //     						USERELATIONSHIP('Calendar'[Dates], 'fact-purchases'[SHIPPED_DATE])))
	// 						) 		

DEFINE
MEASURE 'Purch-Matrix'[Mid Days Delivery] =
	VAR shipped_prod =
		FILTER(
		'fact-purchases',
		'fact-purchases'[SHIPPED_DATE] <> BLANK()
		)
	VAR orders_days =
		SUMMARIZE(
		shipped_prod,
		'fact-purchases'[ORDER_ID],
		'fact-purchases'[DELIVERY_TIME]
		)
	RETURN
		AVERAGEX(
			orders_days,
			[DELIVERY_TIME]
		)
EVALUATE
	SUMMARIZECOLUMNS(
	"AVG Del days", [Mid Days Delivery]
	)										
