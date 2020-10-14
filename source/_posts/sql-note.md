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

```python
from datetime import datetime

from sqlalchemy import (MetaData, Table, Column, Integer, Numeric, String, DateTime, ForeignKey, Boolean, create_engine, CheckConstraint)
metadata = MetaData()

cookies = Table('cookies', metadata,
    Column('cookie_id', Integer(), primary_key=True),
    Column('cookie_name', String(50), index=True),
    Column('cookie_recipe_url', String(255)),
    Column('cookie_sku', String(55)),
    Column('quantity', Integer()),
    Column('unit_cost', Numeric(12, 2)),
    CheckConstraint('quantity >= 0', name='quantity_positive')
)

users = Table('users', metadata,
    Column('user_id', Integer(), primary_key=True),
    Column('username', String(15), nullable=False, unique=True),
    Column('email_address', String(255), nullable=False),
    Column('phone', String(20), nullable=False),
    Column('password', String(25), nullable=False),
    Column('created_on', DateTime(), default=datetime.now),
    Column('updated_on', DateTime(), default=datetime.now, onupdate=datetime.now)
)

orders = Table('orders', metadata,
    Column('order_id', Integer()),
    Column('user_id', ForeignKey('users.user_id')),
    Column('shipped', Boolean(), default=False)
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
connection = engine.connect()
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

## Exception and Transaction

Transaction Example:

```python
# Use second schema

from sqlalchemy import select, insert, update
ins = insert(users).values(
    username="cookiemon",
    email_address="mon@cookie.com",
    phone="111-111-1111",
    password="password"
)
result = connection.execute(ins)

ins = cookies.insert()
inventory_list = [
    {
        'cookie_name': 'chocolate chip',
        'cookie_recipe_url': 'http://some.aweso.me/cookie/recipe.html',
        'cookie_sku': 'CC01',
        'quantity': '12',
        'unit_cost': '0.50'
    },
    {
        'cookie_name': 'dark chocolate chip',
        'cookie_recipe_url': 'http://some.aweso.me/cookie/recipe_dark.html',
        'cookie_sku': 'CC02',
        'quantity': '1',
        'unit_cost': '0.75'
    }
]
result = connection.execute(ins, inventory_list)

ins = insert(orders).values(user_id=1, order_id=1)
result = connection.execute(ins)
ins = insert(line_items)
order_items = [
    {
        'order_id': 1,
        'cookie_id': 1,
        'quantity': 9,
        'extended_cost': 4.50
    }
]
result = connection.execute(ins, order_items)


ins = insert(orders).values(user_id=1, order_id=2)
result = connection.execute(ins)
ins = insert(line_items)
order_items = [
    {
        'order_id': 2,
        'cookie_id': 2,
        'quantity': 1,
        'extended_cost': 1.50
    },
    {
        'order_id': 2,
        'cookie_id': 1,
        'quantity': 4,
        'extended_cost': 4.50
    }
]
result = connection.execute(ins, order_items)

def ship_it(order_id):

    s = select([line_items.c.cookie_id, line_items.c.quantity])
    s = s.where(line_items.c.order_id == order_id)
    cookies_to_ship = connection.execute(s)
    for cookie in cookies_to_ship:
        u = update(cookies).where(cookies.c.cookie_id == cookie.cookie_id)
        u = u.values(quantity = cookies.c.quantity - cookie.quantity)
        result = connection.execute(u)
    u = update(orders).where(orders.c.order_id == order_id)
    u = u.values(shipped=True)
    result = connection.execute(u)
    print("Shipped order ID: {}".format(order_id))

ship_it(1)

s = select([cookies.c.cookie_name, cookies.c.quantity])
connection.execute(s).fetchall()

ship_it(2)

u = update(cookies).where(cookies.c.cookie_name == "dark chocolate chip")
u = u.values(quantity = 1)
result = connection.execute(u)

from sqlalchemy.exc import IntegrityError
def ship_it(order_id):
    s = select([line_items.c.cookie_id, line_items.c.quantity])
    s = s.where(line_items.c.order_id == order_id)
    transaction = connection.begin()
    cookies_to_ship = connection.execute(s).fetchall()
    try:
        for cookie in cookies_to_ship:
            u = update(cookies).where(cookies.c.cookie_id == cookie.cookie_id)
            u = u.values(quantity = cookies.c.quantity-cookie.quantity)
            result = connection.execute(u)
        u = update(orders).where(orders.c.order_id == order_id)
        u = u.values(shipped=True)
        result = connection.execute(u)
        print("Shipped order ID: {}".format(order_id))
        transaction.commit()
    except IntegrityError as error:
        transaction.rollback()
        print(error)
