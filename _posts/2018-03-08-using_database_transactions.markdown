---
layout: post
title:      "Using Database Transactions"
date:       2018-03-08 21:02:58 -0500
permalink:  using_database_transactions
---


When we use SQL, we are using database transactions. By default, every SQL statement (select, insert, update, etc.) executes within its own transaction. A transaction incurs a lot of overhead. When we need to process lots of SQL statements at a time, our application's performance can suffer greatly.
### Background
A database transaction represents a unit of work in a database. A transaction exhibits the [ACID](https://en.wikipedia.org/wiki/ACID) properties. For our discussion here, we'll focus on the atomic or 'all or nothing' property of a transaction. That is, all the work in a transaction must succeed or none will succeed, leaving the database in a consistent state. This propery is important, for example, when transferring money from a savings account to a checking account. 
```
UPDATE savings  SET balance = balance - 100.0 WHERE account_id = 100
UPDATE checking SET balance = balance + 100.0 WHERE account_id = 100
```
We wouldn't want the withdrawal from savings to succeed, but the deposit into checking to fail. If both of these operations were in the same transaction, the the savings withdrawal would fail if the checking deposit fails. This would leave the money to be transferred in the savings account, rather than be lost.

A transaction incurs substantial overhead - transaction setup, recording the intermediate results of each SQL statement isolated from the rest of the database, applying the those changes to the main database, and cleaning up transaction resources. Typically, all this work is done to persistent storage (e.g., disk drives), further slowing the process. Unless specified otherwise, every SQL statement executes in its own transaction.
### *The Problem*
For one of my projects, I needed to update equity prices in an 9,000 row SQLite database table. This was done in a background process using SideKiq. For each equity price received from a third-party market data provider, I would perform an update statement to a row in that table if the price had changed from what was already stored.

Updating 9,000 rows, each in their own transaction took 30 minutes and put a substantial strain on the database. As well as starving other users of the database from speedy access.
### *The Solution*
By using the Begin and End transaction statements, I could cause all my update statements to execute within a single transaction. This cuts down substantially on overhead. The time to update 9,000 records fell from 30 minutes to 30 seconds. Or, from 5 updates per second to 300!

In [ActiveRecord](http://api.rubyonrails.org/v5.0/classes/ActiveRecord/Transactions/ClassMethods.html), the process looks something like this -

```
Trade.transaction do
  trades.each do |trade|
    *some processing here*
   trade.save
  end
end
```

Using this method, all the updates will fail if only one update fails.  For this project, that wasn't a problem. The updates were fairly simple and should not fail unless there were some major database problem or if the app crashed. In that case, restarting would not be a problem. In the checking account transfer example given earlier, if we were to batch multiple account's transfers into a single transaction, we would need to provide a way to restart the updates while excluding the failing updates. Unless this was needed to yield acceptable performance, for code simplicity's sake, it would be best to have just the statements for a single account in a single transaction.
### *Conclusion*
Database software packages (MySQL, Postgres, SQLite, etc.) tend to have their own way of dealing with large or long running transactions in terms of locking, rolling back, mulituser concurrency, etc. So, it pays to read up on the particulars of the package you will be using.

