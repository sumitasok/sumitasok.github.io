# Handling ranges of data and queries

When we are implementing a calendar, where we block a user's datetime from a period to a period. We need ability to check if the user is already completly or partially blocked at a specific time. We are evaluating how to structure data in postgres db to handle some of these scenarios.

https://www.postgresql.org/docs/current/rangetypes.html

## How to use range types in golang

https://stackoverflow.com/questions/53203464/how-to-extract-postgres-timestamp-range-with-go


Custom DataTypes in GORM:

https://gorm.io/docs/data_types.html

Creating Compount Index using BTREE_GIST

## Setting Exclusion constraint for time for a user.

A User's calendar should not be double booked. To avoid the same, for each user, their time entries should be mutually exclusive under the scope of user_id. At the same time, we should be able to soft delete an entry by filling deleted_at timestamp and those rows should be ignored from the exclusion CONSTRAINT.



TODO: PR to Gorm if this can be extended as a feature.


## Queries

### Create a table with range datatype columns

#### int4range

```
CREATE TABLE intg (room int, during int4range);
insert into intg values (102, '[100, 200]'), (101, '[140, 230]'), (102, '[190, 230]')
```

| room | during |
| --- | --- |
| 100 | [100,201)|
| 101 | [140,230)|
| 102 | [190,230)|

### @> operator is used to find if the specified element on the rhs is in the lhs.
```
select * from intg where during @> 99;
```

No results

| room | during |
| --- | --- |

```
select * from intg where during @> 100;
```

| room | during |
| --- | --- |
| 100 | [100,201)|

```
select * from intg where during @> 140;
```

| room | during |
| --- | --- |
| 100 | [100,201)|
| 101 | [140,230)|

```
select * from intg where during @> 190;
```

| room | during |
| --- | --- |
| 100 | [100,201)|
| 101 | [140,230)|
| 102 | [190,230)|

```
select * from intg where during @> 230;
```

| room | during |
| --- | --- |
| 101 | [140,230)|
| 102 | [190,230)|

```
select * from intg where during @> 231;
```

| room | during |
| --- | --- |
| 101 | [140,230)|
| 102 | [190,230)|

No results

| room | during |
| --- | --- |


#### Tstzrange

```
CREATE TABLE tstzrgt (room int, dttm tstzrange)
```

```
insert into tstzrgt values (102, '[2010-01-02, 2010-01-03]');
insert into tstzrgt values (103, '[2010-01-02 01:01:02+5:30, 2010-01-03 02:04:07+5:30]');
insert into tstzrgt values (104, '[2010-01-02 01:01:02+5:30, 2010-01-03 02:04:07+7:30]');
```


| room | dttm |
| --- | --- |
| 102 |	"[""2010-01-02 01:01:02+05:30"",""2010-01-03 00:00:00+05:30""]" |
| 103 | "[""2010-01-02 01:01:02+05:30"",""2010-01-03 02:04:07+05:30""]" |
| 104 |	"[""2010-01-02 01:01:02+05:30"",""2010-01-03 00:04:07+05:30""]" |


Notice that "2010-01-03 02:04:07+7:30" got converted to "2010-01-03 00:04:07+05:30"

```
insert into tstzrgt values (102, '[2010-01-02 01:01:02+7:30, 2010-01-03 02:04:07+7:30]');
insert into tstzrgt values (102, '[2010-01-02 01:01:02+7:30, 2010-01-03 02:04:07+5:30]');
```

| room | dttm |
| --- | --- |
| 102 |	"[""2010-01-02 01:01:02+05:30"",""2010-01-03 00:00:00+05:30""]" |
| 103 | "[""2010-01-02 01:01:02+05:30"",""2010-01-03 02:04:07+05:30""]" |
| 104 |	"[""2010-01-02 01:01:02+05:30"",""2010-01-03 00:04:07+05:30""]" |
| 105 |	"[""2010-01-01 23:01:02+05:30"",""2010-01-03 00:04:07+05:30""]" |
| 106 |	"[""2010-01-01 23:01:02+05:30"",""2010-01-03 02:04:07+05:30""]" |

All data is converted to pg instances 's local time.

```
select * from tstzrgt where dttm @> '2010-01-03 02:04:07+5:30'::timestamptz;
```

| room | dttm |
| --- | --- |
| 102 |	"[""2010-01-02 01:01:02+05:30"",""2010-01-03 02:04:07+05:30""]" |
| 102 |	"[""2010-01-01 23:01:02+05:30"",""2010-01-03 02:04:07+05:30""]" |

