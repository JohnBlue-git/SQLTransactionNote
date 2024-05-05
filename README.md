# What is Transaction
A sequence of one or more operations, such as insertions, updates, or deletions, performed on a database as a single unit of work.

This mechanism ensures the operations' and data's consistency.

# Locks in SQL Server

## Shared Locks
Shared locks are used when a transaction wants to read a resource.

Note: transactions can hold shared locks on the same resource (not exclusive).

\
In this example, SQL Server will acquire a shared lock on the row with EmployeeID = 1 during the SELECT statement execution.

BEGIN TRANSACTION\
SELECT * FROM Employees WHERE EmployeeID = 1\
COMMIT TRANSACTION

## Exclusive Locks
Exclusive locks are used when a transaction wants to modify a resource, such as when inserting, updating, or deleting data.

Note: no transaction can hold a shared or exclusive lock on the resource simultaneously (exclusive).

\
In this example, SQL Server will acquire an exclusive lock on the row with EmployeeID = 1 during the UPDATE statement execution.

BEGIN TRANSACTION\
UPDATE Employees SET FirstName = 'John' WHERE EmployeeID = 1\
COMMIT TRANSACTION

## Update Locks
Update locks are used when a transaction intends to modify a 
resource but first needs to read the data.

Note: an update lock combines shared lock and exclusive lock inside, and trackle dealock problem, since shared lock and exclusive lock are mutually exclusive.\
(read:shared lock > release: update lock > modify: exculsive lock)
\
Note: select and update sytax is integratable:\
https://blog.quest.com/how-to-use-update-from-select-in-sql-server/

\
In this example, SQL Server will acquire an update lock on the row with EmployeeID = 1.

BEGIN TRANSACTION\
SELECT * FROM Employees WHERE EmployeeID = 1 FOR UPDATE\
COMMIT TRANSACTION

# Isolation Level
Isolation levels determine the degree to which a transaction is isolated from other concurrently running transactions.

## READ UNCOMMITTED
function:\
allows a transaction to read uncommitted (dirty) data changes made by other transactions.\
allow:\
dirty reads, non-repeatable reads, and phantom reads.

## READ COMMITTED
function:\
ensures that a transaction can only read data changes committed by other transactions.\
prevent:\
dirty reads.\
allow:\
non-repeatable reads, and phantom reads.

## REPEATABLE READ
function:\
ensures that a transaction can only read data changes committed by other transactions and that any data read during the transaction will remain unchanged for the duration of the transaction.\
prevent:\
dirty reads and non-repeatable reads.\
allow:\
phantom reads.

## SERIALIZABLE
function:\
provides the highest level of isolation by ensuring that a transaction can only read data changes committed by other transactions, that any data read during the transaction will remain unchanged for the duration of the transaction.\
prevent:\
dirty reads, non-repeatable reads, and phantom reads.

## SNAPSHOT
function:\
allows a transaction to work with a snapshot of the committed data as it existed at the start of the transaction. This means that any changes made by other transactions after the snapshot was created will not be visible to the current transaction.\
prevent:\
dirty reads, non-repeatable reads, and phantom reads

reference:\
https://codedamn.com/news/sql/diving-into-sql-server-locking-isolation-levels
\
more deep about phantom read:\
repeatable level would not prevent operations to read newer commits like INSERT UPDATE DELETE
and could cause phantom reads (when repeating a read operation on the same records, new records in the results set)
https://notes.andywu.tw/2022/%E6%B7%BA%E8%AB%87mysql%E9%9A%94%E9%9B%A2%E5%B1%A4%E7%B4%9A%E7%82%BA%E5%8F%AF%E9%87%8D%E5%BE%A9%E8%AE%80%E6%99%82%E4%B8%8D%E8%83%BD%E9%81%BF%E5%85%8D%E5%B9%BB%E8%AE%80/

# SQL WITH syntax (to set lock / isolation level)
if you want to tag the setting in code\
https://learn.microsoft.com/en-us/answers/questions/150070/sql-server-when-to-use-rowlock-updlock-etc
\
Nevertheless, sql server will watch over the transaction and attch the default setting
\
, such as seeing "update" then implicitly acquires update lock and sets the isolation level to REPEATABLE READ

# C# code about setting isolation level

```
using (SqlConnection objConn = new SqlConnection(strConnString))
{
   objConn.Open();

   // Begin Transaction
   SqlTransaction objTrans = objConn.BeginTransaction();
   
   SqlCommand objCmd1 = new SqlCommand(sql1, objConn);
   SqlCommand objCmd2 = new SqlCommand(sql2, objConn);
   try
   {
      objCmd1.ExecuteNonQuery();
      objCmd2.ExecuteNonQuery();

      // Commit and End of Transaction
      objTrans.Commit();
   }
   catch (Exception)
   {
      // Fail ... then Rollback
      objTrans.Rollback();
   }
   finally
   {
      objConn.Close();
   }
}
```

In addition, at c# level, we can also set the isolation level on transcation section:
\
https://learn.microsoft.com/zh-tw/dotnet/api/system.transactions.transactionoptions.isolationlevel?view=net-8.0

Moreover, we can replace the above into Dapper style or SqlBulkTools style:
\
https://github.com/DapperLib/Dapper/blob/main/Readme.md
\
https://github.com/olegil/SqlBulkTools/blob/master/README.md



# (Optional) Optimistic vs pessimistic locking

Pessimistic locking:\
lock whenever something is going to change.
pros: make sure transaction's concurrency.
implement: sql as default.

Optimistic locking:
\
letting everyone have a chance at updating the records, which assumes no conflicts. 
pros: no blocking will happend among transactions
implement: 
* Perform no conflicting operations
* Under transaction, verify that no conflicts have occurred and abort otherwise (Using a version number or timestamp that changes ever time the data changes)

https://learning-notes.mistermicheels.com/data/sql/optimistic-pessimistic-locking-sql/
