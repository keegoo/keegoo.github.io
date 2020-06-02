---
layout: post
title:  "setup Ruby local ENV with Docker"
date:   2020-06-02 11:35:00 +0800
categories: notes
---

In Ruby, [rbenv][rbenv-github] was used to manage different Ruby versions. But if you have the needs to develop in multiple languages, it's still going to be a mess even with the help of [pyenv][pyenv-github], [nvm][nvm-github] or similarities. Because they themselves have to be installed first if you would like to hand on these languages in the long run.

With Docker, the situation could be alleviated a lot. E.g. if you would like to learn Python but don't know if you're going to use it primarily in the future, it's not a bad idea to try out everything inside a Docker container.

The Docker solution could be very clean regarding deleting a specific Ruby version and the associated libraries. All you need to do is remove the Docker image. But the downside is the slowness and inconvenience some times.

### Ruby irb in Docker

Firstly check the Ruby version you would like to install from [Docker Hub][ruby-docker-hub].

Get the image for that version, say Ruby 2.7.

```shell
~$ docker pull ruby:2.7
```

Run following command to get the irb.

```shell
~$ docker run -it --rm ruby:2.7 
# irb(main):001:0>
```

### Execute single file

Say you have a script `hello.rb` in the CURRENT directory.

```ruby
# hello.rb
puts 'Hello World!'
```

Then you can execute it with the following command.

```shell
~$ docker run -it --rm -v "$PWD":/usr/src/app -w /usr/src/app ruby:2.7 ruby hello.rb
# Hello World!

# note:
# -v "$PWD":/usr/src/app  => mount current folder to /usr/src/app in Docker container.
# -w /usr/src/app         => make working directory as /usr/src/app
# ruby hello.rb           => execute command
```

If [gems][gem-homepage] were needed as dependency, you can put `gem install` before executing the file.

Take the following as an example. [Nokogiri][nokogiri-github] was needed.

```ruby
# nokogiri_demo.rb
require nokogiri
doc = Nokogiri::XML("<greeting>Hello World!</greeting>")
puts doc.text
```

```shell
~$ docker run -it --rm -v "$PWD":/usr/src/app -w /usr/src/app ruby:2.7 gem install nokogiri && ruby nokogiri_demo.rb
# output:
# Fetching mini_portile2-2.4.0.gem
# Fetching nokogiri-1.10.9.gem
# Successfully installed mini_portile2-2.4.0
# Building native extensions. This could take a while...
# Successfully installed nokogiri-1.10.9
# 2 gems installed
# Hello World!
```

Installing gems on demand is not convenient most of the time, especially for the above example when `nokogiri` has to be compiled during the installation which is quite slow.

### Execute a project

The idea is to mount the entire project directory into Docker container. Then set up the ENV and run the project within the container.

In this case, two consoles are needed. One is to make sure the container is hanging there without exit. The other one is to connect to the container for us to use primarily.

Take [opensaz][opensaz-github] for example.

```shell
~$ git clone https://github.com/keegoo/opensaz
~$ cd opensaz
opensaz $ docker run -it --rm -v "$PWD":/usr/src/opensaz -w /usr/src/opensaz ruby:2.7
# irb(main):001:0>

# open another console!!!
# show container list
~$ docker ps -a
# connect with container ID
~$ docker exec -it 45a2ffb62b06 bash
# then resolve dependency and run tests
root@45a2ffb62b06:/usr/src/opensaz# bundle install
root@45a2ffb62b06:/usr/src/opensaz# ruby test/opensaz_test/*_test.rb
```

Because the current directory is mounted into the container, any changes in the host will be reflected inside the container. Which means you can use the favorite editor for editing as always.

Further on, if you want docker containers to persist data, you could create a docker volume. If multiple languages are needed, you could create a customized docker image.

[rbenv-github]: https://github.com/rbenv/rbenv
[pyenv-github]: https://github.com/pyenv/pyenv
[nvm-github]: https://github.com/nvm-sh/nvm
[gem-homepage]: https://rubygems.org/
[nokogiri-github]: https://github.com/sparklemotion/nokogiri
[ruby-docker-hub]: https://hub.docker.com/_/ruby
[opensaz-github]: https://github.com/keegoo/opensaz
