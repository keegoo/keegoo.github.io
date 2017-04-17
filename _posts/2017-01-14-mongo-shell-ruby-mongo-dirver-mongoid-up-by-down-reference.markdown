---
layout: post
title:  "mongo shell, ruby mongo dirver, mongoid up by down reference"
date:   2017-01-14 21:00:00 +0800
categories: jekyll update
---

There're two MongoDB drivers in Ruby: [Ruby MongoDB Driver](https://github.com/mongodb/mongo-ruby-driver) and [Mongoid](https://github.com/mongodb/mongoid).

`Ruby MongoDB Driver`'s syntax is very similiar with `Mongo Shell`, which is meant to.

`Mongoid`, on the other hand, tried to provide a familiar API to Active Record. Even though `Mongoid` is built on top of `Ruby MongoDB Driver`, they're quite different in syntax. Plus `Mongoid` includes extra features like validation and relations which are quite common in Rails.

I always found myself lost in the syntax of these two drivers. So I wrote this article to help me out when I trapped again.


## Sections

I'm going to write examples in `Mongo Shell`, `Ruby MongoDB Driver` and `Mongoid` for the same MongoDB operations. 

[Versions](#versions)

[Test Data](#test-data)

[Connection](#connection)

[Create Operation](#create-operation)

[Query Operation](#query-operation)

[Others](#others)


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
  - *restaurants grades* is an array.

{% highlight json %}
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
  ...
]
{% endhighlight %}


## Connection

`Mongo Shell`

```shell
MyBook:~ keegoo$ mongo 127.0.0.1:27017/test -u <dbuser> -p <dbpassword>
# if your DB instance is running in local and security was not enabled, simply type:
# MyBook:~ keegoo$ mongo
```

`Ruby Mongo Driver`

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

`Mongoid`

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

Mongoid.load!("mongoid.yml", :production)
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

## Query Operation

`mongo shell`

```javascript
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

## Others

#### atomic writes

In MongoDB, a write operation is atomic on the level of a single document, even if the operation modifies multiple embedded documents within a single document.