# AdventureWorks2022 Data Analytics Project
This project analyzes the AdventureWorks2022 sample database — a fictitious manufacturing company that sells bicycles and related products worldwide. The goal is to demonstrate end-to-end data analysis, reporting, and business insights using SQL Serve

## Project Overview

**Project Title**: AdventureWorks2022 Data Analytics
**Level**: Advanced  
**Database**: `AdventureWorks2022`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. 

## Objectives

1. **Understand the business**: Identify key performance areas such as sales, products, customers, and employee performance.
2. **Apply SQL Skills**: Perform advanced queries, joins, CTEs, window functions, and aggregations.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.
5. **Visualize results**: Create interactive Power BI dashboards to communicate findings effectively.

## Project Structure

### 1. Key Business Questions
1. **Sales Performance**
    - What are the top 10 products by total sales and quantity sold?
   ```SQL
   select
       PD.Name as ProductName ,
       sum(SOD.OrderQty)as TotalQuantitySold
    from Sales.SalesOrderDetail SOD 
    join Production.Product PD 
        on SOD.ProductID = PD.ProductID
    group by PD.Name ,SOD.ProductID	
    order by TotalQuantitySold desc 
   ```
   <img width="218" height="136" alt="Screenshot 2025-10-30 163627" src="https://github.com/user-attachments/assets/4f080f1b-6e93-4502-8f21-2c1d06a0a982" />
   
    - What are the top 10 products by revenue?
   ```SQL
   select
       PP.Name,
       sum(SS.OrderQty*SS.UnitPrice) as TotalRevenue
    from Sales.SalesOrderDetail as SS
    join Production.Product as PP
        on PP.ProductID = SS.ProductID 
    group by PP.Name, SS.ProductID
    order by TotalRevenue desc
   ```
   
   <img width="189" height="145" alt="Screenshot 2025-10-31 062517" src="https://github.com/user-attachments/assets/1b9d5550-3549-43b3-ad31-7a808058276a" />

    - Which sales territories generate the most revenue?
   ``` SQL
   select
       ST.Name, ST.CountryRegionCode,
       sum(SOD.OrderQty* SOD.UnitPrice) as TotalRevenue 
    from Sales.SalesOrderHeader as SS 
    join Sales.SalesOrderDetail as SOD
       on SOD.SalesOrderID = SS.SalesOrderID
    join sales.SalesTerritory as ST
       on ST.TerritoryID=SS.TerritoryID
    group by ST.Name, ST.CountryRegionCode
    order by TotalRevenue desc
   ```
   <img width="250" height="149" alt="Screenshot 2025-10-31 064404" src="https://github.com/user-attachments/assets/098b8a44-7086-413d-9601-a2eb61f4c311" />


    - How have monthly sales trends evolved over time?
    ``` SQL
    select 
        year(SOH.orderDate) aS orderYear , 
        MONTH(SOH.orderDate) as Ordermonth ,
        sum(SOD.OrderQty*SOD.UnitPrice)as TotalRevenue 
        from Sales.SalesOrderDetail as SOD 
        join Sales.SalesOrderHeader as SOH
        on SOD.SalesOrderID = SOH.SalesOrderID
        group by year(SOH.OrderDate),MONTH(SOH.OrderDate)
        ORDER BY OrderYear, OrderMonth;
    ```  
    <img width="199" height="240" alt="Screenshot 2025-10-31 160557" src="https://github.com/user-attachments/assets/3c6719d7-06b5-44e9-b786-e4ae96603020" />

    - What is the average order value (AOV) per region?
  ``` SQL
    select 
    	ST.Name as Region,
    	count(SOH.salesorderID) as TotalOrders,
    	sum(SOH.SubTotal) as TotalRevenue,
    	AVG(SOh.SubTotal) as AvrageOrderValue
    from Sales.salesOrderHeader as SOH 
    join.Sales.SalesTerritory AS ST 
    	on ST.TerritoryID = SOH.TerritoryID
    group by ST.Name
    order by AvrageOrderValue desc
```
<img width="290" height="151" alt="Screenshot 2025-11-01 091213" src="https://github.com/user-attachments/assets/4dbb25f9-c916-4690-87da-e410f855783b" />



