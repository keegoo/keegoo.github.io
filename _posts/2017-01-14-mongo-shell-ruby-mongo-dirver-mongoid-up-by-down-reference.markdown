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

## create operation

`mongo shell`

{% highlight ruby %}
db.users.insert({ name: "abc",age: 19 })

db.users.insertMany([
  { name: "bcd", age: 20 },
  { name: "efg", age: 21 }
])
{% endhighlight %}

`ruby mongodb driver`

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

`ruby mongodb driver`

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
