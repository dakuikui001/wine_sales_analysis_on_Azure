# Data Table Schema Documentation

This document provides detailed descriptions of all data tables in the project, including column names, data types, and remarks.

**Generated Date**: 2025-01-27  
**Project**: Wine Sales Analysis on Azure

---

## Table of Contents

1. [Gold Layer Data Tables](#gold-layer-data-tables)
   - [product (Product Table)](#product-product-table)
   - [processed_sales (Processed Sales Data Table)](#processed_sales-processed-sales-data-table)
   - [stores (Stores Table)](#stores-stores-table)
   - [territory (Territory Table)](#territory-territory-table)
   - [dates (Date Dimension Table)](#dates-date-dimension-table)
   - [currency (Currency Table)](#currency-currency-table)
   - [currency_rate (Currency Exchange Rate Table)](#currency_rate-currency-exchange-rate-table)

2. [Silver Layer Data Tables](#silver-layer-data-tables)
   - [Silver Sales Data](#silver-sales-data)

3. [Bronze Layer Data Tables](#bronze-layer-data-tables)
   - [Bronze Product Data](#bronze-product-data)
   - [Bronze Sales Data](#bronze-sales-data)
   - [Bronze Metadata](#bronze-metadata)

---

## Gold Layer Data Tables

The Gold layer is a business-ready aggregated data layer stored in Delta Lake format.

### product (Product Table)

**Table Description**: Master product data table containing detailed information for all products, including product ID, price, score, origin, etc.

**Storage Location**: `medallion/gold/product`  
**Data Format**: Delta Lake  
**Data Source**: Processed and loaded from Bronze layer product data

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| ProductId | int64 | Product ID | Primary key, unique product identifier |
| Country | string | Country | Product origin country |
| Score | int64 | Score | Product score (integer) |
| DealerPrice | double | Dealer Price | Dealer purchase price |
| Markup | double | Markup Rate | Price markup ratio |
| ListPrice | double | List Price | Product list price/retail price |
| Province | string | Province | Product origin province |
| Region_1 | string | Region 1 | Product origin level 1 region |
| Region_2 | string | Region 2 | Product origin level 2 region |
| Title | string | Title | Product name/title |
| Vintage | int64 | Vintage | Wine vintage year |
| Variety | string | Variety | Grape variety |
| Winery | string | Winery | Producer winery name |
| Year | int64 | Year | Production year (may be the same as Vintage) |
| Store | string | Store | Sales store/website name |
| load_timestamp | dateTime | Load Timestamp | Time when data was loaded to Gold layer |

---

### processed_sales (Processed Sales Data Table)

**Table Description**: Processed sales transaction data table containing detailed sales transaction information, already associated with product data (includes ProductId).

**Storage Location**: `medallion/gold/processed_sales`  
**Data Format**: Delta Lake  
**Data Source**: Generated from Gold layer sales data associated with product data

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| OnlineRetailer | string | Online Retailer | Sales channel/retailer name |
| SalesMonth | dateTime | Sales Month | Month when sales occurred (last day of month) |
| ProductId | int64 | Product ID | Foreign key linking to product table |
| Title | string | Title | Product name/title |
| Vintage | int64 | Vintage | Wine vintage year |
| Variety | string | Variety | Grape variety |
| Score | double | Score | Product score |
| ListPrice | double | List Price | Product list price/retail price |
| Quantity | int64 | Quantity | Sales quantity |
| Load_timestamp | dateTime | Load Timestamp | Time when data was loaded to Gold layer |

**Table Relationships**:
- `OnlineRetailer` → `stores.StoreName`
- `ProductId` → `product.ProductId`
- `SalesMonth` → `dates.LastDayOfMonth`

---

### stores (Stores Table)

**Table Description**: Stores/retailers master data table containing basic information for all retailers.

**Storage Location**: `medallion/gold/stores`  
**Data Format**: Delta Lake  
**Data Source**: Bronze layer MasterData/Store.csv file

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| StoreName | string | Store Name | Primary key, retailer/store name |
| StoreType | string | Store Type | Store type classification |
| Description | string | Description | Store description information |
| load_timestamp | dateTime | Load Timestamp | Time when data was loaded to Gold layer |

---

### territory (Territory Table)

**Table Description**: Territory master data table containing geographic region hierarchy information.

**Storage Location**: `medallion/gold/territory`  
**Data Format**: Delta Lake  
**Data Source**: Bronze layer MasterData/Territory.csv file

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| TerritoryCode | string | Territory Code | Primary key, unique territory code |
| TerritoryName | string | Territory Name | Territory name |
| TradeRegion | string | Trade Region | Trade region classification |
| Continent | string | Continent | Associated continent |
| load_timestamp | dateTime | Load Timestamp | Time when data was loaded to Gold layer |

---

### dates (Date Dimension Table)

**Table Description**: Date dimension table for time series analysis and reporting.

**Storage Location**: `medallion/gold/dates`  
**Data Format**: Delta Lake  
**Data Source**: Bronze layer MasterData/Dates.csv file

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| DateYear | int64 | Year | Year (integer) |
| DateMonth | int64 | Month | Month (1-12) |
| YearMonth | string | Year-Month | Year-month string (e.g., "2023-01") |
| LastDayOfMonth | dateTime | Last Day of Month | Last day of month (used for association) |
| Quarter | int64 | Quarter | Quarter (1-4) |
| Season | string | Season | Season name |
| load_timestamp | dateTime | Load Timestamp | Time when data was loaded to Gold layer |

**Hierarchy**: Date Hierarchy
- DateYear (Year)
  - Quarter (Quarter)
    - YearMonth (Year-Month)

---

### currency (Currency Table)

**Table Description**: Currency master data table containing currency codes and names.

**Storage Location**: `medallion/gold/currency`  
**Data Format**: Delta Lake  
**Data Source**: Bronze layer MasterData/Currency.csv file

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| CurrencyCode | string | Currency Code | Primary key, currency code (e.g., EUR, GBP, USD) |
| CurrencyName | string | Currency Name | Full currency name |
| load_timestamp | timestamp | Load Timestamp | Time when data was loaded to Gold layer |

---

### currency_rate (Currency Exchange Rate Table)

**Table Description**: Currency exchange rate table containing exchange rate information between different currencies.

**Storage Location**: `medallion/gold/currency_rate`  
**Data Format**: Delta Lake  
**Data Source**: Bronze layer MasterData/ExchangeRates.csv file

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| FromCurrency | string | From Currency | Source currency code |
| ToCurrency | string | To Currency | Target currency code |
| EffectiveDate | date | Effective Date | Exchange rate effective date |
| AverageRate | double | Average Rate | Average exchange rate value |
| load_timestamp | timestamp | Load Timestamp | Time when data was loaded to Gold layer |

**Usage Notes**: This table is used to convert sales data from non-EUR currencies to EUR (e.g., GBP→EUR conversion).

---

## Silver Layer Data Tables

The Silver layer is a cleaned, validated, and enriched data layer stored in Parquet format.

### Silver Sales Data

**Table Description**: Silver layer sales data, stored by store category (Celeste, Arancione, Verde). Data has been validated, currency converted, and error processed.

**Storage Location**: 
- `medallion/silver/Celeste` (Parquet format)
- `medallion/silver/Arancione` (Parquet format)
- `medallion/silver/Verde` (Parquet format)
- `medallion/silver/salesdata/` (Merged sales data)

**Data Format**: Parquet

#### Silver Layer Sales Data Common Schema

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| OnlineRetailer | string | Online Retailer | Sales channel/retailer name |
| SalesMonth | date | Sales Month | Month when sales occurred |
| Title | string | Title | Product name/title |
| Vintage | short/int64 | Vintage | Wine vintage year |
| Variety | string | Variety | Grape variety |
| Score | double | Score | Product score |
| ListPrice | double | List Price | Product list price (converted to EUR) |
| Quantity | long/int64 | Quantity | Sales quantity |

**Data Processing Notes**:
- Data from Celeste and Arancione stores is converted from CSV format
- Data from Verde store is converted from JSON format
- All sales prices in non-EUR currencies have been converted to EUR (using currency_rate table)
- Currency validation has been performed (only EUR/GBP allowed)
- Error rows have been separately captured in ErrorRows table
- Data has been aggregated by month

---

## Bronze Layer Data Tables

The Bronze layer is the raw, unprocessed data layer that preserves the original data format.

### Bronze Product Data

**Table Description**: Bronze layer product data from CSV files in the ProductData folder.

**Storage Location**: `medallion/bronze/ProductData/`  
**Data Format**: CSV

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| ProductId | int32 | Product ID | Unique product identifier |
| Country | string | Country | Product origin country |
| Score | int16 | Score | Product score |
| DealerPrice | int16 | Dealer Price | Dealer purchase price |
| Markup | double | Markup Rate | Price markup ratio |
| ListPrice | double | List Price | Product list price |
| Province | string | Province | Product origin province |
| Region_1 | string | Region 1 | Product origin level 1 region |
| Region_2 | string | Region 2 | Product origin level 2 region |
| Title | string | Title | Product name |
| Vintage | int16 | Vintage | Wine vintage year |
| Variety | string | Variety | Grape variety |
| Winery | string | Winery | Producer winery name |
| Year | int16 | Year | Production year |
| website | string | Website | Sales website name (extracted from filename) |

---

### Bronze Sales Data

#### Celeste Store Sales Data (CSV)

**Storage Location**: `medallion/bronze/Celeste/`  
**Data Format**: CSV

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| TransactionId | string | Transaction ID | Unique transaction identifier |
| TransactionDate | string | Transaction Date | Transaction date (string format) |
| OnlineRetailer | string | Online Retailer | Retailer name |
| SalesMonth | string | Sales Month | Sales month (string format) |
| SalesRegion | string | Sales Region | Sales region |
| SalesCurrency | string | Sales Currency | Currency code (EUR or GBP) |
| Title | string | Title | Product name |
| Vintage | string | Vintage | Wine vintage year (string format) |
| Variety | string | Variety | Grape variety |
| Score | string | Score | Product score (string format) |
| ListPrice | string | List Price | Product list price (string format) |
| Quantity | string | Quantity | Sales quantity (string format) |

#### Arancione Store Sales Data (CSV)

**Storage Location**: `medallion/bronze/Arancione/`  
**Data Format**: CSV

Schema is similar to Celeste, containing the following main columns:
- OnlineRetailer
- SalesMonth
- Title
- Vintage
- Variety
- Score
- ListPrice
- Quantity

#### Verde Store Sales Data (JSON)

**Storage Location**: `medallion/bronze/Verde/`  
**Data Format**: JSON

**JSON Structure**:
```json
{
  "YearMonth": "string",      // Year-month
  "StoreName": "string",      // Store name
  "Sales": {
    "Product": "string",      // Product name
    "Vintage": "string",      // Vintage
    "Variety": "string",      // Variety
    "Score": "string",        // Score
    "SalesPrice": "string",   // Sales price
    "SalesQty": "string"      // Sales quantity
  }
}
```

---

### Bronze Metadata

#### Currency.csv (Currency Master Data)

**Storage Location**: `medallion/bronze/MasterData/Currency.csv`  
**Data Format**: CSV

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| CurrencyCode | string | Currency Code | Currency code (e.g., EUR, GBP) |
| CurrencyName | string | Currency Name | Full currency name |

#### Dates.csv (Date Master Data)

**Storage Location**: `medallion/bronze/MasterData/Dates.csv`  
**Data Format**: CSV

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| DateYear | short | Year | Year |
| DateMonth | short | Month | Month (1-12) |
| YearMonth | string | Year-Month | Year-month string |
| LastDayOfMonth | string | Last Day of Month | Last day of month (string, format: M/d/yyyy) |
| Quarter | short | Quarter | Quarter (1-4) |
| Season | string | Season | Season name |

#### ExchangeRates.csv (Exchange Rate Master Data)

**Storage Location**: `medallion/bronze/MasterData/ExchangeRates.csv`  
**Data Format**: CSV

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| FromCurrency | string | From Currency | Source currency code |
| ToCurrency | string | To Currency | Target currency code |
| EffectiveDate | string | Effective Date | Effective date (string, format: M/d/yyyy) |
| AverageRate | double | Average Rate | Average exchange rate value |

#### Store.csv (Store Master Data)

**Storage Location**: `medallion/bronze/MasterData/Store.csv`  
**Data Format**: CSV

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| StoreName | string | Store Name | Retailer/store name |
| StoreType | string | Store Type | Store type classification |
| Description | string | Description | Store description information |

#### Territory.csv (Territory Master Data)

**Storage Location**: `medallion/bronze/MasterData/Territory.csv`  
**Data Format**: CSV

| Column Name | Data Type | Description | Remarks |
|------------|-----------|-------------|---------|
| TerritoryCode | string | Territory Code | Unique territory code |
| TerritoryName | string | Territory Name | Territory name |
| TradeRegion | string | Trade Region | Trade region classification |
| Continent | string | Continent | Associated continent |

---

## Semantic Model Auxiliary Tables

The following tables are used in the Microsoft Fabric semantic model but are not directly stored in the data lake:

### Measure Table

**Table Description**: Table containing definitions for all business measure calculations.

**Measure Definitions**:
- **Total Sales**: Total Sales Amount = SUMX(processed_sales, ListPrice * Quantity)
- **Total Orders**: Total Order Count = SUM(Quantity)
- **Total Cost**: Total Cost = SUMX(processed_sales, Quantity * DealerPrice)
- **Gross Profit**: Gross Profit = Total Sales - Total Cost
- **Gross Margin %**: Gross Margin % = Gross Profit / Total Sales
- **Previous Month Orders**: Previous Month Order Count
- **Previous Month Sales**: Previous Month Sales Amount
- **Previous Month Profit**: Previous Month Profit
- **Target Gross Profit**: Target Gross Profit = Previous Month Profit * 1.2
- **Target Orders**: Target Order Count = Previous Month Orders * 1.2
- **Target Sales**: Target Sales Amount = Previous Month Sales * 1.2
- **Average Score**: Average Score = AVERAGE(product[Score])
- **Adjusted Sales**: Adjusted Sales Amount = SUMX(processed_sales, Quantity * ListPrice * (1 + price adjustment))
- **Adjusted Profit**: Adjusted Profit = Adjusted Sales - Total Cost

---

### price adjustment (Price Adjustment Table)

**Table Description**: Parameter table for price sensitivity analysis.

**Column Definitions**:
- **price adjustment**: double type, range from -1 to 1, step size 0.1
- **price adjustment Value**: Measure that returns the selected price adjustment value

---

### Orders Distribution Metrics Selection (Orders Distribution Metrics Selection Table)

**Table Description**: Selection table for order distribution analysis in reports.

**Column Definitions**:
- **Orders Distribution Metrics Selection**: Selectable metric fields (Score, Store, Variety, Vintage)
- **Orders Distribution Metrics Selection Fields**: Hidden column storing field names
- **Orders Distribution Metrics Selection Order**: Hidden column storing sort order

---

## Data Type Mapping Notes

### Data Type Comparison Table

| Semantic Model Type | Parquet/Delta Type | Description |
|---------------------|-------------------|-------------|
| int64 | INT64/LONG | 64-bit integer |
| double | DOUBLE | Double precision floating point |
| string | UTF8/STRING | String |
| dateTime | TIMESTAMP/DATE | Date and time |
| date | DATE | Date |
| short | INT16/SHORT | 16-bit integer |
| int32 | INT32 | 32-bit integer |

---

## Data Flow Notes

### Data Flow Path

1. **Landing → Bronze**: Original ZIP files are extracted and copied to Bronze layer
2. **Bronze → Silver**: 
   - Product data: Add Store column
   - Sales data: Validation, currency conversion, error processing
3. **Silver → Gold**:
   - Product data: Type conversion, null filtering
   - Sales data: Merge all store data, add timestamp
   - Metadata: Type conversion, add timestamp
4. **Gold → Semantic Model**: Direct access to Gold layer data via DirectLake mode

### Key Transformations

- **Currency Conversion**: GBP → EUR (using currency_rate table)
- **Date Conversion**: String dates → Standard date format
- **Aggregation**: Aggregate sales quantity by month
- **Association**: Sales data associated with product data via Variety, Title, Vintage

---

## Important Notes

1. **Timestamp Fields**: All Gold layer tables contain a `load_timestamp` field for data lineage tracking
2. **Currency Unification**: All sales data is unified to EUR in the Silver layer
3. **Error Handling**: Data rows that do not meet requirements are captured in the ErrorRows table
4. **Data Format**: Gold layer uses Delta Lake format, supporting ACID transactions, time travel, and schema evolution
5. **Association Keys**: 
   - `processed_sales.OnlineRetailer` → `stores.StoreName`
   - `processed_sales.ProductId` → `product.ProductId`
   - `processed_sales.SalesMonth` → `dates.LastDayOfMonth`

---

## Appendix

### Data File Path Structure

```
wine-project/
├── landing/                          # Raw data (ZIP files)
│   └── sampledata.zip
└── medallion/
    ├── bronze/                       # Bronze layer (raw data)
    │   ├── MasterData/              # Master data files
    │   │   ├── Currency.csv
    │   │   ├── Dates.csv
    │   │   ├── ExchangeRates.csv
    │   │   ├── Store.csv
    │   │   └── Territory.csv
    │   ├── ProductData/             # Product data
    │   ├── Celeste/                 # Celeste store sales data
    │   ├── Arancione/               # Arancione store sales data
    │   └── Verde/                   # Verde store sales data
    ├── silver/                       # Silver layer (cleaned data)
    │   ├── Celeste/                 # Parquet format
    │   ├── Arancione/               # Parquet format
    │   ├── Verde/                   # Parquet format
    │   └── salesdata/               # Merged sales data
    └── gold/                         # Gold layer (business-ready data)
        ├── product/                 # Delta format
        ├── processed_sales/         # Delta format
        ├── stores/                  # Delta format
        ├── territory/               # Delta format
        ├── dates/                   # Delta format
        ├── currency/                # Delta format
        └── currency_rate/           # Delta format
```

---

**Document Version**: 1.0  
**Last Updated**: 2025-01-27

