---
layout: single
title: Cross Domain Access In Rails
excerpt: 使用 CORS , JSONP, rack-cors等不同的方法來實作 Cross Domain Access。
tags:
  - Rails
  - CORS
  - JSONP
  - Cross Domain Access
---
# 前言
透過 ajax 發送跨網域存取的 Request, 會因為 broswer 的 [Same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy) 的安全協定而被擋了下來，如果需要開放部份資料的 API ，來讓外部網域存取，此時就會需要 cross domain 來取得資料，下面就是紀錄一下在 rails 裡要怎麼設定 Cross-Origin。
# 使用Cross-Origin Resource Sharing (CORS)
## js 實作
如果需要把自己的 cookie 帶給 server 的話，要設定 `withCredentials` 為 `true`
詳細可參考
[Requests with credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Requests_with_credentials)
```js
$.ajax({
          url:         "http://localhost:3000/api/user/info",
          accepts:     "application/json",
          contentType: "application/json",
          method:      "GET",
          xhrFields: {
            withCredentials: true
          }
        })
```

## 設定 header
如果從 broswer 有發出跨網域的 request 的話，broswer 會先送一個 http method 為 `OPTION` 的 request，目的是為了要跟 server 確認伺服器是否接受跨網域存取，
如果 server 允許跨網域存取就會在 header 裡寫入相關的訊息，然後 broswer 接收到後就會再一次發出 request 去跟 server 要資料。

Access-Control-Allow-Origin: 允許跨網域存取的 domain white list。

Access-Control-Allow-Methods: 允許使用哪幾種 http method。

Access-Control-Allow-Headers: 允許夾帶哪幾種 header。

Access-Control-Allow-Credentials: 允許 broswer 帶 cookie 上來和伺服器端交換資料。

因為 broswer會送兩次 request 一次為 `OPTIONS` 另一次為 `GET`, 所以 routes 要設定。
```ruby
# controller
module API
  class UsersController < BaseController
    def info
      headers['Access-Control-Allow-Headers'] =
        'Origin, X-Requested-With, Content-Type, Accept'
      headers['Access-Control-Allow-Methods'] = 'POST, GET, OPTIONS'
      headers['Access-Control-Allow-Origin'] =  'http://localhost:3001'
      headers['Access-Control-Allow-Credentials'] = 'true'

      render status: 200, json: current_user
    end
  end
end
# routes
match '/api/user/info', to: 'users#info', via: [:options, :get]
```

# 使用 JSONP（JSON with Padding）來實作
[JSONP wiki](https://zh.wikipedia.org/wiki/JSONP)
## js 實作
### 方法ㄧ
```js
$.ajax({
  url:      "http://localhost:3000/api/v1/user/info",
  dataType: "jsonp"
})
.done((data) => {
  console.log(data);
});
```
### 方法二
這邊需要注意的是如果是用 `$.getJSON` 的話，在 URL 後面一定要加 `?callback=?`，不然 broswer 還是會跟你說沒有設定Access-Control-Allow-Origin
```js
$.getJSON("http://localhost:3000/api/v1/user/info?callback=?", function(data){
  console.log(data)
});
```
## rails server 端實作
這邊要注意的點是 return 的格式要是 json，然後要記得多 return 一個 `callback` 的值回去，不然會吐下面的error
> Refused to execute script from 'http://localhost:3000/api/user/info?callback=jQuery32103560386678369363_1497340429353&_=1497340429354' because its MIME type ('application/json') is not executable, and strict MIME type checking is enabled.

另外因為 jsonp 只能用 GET method, 所以 routes 也設定 get 就可以了。
```ruby
module API
  class UsersController < BaseController
    def info
      render status: 200, json: current_user, callback: params[:callback]
    end
  end
end
# 透過jsonp送到server的params會是長這個樣子
{"callback"=>"jQuery32102229570348418879_1497337933526",
 "_"=>"1497337933527",
 "format"=>:json,
 "controller"=>"api/users",
 "action"=>"info"}
```


# 使用 rack-cors gem 實作
跟前面兩種 `CORS` 或 `JOSNP` 的方式相比，用[Rack CORS Middleware](https://github.com/cyu/rack-cors)方式是最簡單的（因為大大都幫忙寫好了，主要是透過 Middleware 的方式去做設定。
## Indtall
```ruby
gem 'rack-cors', :require => 'rack/cors'
```
## setting
需要注意的是 rails3,4 跟 rails 5 的設定方法是不同的，如果有需要 `withCredentials` 的話可以再加上 `credentials: true` 的參數。

origins: 跟上面一樣是指定允許跨網域存取的 Domain。

resource: 可以指定特定的URL。

還有其他的設定可以到 ack-cors 的 readme 看。

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application

    # ...

    # Rails 3/4

    config.middleware.insert_before 0, "Rack::Cors" do
      allow do
        origins '*'
        resource '*', :headers => :any, :methods => [:get, :post, :options]
      end
    end

    # Rails 5

    config.middleware.insert_before 0, Rack::Cors do
      allow do
        origins 'localhost:3000'
        resource '*', :headers => :any, :methods => [:get, :post, :options]
      end
    end

  end
end
```
# Referance
[HTTP access control (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Requests_with_credentials)

[Changing getJSON to JSONP](https://stackoverflow.com/a/11916804)

[Supporting Cross-Domain AJAX in Rails using JSONP and CORS](http://blog.carbonfive.com/2012/02/27/supporting-cross-domain-ajax-in-rails-using-jsonp-and-cors/)

[jQuery’s JSONP Explained with Examples](https://www.sitepoint.com/jsonp-examples/)