refer [link](https://www.postgresql.org/docs/9.3/functions-range.html) for Postgres range functions and operations

### @> is also used to check if the range provided in rhs is compltely available in the lhs data.

We are going to use this data for our examples

![img](range-data-assets/Screenshot%202021-08-31%20at%2011.07.07%20AM.png)

The colors represent the date range each room is occupied.

```
CREATE TABLE tstzrgt (room int, dttm tstzrange)
insert into tstzrgt values (100, '[2000-01-02 04:00:00+5:30, 2000-01-02 16:00:00+5:30]');
insert into tstzrgt values (101, '[2000-01-02 12:00:00+5:30, 2000-01-03 08:00:00+5:30]');
insert into tstzrgt values (102, '[2000-01-02 08:00:00+5:30, 2000-01-03 00:00:00+5:30]');
insert into tstzrgt values (103, '[2000-01-03 16:00:00+5:30, 2000-01-04 08:00:00+5:30]');
insert into tstzrgt values (104, '[2000-01-03 04:00:00+5:30, 2000-01-03 20:00:00+5:30]');
```

![img](range-data-assets/Screenshot%202021-08-31%20at%2011.23.11%20AM.png)

---

The range, always has to be progressive.

If you try to insert daterange is wrong order, it will throw an error. The below code is not valid.
```
insert into tstzrgt values (103, '[2000-01-03 16:00:00+5:30, 2000-01-03 08:00:00+5:30]');
```

Notice that the range above is between today evening 4 pm to today morning 8 am, which is a wrong way of defining range.

---

If you want to understand which all rooms are completely occupied at the time range we check

We use @> Operator, for checking `contains range`

```
select * from tstzrgt where dttm @> tstzrange('2000-01-02 12:00:00+5:30', '2000-01-02 20:00:00+5:30');
```

- returns the rows which fully covers the range we are searching with

![img](range-data-assets/Screenshot%202021-08-31%20at%2012.04.52%20PM.png)

Only 101, 102 are fully contained in the range with which we checked using `contains range`

Similarly, if we query from 2nd 12pm to 3rd 4pm.
```
select * from tstzrgt where dttm @> tstzrange('2000-01-02 12:00:00+5:30', '2000-01-03 04:00:00+5:30');
```

Will only return 101.

---

As a User, I want to see which all rooms are unavailable at a specific period of time.

- In this case, we need to consider rooms which are partially occupied and fully occupied at the range as both are not useful, if we need to book for the range.

To find rooms booked partially or fully from 2nd 12pm to 3rd 4am.

```
select * from tstzrgt where dttm && tstzrange('2000-01-02 12:00:00+5:30', '2000-01-03 04:00:00+5:30');
```

We should be able to find that 100, 101, 102 are unavailable.

![img](range-data-assets/Screenshot%202021-08-31%20at%2012.19.27%20PM.png)

We use `overlap` operator - `&&` - for finding which all rows are disqualified.

But,

As a User, I am more interested in finding the roooms that are available at a specific time period.

In which case, we need 103, 104 as result.

We can acheive this by negating the condition using `NOT` operator

```
select * from tstzrgt where NOT dttm && tstzrange('2000-01-02 12:00:00+5:30', '2000-01-03 04:00:00+5:30');
```


However, this never is a way to identify which room is avaialble are a specific time. COnsider if we had another row for room 104 with time blocked between 2nd 12pm and 3rd 4am, which is quite possible, the above query would still return us the above result.

---

As a user, I want to understand which rooms are available at the time range i request.

We will update thetable with more rows to show same room booked for multiple instances at different times.

```
insert into tstzrgt values (101, '[2000-01-03 16:00:00+5:30, 2000-01-04 08:00:00+5:30]');
insert into tstzrgt values (103, '[2000-01-02 04:00:00+5:30, 2000-01-02 16:00:00+5:30]');
```

![img](range-data-assets/Screenshot%202021-08-31%20at%2012.52.42%20PM.png)

After the above change, only room 104 should be available at 2nd 12pm to 3rd 4am

![img](range-data-assets/Screenshot%202021-08-31%20at%2012.41.26%20PM.png)

From an application point of view, there are various ways to approach the query.

1. pass the list of rooms by ID: We pass the list of rooms and check which  of them have an overlaping booking and return the room ids that dont have overlapping booking.
   1. In this case, when a new room is added, which dont have a booking yet, that will be returned.
2. from the list of rooms already had booking entries, return the ones that are free at the desired range.
   1. In this case, if a new room is added, but not yet booked, the booking service willnot be able to account for the room.

We can always filter the data based on additional parameters like roomType.

Let us evaluate both approached using query.

1. passing the list of rooms by ID.

```
select unnest(array[100, 101, 102, 103, 104, 107])
    EXCEPT
    select room from tstzrgt where dttm && tstzrange('2000-01-02 12:00:00+5:30', '2000-01-03 04:00:00+5:30');
```

![img](/blog/range-data-assets/Screenshot%202021-08-31%20at%201.11.06%20PM.png)

Which also returned 107, which was not already in the booking table.

Note that, in a microservices architecture, the booking itself would be a service and the rooms data will be with a different service. Both wouldnt be sharing database. Booking service might be managing bokkings for entities other than room as well in a same table. We will check, how to fetch availaibity of various entites together in a later post. Redundantly storing entity types with the booking service is advice-able

In a simple scenario, we will be having different room types, where room types information willnot be stored in booking db. Even the room ID would be a uuid.

