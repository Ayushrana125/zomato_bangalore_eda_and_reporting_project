## Data Cleaning & Transformation Steps

### 1. Loaded the Dataset

The original CSV file was loaded using Python and pandas because the dataset was large and Excel was not rendering the file reliably.

```python
df = pd.read_csv(file_path)
```

Initial checks performed:

```python
df.head()
df.shape
df.columns
df.info()
```

The dataset contained `51,717` rows and `17` columns.

---

### 2. Preserved Raw Data

The original dataframe `df` was kept unchanged as the raw source table.

A separate working dataframe was created for transformation:

```python
outlets = df.copy()
```

This ensures the raw dataset remains available for reference and validation.

---

### 3. Created Outlet and Restaurant IDs

Each row was treated as one restaurant outlet/listing.

Two IDs were created:

- `outlet_id`: unique ID for every row/outlet
- `restaurant_id`: common ID for outlets with the same restaurant name

```python
outlets["outlet_id"] = ["O" + str(i) for i in range(1, len(outlets) + 1)]

outlets["restaurant_id"] = pd.Series(
    pd.factorize(outlets["name"])[0] + 1,
    index=outlets.index
).astype(str)

outlets["restaurant_id"] = "R" + outlets["restaurant_id"]
```

`pd.factorize()` was used so restaurant IDs are assigned based on first appearance in the dataset.

---

### 4. Cleaned the Rate Column

The `rate` column contained values like `4.1/5`.

The `/5` part was removed and the column was converted to numeric format.

```python
outlets["rate"] = pd.to_numeric(
    outlets["rate"].str.replace("/5", "", regex=False),
    errors="coerce"
)
```

Invalid or missing rating values were converted to `NaN`.

---

### 5. Normalized Dish Data

The `dish_liked` column contained multiple comma-separated dishes in a single cell.

A separate dish dimension and mapping table were created:

- `dishes`
- `outlet_dish_mapping`

Steps performed:

```python
outlet_dish_mapping = (
    outlets[["outlet_id", "restaurant_id", "dish_liked"]]
    .dropna(subset=["dish_liked"])
    .copy()
)

outlet_dish_mapping["dish_liked"] = outlet_dish_mapping["dish_liked"].str.split(",")
outlet_dish_mapping = outlet_dish_mapping.explode("dish_liked")
outlet_dish_mapping["dish_liked"] = outlet_dish_mapping["dish_liked"].str.strip()
outlet_dish_mapping = outlet_dish_mapping[outlet_dish_mapping["dish_liked"] != ""]
```

Created the `dishes` dimension table:

```python
dishes = (
    outlet_dish_mapping[["dish_liked"]]
    .drop_duplicates()
    .reset_index(drop=True)
)

dishes["dish_id"] = ["D" + str(i) for i in range(1, len(dishes) + 1)]

dishes = dishes[["dish_id", "dish_liked"]]
dishes = dishes.rename(columns={"dish_liked": "dish_name"})
```

Mapped dish IDs back to outlet-level data.

---

### 6. Normalized Cuisine Data

The `cuisines` column also contained multiple comma-separated values.

Two tables were created:

- `cuisines`
- `outlet_cuisine_mapping`

```python
outlet_cuisine_mapping = (
    outlets[["outlet_id", "restaurant_id", "cuisines"]]
    .dropna(subset=["cuisines"])
    .copy()
)

outlet_cuisine_mapping["cuisines"] = outlet_cuisine_mapping["cuisines"].str.split(",")
outlet_cuisine_mapping = outlet_cuisine_mapping.explode("cuisines")
outlet_cuisine_mapping["cuisines"] = outlet_cuisine_mapping["cuisines"].str.strip()
outlet_cuisine_mapping = outlet_cuisine_mapping[outlet_cuisine_mapping["cuisines"] != ""]
```

Created the `cuisines` dimension table:

```python
cuisines = (
    outlet_cuisine_mapping[["cuisines"]]
    .drop_duplicates()
    .reset_index(drop=True)
)

cuisines["cuisine_id"] = ["C" + str(i) for i in range(1, len(cuisines) + 1)]

cuisines = cuisines[["cuisine_id", "cuisines"]]
cuisines = cuisines.rename(columns={"cuisines": "cuisine_name"})
```

