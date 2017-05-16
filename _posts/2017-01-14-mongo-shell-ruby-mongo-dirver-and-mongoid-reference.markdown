---
layout: post
title:  "mongo shell, ruby mongo dirver and mongoid reference"
date:   2017-01-14 21:00:00 +0800
categories: jekyll update
---

There're two MongoDB drivers in Ruby: [Ruby MongoDB Driver](https://github.com/mongodb/mongo-ruby-driver) and [Mongoid](https://github.com/mongodb/mongoid).

`Ruby MongoDB Driver`'s syntax is very similiar with `Mongo Shell`, which is meant to.

`Mongoid`, on the other hand, tried to provide a familiar API to Active Record. Even though `Mongoid` is built on top of `Ruby MongoDB Driver`, they're quite different in syntax. Plus `Mongoid` includes extra features like validation and relations which are quite common in Rails.

I always found myself lost in the syntax of these two drivers. So I wrote this article to help me out when I trapped again.

I'm going to show examples in `Mongo Shell`, `Ruby MongoDB Driver` and `Mongoid` for the same MongoDB operations. 

  - [Versions](#versions)
  - [Test Data](#test-data)
  - [Connection](#connection)
  - [Create Operation](#create-operation)
  - [Exists](#exists)
  - [Query Operation](#query-operation)
  - [Arrays and Hashes](#arrays-and-hashes)
  - [Update Operation](#update-operation)
  - [Field Update Operation](#field-update-operation)
  - [Limit Fields](#limit-fields)
  - [Distinct](#distinct)


## Versions

Here's the versions I used for this article.

{% highlight shell %}
MyBook:~ keegoo$ mongod --version
# db version v3.2.11
# git version: 009580ad490190ba33d1c6253ebd8d91808923e4
# OpenSSL version: OpenSSL 1.0.2k  26 Jan 2017
# allocator: system
# modules: none
# build environment:
#     distarch: x86_64
#     target_arch: x86_64

MyBook:~ keegoo$ mongo --version
# MongoDB shell version: 3.2.11

MyBook:~ keegoo$ gem list | grep mongo
# mongo (2.4.1)
# mongoid (6.1.0)
{% endhighlight %}


## Test Data

Mongo provides a very good test data for you to play with. You could import these data by following the [steps](https://docs.mongodb.com/getting-started/shell/import-data/).

We don't need to import these data. But we need to keep these data in mind. Basically, we should know:
  - each document represent a *restanrant*; 
  - *restaurant address* is a hash; 
  - *restaurants grades* is an array;
  - *restaurants* has borough, cuisine and restaurant_id.

```javascript
[
  {
    "address": {
       "building": "1007",
       "coord": [ -73.856077, 40.848447 ],
       "street": "Morris Park Ave",
       "zipcode": "10462"
    },
    "borough": "Bronx",
    "cuisine": "Bakery",
    "grades": [
       { "date": { "$date": 1393804800000 }, "grade": "A", "score": 2 },
       { "date": { "$date": 1378857600000 }, "grade": "A", "score": 6 },
       { "date": { "$date": 1358985600000 }, "grade": "A", "score": 10 },
       { "date": { "$date": 1322006400000 }, "grade": "A", "score": 9 },
       { "date": { "$date": 1299715200000 }, "grade": "B", "score": 14 }
    ],
    "name": "Morris Park Bake Shop",
    "restaurant_id": "30075445"
  },
  // ...
]
```


## Connection

`mongo shell`

```shell
MyBook:~ keegoo$ mongo 127.0.0.1:27017/test -u <dbuser> -p <dbpassword>
# if your DB instance is running in local and security was not enabled, simply type:
# MyBook:~ keegoo$ mongo
```

`ruby mongo driver`

```ruby
require 'mongo'

# output less codes from console
Mongo::Logger.logger.level = Logger::WARN

host = ["127.0.0.1:27017"]
# remove user and password if don't have
options = {
  database: "test",
  user: 'dbuser',
  password: 'dbpassword'
}
client = Mongo::Client.new(host, options)
```

`mongoid`

Create a `mongoid.yml` file with following content.

```yml
development:
  clients:
    default:
      database: test
      hosts:
        - 127.0.0.1:27017
```
or

```yml
development:
  clients:
    default:
      # remove user and password if don't have
      uri: mongodb://<user>:<password>@127.0.0.1:27017/test
```

Connect.

```ruby
require 'mongoid'

Mongoid.load!("mongoid.yml", :development)
```

## Create Operation

`Mongo Shell`

```javascript
db.restaurants.insert({name: "my restaurants", borough: "Shenzhen", cuisine: "Soup" })

db.restaurants.insertMany([
  {name: "my restaurants 01", borough: "Shenzhen", cuisine: "Soup" }, 
  {name: "my restaurants 02", borough: "Shenzhen", cuisine: "Soup" }
])
```

`Ruby Mongo Driver`

```ruby
client["restaurants"].insert_one({name: "my restaurants", borough: "Shenzhen", cuisine: "Soup"})

client["restaurants"].insert_many([
  {name: "my restaurants 01", borough: "Shenzhen", cuisine: "Soup"},
  {name: "my restaurants 02", borough: "Shenzhen", cuisine: "Soup"}
])
```

`Mongoid`

```ruby
# note: Restanrant<class name> is singular!
class Restaurant
  include Mongoid::Document

  field :name,    type: String
  field :borough, type: String
  field :cuisine, type: String
  field :address, type: Hash
  field :grades,  type: Array
end

Restaurant.create({name: "my restaurants", borough: "Shenzhen", cuisine: "Soup"})
Restaurant.create([
  {name: "my restaurants 01", borough: "Shenzhen", cuisine: "Soup"},
  {name: "my restaurants 02", borough: "Shenzhen", cuisine: "Soup"}
])
```


## Exists

`mongo shell`

```javascript
db.generators.find({name: 'my restaurants 01'}).count()
```

`Ruby Mongo Driver`

```ruby
client["restaurants"].find({name: "my restaurants 01"}).count()
```

`Mongoid`

```ruby
Restaurant.where({name: "my restaurants 01"}).exists?
```


## Query Operation

`mongo shell`

```javascript
// == find by ID ==
db.restaurants.find(ObjectId('58f4baa603bb14a0c4306289'))
// === find one ===
db.restaurants.findOne({name: 'my restaurants'})
// === find many ==
db.restaurants.find({name: 'my restaurants'})
// ===   and    ===
db.restaurants.find({name: 'my restaurants', borough: 'Shenzhen'})
// ===    or    ===
db.restaurants.find({
  $or: [ {name: 'my restaurants'}, { borough: 'Shenzhen'} ]
})
// ===    in    ===
db.restaurants.find({ 
  borough: { $in: ['Shenzhen', 'Bronx'] } 
})
// === and & or ===
db.restaurants.find({ 
  borough: 'Bronx', 
  $or: [{cuisine: 'Bakery'}, {name: 'my restaurants'}] 
})
// == comparison ==
db.restaurants.find({ 
  $or: [
    { borough: 'Bronx' }, 
    { restaurant_id: { $gt: '30075445' } }
  ] 
})
```

`ruby mongo driver`

```ruby
# == find by ID ==
client["restaurants"].find({ _id: BSON::ObjectId("58f4baa603bb14a0c4306289")})
# === find one ===
client["restaurants"].find({name: "my restaurants"}).first
# === find many ==
ary = client["restaurants"].find({name: "my restaurants"})
# ===   and    ===
client["restaurants"].find({name: "my restaurants", borough: "Shenzhen"})
# ===    or    ===
client["restaurants"].find(
  "$or" => [ {name: "my restaurants"}, { borough: "Shenzhen"} ]
)
# ===    in    ===
client["restaurants"].find( 
  borough: { "$in" => ["Shenzhen", "Bronx"] }
)
# === and & or ===
client["restaurants"].find({
  borough: "Bronx",
  "$or" => [ {cuisine: "Bakery"}, {name: "my restaurants"} ]
})
# == comparison ==
client["restaurants"].find({
  "$or" => [
    { borough: "Bronx" },
    { restaurant_id: { "$gt" => "30075445" } }
  ]
})
```

`mongoid`

```ruby
class Restaurant
  include Mongoid::Document

  field :name,    type: String
  field :borough, type: String
  field :cuisine, type: String
  field :address, type: Hash
  field :grades,  type: Array
end
# == find by ID ==
Restaurant.find("58f4baa603bb14a0c4306289")
# === find one ===
Restaurant.where({name: "my restaurants"}).first
# === find many ==
Restaurant.where({name: "my restaurants"})
# ===   and    ===
Restaurant.where({name: "my restaurants", borough: "Shenzhen"})
# ===    or    ===
Restaurant.where(
  "$or" => [ {name: "my restaurants"}, { borough: "Shenzhen"} ]
)
# ===    in    ===
Restaurant.where(
  borough: { "$in" => ["Shenzhen", "Bronx"] }
)
# === and & or === 
Restaurant.where({
  borough: "Bronx",
  "$or" => [ {cuisine: "Bakery"}, {name: "my restaurants"} ]
})
# == comparison ==
Restaurant.where({
  "$or" => [ 
    { borough: "Bronx" },
    { restaurant_id: { "$gt" => "30075445" } }
  ]
})

Restaurant.where({
  "$or" => [ 
    { borough: "Bronx" },
    { :restaurant_id.gt => "30075445" }
  ]
})
```

## Arrays and Hashes

`mongo shell`

```javascript
// ====  hash  ====
db.restaurants.find({ 'address.building': '1007' })
// ====  array ====
db.restaurants.find({ 'address.coord.0': -73.856077 })
// == array size ==
db.restaurants.find({ 'address.coord': {$size: 2} })
// = array & hash =
db.restaurants.find({ grades: {$elemMatch: {grade: 'C', score: 10} } })
```

`ruby mongo driver`

```ruby
# ====  hash  ====
client["restaurants"].find({ "address.building": "1007" })
# ====  array ====
client["restaurants"].find({ "address.coord.0": -73.856077 })
# == array size ==
client["restaurants"].find({ "address.coord" => {"$size" => 2} })
# = array & hash =
client["restaurants"].find({ grades: {"$elemMatch" => {grade: "C", score: 10}} })
```

`mongoid`

```ruby
# ====  hash  ====
Restaurant.where({ "address.building": "1007" })
# ====  array ====
Restaurant.where({ "address.coord.0": -73.856077 })
# == array size ==
Restaurant.where({ "address.coord" => {"$size" => 2} })
# = array & hash =
Restaurant.where({ grades: {"$elemMatch" => {grade: "C", score: 10}} })
```

## Update Operation

`mongo shell`

```javascript
// == update one ==
db.restaurants.update(
  { _id: ObjectId('58f4cad08fb238094bd9f518') }, 
  { $set: {name: 'keegoo'} }
)
// = update multi =
db.restaurants.update(
  { name: 'my restaurants 01' }, 
  { $set: {name: 'my restaurants 02'} }, 
  { multi: true }
)
// = create if not find =
db.restaurants.update(
  { name: 'a wierd name' }, 
  { $set: {name: 'my restaurants 01', borough: 'Shenzhen'} }, 
  { upsert: true }
)
// ====  array ====
db.restaurants.update(
  { _id: ObjectId('58f4baa603bb14a0c4306289') }, 
  { $set: {'address.coord.0': 25.0000} }
)
db.restaurants.update(
  { 
    _id: ObjectId('58f4baa603bb14a0c4304e19'), 
    // 'address.coord': -58 match a position in 'coord' array.
    'address.coord': -58 
  }, 
  // '$' in 'address.coord.$' is this position.
  { $set: {'address.coord.$': 21} } 
)
// ====  hash  ====
db.restaurants.update(
  { _id: ObjectId("58f4baa603bb14a0c4306289") }, 
  { $set: {'address.zipcode': '438620'} }
)
// = array & hash =
db.restaurants.update(
  { 
    _id: ObjectId('58f4baa603bb14a0c4304e19'), 
    // 'grades.date': ISODate('2014-09-06T00:00:00Z') match a position in 'grades' array.
    'grades.date': ISODate('2014-09-06T00:00:00Z')
  }, 
  // '$' in 'grades.$.grade' is this position.
  { $set: {"grades.$.grade": 'B'} }
)
```

`ruby mongo driver`

```ruby
# == update one ==
client["restaurants"].update_one(
  { _id: BSON::ObjectId("58f4cad08fb238094bd9f518") },
  { "$set": {name: "my restaurants 02"} }
)
# = update multi =
client["restaurants"].update_many(
  { name: "my restaurants 02" }, 
  { "$set": {name: "my restaurants 01"} }
)
# = create if not find =
client["restaurants"].update_one(
  { name: "a wierd name" }, 
  { "$set": {name: "my restaurants 01", borough: "Shenzhen"} }, 
  { upsert: true }
)
# ====  array ====
client["restaurants"].update_one(
  { _id: BSON::ObjectId("58f4baa603bb14a0c4306289") },
  { "$set": {"address.coord.0": 25.0000} }
)
client["restaurants"].update_one(
  {
    _id: BSON::ObjectId("58f4baa603bb14a0c4304e19"),
    # "address.coord": -58 match a position in "coord" array.
    "address.coord": -58
  },
  # "$" in "address.coord.$" is this position.
  { "$set": {"address.coord.$": 21} }
)
# ====  hash  ====
client["restaurants"].update_one(
  { _id: BSON::ObjectId("58f4baa603bb14a0c4306289") }, 
  { "$set": {"address.zipcode": "438620"} }
)
# = array & hash =
client["restaurants"].update_one(
  { 
    _id: BSON::ObjectId("58f4baa603bb14a0c4304e19"), 
    # "grades.date": ... match a position in "grades" array.
    "grades.date": DateTime.strptime("2014-09-06", "%Y-%m-%d")
  }, 
  # "$" in "grades.$.grade" is this position.
  { "$set": {"grades.$.grade": "B"} }
)
```

`mongoid`

```ruby
# == update one ==
Restaurant.find("58f4cad08fb238094bd9f518").update(name: "my restaurants 02")
# = update multi =
Restaurant.where({name: "my restaurants 01"}).update_all(name: "my restaurants 02")
# = create if not find =
# ...couldn't find a way...
# ====  array ====
Restaurant.where(_id: "58f4baa603bb14a0c4306289").set({"address.coord.0": 25.000})
Restaurant.where({
  _id: "58f4baa603bb14a0c4304e19",
  # "address.coord": -58 match a position in "coord" array.
  'address.coord': -58
}).set("address.coord.$": 21)
# ====  hash  ====
Restaurant.where(_id: "58f4baa603bb14a0c4306289").set({"address.zipcode": "438620"})
# = array & hash =
Restaurant.where({
  _id: BSON::ObjectId("58f4baa603bb14a0c4304e19"),
  # "grades.date": ... match a position in "grades" array.
  "grades.date": DateTime.strptime("2014-09-06", "%Y-%m-%d")
}).set("grades.$.grade": "B")
```


## Field Update Operation

`mongo shell`

```javascript
// =====  inc =====
db.restaurants.update(
  { _id: ObjectId("58f4baa603bb14a0c4304e19") }, 
  { $inc: {"address.coord.0": 10, "address.coord.1": -20 } }
)

// =====  mul =====
db.restaurants.update(
  { _id: ObjectId("58f4baa603bb14a0c4304e19") }, 
  { $mul: { "address.coord.0": 1.10, "address.coord.1": 2 }}
)

// ==== rename ====

```

`ruby mongo driver`

```ruby

```

`mongoid`

```ruby
# =====  inc =====
Restaurant.find("58f4baa603bb14a0c4304e19").inc({"address.coord.0" => 10, "address.coord.1" => -20})

# =====  mul =====
# couldn't find any ...
```


## Limit Fields

Return specific fields from the query instead of the whole document.

`mongo shell`

```javascript
// ===  single  ===
db.restaurants.find({borough: 'Bronx', cuisine: 'Bakery'}, {name: 1})
// === multiple ===
db.restaurants.find({borough: 'Bronx', cuisine: 'Bakery'}, {name: 1, restaurant_id: 1})
```

`ruby mongo driver`

```ruby
# ===  single  ===
client['restaurants'].find({borough: "Bronx", cuisine: "Bakery"}).projection({name: 1})
# or
client['restaurants'].find({borough: "Bronx", cuisine: "Bakery"}, projection: {name: 1})

# === multiple ===
client['restaurants'].find({borough: "Bronx", cuisine: "Bakery"}).projection({name: 1, restaurant_id: 1})
# or
client['restaurants'].find({borough: "Bronx", cuisine: "Bakery"}, projection: {name: 1, restaurant_id: 1})
```

`mongoid`
```ruby
# ...
# omit class definition
# ===  single  ===
Restaurant.where({borough: "Bronx", cuisine: "Bakery"}).only(:name)
# === multiple ===
Restaurant.where({borough: "Bronx", cuisine: "Bakery"}).only(:name, :restaurant_id)
```

## Distinct 

The values returned by distinct is an array. 

`mongo shell`

```javascript
// == collection ==
db.restaurants.distinct('name')
// ==  on query ===
db.restaurants.distinct('name', {borough: 'Bronx', cuisine: 'Bakery'})
```

`ruby mongo driver`

```ruby
# == collection ==
client["restaurants"].find().distinct("name")
# ==  on query ===
client["restaurants"].find({borough: "Bronx", cuisine: "Bakery"}).distinct("name")
```

`mongoid`

```ruby
# == collection ==
Restaurant.where({}).distinct("name")
# ==  on query ===
Restaurant.where({borough: "Bronx", cuisine: "Bakery"}).distinct("name")
```


## (not finished)things need to be digested later

## sort

`mongo shell`

{% highlight ruby %}
db.schedulers.find().sort({'schedule.time': -1})
{% endhighlight %}

`ruby mongo driver`

`mongoid`

{% highlight ruby %}
Scheduler.order_by("schedule.time" => :desc).limit(30)
{% endhighlight %}


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

## Others

#### atomic writes

In MongoDB, a write operation is atomic on the level of a single document, even if the operation modifies multiple embedded documents within a single document.

#### Date
```javascript
// ====  date  ====
db.restaurants.find({ 'grades.0.date': new Date(1393804800000) })
// new Date(1393804800000) matched { "$date": 1393804800000 }
```