# 📚 Housing

 1. [Data cleaning/exploring](#data-cleaning/exploring)
 2. [Dashboard](#dashboard)


## Data cleaning/exploring 
### Creating copy of the dataset

```sql
select * from `Nashville Housing Data for Data Cleaning.xlsx - Sheet1`;


ALTER TABLE `Nashville Housing Data for Data Cleaning.xlsx - Sheet1`
RENAME TO housing_data;

Select * from housing_data;

```

### Converting datetime to date

```sql
SELECT SaleDate, CAST(SaleDate as Date) FROM housing_data;

UPDATE housing_data
SET SaleDate = CAST(SaleDate as Date);

ALTER TABLE housing_data
ADD SaleDateConverted Date;

UPDATE housing_data
SET SaleDateConverted = CAST(SaleDate as Date);
```
### Populate Property Address
```sql
UPDATE housing_data SET PropertyAddress = NULL WHERE PropertyAddress= '';

SELECT *
FROM housing_data
WHERE PropertyAddress is NULL
Order by ParcelID;

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, a.PropertyAddress, IFNULL(a.PropertyAddress,b.PropertyAddress) 
FROM housing_data a
JOIN housing_data b
	ON a.ParcelID = b.ParcelID
    AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress is Null;


UPDATE housing_data a
JOIN housing_data b
	ON a.ParcelID = b.ParcelID
    AND a.UniqueID <> b.UniqueID
SET a.PropertyAddress = IFNULL(a.PropertyAddress,b.PropertyAddress)
WHERE a.PropertyAddress is Null;
```

### Breaking out address into individual columns (Address, City, State)
```SELECT PropertyAddress
FROM housing_data;

SELECT 
SUBSTRING(PropertyAddress, 1, LOCATE(',', PropertyAddress) - 1 ) AS Address, 
SUBSTRING(PropertyAddress, LOCATE(',', PropertyAddress) +1 , LENGTH(PropertyAddress)) AS Address
FROM housing_data;

ALTER TABLE housing_data
ADD PropertySplitAddress VARCHAR(255);

UPDATE housing_data
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, LOCATE(',', PropertyAddress) - 1 );

ALTER TABLE housing_data
ADD PropertySplitCity VARCHAR(255);

UPDATE housing_data
SET PropertySplitCity = SUBSTRING(PropertyAddress, LOCATE(',', PropertyAddress) +1 , LENGTH(PropertyAddress));

SELECT *
FROM housing_data;

SELECT OwnerAddress
FROM housing_data;

SELECT
SUBSTRING_INDEX(OwnerAddress, ',', 1) as street,
SUBSTRING_INDEX(SUBSTRING_INDEX(OwnerAddress, ',', -2), ',', 1) as city,
SUBSTRING_INDEX(OwnerAddress, ',', -1) as state
FROM housing_data;


ALTER TABLE housing_data
ADD OwnerSplitAddress VARCHAR(255);

UPDATE housing_data
SET OwnerSplitAddress = SUBSTRING_INDEX(OwnerAddress, ',', 1);

ALTER TABLE housing_data
ADD OwnerSplitCity VARCHAR(255);

UPDATE housing_data
SET OwnerSplitCity = SUBSTRING_INDEX(SUBSTRING_INDEX(OwnerAddress, ',', -2), ',', 1);

ALTER TABLE housing_data
ADD OwnerSplitState VARCHAR(255);

UPDATE housing_data
SET OwnerSplitState = SUBSTRING_INDEX(OwnerAddress, ',', -1);

SELECT *
FROM housing_data;
```
### Change Y and Yes and No in 'Sold as Vacant" field

```SELECT DISTINCT(SoldAsVacant), Count(SoldAsVacant)
FROM housing_data
GROUP BY SoldAsVacant
ORDER BY 2;

SELECT
CASE WHEN SoldAsVacant = "Y" THEN "Yes"
	WHEN SoldAsVacant = "N" THEN "No"
	ELSE SoldAsVacant
    END
FROM housing_data;

UPDATE housing_data
SET SoldAsVacant = CASE
	WHEN SoldAsVacant = "Y" THEN "Yes"
	WHEN SoldAsVacant = "N" THEN "No"
	ELSE SoldAsVacant
    END;
    
```

###  Remove Duplicates
```sql
WITH RowNumCTE AS(
SELECT *,
	ROW_NUMBER() OVER(
    PARTITION BY ParcelID,
				PropertyAddress,
				SalePrice,
                SaleDate,
                LegalReference
                ORDER BY 
					UniqueID
				) row_num
FROM housing_data
)

SELECT *
FROM RowNumCTE
WHERE row_num > 1;


DELETE FROM housing_data
WHERE UniqueID IN (
    SELECT UniqueID
    FROM (
        SELECT *,
            ROW_NUMBER() OVER (
                PARTITION BY ParcelID,
                             PropertyAddress,
                             SalePrice,
                             SaleDate,
                             LegalReference
                ORDER BY UniqueID
            ) AS row_num
        FROM housing_data
    ) AS RowNumCTE
    WHERE row_num > 1
);
```


### Deleting unwanted columns
```sql
SELECT *
FROM housing_data;

ALTER TABLE housing_data
DROP COLUMN OwnerAddress;

ALTER TABLE housing_data
DROP COLUMN PropertyAddress;

ALTER TABLE housing_data
DROP COLUMN TaxDistrict;

ALTER TABLE housing_data
DROP COLUMN SaleDate;
    
SELECT SaleDateConverted, SalePrice
FROM housing_data
Order by SalePrice DESC;

SELECT *
FROM housing_data;
```