---

### 7. Checked Cuisine Name Standardization

Cuisine names were checked for case, spacing, and duplicate formatting issues.

```python
cuisines["cuisine_name_clean"] = (
    cuisines["cuisine_name"]
    .str.strip()
    .str.lower()
    .str.replace(r"\s+", " ", regex=True)
)
```

Duplicate checks were performed after normalization.

```python
cuisine_duplicates = (
    cuisines.groupby("cuisine_name_clean")["cuisine_name"]
    .agg(list)
    .reset_index()
)

cuisine_duplicates = cuisine_duplicates[
    cuisine_duplicates["cuisine_name"].apply(len) > 1
]
```

No duplicate cuisine names were found after case and whitespace normalization.

The temporary helper column was removed:

```python
cuisines = cuisines.drop(columns=["cuisine_name_clean"], errors="ignore")
```

---

### 8. Removed Non-Cuisine Labels

Some values in the `cuisines` column represented food categories, beverages, or outlet formats rather than actual cuisines.

Examples included:

- `Cafe`
- `Bakery`
- `Beverages`
- `Desserts`
- `Coffee`
- `Tea`
- `Pizza`
- `Rolls`
- `Sandwich`

These labels were removed from both:

- `cuisines`
- `outlet_cuisine_mapping`

This was a semantic cleaning step based on business interpretation.

---

### 9. Normalized Restaurant Type Data

The `rest_type` column also contained multiple comma-separated values.

Two tables were created:

- `restaurant_types`
- `outlet_restaurant_type_mapping`

```python
outlet_restaurant_type_mapping = (
    outlets[["outlet_id", "restaurant_id", "rest_type"]]
    .dropna(subset=["rest_type"])
    .copy()
)

outlet_restaurant_type_mapping["rest_type"] = outlet_restaurant_type_mapping["rest_type"].str.split(",")
outlet_restaurant_type_mapping = outlet_restaurant_type_mapping.explode("rest_type")
outlet_restaurant_type_mapping["rest_type"] = outlet_restaurant_type_mapping["rest_type"].str.strip()
outlet_restaurant_type_mapping = outlet_restaurant_type_mapping[
    outlet_restaurant_type_mapping["rest_type"] != ""
]
```

Created the `restaurant_types` dimension table:

```python
restaurant_types = (
    outlet_restaurant_type_mapping[["rest_type"]]
    .drop_duplicates()
    .reset_index(drop=True)
)

restaurant_types["restaurant_type_id"] = [
    "RT" + str(i) for i in range(1, len(restaurant_types) + 1)
]

restaurant_types = restaurant_types[["restaurant_type_id", "rest_type"]]
restaurant_types = restaurant_types.rename(columns={"rest_type": "restaurant_type_name"})
```

---

### 10. Interpreted Restaurant Type Field

The `rest_type` field was identified as a broader outlet classification field.

It contains a mix of:

- dining formats: `Casual Dining`, `Fine Dining`, `Cafe`, `Pub`
- service models: `Delivery`, `Takeaway`
- shop/outlet types: `Beverage Shop`, `Sweet Shop`, `Meat Shop`, `Kiosk`

Therefore, `rest_type` should not be interpreted strictly as a restaurant dining category.

---

### 11. Exported Final Tables

The cleaned and transformed tables were exported as CSV files for Power BI reporting.

```python
outlets.to_csv("outlets.csv", index=False)
dishes.to_csv("dishes.csv", index=False)
outlet_dish_mapping.to_csv("outlet_dish_mapping.csv", index=False)
cuisines.to_csv("cuisines.csv", index=False)
outlet_cuisine_mapping.to_csv("outlet_cuisine_mapping.csv", index=False)
restaurant_types.to_csv("restaurant_types.csv", index=False)
outlet_restaurant_type_mapping.to_csv("outlet_restaurant_type_mapping.csv", index=False)
```

---

## Final Data Model Tables

The final transformed dataset contains the following tables:

- `outlets.csv`
- `dishes.csv`
- `outlet_dish_mapping.csv`
- `cuisines.csv`
- `outlet_cuisine_mapping.csv`
- `restaurant_types.csv`
- `outlet_restaurant_type_mapping.csv`

These tables were prepared for relational modeling and dashboard development in Power BI.