ship_it(2)

s = select([cookies.c.cookie_name, cookies.c.quantity])
connection.execute(s).fetchall()
```

## Reflection

### Reflecting Individual Tables

```python
from sqlalchemy import MetaData, Table, create_engine
metadata = MetaData()
engine = create_engine('sqlite:///Chinook_Sqlite.sqlite')


artist = Table('artist', metadata, autoload=True, autoload_with=engine)
album = Table('album', metadata, autoload=True, autoload_with=engine)

artist.columns.keys()
from sqlalchemy import select
s = select([artist]).limit(10)
engine.execute(s).fetchall()

album.foreign_keys
from sqlalchemy import ForeignKeyConstraint
album.append_constraint(
    ForeignKeyConstraint(['ArtistId'], ['artist.ArtistId'])
)
metadata.tables['album']
str(artist.join(album))
```

### Reflecting a Whole Database

```python
metadata.reflect(bind=engine)

metadata.tables.keys()

playlist = metadata.tables['Playlist']
from sqlalchemy import select
s = select([playlist]).limit(10)
engine.execute(s).fetchall()
```

# SQLAlchemy ORM

## Defining Tables via ORM Classes

### sql_orm_setup
```python
from datetime import datetime

from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Table, Column, Integer, Numeric, String, Boolean, DateTime, ForeignKey
from sqlalchemy import create_engine
from sqlalchemy.orm import relationship, backref

Base = declarative_base()


class Cookie(Base):
    __tablename__ = 'cookies'

    cookie_id = Column(Integer(), primary_key=True, )
    cookie_name = Column(String(50), index=True)
    cookie_recipe_url = Column(String(255))
    cookie_sku = Column(String(55))
    quantity = Column(Integer())
    unit_cost = Column(Numeric(12, 2))

    def __repr__(self):
        return "Cookie(cookie_name='{self.cookie_name}', " \
            "cookie_recipe_url='{self.cookie_recipe_url}', " \
            "cookie_sku='{self.cookie_sku}', " \
            "quantity={self.quantity}, " \
            "unit_cost={self.unit_cost})".format(self=self)


class User(Base):
    __tablename__ = 'users'

    user_id = Column(Integer(), primary_key=True)
    username = Column(String(15), nullable=False, unique=True)
    email_address = Column(String(255), nullable=False)
    phone = Column(String(20), nullable=False)
    password = Column(String(25), nullable=False)
    created_on = Column(DateTime(), default=datetime.now)
    updated_on = Column(DateTime(), default=datetime.now,
                        onupdate=datetime.now)

    def __repr__(self):
        return "User(username='{self.username}', " \
            "email_address='{self.email_address}', " \
            "phone='{self.phone}', " \
            "password='{self.password}')".format(self=self)


class Order(Base):
    __tablename__ = 'orders'
    order_id = Column(Integer(), primary_key=True)
    user_id = Column(Integer(), ForeignKey('users.user_id'))
    shipped = Column(Boolean(), default=False)
    user = relationship("User", backref=backref(
        'orders', order_by=order_id))  # one-to-many relationship

    def __repr__(self):
        return "Order(user_id={self.user_id}, " \
            "shipped={self.shipped})".format(self=self)


class LineItem(Base):
    __tablename__ = 'line_items'
    line_items_id = Column(Integer(), primary_key=True)
    order_id = Column(Integer(), ForeignKey('orders.order_id'))
    cookie_id = Column(Integer(), ForeignKey('cookies.cookie_id'))
    quantity = Column(Integer())
    extended_cost = Column(Numeric(12, 2))
    order = relationship("Order", backref=backref(
        'line_items', order_by=line_items_id))
    cookie = relationship("Cookie", uselist=False)  # one-to-one relationship

    def __repr__(self):
        return "LineItems(order_id={self.order_id}, " \
            "cookie_id={self.cookie_id}, " \
            "quantity={self.quantity}, " \
            "extended_cost={self.extended_cost})".format(
                self=self)


engine = create_engine('sqlite:///sql_orm_setup.sqlite', echo=True)
Base.metadata.create_all(engine)
```

### sql_orm_setup_cons
```python
import os
from sqlalchemy.orm import relationship, backref
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Table, Column, Integer, Numeric, String, DateTime, ForeignKey, Boolean, CheckConstraint
from datetime import datetime
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

dbname = os.path.basename(__file__).replace('.py', '.sqlite')
engine = create_engine('sqlite:///{}'.format(dbname), echo=True)

Session = sessionmaker(bind=engine)

