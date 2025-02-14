```python
from zlib import crc32

def is_id_in_test_set(identifier,test_ratio):
return crc32(np.int64(identifier)) < test_ratio \* 2\*\*32

def split*data_with_id_hash(data,test_ratio,id_column):
ids = data[id_column]
in_test_set = ids.apply(lambda id* : is*id_in_test_set(id*,test_ratio))
return data.loc[~in_test_set],data.loc[in_test_set]
```

## 1. Importing the CRC32 Function

```python
from zlib import crc32
```

#### What it does:

Imports the crc32 function from Python’s built-in zlib library.

#### Purpose:

crc32 computes a 32-bit hash (a number) for a given input. In this context, it’s used to generate a stable hash from a unique identifier.

## Defining is_id_in_test_set Function

```python
def is_id_in_test_set(identifier, test_ratio):
return crc32(np.int64(identifier)) < test_ratio \* 2\*\*32
```

#### Input Parameters:

- identifier: A unique value for each instance (for example, a row index or a combination of features).
- test_ratio: A decimal representing the fraction of data you want in the test set (e.g., 0.2 for 20%).

#### What it does:

It converts the identifier into a 64-bit integer using np.int64(identifier).
Then it computes the 32-bit hash of that number using crc32(...).
The expression test_ratio \* 2**32 calculates a threshold value.
Since crc32 returns an integer in the range [0, 2**32), multiplying the desired test ratio by 2\*\*32 gives a cutoff value.
The function returns True if the hash is less than this threshold, meaning that the instance should be in the test set.

#### Why it works:

This method assigns each instance a number based on its identifier. If the number is within the lowest test_ratio portion of the possible hash values, it goes into the test set. This decision is deterministic—meaning that every time you run it on the same identifier, it will produce the same result.

## 3. Defining split_data_with_id_hash Function

```python
def split*data_with_id_hash(data, test_ratio, id_column):
ids = data[id_column]
in_test_set = ids.apply(lambda id* : is*id_in_test_set(id*, test_ratio))
return data.loc[~in_test_set], data.loc[in_test_set]
```

#### Input Parameters:

- data: The entire dataset (typically a Pandas DataFrame).
- test_ratio: The proportion of data to be set aside as the test set.
- id_column: The name of the column in data that contains the unique identifier.

#### What it does:

- Extract Identifiers:
  ids = data[id_column]
- Determine Test Set Membership:
  in*test_set = ids.apply(lambda id* : is*id_in_test_set(id*, test_ratio))
- Split the Data:
  return data.loc[~in_test_set], data.loc[in_test_set]

data.loc[~in_test_set]: Selects rows that are not in the test set (the training set).
data.loc[in_test_set]: Selects rows that are in the test set.

#### Why it’s useful:

This function provides a stable way to split data into train and test sets based solely on a unique identifier. Even if you update the dataset, instances that have the same ID will always be assigned to the same set.

## 4. Using the Row Index as an Identifier

```python
housing_with_id = data.reset_index() ## Adding index column
train_set, test_set = split_data_with_id_hash(housing_with_id, 0.2, "index") ## Calling the function
```

data.reset_index():
This resets the DataFrame’s index and adds the old index as a new column named "index".
It creates a unique identifier for each row, based on its original position.

Splitting the Data:

- The function split_data_with_id_hash is then called with:
  - housing_with_id: The DataFrame that now includes an "index" column.
  - 0.2: The test ratio (20% of the data).
  - "index": The name of the column used as the unique identifier.
- This returns the training set and test set with a consistent split.

#### Important Note:

When using the row index as a unique identifier, ensure that new data is always appended (so indices remain stable) and that no row is ever deleted, otherwise the split may change.

## 5. Creating a Unique ID from Stable Features

```python
housing_with_id["id"] = data["longitude"] \* 1000 + data["latitude"] ## Creating a unique id
train_set, test_set = split_data_with_id_hash(housing_with_id, 0.2, "index") ## Calling the function
```

#### Creating an ID:

- This line creates a new column "id" by combining two stable features: "longitude" and "latitude".
  Multiplying the longitude by 1000 and adding the latitude is one way to generate a unique number for each instance, assuming that the combination of these two is unique for every data point.

#### Using the ID for Splitting:

Ideally, after creating the "id" column, you would use it as the identifier for splitting:
train_set, test_set = split_data_with_id_hash(housing_with_id, 0.2, "id")

This ensures that the assignment to train or test set is based on stable geographic features.

#### Why use stable features:

If the row index isn’t reliable (e.g., because rows might be deleted or reordered when new data comes in), using features like longitude and latitude—which don’t change—will result in a more robust and consistent train/test split.

# In Summary:

### is_id_in_test_set:

Decides whether an instance should be in the test set by comparing a hash of its unique identifier with a threshold determined by the desired test set ratio.

### split_data_with_id_hash:

Uses the above function to split the dataset into training and test sets based on a specified unique identifier column.

### Usage with Row Index:

You can create a unique identifier using the DataFrame’s row index (via reset_index()), but you must ensure the index remains stable over time.

### Alternative Unique ID:

If the row index isn’t stable, you can create a unique identifier using unchanging features like longitude and latitude.
