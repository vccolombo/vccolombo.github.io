---
title: "CTFLearn Inj3ction Time Writeup"
excerpt: "SQL Injection challenge from [CTFLearn](https://ctflearn.com/challenge/149). This challenge consists of a forms that is vulnerable to SQL injection."
date: 2020-08-16T13:46:30-03:00
categories:
  - Cybersecurity
tags:
  - SQL Injection
  - Web
  - CTFLearn
  - CTF
---

This challenge is a [website](https://web.ctflearn.com/web8/) with a single input where we can search for an id. The result seems to be information about dogs.

![Challenge home page](/assets/images/cybersec/ctflearn-Inj3ction-Time-preview.png)

## Solving

### Checking if the input is vulnerable

To check if the ID field is vulnerable to SQLi, the first payload I used was a simple `1'`, which returned no results.

I then tried `1 OR 1=1`:

![ID field is vulnerable to SQLi](/assets/images/cybersec/ctflearn-Inj3ction-Time-checking-vuln.png)

With this result, I started suspecting that the quotes character is blacklisted.

`1 OR 1=1` worked but did not gave me anything interesting, so the flag must be somewhere else in the database.

### Finding how many columns the query needs

Next, I started to look for how many columns the query needs to work. This information will be needed in a UNION attack.

To find it, I crafted a UNION SELECT query with NULL columns until I found the correct number, which is 4 (even though one of them does not appear in the results).

![Finding number of columns in the query](/assets/images/cybersec/ctflearn-Inj3ction-Time-checking-n-of-columns.png)

Apparently, all columns accept strings (as we can see in the results), but to confirm I replaced NULL with @@VERSION in every field, and every one of them worked.

@@VERSION also gave us the information about what SQL server the site is running. The result of the query is `5.5.58-0ubuntu0.14.04.1`. A quick Google search confirms this is a MySQL version.

### Finding the table

My hypothesis was that the flag was in another table in the database, so I started looking for them.

As we know the challenge is running a MySQL server, I queried `table_name` from `information_schema.tables`, which resulted in all available tables:

![Found the tables](/assets/images/cybersec/ctflearn-Inj3ction-Time-tables.png)

Scrolling down there is a table with the very suggestive name `w0w_y0u_f0und_m3`.

### Finding the columns

I first tried `1 UNION SELECT * FROM w0w_y0u_f0und_m3`, but it didn't work, so I went to look for the column names. I crafted the query `1 UNION SELECT NULL, COLUMN_NAME, NULL, NULL FROM INFORMATION_SCHEMA.COLUMNS`, that results in all column names in the database. Remember that the query is not accepting quotes, so I was not able to filter by the table name. Anyway, this was enough and scrolling down I found a column named `f0und_m3`.

### Finding the flag

Now the easy part: just put the column in the query's second field: `1 UNION SELECT NULL, f0und_m3, NULL, NULL FROM w0w_y0u_f0und_m3`:

![Found the flag](/assets/images/cybersec/ctflearn-Inj3ction-Time-flag.png)

And we have the flag.