2. **Customer Insights**
    - Who are the top 10 customers by revenue?
    ```SQL
    select top 10
    SS.BusinessEntityID AS StoreID , SS.Name as StoreName, sum(orderQty*UnitPrice) as Revenue
    from Sales.SalesOrderDetail as SOD
    join Sales.SalesOrderHeader as SOH 
    	on SOD.SalesOrderID=SOH.SalesOrderID 
    join Sales.Customer as SC
        on SC.CustomerID = SOH.CustomerID
    join Sales.Store as SS
        on SC.StoreID=SS.BusinessEntityID
    group by SS.BusinessEntityID , SS.Name
    order by Revenue desc
    ```
   <img width="267" height="158" alt="Screenshot 2025-11-02 121929" src="https://github.com/user-attachments/assets/b3fa91f1-cf9d-47f0-ac7b-f63341d0ed8c" />


    - How many customers are repeat vs new?
   ``` sql
    with customerOrders as 
    (select CustomerID , count(SalesOrderID) as ordercount from Sales.SalesOrderHeader
    group by CustomerID)
    select 
    	case 
    		when ordercount = 1 then 'New Customer'
    		else 'repeat Customer'
    	end as CustomerType,
    count(*) as CustomerCount from customerOrders
    group by
    	case 
    		when ordercount = 1 then 'New Customer'
    		else 'repeat Customer'
    	end
      ```
   <img width="242" height="73" alt="Screenshot 2025-11-04 064036" src="https://github.com/user-attachments/assets/003dda5e-9f71-4034-a2a0-c3bb92c42127" />

      
    - What are the most profitable Territories?
   ``` sql
    select st.name As Territory, 
    st.countryregioncode as TerritoryCode,
        sum(sod.LineTotal - (pp.standardcost* SOD.OrderQty)) as TotalProfit,
        count(DIStinct SOH.customerID)as TotalCustomers,
        avg(sod.LineTotal - (pp.standardcost* SOD.OrderQty)) as AvrageProfitPerOrder
    from Sales.SalesOrderDetail as SOD 
    join sales.SalesOrderHeader as SOH
       on SOH.SalesOrderID = SOD.SalesOrderID
    join Production.Product as pp
       on SOD.ProductID = pp.ProductID
    join Sales.Customer as CS
       on Cs.CustomerID =SOh.CustomerID
    join Sales.SalesTerritory as ST
       on SOh.TerritoryID = St.TerritoryID
    group by st.Name , st.countryregioncode
    order by TotalProfit desc
    ```
   <img width="383" height="170" alt="Screenshot 2025-11-06 061147" src="https://github.com/user-attachments/assets/9a392f4f-8378-4012-b982-a6e19893ce3e" />

    
    - Which region shows the highest customer growth??
      
3. **Product & Inventory**
    - Which product categories perform best by sales?
   ``` sql
    select top 10 PPs.Name as SubCategoryName, sum(sod.LineTotal) as TotalSold 
    from Sales.SalesOrderDetail  as SOD
    join Production.Product as pp 
    	on pp.ProductID=sod.ProductID 
    join Production.ProductSubcategory as PPS 
    	on PPS.ProductSubcategoryID = pp.ProductSubcategoryID
    group by PPs.Name
    order by TotalSold desc
      ```
   <img width="197" height="176" alt="Screenshot 2025-11-06 165104" src="https://github.com/user-attachments/assets/15d0e48a-f64d-4a38-9cf3-865eecf8af9b" />


    - Which product categories perform best by profit?
    ``` sql
    select top 10 PPs.Name as SubCategoryName, 
    sum(sod.LineTotal - (pp.standardcost* SOD.OrderQty)) as TotalProfit,
    avg(sod.LineTotal - (pp.standardcost* SOD.OrderQty)) as AvrageProfitPerOrder
    from Sales.SalesOrderDetail  as SOD
    join Sales.SalesOrderHeader as SOH
    	on soh.SalesOrderID = Sod.SalesOrderID
    join Production.Product as pp 
    	on pp.ProductID=sod.ProductID 
    join Production.ProductSubcategory as PPS 
    	on PPS.ProductSubcategoryID = pp.ProductSubcategoryID
    group by PPs.Name
    order by TotalProfit desc
    ```
    <img width="259" height="152" alt="Screenshot 2025-11-07 053844" src="https://github.com/user-attachments/assets/092e5be6-9dbf-41c8-9032-6b8b63e506c7" />


    - What are the slow-moving product?
    ```sql
    select SOD.ProductID, pp.name, 
    sum(SOD.orderqty) as totalorders 
    from Sales.SalesOrderDetail as SOD
    join Production.Product as pp 
    	on pp.ProductID = SOD.ProductID
    group by pp.name, SOD.ProductID
    having sum(SOD.orderqty) <20
    order by totalorders
    ```
    <img width="251" height="105" alt="Screenshot 2025-11-07 062846" src="https://github.com/user-attachments/assets/b378ce61-4dcb-4276-b90d-20311384127f" />

    - What are the low-margin products?
   ```sql
   select pp.name as 'Product Name',
    sum(sod.OrderQty) as 'Total Sold',
    sum(sod.linetotal) as Revenue,
    sum(sod.LineTotal - (pp.StandardCost*sod.OrderQty)) as 'Profit/Loss' ,
    (sum(sod.LineTotal - (pp.StandardCost*sod.OrderQty))/sum(sod.linetotal))*100 as Margin
    from Sales.SalesOrderDetail as SOD 
    join Production.Product as pp 
    	on SOD.ProductID = pp.ProductID		
    group by pp.name
    order by margin 
   ```
    <img width="389" height="240" alt="Screenshot 2025-11-09 081256" src="https://github.com/user-attachments/assets/7dd13b3c-ffe4-48f7-980d-34369ee743c8" />


    - How do production costs compare to sales prices?
      ```sql
      select pp.name,
        avg(sod.UnitPrice) as 'Avg Product Price',
        avg(PP.StandardCost) as 'Avg Product Cost',
        avg(sod.UnitPrice-PP.StandardCost) as 'Avg Profit PerUnit',
        (avg(sod.UnitPrice-PP.StandardCost)/avg(PP.StandardCost)*100) as 'Markup %',
        (avg(sod.UnitPrice-PP.StandardCost)/avg(sod.UnitPrice)*100) as 'Margin %'
        from Sales.SalesOrderDetail as sod
        join Production.Product as pp
            on sod.ProductID = pp.ProductID
        group by pp.name
        order by 'Avg Profit PerUnit'
        ```
    <img width="461" height="257" alt="Screenshot 2025-11-10 084925" src="https://github.com/user-attachments/assets/821f889a-7fbc-460d-ae4a-2127f4f91a0f" />