session = Session()


Base = declarative_base()


class Cookie(Base):
    __tablename__ = 'cookies'
    __table_args__ = (CheckConstraint(
        'quantity >= 0', name='quantity_positive'),)

    cookie_id = Column(Integer, primary_key=True)
    cookie_name = Column(String(50), index=True)
    cookie_recipe_url = Column(String(255))
    cookie_sku = Column(String(55))
    quantity = Column(Integer())
    unit_cost = Column(Numeric(12, 2))

    def __init__(self, name, recipe_url=None, sku=None, quantity=0, unit_cost=0.00):
        self.cookie_name = name
        self.cookie_recipe_url = recipe_url
        self.cookie_sku = sku
        self.quantity = quantity
        self.unit_cost = unit_cost

    def __repr__(self):
        return "Cookie(cookie_name='{self.cookie_name}', " \
            "cookie_recipe_url='{self.cookie_recipe_url}', " \
            "cookie_sku='{self.cookie_sku}', " \
            "quantity={self.quantity}, " \
            "unit_cost={self.unit_cost})".format(self=self)


class User(Base):
    __tablename__ = 'users'

    user_id = Column(Integer(), primary_key=True)
    username = Column(String(15), nullable=False, unique=True)
    email_address = Column(String(255), nullable=False)
    phone = Column(String(20), nullable=False)
    password = Column(String(25), nullable=False)
    created_on = Column(DateTime(), default=datetime.now)
    updated_on = Column(DateTime(), default=datetime.now,
                        onupdate=datetime.now)

    def __init__(self, username, email_address, phone, password):
        self.username = username
        self.email_address = email_address
        self.phone = phone
        self.password = password

    def __repr__(self):
        return "User(username='{self.username}', " \
            "email_address='{self.email_address}', " \
            "phone='{self.phone}', " \
            "password='{self.password}')".format(self=self)


class Order(Base):
    __tablename__ = 'orders'
    order_id = Column(Integer(), primary_key=True)
    user_id = Column(Integer(), ForeignKey('users.user_id'))
    shipped = Column(Boolean(), default=False)

    user = relationship("User", backref=backref('orders', order_by=order_id))

    def __repr__(self):
        return "Order(user_id={self.user_id}, " \
            "shipped={self.shipped})".format(self=self)


class LineItem(Base):
    __tablename__ = 'line_items'
    line_item_id = Column(Integer(), primary_key=True)
    order_id = Column(Integer(), ForeignKey('orders.order_id'))
    cookie_id = Column(Integer(), ForeignKey('cookies.cookie_id'))
    quantity = Column(Integer())
    extended_cost = Column(Numeric(12, 2))

    order = relationship("Order", backref=backref(
        'line_items', order_by=line_item_id))
    cookie = relationship("Cookie", uselist=False)

    def __repr__(self):
        return "LineItems(order_id={self.order_id}, " \
            "cookie_id={self.cookie_id}, " \
            "quantity={self.quantity}, " \
            "extended_cost={self.extended_cost})".format(
                self=self)


Base.metadata.create_all(engine)


cookiemon = User('cookiemon', 'mon@cookie.com', '111-111-1111', 'password')
cc = Cookie('chocolate chip', 'http://some.aweso.me/cookie/recipe.html', 'CC01', 12, 0.50)
dcc = Cookie('dark chocolate chip',
             'http://some.aweso.me/cookie/recipe_dark.html',
             'CC02',
             1,
             0.75)
session.add(cookiemon)
session.add(cc)
session.add(dcc)


o1 = Order()
o1.user = cookiemon
session.add(o1)

line1 = LineItem(order=o1, cookie=cc, quantity=9, extended_cost=4.50)


session.add(line1)
session.commit()
o2 = Order()
o2.user = cookiemon
session.add(o2)

line1 = LineItem(order=o2, cookie=cc, quantity=2, extended_cost=1.50)
line2 = LineItem(order=o2, cookie=dcc, quantity=9, extended_cost=6.75)


session.add(line1)
session.add(line2)
session.commit()
```

### sql_orm_setup_test
```python
import os
from datetime import datetime

from sqlalchemy import (Column, Integer, Numeric, String, DateTime, ForeignKey,
                        Boolean, create_engine)
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, backref, sessionmaker


conn_string = os.path.basename(__file__).replace('.py', '.sqlite')
Base = declarative_base()


