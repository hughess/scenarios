# Scenario Planning & Forecasting

```sql orders
select date_trunc('month', order_datetime) as month, count(*) as orders, sum(sales) as sales from needful_things.orders
group by all
order by month asc
```

## Growth Rate Input

<Slider
    title="Growth Rate" 
    name=growth
    min=-0.05
    max=0.15
    step=0.01
    fmt=pct
    defaultValue=0.05
/>

```sql forecast
WITH future_months AS (
    SELECT
        (SELECT MAX(month) FROM ${orders}) + INTERVAL '1 month' * i AS future_month,
        i
    FROM
        range(1, 13) AS t(i) -- Generate the next 12 months
),
sales_forecast AS (
    SELECT
        future_month,
        last_month.orders,
        last_month.sales * POWER(1 + ${inputs.growth}, future_months.i) AS forecasted_sales
    FROM
        future_months
    CROSS JOIN
        (SELECT orders, sales FROM ${orders} ORDER BY month DESC LIMIT 1) AS last_month -- Fetch the most recent month data
)
SELECT * FROM sales_forecast
```

<LineChart
  data={forecast}
  x=future_month
  y=forecasted_sales
  yFmt=gbp
  title="Sales Forecast: Next 12 Months"
/>

## Scenario Selection

<ButtonGroup name=growth_scenario>
    <ButtonGroupItem valueLabel="Downside Case" value="-0.05" />
    <ButtonGroupItem valueLabel="Base Case" value="0.05" default/>
    <ButtonGroupItem valueLabel="Upside Case" value="0.15" />
</ButtonGroup>


```sql forecast2
WITH future_months AS (
    SELECT
        (SELECT MAX(month) FROM ${orders}) + INTERVAL '1 month' * i AS future_month,
        i
    FROM
        range(1, 13) AS t(i) -- Generate the next 12 months
),
sales_forecast AS (
    SELECT
        future_month,
        last_month.orders,
        last_month.sales * POWER(1 + ${inputs.growth_scenario}, future_months.i) AS forecasted_sales
    FROM
        future_months
    CROSS JOIN
        (SELECT orders, sales FROM ${orders} ORDER BY month DESC LIMIT 1) AS last_month -- Fetch the most recent month data
)
SELECT * FROM sales_forecast
```

```sql union
select month, sales from ${orders}
union all
(select future_month as month, forecasted_sales as sales from ${forecast2})
```

```sql min_date
select max(month) as min_date from ${orders}
```

<LineChart
  data={union}
  x=month
  y=sales
  yFmt=gbp
  title="Sales Forecast: Next 12 Months"
>
  <ReferenceArea data={min_date} xMin=min_date label=Forecast />
</LineChart>