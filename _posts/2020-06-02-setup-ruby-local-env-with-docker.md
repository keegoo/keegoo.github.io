---
layout: post
title:  "Setup Ruby local ENV with Docker"
date:   2020-06-02 11:35:00 +0800
categories: notes
---

In Ruby you can use [rbenv][rbenv-github] to manage different versions. But if you have multiple languagues, Python, Javascript, it's still going to be a mess even with the help of [pyenv][pyenv-github] or [nvm][nvm-github].

I have a personal laptop and another one from company. In the company laptop, I usually installed development ENV locally. 

For my personal laptop, as I would like to try differennt languages, I usually setup the ENV in Docker. Then it will be very convient to remove them any time in the future.

### irb in Docker

Firstly check the Ruby version you would like to install from [Docker Hub][ruby-docker-hub].

Get the image for that version, say Ruby 2.7.

```shell
~$ docker pull ruby:2.7
```

Run.

```shell
~$ docker run -it --rm ruby:2.7 
# irb(main):001:0>
```

### execute single file

Say you have following script in current directory.

```ruby
# hello.rb
puts 'Hello World!'
```

```shell
~$ docker run -it --rm -v "$PWD":/usr/src/app -w /usr/src/app ruby:2.7 ruby hello.rb
# Hello World!

# note:
# -v "$PWD":/usr/src/app  => mount current folder to /usr/src/app in Docker container.
# -w /usr/src/app         => make working directory as /usr/src/app
# ruby hello.rb           => execute command
```

We have to install the gems very time if it has dependency on third party libraries.

say

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

Install gems on demand is not convenient most of time, especially for the above example, `nokogiri` need to be compiled during the installation and it's quite slow.

### execute a project

The idea is to mount the entire project directory into Docker container. Then setup the ENV in the container. And run and execute the project in the container as well.

Take [opensaz][opensaz-github] for example.

```shell
~$ git clone https://github.com/keegoo/opensaz
~$ cd opensaz
opensaz $ docker run -it --rm -v "$PWD":/usr/src/opensaz -w /usr/src/opensaz ruby:2.7
# irb(main):001:0>

# open another console to enter the container
# get the container ID
~$ docker ps -a
# CONTAINER ID    IMAGE       COMMAND      CREATED          STATUS           PORTS       NAMES
# 45a2ffb62b06    ruby:2.7    "irb"        35 seconds ago   Up 34 seconds                charming_moore
# connect to container
~$ docker exec -it 45a2ffb62b06 bash
# root@45a2ffb62b06:/usr/src/opensaz#
root@45a2ffb62b06:/usr/src/opensaz# bundle install
root@45a2ffb62b06:/usr/src/opensaz# ruby test/opensaz_test/*_test.rb
```

Further on, if you want docker container to persist data, you can create a docker volumes. And you could create a customized docker image if multiple languages are needed.


[rbenv-github]: https://github.com/rbenv/rbenv
[pyenv-github]: https://github.com/pyenv/pyenv
[nvm-github]: https://github.com/nvm-sh/nvm
[ruby-docker-hub]: https://hub.docker.com/_/ruby
[opensaz-github]: https://github.com/keegoo/opensaz
