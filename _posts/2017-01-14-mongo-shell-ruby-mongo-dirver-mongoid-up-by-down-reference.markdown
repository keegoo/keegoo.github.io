---
layout: post
title:  "mongo shell, ruby mongo dirver, mongoid up by down reference"
date:   2017-01-14 21:00:00 +0800
categories: jekyll update
---

After looking through docs and online resources of the three drivers, seems like `ruby mongodb driver` and `mongodb shell` are very much the same in syntax. 

As `mongoid` is based on `ruby mongodb driver`, `mongoid` suppors what `ruby mongodb dirver` supports. Plus `mongoid` includes some extra features like data validation, relations etc, as well as some helpful methods like `in`, `find_one_and_update` etc.

## atomic writes

In MongoDB, a write operation is atomic on the level of a single document, even if the operation modifies multiple embedded documents within a single document.

## versions

    # mongo --version
    MongoDB shell version: 3.2.11

    # gem list | grep mongo
    mongo (2.3.1)
    mongoid (6.0.2)

## data

Following data is just an sample. You could imagine there're many many such examples(documents) in 'users' collection.

{% highlight ruby %}
{
  name: "Bob",
  age: 59,
  gender: "male"
  skills: ["C ++", "Java", "Ruby", "Python"],
  published: {
    programming: ["10 days to learn C ++", "20 days to learn Java", "1 day to learn Ruby"],
    sport: ["basketball", "training"]
  }
},
...
{% endhighlight %}

## create operation

`mongo shell`

{% highlight ruby %}
db.users.insert({ name: "abc",age: 19 })

db.users.insertMany([
  { name: "bcd", age: 20 },
  { name: "efg", age: 21 }
])
{% endhighlight %}

`ruby mongo driver`

{% highlight ruby %}
result = users.insert_one({ name: 'Steve', age: 39 })
result.n # returns 1, because one document was inserted

result = users.insert_many([
  { name: 'Steve', age: 39 },
  { name: 'Amy', age: 29 }
])
result.inserted_count # returns 2 because two documents were inserted
{% endhighlight %}

`mongoid`

{% highlight ruby %}
class User
  include Mongoid::Document

  field :name, type: String
  field :age, type: Integer
end

User.create({ name: "Bob", age: 59 })
User.create([
  { name: "Steve", age: 71 },
  { name: "Jimmy", age: 35 }
])

{% endhighlight %}

## Read operation

`mongo shell`

{% highlight ruby %}
# === find one ===
db.users.findOne({age: 21})
# === find many ==
db.users.find({age: 21})
# ===   and    ===
db.users.find({name: "Bob", age: 59})
# ===    or    ===
db.users.find({
  $or: [ {age: "Bob"}, {age: 21} ]
})
# ===    in    ===
db.users.find({
  age: { $in: [ 21, 59 ]}
})
# === and & or ===
db.users.find({
  gender: "male",
  $or: [ {name: "Bob"}, {age: 21} ]
})
# == comparison ==
db.users.find({
  $or: [ {name: "Bob"}, {age: { $gt: 20}} ]
})
{% endhighlight %}

`ruby mongo driver`

{% highlight ruby %}
# === find one ===
users.find({age: 39}).first # {"_id"=>BSON::ObjectId('587b32c48fb23804c57ae03b'), "name"=>"Steve", "age"=>39}
# === find many ==
ary = users.find({age: 39}) #<Mongo::Collection::View:0x007ff97c5ab888>
ary.each{ |user| user }
# {"_id"=>BSON::ObjectId('587b32c48fb23804c57ae03b'), "name"=>"Steve", "age"=>39}
# {"_id"=>BSON::ObjectId('587b331a8fb23804e484435e'), "name"=>"Steve", "age"=>39}
# ===   and    ===
users.find({name: "Steve", age: 39})
# ===    or    ===
users.find(
  "$or" => [ {name: "Steve"}, {age: 21} ]
)
# ===    in    ===
users.find( 
  age: { "$in" => [21, 39]}
)
# === and & or ===
users.find({
  gender: "male",
  "$or" => [ {name: "Steve"}, {age: 21} ]
})
# == comparison ==
users.find({
  "$or": [ {name: "Bob"}, {age: {"$gt" => 20}}]
})
{% endhighlight %}

