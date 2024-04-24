        
# My SQL Codes for data cleaning

## Data cleaning

        create database addidas; 
        Use addidas;

        SELECT 
        *
        FROM
        addidas_sales;

        -- + ------------------------------------- + --
        -- Change the name of the column
        Alter table addidas_sales
        rename column `ï»¿Retailer` to Retailer;

        Alter table addidas_sales
        rename column `Retailer ID` to Retailer_id;

        Alter table addidas_sales
        rename column `Invoice Date` to Invoice_date;

        Alter table addidas_sales
        rename column `Price per Unit` to Price_per_unit;

        Alter table addidas_sales
        rename column `Units Sold` to Unit_sold;

        Alter table addidas_sales
        rename column `Total Sales` to Total_Sales;

        Alter table addidas_sales
        rename column `Operating Profit` to Operating_profit;

        Alter table addidas_sales
        rename column `Sales Method` to Sales_method;

        Select column_name , data_type from information_schema.columns
        where table_schema = "addidas" and table_name = "addidas_sales";

        -- Find out the blank value in each column

        SELECT 
        *
        FROM
        addidas_sales
        WHERE
        Price_per_unit = '' OR Retailer = ''
                OR Retailer_id = ''
                OR Region = ''
                OR State = ''
                OR City = ''
                OR Product = ''
                OR Unit_sold = ''
                OR Total_Sales = ''
                OR Operating_profit = ''
                OR Sales_method = '';

        -- Deleting rows with missing values 
        Delete	from addidas_sales
        Where Unit_sold = ''; 

        -- Find out the null value in each column
        SELECT 
        *
        FROM
        addidas_sales
        WHERE
        Price_per_unit IS NULL
                OR Retailer IS NULL
                OR Retailer_id IS NULL
                OR Region IS NULL
                OR State IS NULL
                OR City IS NULL
                OR Product IS NULL
                OR Unit_sold IS NULL
                OR Total_Sales IS NULL
                OR Operating_profit IS NULL
                OR Sales_method IS NULL
                OR Invoice_date IS NULL;

        -- + ------------------------------------- + --

        UPDATE addidas_sales
        SET Invoice_date = STR_TO_DATE(Invoice_date, '%Y-%m-%d');

        -- get an error Error Code: 1292. Truncated incorrect date value: '22-01-2021'

        SELECT * FROM addidas_sales where Invoice_date = 22-01-2021; 

        -- + ------------------------------------- + --
        -- updating the rows of Invoice Date column
        UPDATE addidas_sales 
        SET 
        Invoice_date = CASE
                WHEN Invoice_date LIKE '____-__-__' THEN STR_TO_DATE(`Invoice Date`, '%Y-%m-%d')
                WHEN Invoice_date LIKE '__-__-____' THEN STR_TO_DATE(`Invoice Date`, '%d-%m-%Y')
        END;

        -- Converting the column to convert the text dates to date datatype    
        ALTER TABLE addidas_sales MODIFY COLUMN Invoice_date DATE;  

        -- updating the rows of Total Sales column

        UPDATE addidas_sales 
        SET 
        Total_Sales = REPLACE(Total_Sales, ',', '');

        -- Converting the column to convert the text to int datatype 

        ALTER TABLE addidas_sales MODIFY COLUMN Total_Sales int;

        -- updating the rows of operating profit column

        UPDATE addidas_sales 
        SET Operating_profit = REPLACE(REPLACE(Operating_profit, ',', ''),'$','');

        -- Converting the column to convert the text to int datatype 

        ALTER TABLE addidas_sales MODIFY COLUMN Operating_profit int;


        -- updating the rows of price per unit column

        UPDATE addidas_sales 
        SET 
        Price_per_unit = REPLACE(Price_per_unit, '$', '');

        -- Converting the column to convert the text to int datatype 

        ALTER TABLE addidas_sales MODIFY COLUMN Price_per_unit int;

        -- Filling the null value of price per unit column  
        UPDATE addidas_sales 
        SET 
        Price_per_unit = Total_sales / unit_sold
        WHERE
        Price_per_unit IS NULL;

        -- update the value of total sales
        Update addidas_sales 
        Set Total_sales = Price_per_unit * Unit_sold; -- 5758 Rows affected 

        -- update the value of Product column
        Update addidas_sales 
        Set Product = replace(replace(product, "Men's aparel", "Men's Apparel"),"Men''s Apparel","Men's Apparel");

        -- Checking the duplicate values 

        SELECT Retailer, Retailer_id, Invoice_date, Region, State, City, Product, 
        Price_per_unit, Unit_sold, Total_Sales, Operating_profit, Sales_method, 
        COUNT(*) AS num_duplicates
        FROM addidas_sales
        GROUP BY Retailer, Retailer_id, Invoice_date, Region, State, City, 
        Product, Price_per_unit, Unit_sold, Total_Sales, Operating_profit, Sales_method
        HAVING COUNT(*) > 1; -- No Duplicate found 