class Cookie(Base):
    __tablename__ = 'cookies'

    cookie_id = Column(Integer, primary_key=True)
    cookie_name = Column(String(50), index=True)
    cookie_recipe_url = Column(String(255))
    cookie_sku = Column(String(55))
    quantity = Column(Integer())
    unit_cost = Column(Numeric(12, 2))

    def __repr__(self):
        return "Cookie(cookie_name='{self.cookie_name}', " \
            "cookie_recipe_url='{self.cookie_recipe_url}', " \
            "cookie_sku='{self.cookie_sku}', " \
            "quantity={self.quantity}, " \
            "unit_cost={self.unit_cost})".format(self=self)


class User(Base):
    __tablename__ = 'users'

    user_id = Column(Integer(), primary_key=True)
    username = Column(String(15), nullable=False, unique=True)
    email_address = Column(String(255), nullable=False)
    phone = Column(String(20), nullable=False)
    password = Column(String(25), nullable=False)
    created_on = Column(DateTime(), default=datetime.now)
    updated_on = Column(DateTime(), default=datetime.now, onupdate=datetime.now)

    def __repr__(self):
        return "User(username='{self.username}', " \
            "email_address='{self.email_address}', " \
            "phone='{self.phone}', " \
            "password='{self.password}')".format(self=self)


class Order(Base):
    __tablename__ = 'orders'
    order_id = Column(Integer(), primary_key=True)
    user_id = Column(Integer(), ForeignKey('users.user_id'))
    shipped = Column(Boolean(), default=False)

    user = relationship("User", backref=backref('orders', order_by=order_id))

    def __repr__(self):
        return "Order(user_id={self.user_id}, " \
            "shipped={self.shipped})".format(self=self)


class LineItem(Base):
    __tablename__ = 'line_items'
    line_item_id = Column(Integer(), primary_key=True)
    order_id = Column(Integer(), ForeignKey('orders.order_id'))
    cookie_id = Column(Integer(), ForeignKey('cookies.cookie_id'))
    quantity = Column(Integer())
    extended_cost = Column(Numeric(12, 2))

    order = relationship("Order", backref=backref('line_items',
                                                  order_by=line_item_id))
    cookie = relationship("Cookie", uselist=False)

    def __repr__(self):
        return "LineItems(order_id={self.order_id}, " \
            "cookie_id={self.cookie_id}, " \
            "quantity={self.quantity}, " \
            "extended_cost={self.extended_cost})".format(
                self=self)


class DataAccessLayer:

    def __init__(self):
        self.engine = None
        self.session = None
        self.conn_string = conn_string

    def connect(self):
        self.engine = create_engine(self.conn_string)
        Base.metadata.create_all(self.engine)
        self.Session = sessionmaker(bind=self.engine)


dal = DataAccessLayer()


def prep_db(session):
    c1 = Cookie(cookie_name='dark chocolate chip',
                cookie_recipe_url='http://some.aweso.me/cookie/dark_cc.html',
                cookie_sku='CC02',
                quantity=1,
                unit_cost=0.75)
    c2 = Cookie(cookie_name='peanut butter',
                cookie_recipe_url='http://some.aweso.me/cookie/peanut.html',
                cookie_sku='PB01',
                quantity=24,
                unit_cost=0.25)
    c3 = Cookie(cookie_name='oatmeal raisin',
                cookie_recipe_url='http://some.okay.me/cookie/raisin.html',
                cookie_sku='EWW01',
                quantity=100,
                unit_cost=1.00)
    session.bulk_save_objects([c1, c2, c3])
    session.commit()

    cookiemon = User(username='cookiemon',
                     email_address='mon@cookie.com',
                     phone='111-111-1111',
                     password='password')
    cakeeater = User(username='cakeeater',
                     email_address='cakeeater@cake.com',
                     phone='222-222-2222',
                     password='password')
    pieperson = User(username='pieperson',
                     email_address='person@pie.com',
                     phone='333-333-3333',
                     password='password')
    session.add(cookiemon)
    session.add(cakeeater)
    session.add(pieperson)
    session.commit()

    o1 = Order()
    o1.user = cookiemon
    session.add(o1)

    line1 = LineItem(cookie=c1, quantity=2, extended_cost=1.00)

    line2 = LineItem(cookie=c3, quantity=12, extended_cost=3.00)

    o1.line_items.append(line1)
    o1.line_items.append(line2)
    session.commit()

    o2 = Order()
    o2.user = cakeeater

    line1 = LineItem(cookie=c1, quantity=24, extended_cost=12.00)
    line2 = LineItem(cookie=c3, quantity=6, extended_cost=6.00)

    o2.line_items.append(line1)
    o2.line_items.append(line2)

    session.add(o2)
    session.commit()
```
## Data Operation

### Session

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)

session = Session()
```

### Insert

