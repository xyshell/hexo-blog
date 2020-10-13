---
title: sql-note
date: 2020-10-12 23:27:52
tags: sql
toc: true
---

This note follows Essential SQLAlchemy 2nd Edition

Github repo: https://github.com/oreillymedia/essential-sqlalchemy-2e/

# SQLAlchemy Core

## Create Schema

![Relationship-visualization](/photos/sql-note/Relationship-visualization.png)

Full in-memory SQLite code sample

```python
from datetime import datetime
from sqlalchemy import (MetaData, Table, Column, Integer, Numeric, String, DateTime, ForeignKey, create_engine)

metadata = MetaData()
cookies = Table('cookies', metadata,
    Column('cookie_id', Integer(), primary_key=True),
    Column('cookie_name', String(50), index=True),
    Column('cookie_recipe_url', String(255)),
    Column('cookie_sku', String(55)),
    Column('quantity', Integer()),
    Column('unit_cost', Numeric(12, 2))
)

users = Table('users', metadata,
    Column('user_id', Integer(), primary_key=True),
    Column('customer_number', Integer(), autoincrement=True),
    Column('username', String(15), nullable=False, unique=True),
    Column('email_address', String(255), nullable=False),
    Column('phone', String(20), nullable=False),
    Column('password', String(25), nullable=False),
    Column('created_on', DateTime(), default=datetime.now),
    Column('updated_on', DateTime(), default=datetime.now, onupdate=datetime.now)
)

orders = Table('orders', metadata,
    Column('order_id', Integer(), primary_key=True),
    Column('user_id', ForeignKey('users.user_id'))
)

line_items = Table('line_items', metadata,
    Column('line_items_id', Integer(), primary_key=True),
    Column('order_id', ForeignKey('orders.order_id')),
    Column('cookie_id', ForeignKey('cookies.cookie_id')),
    Column('quantity', Integer()),
    Column('extended_cost', Numeric(12, 2))
)

engine = create_engine('sqlite:///:memory:')
metadata.create_all(engine)
```

## Data Operation

### Insert

Way1:
```python
ins = cookies.insert().values(
    cookie_name="chocolate chip",
    cookie_recipe_url="http://some.aweso.me/cookie/recipe.html",
    cookie_sku="CC01",
    quantity="12",
    unit_cost="0.50"
)
# str(ins)
result = connection.execute(ins)
# result.inserted_primary_key
```

Way2:
```python
from sqlalchemy import insert
ins = insert(cookies).values(
    cookie_name="chocolate chip",
    cookie_recipe_url="http://some.aweso.me/cookie/recipe.html",
    cookie_sku="CC01",
    quantity="12",
    unit_cost="0.50"
)
```

Way3:
```python
ins = cookies.insert()
result = connection.execute(
    ins,
    cookie_name='dark chocolate chip',
    cookie_recipe_url='http://some.aweso.me/cookie/recipe_dark.html',
    cookie_sku='CC02',
    quantity='1',
    unit_cost='0.75')
```

Multiple:
```python
inventory_list = [
    {
        'cookie_name': 'peanut butter',
        'cookie_recipe_url': 'http://some.aweso.me/cookie/peanut.html',
        'cookie_sku': 'PB01',
        'quantity': '24',
        'unit_cost': '0.25'
    },
    {
        'cookie_name': 'oatmeal raisin',
        'cookie_recipe_url': 'http://some.okay.me/cookie/raisin.html',
        'cookie_sku': 'EWW01',
        'quantity': '100',
        'unit_cost': '1.00'
    }
]
result = connection.execute(ins, inventory_list)
```

### Select

Way1:
```python
from sqlalchemy.sql import select
s = select([cookies])
# str(s)
rp = connection.execute(s)
results = rp.fetchall()
```

Way2:
```python
s = cookies.select()
rp = connection.execute(s)
for record in rp:
    print(record.cookie_name)
```

Sepcify columns, order by, limit, cast:
```python
from sqlalchemy import desc
from sqlalchemy import cast

s = select([cookies.c.cookie_name, cookies.c.quantity, 'SKU-' + cookies.c.cookie_sku,
cast((cookies.c.quantity * cookies.c.unit_cost), Numeric(12,2)).label('inv_cost')]])
s = s.order_by(desc(cookies.c.quantity), cookies.c.cookie_name)
s = s.limit(2) # Also by, s = cookies.select(limit=1)
rp = connection.execute(s)
# rp.keys()
for cookie in rp:
    print('{} - {}'.format(cookie.quantity, cookie.cookie_name))
```

Aggregate:
```python
from sqlalchemy.sql import func

s = select([func.count(cookies.c.cookie_name).label('inventory_count')])
rp = connection.execute(s)
record = rp.first()
# record.keys()
# record.inventory_count
```

Filter(Where):
```python
s = select([cookies]).where(cookies.c.cookie_name == 'chocolate chip')
rp = connection.execute(s)
record = rp.first()
# record.items()

s = select([cookies]).where(cookies.c.cookie_name.like('%chocolate%')).where(cookies.c.quantity == 12)
# str(s)
rp = connection.execute(s)
for record in rp.fetchall():
    print(record.cookie_name)

from sqlalchemy import and_, or_, not_
s = select([cookies]).where(or_(
    cookies.c.quantity.between(10, 50),
    cookies.c.cookie_name.contains('chip')
))
for row in connection.execute(s):
    print(row.cookie_name)
```

### Update

```python
from sqlalchemy import update
u = update(cookies).where(cookies.c.cookie_name == "chocolate chip")
u = u.values(quantity=(cookies.c.quantity + 120))
result = connection.execute(u)
print(result.rowcount)
```

### Delete

```python
from sqlalchemy import delete
u = delete(cookies).where(cookies.c.cookie_name == "dark chocolate chip")
result = connection.execute(u)
print(result.rowcount)
```

### Join

```python
columns = [orders.c.order_id, users.c.username, users.c.phone, cookies.c.cookie_name, line_items.c.quantity, line_items.c.extended_cost]
cookiemon_orders = select(columns)
cookiemon_orders = cookiemon_orders.select_from(users.join(orders).join(line_items).join(cookies)).where(users.c.username == 'cookiemon')
result = connection.execute(cookiemon_orders).fetchall()
for row in result:
    print(row)
```

outerjoin:
```python
columns = [users.c.username, orders.c.order_id]
all_orders = select(columns)
all_orders = all_orders.select_from(users.outerjoin(orders))
result = connection.execute(all_orders).fetchall()
for row in result:
    print(row)
```

### Alias

```python
manager = employee_table.alias()
stmt = select([employee_table.c.name], and_(employee_table.c.manager_id==manager.c.id, manager.c.name=='Fred'))
print(stmt)
```

### Groupby

```python
columns = [users.c.username, func.count(orders.c.order_id)]
all_orders = select(columns)
all_orders = all_orders.select_from(users.outerjoin(orders)).group_by(users.c.username)
print(str(all_orders))
result = connection.execute(all_orders).fetchall()
for row in result:
    print(row)
```

### Raw Queries

```python
result = connection.execute("select * from orders").fetchall()
```

```python
from sqlalchemy import text
stmt = select([users]).where(text('username="cookiemon"'))
print(connection.execute(stmt).fetchall())
```