## ANALYSING THE DATA SET
        -- -------------------------------------- --
        
        --> Check the distict values

        Select distinct Retailer from addidas_sales; -- There are total 6 retailers 
        Select distinct Region from addidas_sales; -- There are toal 5 Regions
        Select distinct Product from addidas_sales; -- There are total 6 Product in this dataset
        Select distinct state from addidas_sales; -- There are total 50 state
        Select distinct city from addidas_sales; -- There are total 52 city
        Select distinct Sales_method from addidas_sales; -- There are 3 Sales method


        -- Before analyze I created the year , month & season column

        -- Add the new columns to the table
        ALTER TABLE addidas_sales
        ADD COLUMN Year INT,
        ADD COLUMN Month VARCHAR(10),
        ADD COLUMN Season VARCHAR(10);

        -- Update the values of the newly added columns
        UPDATE addidas_sales
        SET 
        Year = YEAR(Invoice_date),
        Month = DATE_FORMAT(Invoice_date, '%b'),
        Season = CASE 
                WHEN MONTH(Invoice_date) IN (3,4,5) THEN 'Spring'
                WHEN MONTH(Invoice_date) IN (6,7,8) THEN 'Summer'
                WHEN MONTH(Invoice_date) IN (9,10,11) THEN 'Autumn'
                ELSE 'Winter'
        END;


        -- Questions --    

        --> Analyzing repeat customers
          ----------------------------     
        SELECT Retailer, COUNT(*) AS Purchase_Count
        FROM addidas_sales
        GROUP BY Retailer
        HAVING Purchase_Count > 1
        order by Purchase_Count desc;

        --> Geospatial Analysis:
        -------------------------

        -- On the basis of Region

        select Region, Sum(Round(Total_sales/1000000,2)) as total_sales
        From addidas_sales
        Group by Region
        Order by total_sales desc,Operating_profit desc, total_customer desc;

        -- On the basis of state
        
        -- Top 10 
        select State, Sum(Round(Total_sales/1000000,2)) as total_sales
        Group by State
        Order by total_sales desc, total_customer desc
        Limit 10;

        -- Bottom 10 
        
        select State, Sum(Round(Total_sales/1000000,2)) as total_sales
        Group by State
        Order by total_sales asc, total_customer asc
        Limit 10;

        -- > On the basis of city

        -- Top 10 

        select City, Sum(Round(Total_sales/1000000,2)) as total_sales From addidas_sales
        Group by City 
        Order by total_sales desc
        limit 10;

        -- -------------------------------------- --

        -- Bottom 10 city

        select City, Sum(Round(Total_sales/1000000,2)) as total_sales From addidas_sales
        Group by City
        Order by total_sales asc
        limit 10;

        --> Product analyse

        -- Best-selling products

        select Product, Sum(Round(Total_sales/1000000,2)) as total_sales,
        SUM(Unit_Sold) AS Total_Units_Sold
        From addidas_sales
        Group by Product
        Order by total_sales desc;
        -- ------------------------------------------------------------------
        
        -- Product with hieghest growth rate 

        with Cte as (SELECT Product,  
        SUM(CASE WHEN year = 2020 THEN Round(Total_Sales/1000000,2) END) AS Total_sales_2020, 
        SUM(CASE WHEN year = 2021 THEN Round(Total_Sales/1000000,2) END) AS Total_sales_2021
        FROM 
        addidas_sales
        group by product)
        Select Product,Total_sales_2020, Total_sales_2021, 
        Round(((Total_sales_2021 - Total_sales_2020) / Total_sales_2020)*100,2) as Growth_rate     
        From Cte
        Order by Growth_rate desc;

        -- > Seasonal Analysis:

        select Season, Sum(Round(Total_sales/1000000,2)) as total_sales,
        SUM(Unit_Sold) AS Total_Units_Sold  From addidas_sales
        Group by Season
        Order by total_sales desc;

        --> Year analyse

        -- Monthly Sales Analysis for 2020 and 2021

        SELECT 
        DATE_FORMAT(Invoice_Date, '%b') AS Month,
        SUM(CASE WHEN YEAR(Invoice_Date) = 2020 THEN Total_Sales ELSE 0 END) AS Total_Sales_2020,
        SUM(CASE WHEN YEAR(Invoice_Date) = 2021 THEN Total_Sales ELSE 0 END) AS Total_Sales_2021
        FROM 
        addidas_sales
        GROUP BY 
        DATE_FORMAT(Invoice_Date, '%b')
        ORDER BY 
        Month;

        -- --------------------------------------------------------------

        -- Growth Rate on the basis of year

        with Cte as (SELECT  
        SUM(CASE WHEN year = 2020 THEN Round(Total_Sales/1000000,2) END) AS Total_sales_2020, 
        SUM(CASE WHEN year = 2021 THEN Round(Total_Sales/1000000,2) END) AS Total_sales_2021
        FROM 
        addidas_sales)
        Select Total_sales_2020, Total_sales_2021, 
        Round(((Total_sales_2021 - Total_sales_2020) / Total_sales_2020)*100,2) as Growth_rate     
        From Cte;

        -- > Sale method analyse

        SELECT 
        Sales_method,
        SUM(ROUND(Total_sales / 1000000, 2)) AS total_sales,
        SUM(Unit_Sold) AS Total_Units_Sold
        FROM
        addidas_sales
        GROUP BY Sales_method
        ORDER BY total_sales DESC , Total_Units_Sold DESC