Inserting a single object:

```python
cc_cookie = Cookie(cookie_name='chocolate chip',
                   cookie_recipe_url='http://some.aweso.me/cookie/recipe.html',
                   cookie_sku='CC01',
                   quantity=12,
                   unit_cost=0.50)
session.add(cc_cookie)
session.commit()

cc_cookie.cookie_id
```

Multiple inserts:

```python
dcc = Cookie(cookie_name='dark chocolate chip',
             cookie_recipe_url='http://some.aweso.me/cookie/recipe_dark.html',
             cookie_sku='CC02',
             quantity=1,
             unit_cost=0.75)
mol = Cookie(cookie_name='molasses',
             cookie_recipe_url='http://some.aweso.me/cookie/recipe_molasses.html',
             cookie_sku='MOL01',
             quantity=1,
             unit_cost=0.80)
session.add(dcc)
session.add(mol)
session.flush()

print(dcc.cookie_id)
print(mol.cookie_id)
```

Bulk inserting multiple records:

```python
c1 = Cookie(cookie_name='peanut butter',
            cookie_recipe_url='http://some.aweso.me/cookie/peanut.html',
            cookie_sku='PB01',
            quantity=24,
            unit_cost=0.25)
c2 = Cookie(cookie_name='oatmeal raisin',
            cookie_recipe_url='http://some.okay.me/cookie/raisin.html',
            cookie_sku='EWW01',
            quantity=100,
            unit_cost=1.00)
session.bulk_save_objects([c1,c2])
session.commit()
# If you are inserting multiple records and don’t need access to relationships or the inserted primary key, use bulk_save_objects or its related methods.
c1.cookie_id
```

### Query

```python
cookies = session.query(Cookie).all()
print(cookies)

for cookie in session.query(Cookie):
    print(cookie)
```

Controlling the Columns in the Query:

```python
print(session.query(Cookie.cookie_name, Cookie.quantity).first())
```

Ordering:

```python
for cookie in session.query(Cookie).order_by(Cookie.quantity):
    print('{:3} - {}'.format(cookie.quantity, cookie.cookie_name))


from sqlalchemy import desc
for cookie in session.query(Cookie).order_by(desc(Cookie.quantity)):
    print('{:3} - {}'.format(cookie.quantity, cookie.cookie_name))
```

Limit:

```python
query = session.query(Cookie).order_by(Cookie.quantity)[:2] # inefficient
print([result.cookie_name for result in query])

query = session.query(Cookie).order_by(Cookie.quantity).limit(2)
print([result.cookie_name for result in query])
```

Built-In SQL Functions and Labels:

```python
from sqlalchemy import func
inv_count = session.query(func.sum(Cookie.quantity)).scalar()
print(inv_count)

rec_count = session.query(func.count(Cookie.cookie_name)).first()
print(rec_count)

rec_count = session.query(func.count(Cookie.cookie_name) \
                          .label('inventory_count')).first()
print(rec_count.keys())
print(rec_count.inventory_count)
```

Filter:

```python
record = session.query(Cookie).filter(Cookie.cookie_name == 'chocolate chip').first()
print(record)

record = session.query(Cookie).filter_by(cookie_name='chocolate chip').first()
print(record)

query = session.query(Cookie).filter(Cookie.cookie_name.like('%chocolate%'))
for record in query:
    print(record.cookie_name)
```

Operators:

```python
results = session.query(Cookie.cookie_name, 'SKU-' + Cookie.cookie_sku).all()
for row in results:
    print(row)


from sqlalchemy import cast
query = session.query(Cookie.cookie_name,
                      cast((Cookie.quantity * Cookie.unit_cost),
                           Numeric(12,2)).label('inv_cost'))
for result in query:
    print('{} - {}'.format(result.cookie_name, result.inv_cost))
```

Boolean Operators:

```python
from sqlalchemy import and_, or_, not_
query = session.query(Cookie).filter(
    Cookie.quantity > 23,
    Cookie.unit_cost < 0.40
)
for result in query:
    print(result.cookie_name)


from sqlalchemy import and_, or_, not_
query = session.query(Cookie).filter(
    or_(
        Cookie.quantity.between(10, 50),
        Cookie.cookie_name.contains('chip')
    )
)
for result in query:
    print(result.cookie_name)
```

### Update

```python
query = session.query(Cookie)
cc_cookie = query.filter(Cookie.cookie_name == "chocolate chip").first()
cc_cookie.quantity = cc_cookie.quantity + 120
session.commit()
print(cc_cookie.quantity)
```

