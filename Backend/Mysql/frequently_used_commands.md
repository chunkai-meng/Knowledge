`mysqladmin -u root password 'yourpassword'`

```SQL
UPDATE `database_name`.`table_name` SET `column_name`='value' WHERE `id`='1';
# e.g.
UPDATE Hats SET Price=12.9 WHERE `id`='3';

# Delete all rows in a table
truncate table hats;

# SHOW COLUMNS
SHOW COLUMNS FROM mytable

# Alter column name
ALTER TABLE `Users` CHANGE COLUMN `UserName` `Name` VARCHAR(255) NOT NULL;

# Add a column
ALTER TABLE `orders` ADD `Status` ENUM('Submitted','Shipped','Delivered') NULL AFTER `ID`;
```
