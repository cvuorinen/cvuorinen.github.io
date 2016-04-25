---
title: Validating data with triggers in MySQL
categories:
  - Programming
tags:
  - MySQL
---

MySQL triggers can be used to create some validation conditions that are a little bit more complex than what can be achieved with basic data types and unique index for example. The reason why data validation is better kept at the database level rather than application level is that in case the same data source is used by multiple applications, or even multiple interfaces within the same application, is that you can rely on the data being consistent and valid regardless the validation logic on the application side, which might not always be consistent across different implementations.

<!--more-->

So why triggers are ideal for this kind of logic? Triggers can be executed before data is inserted or updated into the database, and you have the values that would be inserted to the database at your disposal, as well as the old values of the row in case of an update. The insert or update can also be prevented from actually executing from the trigger with an error message. This makes it an ideal place to consistently enforce some validation logic in my opinion.

There are many great tutorials about MySQL triggers, so I will not write about that. If you are not familiar with triggers, I suggest you first read a [tutorial](http://net.tutsplus.com/tutorials/databases/introduction-to-mysql-triggers/) or [a few](http://www.sitepoint.com/how-to-create-mysql-triggers/), or check out the [MySQL manual](http://dev.mysql.com/doc/refman/5.5/en/triggers.html).

## SQL Error: Cannot add or update a row

So we can do the validation in an IF statement inside the trigger, but how can we cancel the insert/update and throw an error? MySQL 5.5 introduced the handy *[SIGNAL](http://dev.mysql.com/doc/refman/5.5/en/signal.html)* operator that can be used to do just that. It allows to set a specific error condition (or SQLSTATE) and a custom error message. This is exactly what we need in this case, as it will return a native MySQL error and thus also prevent the insert or update clause (as long as we use it in a trigger that is specified to run before insert or update).

But what about MySQL 5.1 (or even 5.0)? If you are still running old versions, the *SIGNAL* statement will trigger a generic MySQL syntax error about the *SIGNAL* statement when creating the trigger so you can't use that. A simple trick is to call a non existing stored procedure to trigger a MySQL error. If you use some generic name for the non existing procedure, like *throw_error()*, you will not be getting a very useful error message so it can be quite confusing especially for people that do not know about this trigger and the constraint it enforces. This can be improved a little bit by using the error message in the actual call, so the error message becomes a little bit more relevant.

## Example

So here is an example of a case where a table has a foreign key to another table and the relationship between them is one-to-many, but the table also has an *active* boolean field (0&#124;1) and there should always be only one active row per relationship.

```sql
DELIMITER $$

CREATE TRIGGER example_before_insert_allow_only_one_active
     BEFORE INSERT ON example_tbl FOR EACH ROW
     BEGIN
          IF NEW.active = 1 AND (SELECT COUNT(id) FROM example_tbl 
               WHERE active=1 AND foreign_key_id=NEW.foreign_key_id) > 0
          THEN
               SIGNAL SQLSTATE '45000'
                    SET MESSAGE_TEXT = 'Cannot add or update row: only one active row allowed per type';
          END IF;
     END;
$$

CREATE TRIGGER example_before_update_allow_only_one_active
     BEFORE UPDATE ON example_tbl  FOR EACH ROW
     BEGIN
          IF NEW.active = 1 AND (SELECT COUNT(id) FROM example_tbl
               WHERE id<>NEW.id AND active=1 AND foreign_key_id=NEW.foreign_key_id) > 0
          THEN
               SIGNAL SQLSTATE '45000'
                    SET MESSAGE_TEXT = 'Cannot add or update row: only one active row allowed per type';
          END IF;
     END;
$$
```

Here is a sample output from an update statement that does not pass the validation logic:

```
mysql> UPDATE example_tbl SET active=1 WHERE id=2;
ERROR 1644 (45000): Cannot add or update row: only one active row allowed per type
```

Here is the same example for older MySQL versions that do not support the *SIGNAL* statement. Here we are using the same error message, but this time we are trying to call it as a stored procedure. Not that pretty, but it works.

```sql
DELIMITER $$

CREATE TRIGGER example_before_insert_allow_only_one_active
     BEFORE INSERT ON example_tbl FOR EACH ROW
     BEGIN
          IF NEW.active = 1 AND (SELECT COUNT(id) FROM example_tbl
               WHERE active=1 AND foreign_key_id=NEW.foreign_key_id) > 0
          THEN
               CALL `'Cannot add or update row: only one active row allowed per type'`;
          END IF;
     END;
$$

CREATE TRIGGER example_before_update_allow_only_one_active
     BEFORE UPDATE ON example_tbl  FOR EACH ROW
     BEGIN
          IF NEW.active = 1 AND (SELECT COUNT(id) FROM example_tbl
               WHERE id<>NEW.id AND active=1 AND foreign_key_id=NEW.foreign_key_id) > 0
          THEN
               CALL `'Cannot add or update row: only one active row allowed per type'`;
          END IF;
     END;
$$
```

And here is the sample output from an update statement:

```
mysql> UPDATE example_tbl SET active=1 WHERE id=2;
ERROR 1305 (42000): PROCEDURE database.'Cannot add or update row: only one active row allowed per type' does not exist
```

## Conclusion

Data validation should be consistent across all applications or interfaces using given data source to enforce data integrity and validity. This can be hard to achieve especially if multiple developers use the same data source in different applications. Keeping the validation logic at the data source itself, in this case in a MySQL database, makes sure that the same validation logic is applied always and across all applications and even command line users accessing the database. Triggers are an ideal place to create such validation when basic data type and index rules are not enough.

Of course this does not mean you can forget about data validation in your application, you should always filter or validate user submitted data and give users meaningful error messages.
