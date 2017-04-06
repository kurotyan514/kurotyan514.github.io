---
layout: single
title: INTRODUCTION TO PROGRAMMING WITH RUBY 筆記
excerpt: 
---
# variable
在 `times` & `each` 裡無法初始化變數
```ruby
 3.times do
   a = 5
 end
 puts a #=> NameError: undefined local variable or method `a' for main:Object
```
在 `for` 裡面可以初始化變數
```ruby
for i in [1,2,3] do
  a = 5
end
puts a #=> 5
```
> The reason is because the for...do/end code did not create a new inner scope, since for is part of Ruby language and not a method invocation. When we use each, times and other methods, followed by {} or do/end, that's when a new block is created.

# methods
## parameter & Arguments 意思
`parameter` 是指呼叫 methods 時需要帶的變數(名稱)。以下面的 method 來看 `words` 就是 parameter。
```ruby
def say(words)
  puts words + '.'
end
```
`Arguments` 是指帶入呼叫 methods 時需要帶入的`值`。
字串 `hello` 就是 Arguments。
```ruby
say('hello')
```
> You can name parameters whatever you'd like, but like we said earlier, it is always the goal of a good programmer to give things meaningful and explicit names. We name the parameter words because the say method expects some words to be passed in so it knows what to say! Arguments are pieces of information that are sent to a method to be modified or used to return a specific result. We "pass" arguments to a method when we call it. Here, we are using an argument to pass the word, or string of words, that we want to use in the say method. When we pass those words into the method, they're assigned to the variable words and we can use them how we please from within the method. Note that the words variable is scoped at the method level; that is, you cannot reference this variable outside of the say method.
When we call say("hello"), we pass in the string "hello" as the argument in place for the words parameter. Then the code within the method is executed with the words variable evaluated to "hello".

## method () 可省略
在呼叫不需要參數的 method 可以省略 `()` 。
在呼叫需要參數的的 method 也可以省略 `()` 。

```ruby
def say(words)
  puts words + '.'
end
#call method without parameter
say
#call method with parameter
say 'hi'
```

## return
> Ruby methods ALWAYS return the evaluated result of the last line of the expression unless an explicit return comes before it

# Loops & iterators
`next` 跳出這次的迴圈，`break` 則是跳出整個迴圈
```ruby
i = 0
loop do
  i += 2
  if i == 4
    next        # skip rest of the code in this iteration
  end
  puts i
  if i == 10
    break
  end
end
#output
2
6
8
10
```

# array
`delete_at` , delete value by index.
`delete` , delete value by value.
`uniq`, deletes any duplicate values and return new array.
`uniq!`, deletes any duplicate values and mutate original array.
## flatten
The flatten method can be used to take an array that contains nested arrays and create a one-dimensional array.
```ruby
irb: 001 > a = [1, 2, [3, 4, 5], [6, 7]]
=> [1, 2, [3, 4, 5], [6, 7]]
irb: 002 > a.flatten
=> [1, 2, 3, 4, 5, 6, 7]
```
## product

The product method can be used to combine two arrays in an interesting way. It returns an array that is a combination of all elements from all arrays.
```ruby
irb :001 > [1, 2, 3].product([4, 5])
=> [[1, 4], [1, 5], [2, 4], [2, 5], [3, 4], [3, 5]]
```

# hash
## fetch
在 hash 裡 `fetch` 可以用來找 key ，他的另一個好用的地方是可以設預設值，當你找的 key 值不存在的話。
```ruby
h = { a: 1, b: 2 }
h.fetch(:a) #=> 1
h.fetch(:c) #=> KeyError: key not found: :c
h.fetch(:c, 3) #=> 3
puts h #=>  { a: 1, b: 2 }
```
## hash to array
```ruby
{"Bob"=>42, "Steve"=>31, "Joe"=>19}.to_a
#=> [["Bob", 42], ["Steve", 31], ["Joe", 19]]
```
## array to hash
```ruby
[["Bob", 42], ["Steve", 31], ["Joe", 19]].to_h
#=> {"Bob"=>42, "Steve"=>31, "Joe"=>19}
```

# variable & pointer
[variable & pointer](https://launchschool.com/books/ruby/read/more_stuff#variables_as_pointers)

# Exercises
<div class="tabs">
  <ul>
    <li><a href="#tabs-1">Question</a></li>
    <li><a href="#tabs-2">Answer1</a></li>
    <li><a href="#tabs-3">Answer2</a></li>
  </ul>
  <div id="tabs-1">
    Given the following data structures. Write a program that moves the information from the array into the empty hash that applies to the correct person
    {% highlight ruby linenos %}
    contact_data = [["joe@email.com", "123 Main st.","555-123-4567"],
    ["sally@email.com", "404 Not Found Dr.", "123-234-3454"]]

    contacts = {"Joe Smith" => {}, "Sally Johnson" => {}}{% endhighlight %}
  </div>

<div id="tabs-2">
{% highlight ruby linenos %}
contact_data.each do |data|
  name = data.first.split('@').first.capitalize
  contacts.keys.each do |key|
    if /#{name}/.match(key)
      contacts[key].merge!({email: data[0], address: data[1], phone: data[2]})
      break
    end
  end
end
puts contacts
#=> {"Joe Smith"=>{:email=>"joe@email.com", :address=>"123 Main st.", :phone=>"555-123-4567"},
#=> "Sally Johnson"=>{:email=>"sally@email.com", :address=>"404 Not Found Dr.", :phone=>"123-234-3454"}}
{% endhighlight %}
</div>

<div id="tabs-3">
{% highlight ruby linenos %}
contact_data = [["joe@email.com", "123 Main st.", "555-123-4567"],
          ["sally@email.com", "404 Not Found Dr.", "123-234-3454"]]

contacts = {"Joe Smith" => {}, "Sally Johnson" => {}}

fields = [:email, :address, :phone]
contact_data.each do |data|
name = data.first.split('@').first.capitalize
  contacts.keys.each do |key|
    if /#{name}/.match(key)
      fields.each do |field|
        contacts[key][field] = data.shift
      end
      break
    end
  end
end
puts contacts
{% endhighlight %}
</div>

</div>