```python
query = session.query(Cookie)
query = query.filter(Cookie.cookie_name == "chocolate chip")
query.update({Cookie.quantity: Cookie.quantity - 20})

cc_cookie = query.first()
print(cc_cookie.quantity)
```

### Delete

```python
query = session.query(Cookie)
query = query.filter(Cookie.cookie_name == "dark chocolate chip")
dcc_cookie = query.one()
session.delete(dcc_cookie)
session.commit()
dcc_cookie = query.first()
print(dcc_cookie)
```

```python
query = session.query(Cookie)
query = query.filter(Cookie.cookie_name == "molasses")
query.delete()
mol_cookie = query.first()
print(mol_cookie)
```

### Add related objects

setup:

```python
cookiemon = User(username='cookiemon',
                 email_address='mon@cookie.com',
                 phone='111-111-1111',
                 password='password')
cakeeater = User(username='cakeeater',
                 email_address='cakeeater@cake.com',
                 phone='222-222-2222',
                 password='password')
pieperson = User(username='pieperson',
                 email_address='person@pie.com',
                 phone='333-333-3333',
                 password='password')
session.add(cookiemon)
session.add(cakeeater)
session.add(pieperson)
session.commit()
```

```python
o1 = Order()
o1.user = cookiemon
session.add(o1)

cc = session.query(Cookie).filter(Cookie.cookie_name =="chocolate chip").one()
line1 = LineItem(cookie=cc, quantity=2, extended_cost=1.00)

pb = session.query(Cookie).filter(Cookie.cookie_name =="peanut butter").one()
line2 = LineItem(quantity=12, extended_cost=3.00)
line2.cookie = pb
line2.order = o1

o1.line_items.append(line1)
o1.line_items.append(line2)
session.commit()
```

```python
o2 = Order()
o2.user = cakeeater

cc = session.query(Cookie).filter(Cookie.cookie_name == "chocolate chip").one()
line1 = LineItem(cookie=cc, quantity=24, extended_cost=12.00)

oat = session.query(Cookie).filter(Cookie.cookie_name == "oatmeal raisin").one()
line2 = LineItem(cookie=oat, quantity=6, extended_cost=6.00)

o2.line_items.append(line1)
o2.line_items.append(line2)

session.add(o2)
session.commit()
```

### Joins

Using join to select from multiple tables:

```python
query = session.query(Order.order_id, User.username, User.phone, Cookie.cookie_name, LineItem.quantity, LineItem.extended_cost)
query = query.join(User).join(LineItem).join(Cookie)
results = query.filter(User.username == 'cookiemon').all()
print(results)
```

Using outerjoin to select from multiple tables:

```python
query = session.query(User.username, func.count(Order.order_id))
query = query.outerjoin(Order).group_by(User.username)
for row in query:
    print(row)
```

```python
class Employee(Base):
    __tablename__ = 'employees'

    id = Column(Integer(), primary_key=True)
    manager_id = Column(Integer(), ForeignKey('employees.id'))
    name = Column(String(255), nullable=False)

    manager = relationship("Employee", backref=backref('reports'), remote_side=[id])

Base.metadata.create_all(engine)

marsha = Employee(name='Marsha')
fred = Employee(name='Fred')
marsha.reports.append(fred)
session.add(marsha)
session.commit()

for report in marsha.reports:
    print(report.name)
```

### Group

```python
query = session.query(User.username, func.count(Order.order_id))
query = query.outerjoin(Order).group_by(User.username)
for row in query:
    print(row)
```

### Chain

Chaining:

```python
def get_orders_by_customer(cust_name):
    query = session.query(Order.order_id, User.username, User.phone, Cookie.cookie_name, LineItem.quantity,LineItem.extended_cost)
    query = query.join(User).join(LineItem).join(Cookie)
    results = query.filter(User.username == cust_name).all()
    return results

get_orders_by_customer('cakeeater')
```

Conditional chaining:

```python
def get_orders_by_customer(cust_name, shipped=None, details=False):
    query = session.query(Order.order_id, User.username, User.phone)
    query = query.join(User)
    if details:
        query = query.add_columns(Cookie.cookie_name, LineItem.quantity,
                          LineItem.extended_cost)
        query = query.join(LineItem).join(Cookie)
    if shipped is not None:
        query = query.filter(Order.shipped == shipped)
    results = query.filter(User.username == cust_name).all()
    return results

print(get_orders_by_customer('cakeeater'))

print(get_orders_by_customer('cakeeater', details=True))

print(get_orders_by_customer('cakeeater', shipped=True))

print(get_orders_by_customer('cakeeater', shipped=False))

print(get_orders_by_customer('cakeeater', shipped=False, details=True))
```

