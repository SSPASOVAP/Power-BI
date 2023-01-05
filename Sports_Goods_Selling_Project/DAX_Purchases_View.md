
---------------------------------------------------------------
    		----------- Purchases View -----------
---------------------------------------------------------------

## Purchases
	### Products bougth by different client groups 

	//Product categories and quantity by age group:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Age_Range],
		'Dim-Products'[Category_Name],
		"@Qnt", [N Items]
	)
		// + Rank:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Age_Range],
		'Dim-Products'[Category_Name],
		"@Qnt", [N Items],
		"Rank", RANKX(
				ALL('Dim-Products'[Category_Name]),
					[N Items],,
				DESC, DENSE
				)
	)
ORDER BY
	[Age_Range]
,	[Rank]

	//Most purchased product:
DEFINE
	VAR AgeRange_Products = 
		SUMMARIZECOLUMNS(
			'Dim-Customers'[Age_Range],
			'Dim-Products'[Product_Name],
			"@Qnt", [N Items],
			"@RANK", RANKX(
					ALL('Dim-Products'[Product_Name]),
					[N Items],,
					DESC, DENSE
					)
			)
EVALUATE
	FILTER(
		AgeRange_Products,
		[@RANK] = 1
	)

	//Most purchased product - Metric:
DEFINE
	MEASURE 'Calcs-Matrix'[Products Rank] =
			IF(
				ISINSCOPE('Dim-Products'[Product_Name]),
				RANKX(
					ALL('Dim-Products'[Product_Name]),
					[N Items],,
					DESC, DENSE
				),
				"Product Name must be selected!"
			)
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Age_Range],
		'Dim-Products'[Product_Name],
		"Qnt", [N Items],
		"@RANK", [Products Rank] 
	)
ORDER BY
	[Age_Range],
	[@RANK]


	### Clients purchases view 

        // Purchases $, n_items and orders by age group:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Age_Range],
		"@Sum$ by AgeGroup", [Sum of Purch],
		"@Orders by AgeGroup", [N Orders],
		"@Items by AgeGroup", [N Items]
		)
			
ORDER BY
	[Sum of Purch] DESC

		// Purchases $, n_items and orders by gender:

EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Gender],
		"@Sum$ by Gender", [Sum of Purch],
		"@Orders by Gender ", [N Orders],
		"@Items by Gender", [N Items]
		)
			
ORDER BY
	[Sum of Purch] DESC

		// Purchases $, n_items and orders by Marital Status:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Marital_Status],
		"@Sum$ by MS", [Sum of Purch],
		"@Orders by MS ", [N Orders],
		"@Items by MS", [N Items]
		)
			
ORDER BY
	[Sum of Purch] DESC

		// Purchases $, n_items and orders by numb of kids:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Number_of_Children],
		"@Sum$ by N kids", [Sum of Purch],
		"@Orders by N kids", [N Orders],
		"@Items by N kids", [N Items]
		)
			
ORDER BY
	[Sum of Purch] DESC

		// Purchases $, n_items and orders by occupation:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Occupation],
		"@Sum$ by Occupation", [Sum of Purch],
		"@Orders by Occupation", [N Orders],
		"@Items by Occupation", [N Items]
		)
			
ORDER BY
	[Sum of Purch] DESC

		// Purchases $, n_items and orders by yearly incomes:
EVALUATE
	SUMMARIZECOLUMNS(
		'Dim-Customers'[Yearly_Income],
		"@Sum$ by Incomes", [Sum of Purch],
		"@Orders by Incomes", [N Orders],
		"@Items by Incomes", [N Items]
		)
			
ORDER BY
	[Sum of Purch] DESC	

	// Purchases $, n_items and orders by client
EVALUATE
	SELECTCOLUMNS( 					-- selectcolumns must be used because of the name repetitions
		'Dim-Customers',
		'Dim-Customers'[Customer_Name],
		"@Sum", [Sum of Purch],
		"@Items", [N Items]
		)
ORDER BY
	[@Sum] DESC

	// Purchases$ Rank:
DEFINE
	MEASURE 'Calcs-Matrix' [Purch Cust Rank] = 
		IF(
			ISINSCOPE('Dim-Customers'[Customer_Key]),
				RANKX(
					ALL('Dim-Customers'),
						[Sum of Purch],,
						DESC, DENSE
					), --rank
				"Customer Key is missing"
			) --if
EVALUATE
	SUMMARIZECOLUMNS( 					
		'Dim-Customers'[Customer_Key],
		'Dim-Customers'[Customer_Name],
		"@Sum", [Sum of Purch],
		"@Rrank", [Purch Cust Rank]
	)
ORDER BY
	[@Rrank]

	// N Products Rank:
DEFINE
	MEASURE 'Calcs-Matrix' [Items Cust Rank] = 
		IF(
			ISINSCOPE('Dim-Customers'[Customer_Key]),
				RANKX(
					ALL('Dim-Customers'),
						[N Items],,
						DESC, DENSE
					), --rank
				"Customer Key is missing"
			) --if
EVALUATE
	SUMMARIZECOLUMNS( 					
		'Dim-Customers'[Customer_Key],
		'Dim-Customers'[Customer_Name],
		"@Items", [N Items],
		"@Rrank", [Items Cust Rank]
	)
ORDER BY
	[@Rrank]

	### Some dependencies that can be highlighted based on the DATA we worked with:
-- 1. Products coming from category: accessories, same as the bikes are among the most seller.
-- 2. Customers with the most orders and on highest amonunt are in the age group of 31 - 40.
-- 3. Customers having 3-4 cars buy significantly less. Having the fact we sell bikes and sport goods, 
		-- we can guess those are not so active ppl.
-- 4. Man have more orders but on less amount compared to those on the women.
-- 5. Family clients have more orders and on higher amount. 
-- 6. Customers without children buy more than others, and they also represent the highest percentage among the clients network.
-- 7. Customers having 1,2 or 5 kids buy more than those having 3 - 4.
-- 8. Sport-100 and Water Bottle are among most buying products from all clients
-- 9. Women buy more bikes while men buy more clothes.
-- 10. Customers coming from Professional & Skilled Manual, are formed a bit over 50% from all purchases.
-- 11. Customers with yearly incomes between 30 и 70K are formed a bit over 50% from all purchases.
-- 12. In general customers with lower incomes (Below 100K, where MAX = 170K) are buying more, 
	-- but they are also formed a higher percentage compared to those who are taking over 100K.
