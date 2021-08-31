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