# Pesa Pulse: Where Are Our Customers?

Pesa Pulse has a new partner that shares its customer directory through an API. Before marketing plans regional campaigns, this project answers the basics: how many customers there are, where they are, and what the data looks like. It pulls the directory, explores its nested structure, extracts the attributes that matter, answers the question first with plain Python and then with pandas, and produces charts and CSV files for the marketing team.

Source: `https://jsonplaceholder.typicode.com/users` (10 customers).

## Files

| File | What it is |
|---|---|
| `customers.py` | All the code: fetching, exploring, extracting, counting, pandas, charts and CSV export |
| `output/customers.csv` | One row per customer, 12 columns, no index column |
| `output/city_counts.csv` | One row per city (`city`, `customers`), most customers first |
| `output/customer_distribution.png` | Chart: customers per city |
| `output/email_domain_distribution.png` | Chart: customers per email domain ending |

## Running it

```bash
pip install requests pandas matplotlib
python customers.py
```

The same code also runs in a Jupyter notebook: run the function cells first, in order, then the final cell with the `if __name__ == "__main__":` block.

## 1. Which attributes are nested, and how to reach the deepest one

Each customer record is a dictionary with 8 top-level keys. Six hold plain values and two hold nested dictionaries.

| Attribute | Type | Nested? |
|---|---|---|
| `id`, `name`, `username`, `email`, `phone`, `website` | int / str | No |
| `address` | dict | **Yes**, level 1 |
| `address.geo` | dict | **Yes**, level 2 (inside `address`) |
| `company` | dict | **Yes**, level 1 |

What each nested attribute contains:

- **`address`** holds `street`, `suite`, `city`, `zipcode` and the nested `geo`.
- **`address.geo`** holds `lat` and `lng`.
- **`company`** holds `name`, `catchPhrase` and `bs`.

**The deepest values are `lat` and `lng`.** They sit three levels down (record, then `address`, then `geo`, then the value) and are reached by chaining the keys:

```python
customer["address"]["geo"]["lat"]   # "-37.3159"
customer["address"]["geo"]["lng"]   # "81.1496"
```

Both are stored as strings, so they are converted with `float()` during extraction.

`explore(record, indent)` prints this structure for any record, with each level indented two spaces deeper:

```
address: dict
  street: str
  suite: str
  city: str
  zipcode: str
  geo: dict
    lat: str
    lng: str
```

## 2. Extracting the attributes that matter

`extract_attributes(customers)` flattens each record into one dictionary with 12 fields: `id`, `name`, `username`, `email`, `email_domain`, `phone`, `city`, `zipcode`, `latitude`, `longitude`, `company` and `website`.

- `email_domain` is derived from `email` (the part after the `@`, lower-cased).
- `latitude` and `longitude` are converted to floats.
- `company` keeps only the company name.
- A missing nested block, an email without an `@`, or a coordinate that is not a number gives `None` for that field instead of crashing.

## 3. How many cities, compared with the number of customers

| | Count |
|---|---|
| Customers | 10 |
| Unique cities (a Python `set`) | 10 |

**What this tells marketing:** because the number of unique cities equals the number of customers, every customer is in a different city, so no city has enough customers for a city-level campaign and marketing should plan by wider region instead.

pandas agrees with the hand-written counts: `df["city"].value_counts()` matches `city_counts()` exactly, and the DataFrame has shape `(10, 12)` with `latitude` and `longitude` as `float64`.

Note that the city names in this directory are placeholder names, so "wider region" is a conclusion about the shape of the data, not about real Kenyan regions.

## 4. Which chart is more useful, and why

**The email-domain chart is the more useful as a distribution, but the city chart is the one that answers marketing's question.**

- **City chart** (`customer_distribution.png`): with every customer in a different city, it is ten bars of equal length. It is accurate, but nothing stands out, so it gives marketing nothing to act on beyond the finding above.
- **Email-domain chart** (`email_domain_distribution.png`): it has a shape. `.biz` is the largest group (3 of 10 customers) and the other seven endings have one each, so it shows where the customer base concentrates.
- **Limit of the email chart:** an email ending says what kind of address a customer uses, not where they live, so it cannot replace the city chart for planning regional campaigns.

**Recommended next step:** group customers into regions using `latitude` and `longitude`, which the directory already provides. That would give a distribution with a real shape that also answers "where?".

## Method notes

- Plain Python first (loops, `dict.get`, a set comprehension), then pandas, to confirm both give the same answer.
- Network failures print a clear message and return an empty list instead of a traceback
- Charts use one colour for one series, with the value at the end of each bar, and are sized for a slide.