### Raw Queries

```python
from sqlalchemy import text
query = session.query(User).filter(text("username='cookiemon'"))
print(query.all())
```

## Session and Exceptions

`from sql_orm_setup import *`

### Session States

```python

cc_cookie = Cookie(
    cookie_name='chocolate chip', cookie_recipe_url='http://some.aweso.me/cookie/recipe.html', cookie_sku='CC01', quantity=12, unit_cost=0.50)
```

transient:

```python
from sqlalchemy import inspect
insp = inspect(cc_cookie)
for state in ['transient', 'pending', 'persistent', 'detached']:
    print('{:>10}: {}'.format(state, getattr(insp, state)))
```

pending:

```python
session.add(cc_cookie)
for state in ['transient','pending','persistent','detached']:
    print('{:>10}: {}'.format(state, getattr(insp, state)))
```

persistent:

```python
session.commit()
for state in ['transient','pending','persistent','detached']:
    print('{:>10}: {}'.format(state, getattr(insp, state)))
```

detached:

```python
session.expunge(cc_cookie)
for state in ['transient','pending','persistent','detached']:
    print('{:>10}: {}'.format(state, getattr(insp, state)))
```

```python
session.add(cc_cookie)
cc_cookie.cookie_name = 'Change chocolate chip'

insp.modified
for attr, attr_state in insp.attrs.items():
    if attr_state.history.has_changes():
        print('{}: {}'.format(attr, attr_state.value))
        print('History: {}\n'.format(attr_state.history))
```

### MultipleResultsFound Exception

```python
dcc = Cookie('dark chocolate chip',
             'http://some.aweso.me/cookie/recipe_dark.html',
             'CC02', 1, 0.75)
session.add(dcc)
session.commit()

from sqlalchemy.orm.exc import MultipleResultsFound
try:
    results = session.query(Cookie).one()
except MultipleResultsFound as exc:
    print('We found too many cookies... is that even possible?')

session.query(Cookie).all()
```

### DetachedInstanceError

```python
cookiemon = User('cookiemon', 'mon@cookie.com', '111-111-1111', 'password')
session.add(cookiemon)
o1 = Order()
o1.user = cookiemon
session.add(o1)

cc = session.query(Cookie).filter(Cookie.cookie_name ==
                                  "Change chocolate chip").one()
line1 = LineItem(order=o1, cookie=cc, quantity=2, extended_cost=1.00)

session.add(line1)
session.commit()
```

```python
order = session.query(Order).first()
session.expunge(order)
order.line_items.all()
```

### Transactions

```python
from sql_orm_setup_cons import *
```

Defining the ship_it function:

```python
def ship_it(order_id):
    order = session.query(Order).get(order_id)
    for li in order.line_items:
        li.cookie.quantity = li.cookie.quantity - li.quantity
        session.add(li.cookie)
    order.shipped = True
    session.add(order)
    session.commit()
    print("shipped order ID: {}".format(order_id))

ship_it(1)
print(session.query(Cookie.cookie_name, Cookie.quantity).all())

ship_it(2)
print(session.query(Cookie.cookie_name, Cookie.quantity).all())
session.rollback()
print(session.query(Cookie.cookie_name, Cookie.quantity).all())
```

Transactional ship_it:

```python
from sqlalchemy.exc import IntegrityError
def ship_it(order_id):
    order = session.query(Order).get(order_id)
    for li in order.line_items:
        li.cookie.quantity = li.cookie.quantity - li.quantity
        session.add(li.cookie)
    order.shipped = True
    session.add(order)
    try:
        session.commit()
        print("shipped order ID: {}".format(order_id))
    except IntegrityError as error:
        print('ERROR: {!s}'.format(error.orig))
        session.rollback()

ship_it(2)
```

## Testing with SQLAlchemy ORM

```python
from sql_orm_setup_test import *
```

### Testing with a Test Database

```python
from db import Cookie, LineItem, Order, User,  dal


def get_orders_by_customer(cust_name, shipped=None, details=False):
    query = dal.session.query(Order.order_id, User.username, User.phone)
    query = query.join(User)
    if details:
        query = query.add_columns(Cookie.cookie_name, LineItem.quantity,
                                  LineItem.extended_cost)
        query = query.join(LineItem).join(Cookie)
    if shipped is not None:
        query = query.filter(Order.shipped == shipped)
    results = query.filter(User.username == cust_name).all()
    return results
```

