---
layout: single
title: ruby-on-rails-5-tutorial
excerpt: Ruby on Rails Tutorial Rails 5 閱讀筆記。
published: false
---

# Chapter 3 - Mostly static pages
在 view 裡利用 `provide` method 傳值給 layout 動態的改變 title。
<div class="tabs">
  <ul>
    <li><a href="#tabs-1">application.erb.html</a></li>
    <li><a href="#tabs-2">home.erb.html</a></li>
    <li><a href="#tabs-3">about.erb.html</a></li>
  </ul>
  <div id="tabs-1">
{% highlight erb linenos %}
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield :title %></title>
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
{% endhighlight %}
  </div>

<div id="tabs-2">
{% highlight erb linenos %}
<%  provide :title, 'Home' %>
{% endhighlight %}
</div>

<div id="tabs-3">
{% highlight ruby linenos %}
<%  provide :title, 'about' %>
{% endhighlight %}
</div>
</div><br>

可以用 'present' 判斷是否有值，沒值給 default。

```erb
<title><%= yield(:title).present? || 'default title' %></title>
```

# Chapter 4 - Rails-flavored Ruby
## 4.1.2 Custom helpers
利用 helper 去處理有 controller flow 的 view render。
<div class="tabs">
  <ul>
    <li><a href="#tabs-4">application_helper.rb</a></li>
    <li><a href="#tabs-5">application.html.erb</a></li>
  </ul>
<div id="tabs-4">
{% highlight ruby linenos %}
# app/helpers/application_helper.rb
module ApplicationHelper

  # Returns the full title on a per-page basis.
  def full_title(page_title = '')
    base_title = "Ruby on Rails Tutorial Sample App"
    if page_title.empty?
      base_title
    else
      page_title + " | " + base_title
    end
  end
end
{% endhighlight %}
</div>

<div id="tabs-5">
{% highlight erb linenos %}
<!DOCTYPE html>
<html>
  <head>
    <title><%= full_title(yield(:title)) %></title>
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag    'application', media: 'all',
                                              'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
{% endhighlight %}
</div>
</div><br>
## 4.2.3 Objects and message passing
change `nil` to boolean
```ruby
!!nil
#=> true
```

change integer to boolean
```ruby
!!0
#=> false
```
## 4.3.1 Arrays and ranges
split string by `" "` space string
```ruby
"foo bar     baz".split
#=> ["foo", "bar", "baz"]
```

`include?` 檢查元素是否存在
```ruby
a = [05, 14, 96]
a.include?(96) #=> true
```

`shuffle` 隨機元素排列
```ruby
2.3.0 :027 > a = [1,2,5,7,9]
 => [1, 2, 5, 7, 9]
2.3.0 :028 > a.shuffle
 => [5, 2, 9, 1, 7]
2.3.0 :029 > a.shuffle
 => [9, 5, 7, 2, 1]
```

`push` 可用 `<<` 取代
`a.push(100)` 等於 `a << 100`

`<<` 也可用在連續push
```ruby
a = []
a << 100 << 200
#=> [100,200]
```

integer ranges to array
```ruby
(0..9).to_a  
#=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

string ranges to array
```ruby
('a'..'d').to_a
#=> ["a", "b", "c", "d"]
```
<br>
用 `%w(a b c)` 取代 `['a', 'b', 'c']`
<br>

產生 a 到 z 的 array, 並且將結果亂數排列，再取出 array 的前 7 個元素，最後拼在一起成一個字串。
```ruby
('a'..'z').to_a.shuffle[0..7].join
```

## 4.3.2 Blocks
利用 `&:` 取代 block 變數
```ruby
%w(a b c).map { |s| s.upcase }
#=> [A, B, C]
%w(a b c).map(&:upcase)
#=> [A, B, C] same result
```

`inspect`
把 output 轉成 string
```ruby
(1..3).to_s
#=> [1, 2, 3]
(1..3).to_s.inspect
#=> "[1, 2, 3]"
```

## 4.3.4 CSS revisited
在 ruby 在呼叫 mehtod 時可以省略 `()`
```ruby
def test(string)
  p string
end
test('aa') #=> 'aa'
test 'bb' #=> 'bb'
```
如果 method 的最後一個 parameter 為 hash 的時候，在傳值的時候可以省略 `{}`
```ruby
def test2(string, options = {})
  p "#{string} - #{options}"
end
test2('aa', { a:1, b:2 }) #=> "a - {:a=>1, :b=>2}"
test2('aa', a:1, b:2 ) #=> "a - {:a=>1, :b=>2}"
```
所以如果像是上面 test2 method 的話，呼叫時可以省略 `()` & `{}`
```ruby
test2 'aa', a:1, b:2 #=> "a - {:a=>1, :b=>2}"
```
