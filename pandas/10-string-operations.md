# 10 — String Operations

Text data (names, categories, free-form entries) needs cleaning constantly. Pandas gives vectorized string operations via the `.str` accessor — same "no loop needed" philosophy as everything else in pandas/NumPy.

## The .str accessor — the key concept
```python
df["name"].str.upper()          # uppercase every value in the column, no loop needed
df["name"].str.lower()
df["name"].str.title()             # Title Case Each Word
```
Without `.str`, you can't call string methods directly on a whole column — `.str` is what tells pandas "apply this string method to every value in this Series."

## Common cleaning operations
```python
df["name"].str.strip()                  # remove leading/trailing whitespace
df["name"].str.replace(" ", "_")           # replace spaces with underscores
df["email"].str.contains("@gmail")           # boolean -- True where the string contains a substring
df["email"].str.startswith("soham")             # boolean -- starts with a substring
df["email"].str.endswith(".com")                  # boolean -- ends with a substring
df["name"].str.len()                                 # length of each string
```

## Splitting strings into multiple columns
```python
df["full_name"].str.split(" ")                        # splits into a list per row
df["full_name"].str.split(" ", expand=True)               # splits AND creates separate columns directly -- genuinely useful
df[["first_name", "last_name"]] = df["full_name"].str.split(" ", expand=True)     # split and assign to new columns in one line
```

## Extracting parts of a string using regex
```python
df["phone"].str.extract(r"(\d{3})-(\d{3})-(\d{4})")     # extract groups using a regex pattern
df["email"].str.extract(r"@(\w+)\.com")                    # extract the domain name from an email
```

## Filtering rows based on string content (combines with boolean indexing from file 03)
```python
df[df["name"].str.contains("Soham", case=False)]     # case=False -> case-insensitive search
df[df["name"].str.contains("Soham", na=False)]           # na=False -> treat missing values as False instead of erroring
```
That `na=False` is one I learned the hard way — `.str.contains()` on a column with missing values throws an error by default since it can't check "contains" on a NaN, so this parameter avoids that crash.

## Checking string patterns
```python
df["code"].str.isnumeric()      # True if the entire string is numeric characters
df["code"].str.isalpha()          # True if entirely alphabetic
df["code"].str.isupper()             # True if entirely uppercase
```

## Concatenating string columns
```python
df["full_name"] = df["first_name"] + " " + df["last_name"]     # simple concatenation with +
df["full_name"] = df["first_name"].str.cat(df["last_name"], sep=" ")     # alternative using .str.cat()
```

## Replacing using regex
```python
df["phone"].str.replace(r"[^0-9]", "", regex=True)     # strip out everything except digits from a messy phone number column
```
This regex-cleaning pattern comes up a lot with real-world messy data — phone numbers, currency values with symbols, inconsistent formatting.

## Padding strings (useful for IDs, codes)
```python
df["id"].astype(str).str.zfill(5)     # pad with leading zeros to make every ID 5 digits long, e.g. '7' -> '00007'
```

## Encoding categorical text data (brief preview — real depth comes in scikit-learn later)
```python
df["gender_encoded"] = df["gender"].map({"M": 0, "F": 1})     # manual encoding, same map() concept from file 05
```
This connects directly to encoding categorical variables for ML models — I've already touched LabelEncoder/OneHotEncoder conceptually from scikit-learn material, and this manual `.map()` version is essentially what those tools automate at scale.
