# Zomato Bangalore Restaurants EDA & Power BI Report

## Project Overview

This project analyzes the **Zomato Bangalore Restaurants** dataset sourced from Kaggle. The dataset contains restaurant listings from Bengaluru with details such as restaurant name, location, cuisine, restaurant type, rating, votes, approximate cost, liked dishes, and service options.

The main goal of this project is to clean, structure, and analyze the raw restaurant data so it can be used for meaningful exploratory data analysis and Power BI reporting.

Dataset source: [Zomato Bangalore Restaurants on Kaggle](https://www.kaggle.com/datasets/himanshupoddar/zomato-bangalore-restaurants)

## Business Context

Bengaluru has a large and diverse restaurant market, with restaurants serving food from many regions and cuisines. The dataset can help explore questions such as:

- Which cuisines are most widely available across Bengaluru outlets?
- Which dishes appear most frequently in the `dish_liked` field?
- Which restaurant formats are common across the city?
- How are outlets distributed across localities?
- What patterns exist around ratings, votes, cost, online ordering, and table booking?

## Tools Used

- **Python**: data loading, cleaning, transformation, normalization, and EDA
- **pandas**: dataframe manipulation and CSV exports
- **Power BI**: dashboard creation and visual reporting
- **Kaggle**: dataset source

Python was used instead of Excel because the raw CSV was large and contained complex text fields that were not rendering cleanly in Excel.

## Dataset Summary

The raw dataset contains:

- `51,717` rows
- `17` columns
- restaurant details, ratings, votes, cuisine labels, liked dishes, restaurant types, and locality information

The original raw file was preserved, and all transformations were performed on derived working tables.

## What I Did In This Project

### 1. Initial Data Understanding

I started by loading the raw CSV into pandas and inspecting the dataset structure using:

- `head()`
- `shape`
- `columns`
- `info()`
- missing value checks
- value counts for important categorical columns

This helped identify the data grain, column meanings, missing values, and columns that required transformation.

### 2. Identified the Dataset Grain

I identified that each row represents a restaurant outlet/listing rather than a unique restaurant brand.

For example, a restaurant like `Cafe Coffee Day` appears multiple times because it has multiple outlets. Therefore, repeated restaurant names were not treated as duplicate records by default.

### 3. Created Outlet and Restaurant IDs

Two ID columns were created:

- `outlet_id`: unique ID for each row/outlet
- `restaurant_id`: common ID for all outlets sharing the same restaurant name

This made the dataset easier to model and connect with normalized mapping tables.

### 4. Cleaned the Rating Column

The `rate` column had values such as `4.1/5`. I removed the `/5` part and converted the column into a numeric format so it can be used properly in analysis and Power BI visuals.

Invalid values were converted into null values for cleaner handling.

### 5. Normalized Multi-Value Columns

Some columns contained multiple comma-separated values in a single cell. These were split and normalized into separate dimension and mapping tables.

The normalized fields were:

- `dish_liked`
- `cuisines`
- `rest_type`

This transformation helped convert messy text fields into a more analysis-ready relational structure.

### 6. Created Dishes Dimension

The `dish_liked` column was split into individual dishes.

Created tables:

- `dishes.csv`
- `outlet_dish_mapping.csv`

This allows analysis of which dishes appear most frequently across outlets.

### 7. Created Cuisines Dimension

The `cuisines` column was split into individual cuisine labels.

Created tables:

- `cuisines.csv`
- `outlet_cuisine_mapping.csv`

I also reviewed the cuisine labels semantically and removed labels that were not actual cuisines, such as some beverage, dessert, and outlet-format categories.

### 8. Created Restaurant Types Dimension

The `rest_type` column was split into individual restaurant type labels.

Created tables:

- `restaurant_types.csv`
- `outlet_restaurant_type_mapping.csv`

The `rest_type` field was interpreted as a broader outlet classification field because it includes dining formats, service models, and shop/outlet types.

Examples:

- Dining formats: `Casual Dining`, `Fine Dining`, `Cafe`, `Pub`
- Service models: `Delivery`, `Takeaway`
- Shop/outlet types: `Beverage Shop`, `Sweet Shop`, `Meat Shop`, `Kiosk`

### 9. Performed Exploratory Analysis

After cleaning and structuring the data, I performed EDA to answer early analytical questions.

Examples:

- outlet count by cuisine
- outlet count by liked dish
- restaurant count by name
- locality-level restaurant distribution
- interpretation of broad cuisine labels such as `North Indian`, `South Indian`, and `Indian`

One important finding was that `North Indian` appeared as the most widely available cuisine label. This was interpreted carefully as a platform tagging pattern, not necessarily as proof of absolute city-wide food preference.

### 10. Prepared Data for Power BI

The final cleaned tables were exported as CSV files and prepared for Power BI modeling.

The transformed tables can be connected using IDs such as:

- `outlet_id`
- `restaurant_id`
- `dish_id`
- `cuisine_id`
- `restaurant_type_id`

## Final Output Tables

The final transformed tables are stored in the `Tables_Created` folder:

- `outlets.csv`
- `dishes.csv`
- `outlet_dish_mapping.csv`
- `cuisines.csv`
- `outlet_cuisine_mapping.csv`
- `restaurant_types.csv`
- `outlet_restaurant_type_mapping.csv`

## Project Files

- `Zomato_Bangalore_EDA.ipynb`: Python notebook containing EDA, cleaning, and transformation logic
- `Data_Cleaning_and_Transformation.md`: detailed cleaning and transformation documentation
- `Tables_Created/`: folder containing final exported CSV tables
- `zomato_raw.csv`: original raw dataset

## Key Analytical Notes

- The original dataset contains repeated restaurant names because the same restaurant brand can have multiple outlets.
- `listed_in(city)` represents Bengaluru localities/areas rather than different cities.
- `cuisines` is a mixed platform category field and required semantic review before analysis.
- `dish_liked` reflects dishes mentioned as liked/listed in the dataset, not actual sales volume.
- `rest_type` contains restaurant formats, service models, and outlet/shop types.
- Outlet count should be interpreted as outlet presence, not direct customer demand.

## Power BI Reporting Plan

The cleaned tables are intended to support a Power BI dashboard with visuals such as:

- top cuisines by outlet count
- top liked dishes by outlet count
- outlet distribution by locality
- restaurant type distribution
- ratings and votes analysis
- cost analysis by cuisine, locality, or restaurant type
- online ordering and table booking availability

## Future Scope

The dataset also contains a `reviews_list` column, which can be used for advanced text analysis.

Future enhancements may include:

- review sentiment analysis using an LLM API
- theme extraction from customer reviews
- identifying common positive and negative feedback patterns
- comparing review sentiment with ratings and votes
- building a richer recommendation or market analysis layer

## Project Status

The data cleaning, transformation, normalization, and initial EDA are complete. The next phase is Power BI dashboard development and insight storytelling.
