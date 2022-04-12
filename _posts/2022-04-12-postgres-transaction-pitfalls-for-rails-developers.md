---
layout: single
title: Postgres transaction pitfalls for rails developers
date: 2022-04-12 12:32 +0530
description: 'Common mistakes rails devs make with postgres transactions'
tags: ['rails', 'postgres']

---

Rails abstract away of a lot of database stuff away using Active record which is very convenient. But the convince can bite back if we are not careful enough. 

Here I am going to list some common mistakes rails developers make and how to avoid them.

1. Having network calls on rails after_create/before_create callbacks
    
Active record provides callback methods to do some operations based on the changes in data. it includes before_create, after_create, after_commit etc. Rails wrap before_create and after_create inside the same transaction so that it can roll back when the exception is raised from these callbacks. after_commit is raised after the transaction is committed to the database. So if you have calls like

```ruby
class MyModel <ApplicationRecord
  before_create :fetch_data_from_server

  def fetch_data_from_server
    self.attribute = SlowApi.new(self).call
  end
end
```

The transaction will open for a long and that will cause locks and potentially deadlocks on tables. 

Solutions:

    1. Make network calls before transaction begins, ie before the save method is called

    2. Make the API call in after commit callback

2. Scheduling Background jobs from after_create/before_create callbacks

This is very similar to the previous point, as the backgrounds jobs typically use another data store like Redis and the communication will be over the network. This may not be significant on a small scale if the Redis is in the same infra and network latency is not significant. But any performance degradation on redis infra will cause a sudden spike in long transactions and can potentially cause cascading failures on infra.

Solutions:

    1. Schedule background jobs from after_commit callback
    2. Move job scheduling out of rails callbacks


3. Nested transactions
  When you have nested services you might end up with nested transactions as well.


```ruby
class OrderCreator
  def call
    Order.transaction do
      adjust_currency_conversion
      ...
      InventoryUpdateService.new.call
    end
  end
end

class InventoryUpdateService
  def call
   Inventory.transaction(require_new: true) do
     deduct_inventory_from_primary_store
     run_inventory_sync
   end
  end
end
```
	
This can cause performance bottlenecks due to the SAVEPOINT behaviour, Skipping it here as Gilab wrote a great blog on this [here](https://about.gitlab.com/blog/2021/09/29/why-we-spent-the-last-month-eliminating-postgresql-subtransactions/)

4. Last but the not least is adding a transaction block where it is not necessary, for instance if there won't any data corruption if the operations ran individually, or the impact is very minimal which gets fixed on background worker retry, then avoid using database transactions

