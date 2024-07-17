# Clean House selling data and makes it easier to use

# Overview:
1. Project Name
2. Objectives
3. Data source
4. Tools
5. Code


# 1. Project Name:

**Clean House selling data and makes it easier to use**

# 2. Objectives:

The project aims to clean data to make mapping more convenient through: data formatting, split field, data filling, remove duplicate, synchronize data values.

# 3. Data source:

Data on home sales in the US during the period 2013-2019.
- *Source: [Alex The Analyst](https://github.com/AlexTheAnalyst/PortfolioProjects/blob/main/Nashville%20Housing%20Data%20for%20Data%20Cleaning.xlsx)*

# 4. Tools

- Tools:
  * SQL: JOIN, UPDATE, SUBSTRING, PARSENAME, WINDOW FUNCTION, CTE

# 5. Code
	
SELECT * 
FROM Houseselling

-- CREATE TEMP TABLE FOR CLEANING

	SELECT *
	INTO ##DATA
	FROM Houseselling

-- 1.STANDARDIZE DATE FORMAT

	ALTER TABLE ##DATA
		ADD Sale_date DATE 

	UPDATE ##DATA
	SET Sale_date = CONVERT (DATE, SaleDate)

-- 2. POPULATE PROPERTY ADDRESS DATA
	
	UPDATE A
	SET A.PropertyAddress = B.PropertyAddress
	FROM ##DATA A
	JOIN ##DATA B ON A.ParcelID = B.ParcelID
	WHERE B.PropertyAddress IS NOT NULL

-- 3. BREAKING ADRESS INTO INDIVIDUAL COLUMNS (ADDRESS, CITY, STATE)

----- SPLIT PROPERTY ADDRESS

	ALTER TABLE ##DATA
		ADD [PropertyAdress] NVARCHAR(255)

	UPDATE ##DATA
	SET [PropertyAdress] = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1 )
	
	ALTER TABLE ##DATA
		ADD PropertyCity NVARCHAR(255)

	UPDATE ##DATA
	SET PropertyCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress))
	
----- SPLIT OWNERADDRESS INTO ADDRESS, CITY, STATE

	ALTER TABLE ##DATA
		ADD Owneraddress_split NVARCHAR(255)
	
	UPDATE ##DATA
	SET Owneraddress_split = PARSENAME(REPLACE(OWNERADDRESS,',','.'),3)

	ALTER TABLE ##DATA
		ADD Ownercity_split NVARCHAR(255)
	
	UPDATE ##DATA
	SET Ownercity_split = PARSENAME(REPLACE(OWNERADDRESS,',','.'),2)

	ALTER TABLE ##DATA
		ADD Ownerstate_split NVARCHAR(255)
	
	UPDATE ##DATA
	SET Ownerstate_split = PARSENAME(REPLACE(OWNERADDRESS,',','.'),1)

-- 4. CHANGE YES AND NO TO Y AND N IN  "SOLID AS VACANT" FIED

	UPDATE ##DATA
	SET SoldAsVacant = CASE		
							WHEN SoldAsVacant = 'Yes' THEN 'Y'
							WHEN SoldAsVacant = 'No' THEN 'N'
							ELSE SoldAsVacant
						END

-- 5. REVOME DUPLICATE

	WITH ROWNUM AS (
	SELECT *,
			ROW_NUMBER() OVER (	
					PARTITION BY ParcelID,
							PropertyAddress,
							SalePrice,
							Saledate,
							LegalReference
					ORDER BY UniqueID
						) row_num

	FROM ##DATA
	)
	DELETE 
	FROM ROWNUM
	WHERE row_num > 1

-- 6. DELETE UNUSED COLUMN
	
	ALTER TABLE ##DATA
		DROP COLUMN PropertyAddress,OwnerAddress, TaxDistrict, saledate

-- CREATE NEW CLEAN TABLE
	
	DROP TABLE IF EXISTS HOUSESELLING_UPDATE
	SELECT *
	INTO HOUSESELLING_UPDATE
	FROM ##DATA
 
