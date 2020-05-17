---
layout: post
title: "Use Pandas for ETL: Experience and Practical Tips"
categories: [All, Technical]
tags: [python, pandas, etl]
fullview: false
excerpt: This post talks about my experience of building a small scale ETL with Pandas. It also offers some hands-on tips that may help you build ETLs with Pandas.
comments: true
---

## Introduction
This post talks about my experience of building a small scale ETL with Pandas. It also offers some hands-on tips that may help you build ETLs with Pandas.

**Background**: Recently, I was tasked with importing multiple data dumps into our database. The data dumps came from different source, e.g., clients, web. We were lucky that all of our dumps were small, with the largest were under 20 GB. Also, the data sources were updated quarterly, or montly at most, so the ETL doesn't have to be real time, as long as it could re-run. 

## SQL vs. Pandas
For simple transformations, like one-to-one column mappings, caculating extra columns, SQL is good enough.

However, for more complex tasks, e.g., row deduplication, splitting a row into multiple tables, creating new aggregate columns with on custom group-by logic, implementing these in SQL can lead to long queries, which could be hard to read or maintain.

There are [discussions](https://www.reddit.com/r/ETL/comments/cnbl1w/using_python_for_etlelt_transformations/) about building ETLs with SQL vs. Python/Pandas. The major complaints against Pandas are performance:

> Python and Pandas are great for many use cases, but Pandas becomes an issue when the datasets get large because it's grossly inefficient with RAM.

In our case, since the data dumps are not real-time, and small enough to run locally, simplicity is something we want to optimize for. 

Our reasoning goes like this: Since part of our tech stack is built with Python, and we are familiar with the language, using Pandas to write ETLs is just a natural choice besides SQL.

Writing ETL in a high level language like Python means we can use the operative programming styles to manipulate data.

![ETL.png](https://user-images.githubusercontent.com/2715151/71391743-7e914c00-25d3-11ea-9460-309df99edf54.png)

## Practical Tips
### Useful Pandas functions
Most of my ETL code revolve around using the following functions:

- `drop_duplicates`
- `dropna`
- `replace` / `fillna`
-  `df[df['column'] != value]`: filtering
- `apply`: transform, or adding new column
- `merge`: SQL like inner, left, or right join
- `groupby`
- `read_csv` / `to_csv`

Functions like `drop_duplicates` and `drop_na` are nice abstractions and save tens of SQL statements.

And `replace` / `fillna` is a typical step that to manipulate the data array.

One thing that I need to wrap my head around is filtering. Writing

```python
df[df['column'] != value]
```
was a bit awkward at first. This has to do with Python and  the way it overrides operators like `[]`. I haven't peeked into Pandas implementation, but I imagine the class structure and the logic needed to implement the `__getitem__` method.

### Pandas + Jupyter Notebook
Data processing is often exploratory at first. This is especially true for unfamiliar data dumps. We need to see the shape / columns / count / frequencies of the data, and write our next line of code based on our previous output. So the process is iterative.

One tool that Python / Pandas comes in handy is [Jupyter Notebook](https://jupyter.org/). It's like a Python shell, where we write code, execute, and check the output right away. However, it offers a enhanced, modern web UI that makes data exploration more smooth.

Also, for processing data, if we start from a `etl.py` file instead of a notebook, we will need to run the entire `etl.py` many times because of a bug or typo in the code, which could be slow. 

In Jupyter notebook, processing results are kept in memory, so if any section needs fixes, we simply change a line in that seciton, and re-run it again. There is no need to re-run the whole notebook (Note: to be able to do so, we need good conventions, like no reused variable names, see my discussion below about conventions).

My workflow was usually to start with notebook, create a a new section, write a bunch of pandas code, print intermediate results, and keep the output as reference, and move on to write next section. Eventually, when I finish all logic in a notebook, I export the notebook as `.py` file, and delete the notebook.

While writing code in jupyter notebook, I established a few conventions to avoid the mistakes I often made.

- Avoid global variables; no reused variable names across sections.
- Avoid writing logic in root level; Wrap them in functions so that they can reused.
- After seeing the output, write down the findings in code comments before starting the section. Doing so helps clear thinking and not miss some details

### Stablizing IDs
When doing data processing, it's common to generate UUIDs for new rows. For debugging and testing purposes, it's just easier that IDs are deterministic between runs. In other words, running ETL the 2nd time shouldn't change all the new UUIDs.

To support this, we save all generated ids for a temporary file, e.g., `generated/ids.csv`. This file is often the mapping between the old primary key to the newly generated UUIDs. We sort the file based on old primary key column and commit it into git. This way, whenever we re-run the ETL again and see changes to this file, the diffs will us what get changed and help us debug.

A pattern I use often is like below:

```python
existing_ids = pd.read_csv('generated/ids.csv')
data = data.join(existing_ids, how='left', on='old-pk')
data['new-id'].map(lambda i: uuid4() if pd.isnull(i) else i)
data[['old-pk', 'new-id']].sort_values(by=['old-pk']).to_csv('generated/ids.csv')
```

