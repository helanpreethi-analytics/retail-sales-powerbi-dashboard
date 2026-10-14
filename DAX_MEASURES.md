# DAX Reference

## Core measures
```dax
Total Revenue = SUM(sales_fact[total_amount])
Avg Order Value = AVERAGE(sales_fact[total_amount])
Sales Count = COUNT(sales_fact[sale_id])
Unique Customers = DISTINCTCOUNT(sales_fact[customer_id])
Highest Sale = MAX(sales_fact[total_amount])
Avg Price per Unit = DIVIDE([Total Revenue], SUM(sales_fact[quantity]))
```

## Share of total (respects slicers)
```dax
Total Revenue All = CALCULATE([Total Revenue], ALLSELECTED(sales_fact))
% of Total Revenue = DIVIDE([Total Revenue], [Total Revenue All])
```

## Time intelligence (`Calendar` table)
```dax
Revenue MTD = TOTALMTD(SUM(sales_fact[total_amount]), 'Calendar'[Date])
Revenue QTD = TOTALQTD(SUM(sales_fact[total_amount]), 'Calendar'[Date])
Revenue YTD = TOTALYTD(SUM(sales_fact[total_amount]), 'Calendar'[Date])
Revenue LY  = CALCULATE([Total Revenue], SAMEPERIODLASTYEAR('Calendar'[Date]))
Revenue Prev Month = CALCULATE([Total Revenue], DATEADD('Calendar'[Date], -1, MONTH))
```

## Ranking and comparison
```dax
Customer Rank = RANKX(ALL(customers), [Total Revenue])

Product Rank =
VAR CurrentRevenue = [Total Revenue]
RETURN
IF(
    ISINSCOPE(products[product_name]),
    RANKX(ALLSELECTED(products), [Total Revenue], CurrentRevenue, DESC)
)

Revenue vs Avg =
VAR CurrentCustomerRevenue = [Total Revenue]
VAR AverageCustomerRevenue = AVERAGEX(ALLSELECTED(customers), [Total Revenue])
RETURN
IF(ISINSCOPE(customers[customer_id]), CurrentCustomerRevenue - AverageCustomerRevenue)
```

## Filter-context patterns
```dax
Chennai Revenue = CALCULATE([Total Revenue], customers[city] = "Chennai")
Big Orders Revenue = CALCULATE([Total Revenue], FILTER(sales_fact, sales_fact[total_amount] > 5000))
Crossfiltered Products =
CALCULATE(DISTINCTCOUNT(products[product_id]),
          CROSSFILTER(sales_fact[product_id], products[product_id], Both))
Magic Revenue =
VAR SelectedValueFromSlicer = SELECTEDVALUE(sales_fact[payment_mode])
RETURN CALCULATE([Total Revenue], sales_fact[payment_mode] = SelectedValueFromSlicer)
```

## Dynamic title
```dax
Dynamic Title = "Revenue for " & SELECTEDVALUE(customers[city], "All Cities")
```

## Calculated columns
```dax
total_amount = sales_fact[quantity] * RELATED(products[base_price])
Order Size   = IF(sales_fact[quantity] >= 3, "Bulk", "Normal")
Price Tier   = SWITCH(TRUE(), products[base_price] >= 2000, "Expensive",
                              products[base_price] >= 800,  "Medium", "Cheap")
```

## Field parameters
`Select Metric` (Total Revenue / Revenue MTD / Revenue YTD) and `test dial` (city / payment_mode / quantity).