```python
import unittest

from decimal import Decimal

from db import prep_db, dal

from app import get_orders_by_customer


class TestApp(unittest.TestCase):
    cookie_orders = [(1, u'cookiemon', u'111-111-1111')]
    cookie_details = [
        (1, u'cookiemon', u'111-111-1111',
            u'dark chocolate chip', 2, Decimal('1.00')),
        (1, u'cookiemon', u'111-111-1111',
            u'oatmeal raisin', 12, Decimal('3.00'))]

    @classmethod
    def setUpClass(cls):
        dal.conn_string = 'sqlite:///:memory:'
        dal.connect()
        dal.session = dal.Session()
        prep_db(dal.session)
        dal.session.close()

    def setUp(self):
        dal.session = dal.Session()

    def tearDown(self):
        dal.session.rollback()
        dal.session.close()

    def test_orders_by_customer_blank(self):
        results = get_orders_by_customer('')
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_shipped(self):
        results = get_orders_by_customer('', True)
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_notshipped(self):
        results = get_orders_by_customer('', False)
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_details(self):
        results = get_orders_by_customer('', details=True)
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_shipped_details(self):
        results = get_orders_by_customer('', True, True)
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_notshipped_details(self):
        results = get_orders_by_customer('', False, True)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust(self):
        results = get_orders_by_customer('bad name')
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_shipped(self):
        results = get_orders_by_customer('bad name', True)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_notshipped(self):
        results = get_orders_by_customer('bad name', False)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_details(self):
        results = get_orders_by_customer('bad name', details=True)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_shipped_details(self):
        results = get_orders_by_customer('bad name', True, True)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_notshipped_details(self):
        results = get_orders_by_customer('bad name', False, True)
        self.assertEqual(results, [])

    def test_orders_by_customer(self):
        results = get_orders_by_customer('cookiemon')
        self.assertEqual(results, self.cookie_orders)

    def test_orders_by_customer_shipped_only(self):
        results = get_orders_by_customer('cookiemon', True)
        self.assertEqual(results, [])

    def test_orders_by_customer_unshipped_only(self):
        results = get_orders_by_customer('cookiemon', False)
        self.assertEqual(results, self.cookie_orders)

    def test_orders_by_customer_with_details(self):
        results = get_orders_by_customer('cookiemon', details=True)
        self.assertEqual(results, self.cookie_details)

    def test_orders_by_customer_shipped_only_with_details(self):
        results = get_orders_by_customer('cookiemon', True, True)
        self.assertEqual(results, [])

    def test_orders_by_customer_unshipped_only_details(self):
        results = get_orders_by_customer('cookiemon', False, True)
        self.assertEqual(results, self.cookie_details)
```

### Using Mocks

```python
import unittest
from decimal import Decimal

import mock

from app import get_orders_by_customer


class TestApp(unittest.TestCase):
    cookie_orders = [(1, u'cookiemon', u'111-111-1111')]
    cookie_details = [
        (1, u'cookiemon', u'111-111-1111',
            u'dark chocolate chip', 2, Decimal('1.00')),
        (1, u'cookiemon', u'111-111-1111',
            u'oatmeal raisin', 12, Decimal('3.00'))]

    @mock.patch('app.dal.session')
    def test_orders_by_customer_blank(self, mock_dal):
        mock_dal.query.return_value.join.return_value.filter.return_value. \
            all.return_value = []
        results = get_orders_by_customer('')
        self.assertEqual(results, [])

    @mock.patch('app.dal.session')
    def test_orders_by_customer_blank_shipped(self, mock_dal):
        mock_dal.query.return_value.join.return_value.filter.return_value. \
            filter.return_value.all.return_value = []
        results = get_orders_by_customer('', True)
        self.assertEqual(results, [])

    @mock.patch('app.dal.session')
    def test_orders_by_customer(self, mock_dal):
        mock_dal.query.return_value.join.return_value.filter.return_value. \
            all.return_value = self.cookie_orders
        results = get_orders_by_customer('cookiemon')
        self.assertEqual(results, self.cookie_orders)
```

## Reflection with SQLAlchemy ORM and Automap

### Reflecting a Database with Automap

setup:
```python
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine

Base = automap_base()

from sqlalchemy import create_engine

engine = create_engine('sqlite:///Chinook_Sqlite.sqlite')

Base.prepare(engine, reflect=True)
Base.classes.keys()
```

```python
Artist = Base.classes.Artist
Album = Base.classes.Album

from sqlalchemy.orm import Session

session = Session(engine)
for artist in session.query(Artist).limit(10):
    print(artist.ArtistId, artist.Name)

artist = session.query(Artist).first()
for album in artist.album_collection:
    print('{} - {}'.format(artist.Name, album.Title))
```