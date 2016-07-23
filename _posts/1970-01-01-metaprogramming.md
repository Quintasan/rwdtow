---
layout: post
title: Magical world of Metaprogramming
order: 80
---

Metaprogramming is a special feature of some language (Ruby is one of them) to make dynamic code definition in runtime. When code generates code. This is responsible for a lot of Rails "magic", for example: `some_route_path` helpers, ActiveRecord `find_by_%attr_name%`. 

At first look it seems like an awesome feature, until it is misused (unfortunately most of the time it is).
The downsides of metaprogramming are:

* Difficult to locate method source code.
* Hidden intention in the codebase.
* IDEs can't locate these methods for auto-complete.

To recreate the famouse saying, "If you have one problem and think that metaprogramming could help you. Congratulations! Nw you have two problems". Many times the coding challenges that you solve with metaprogramming could be be solved in a simpler way that could result better code quality, separation of concerns, and clearness.

Example of unnecessary metaprogramming in [`rest-client`](https://github.com/rest-client/rest-client/blob/master/bin/restclient) gem.

```ruby
POSSIBLE_VERBS = ['get', 'put', 'post', 'delete']

POSSIBLE_VERBS.each do |m|
  define_method(m.to_sym) do |path, *args, &b|
    r[path].public_send(m.to_sym, *args, &b)
  end
end
```

* [Bala Paranj: 7 Deadly Sins of Ruby Metaprogramming](https://www.codeschool.com/blog/2015/04/24/7-deadly-sins-of-ruby-metaprogramming/)