`mongoid`

{% highlight ruby %}
class User
  include Mongoid::Document

  field :name, type: String
  field :age, type: Integer
  field :gender, type: String
end

# === find one ===
User.where({age: 21}).first
# === find many ==
User.where({age: 21})
# ===   and    ===
User.where({name: "Steve", age: 39})
# ===    or    ===
User.where(
  "$or" => [ {name: "Steve"}, {age: 21} ]
)
# ===    in    ===
User.where(
  age: { "$in" => [21, 39]}
)
# === and & or === 
User.where({
  gender: "male",
  "$or" => [ {name: "Steve"}, {age: 21} ]
})
# == comparison ==
User.where({
  "$or": [ {name: "Bob"}, {age: {"$gt" => 20}}]
})

User.where({
  "$or": [ {name: "Bob"}, {:age.gt => 20}]
})
{% endhighlight %}

## Limit fields

Return specific fields from the query instead of the whole document.

`mongo shell`

{% highlight ruby %}
# ===  single  ===
db.users.find({name: "Bob", age: 59}, {age: 1})
# === multiple ===
db.users.find({name: "Bob", age: 59}, {name:1, gender: 1})
{% endhighlight %}

`ruby mongo driver`

{% highlight ruby %}
# ===  single  ===
users.find({name: "Bob", age: 59}).projection({age: 1})
# or
users.find({name: "Bob", age: 59}, projection: {age: 1})

# === multiple ===
users.find({name: "Bob", age: 59}).projection({name: 1, gender: 1})
# or
users.find({name: "Bob", age: 59}, projection: {name: 1, gender: 1})
{% endhighlight %}

`mongoid`
{% highlight ruby %}
# ===  single  ===
Users.where({name: "Bob", age: 59}).only(:age)
# === multiple ===
Users.where({name: "Bob", age: 59}).only(:name, :gender)
{% endhighlight %}

## distinct 

The values returned by distinct is an array. 

`mongo shell`

{% highlight ruby %}
# == collection ==
db.users.distinct("name")
# ==  on query ===
db.users.distinct("age", {name: "Steve", gender: "male"})
{% endhighlight %}

`ruby mongo driver`

{% highlight ruby %}
# == collection ==
users.find().distinct("name")
# ==  on query ===
users.find({name: "Steve", gender: "male"}).distinct("age")
{% endhighlight %}

`mongoid`

{% highlight ruby %}
# == collection ==
Users.find().distinct("name")
# ==  on query ===
Users.find({name: "Steve", gender: "male"}).distinct("age")
{% endhighlight %}

## things need to be digested later

### weird of mongoid

say we had time series data in mongodb as following.

    {
      "_id" : ObjectId("587c35108fe879199c3a3c07"),
      "server" : "CONGY-7W4",
      "time" : "2017-01-13T07:00:00Z",
      "cpu_percent" : {
        "0" : {
          "0" : 20,
          "6" : 18
        }
      }
    }

you may want to insert another data slice to `cpu_percent`

{% highlight ruby %}
doc = SystemMonitor.find("587c35108fe879199c3a3c07")
doc.set("cpu_percent.0.12" => 22)
{% endhighlight %}

But you would find cpu_percent been overwritten by the new data. 

    {
      ...
      "cpu_percent" : {
        "0" : {
          "12" : 22
        }
      }
    }

What you were expected to see should be.

    {
      ...
      "cpu_percent" : {
        "0" : {
          "0" : 20,
          "6" : 18,
          "12": 22
        }
      }
    }

How to do that? It turns out you need to use `where` instead of `find` to query the doc at first hand. 
{% highlight ruby %}
doc = SystemMonitor.where(server: "CONGY-7W4")
doc.set("cpu_percent.0.12" => 22)
{% endhighlight %}

Why?