6. **Employee & Department Analysis**
    - Which salespersons achieve the highest sales volume?
    ```sql
    select 
    PP.BusinessEntityID as SalesPersonID,
    concat(pp.FirstName, pp.LastName)as SalesPerson ,
    count(soh.SalesOrderID) as TotalOrders,
    sum(soh.SubTotal) as TotalSales
    from Person.Person as pp join Sales.SalesOrderHeader as SOH
    on PP.BusinessEntityID = SOH.SalesPersonID
    GROUP BY pp.FirstName, pp.LastName , PP.BusinessEntityID
    order by TotalSales desc
    ```
    <img width="307" height="230" alt="Screenshot 2025-11-11 071111" src="https://github.com/user-attachments/assets/a125738d-cb4e-4469-af28-8cce52242aeb" />

    - What’s the average commission per employee?
    ```sql
    select
    sp.BusinessEntityID,
    concat(pp.FirstName, pp.LastName)as SalesPerson ,  
    avg(sp.CommissionPct*sp.SalesYTD) as AVGcommission
    from sales.SalesPerson as sp
    join Person.Person as pp
        on pp.BusinessEntityID= sp.BusinessEntityID
    where CommissionPct > 0
    group by sp.BusinessEntityID , pp.FirstName, pp.LastName
    ```
   <img width="288" height="221" alt="Screenshot 2025-11-12 061313" src="https://github.com/user-attachments/assets/7d4fdbd6-c713-40f4-a055-cecfc9d2f4b1" />

    - Which departments have the highest average tenure or employee retention?
    ```sql
    select hd.Name,
    avg(Datediff (day,he.HireDate , isnull(edh.EndDate,getdate()))) /365 as AvgTenureYears, 
    count(distinct HE.BusinessEntityID) as Totalnumber
    from HumanResources.EmployeeDepartmentHistory as EDH 
    join HumanResources.Employee As HE 
        on EDH.BusinessEntityID = HE.BusinessEntityID
    join HumanResources.Department as HD 
        on HD.DepartmentID = EDH.DepartmentID
    group by HD.name
    order by AvgTenureYears desc
    ```
    <img width="259" height="233" alt="Screenshot 2025-11-13 160853" src="https://github.com/user-attachments/assets/5537e1ba-53fe-4494-9391-e33635526498" />


### 2. Reports & Dashboards (Power BI)
1. **Sales Overview Dashboard**
Revenue trends, product and territory performance
2. **Customer Insights Dashboard**
Segmentation, retention, and top customers
3. **Employee Performance Dashboard**
Sales by employee, commission analysis
4. **Product Analysis Dashboard**
Product profitability and inventory movement

### 3. findings_summary
### 4. conclusions
