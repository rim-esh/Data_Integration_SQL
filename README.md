 Step 1: Create Base Population Table (Join + Cleanup)
CREATE TABLE Join_Accommodation AS
SELECT
    A.LA_Code,
    A.LA_Name,
    L.Region_name,
    A.Total,
    A.Detached,
    A.Semi_detached,
    A.Terraced,
    A.Flats
FROM Accommodation A
LEFT JOIN lasregionew2021lookup L ON TRIM(A.LA_Code) = TRIM(L.LA_Code);

Step 2: Update Null Regions
UPDATE Join_Accommodation SET Region_name = 'North West' WHERE LA_Code IN ('E06000063', 'E06000064');
UPDATE Join_Accommodation SET Region_name = 'Yorkshire and The Humber' WHERE LA_Code = 'E06000065';
UPDATE Join_Accommodation SET Region_name = 'South West' WHERE LA_Code = 'E06000066';


Step 3: Create Sales Summary Table
CREATE TABLE Sales_Summary AS
SELECT 
    District,
    SUM(CASE WHEN Property_Type = 'D' THEN 1 ELSE 0 END) AS Sold_D,
    SUM(CASE WHEN Property_Type = 'S' THEN 1 ELSE 0 END) AS Sold_S,
    SUM(CASE WHEN Property_Type = 'T' THEN 1 ELSE 0 END) AS Sold_T,
    SUM(CASE WHEN Property_Type = 'F' THEN 1 ELSE 0 END) AS Sold_F
FROM pp_december
WHERE Record_Status = 'A'
GROUP BY District;


Step 4: Join All table

CREATE TABLE Join_All AS
SELECT 
    B.Region_name,
    B.LA_Name,
    B.Detached AS Total_D,
    S.Sold_D,
    B.Semi_detached AS Total_S,
    S.Sold_S,
    B.Terraced AS Total_T,
    S.Sold_T,
    B.Flats AS Total_F,
    S.Sold_F
FROM Join_Accommodation B
LEFT JOIN Sales_Summary S ON UPPER(B.LA_Name) = UPPER(S.District);



step 5: find the percent and Final output

CREATE TABLE Final_output AS
SELECT 
    Region_name,
    SUM(Sold_D) AS "Sum of D",
    SUM(Sold_F) AS "Sum of F",
    SUM(Sold_S) AS "Sum of S",
    SUM(Sold_T) AS "Sum of T",
    SUM(Total_D) AS "Base D",
    SUM(Total_F) AS "Base F",
    SUM(Total_S) AS "Base S",
    SUM(Total_T) AS "Base T",
	ROUND((SUM(Sold_D) * 100.0) / SUM(Total_D), 3) AS "%_D",
    ROUND((SUM(Sold_F) * 100.0) / SUM(Total_F), 3) AS "%_F",
    ROUND((SUM(Sold_S) * 100.0) / SUM(Total_S), 3) AS "%_S",
    ROUND((SUM(Sold_T) * 100.0) / SUM(Total_T), 3) AS "%_T"
FROM Join_All
GROUP BY Region_name;
