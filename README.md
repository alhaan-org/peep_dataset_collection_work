# Peep Dataset Cleaning And Analysis:

In this repo: I have cleaned peep registry logs and peep occupancy logs in the given dataset according to client requirements. If there are errors in data processing, issues are welcomed and will be further improved and effort to resolve these issues first hand:

Below are the scripts of Data Cleaning techniques using Pandas:

# Step 1

```python
# Listing Datasets and cleaning the peep registry file
import pandas as pd

df = pd.read_csv("pp_peep_registry.csv")

peep_stats = df["peep_status"].isna()
df["peep_status"] = df["peep_status"].fillna(value="Not Available")
df["peep_required"] = df["peep_required"].astype(str).str.lower()

yes_values = ["y", "yes", "true"]
no_values = ["n", "no", "false"]

df["peep_required"] = df["peep_required"].apply(
    lambda x: "Yes" if x in yes_values else ("No" if x in no_values else "Not Specified")
)

mobility_category = df["mobility_category"].isna()
df["mobility_category"] = df["mobility_category"].fillna(value="Category Not Specified")

df["last_review_date"] = pd.to_datetime(df["last_review_date"], format="mixed", dayfirst=True, errors='coerce')

property_names = {"London HQ": "LDN-HQ", "Bristol Harbourside": "BRI-01", "Edinburgh Exchange" : "EDI-EX", "Manchester City Campus" : "MCH-CC", "Leeds Arena Point": "LDS-AR", "Birmingham Paradise": "BHM-PR", "Cardiff Bay": "CDF-BY", "Belfast Lagan": "BEL-LG", "Norwich Quarter": "NRW-QP", "Glasgow Clyde" : "GLW-CL"}
df["property_code"] = df["property_code"].replace(property_names)
df["property_code"] = df["property_code"].str.upper()
df = df[~((df["peep_required"] == "No") & (df["peep_status"] == "Not Available"))]
bad_statuses = ["Not Available", "Pending", "Expired", "Revoked"]
df["risk_status"] = df.apply(
    lambda row: "Potential" if (row["peep_status"] in bad_statuses and row["peep_required"] == "Yes") else "Not Potential",
    axis=1
)
df = df.drop_duplicates()
df.head(100)
# df.to_csv("cleaned_peep_registry.csv", index=False)

```

# Step 2
```python
df2 = pd.read_csv("pp_property_occupancy_logs.csv")
df2["timestamp"] = pd.to_datetime(df2["timestamp"], format="mixed", errors="coerce", dayfirst=True)
# df2["timestamp"] = pd.to_datetime(df2["timestamp"], errors="coerce", utc=True)
# df2["timestamp"] = df2["timestamp"]

df2["property_code"] = df2["property_code"].replace(property_names)
df2["property_code"] = df2["property_code"].str.upper()
df2["property_code"] = df2["property_code"].replace("Unknown", "")
df2["property_code"] = df2["property_code"].fillna("Not Available")
df2["property_code"] = df2["person_id"].map(df.set_index("person_id")["property_code"])

df2.loc[df2["property_code"].isna()]

df2["property_code"] = df2["property_code"].fillna(df["person_id"].map(
    df.set_index("person_id")["property_code"]
))

df2 = df2.sort_values(["event_id"])
direction_in = df2[df2["direction"] == "IN"]
direction_out = df2[df2["direction"] == "OUT"]

in_count = direction_in.groupby("person_id").size()
out_count = direction_out.groupby("person_id").size()

visits = pd.DataFrame({
    "in_count" : in_count,
    "out_count": out_count
}).fillna(0)

visits["total_visits"] = visits[["in_count", "out_count"]].min(axis=1)

df2 = df2.merge(visits["total_visits"], on="person_id", how="left")

df2["property_code"] = df2["property_code"].fillna("Not Specified")
df_merged = df2.merge(df, on="person_id", how="inner")
df_merged = df_merged.drop(columns=["property_code_y"])
df_merged = df_merged.rename(columns={"property_code_x": "property_code", "total_visits": "peep_visits"})
peep_visits = df_merged.groupby("person_id").size().reset_index(name="total_vists_correct")
df_merged = df_merged.merge(peep_visits, on="person_id", how="inner")
df_merged.to_csv("cleaned_pp_property_occupancy_logs.csv", index=False)
df_merged.head(10)

```

Note: Recommendations are welcomed :)
