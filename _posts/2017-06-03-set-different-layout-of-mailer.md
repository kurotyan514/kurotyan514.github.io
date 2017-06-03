---
layout: single
title: Set different  layout of mailer
excerpt: 為 Action mailer 設定不同 layout
---
這是為 mailer 設定 layout的方法
```ruby
class MyMailer < Devise::Mailer
  layout 'MyMailer'
end
```

如果要為不同的 action 設定不同的 layout ，可以這樣做
如果不使用layout的話也可以直接 return `false`
```ruby
class MyMailer < Devise::Mailer
  layout set_layout

  def set_layout
    case action_name
    when 'action1', 'action2'
      'layout_1'
    else
      'layout_2'
    end
  end
end
```

如果想在某個 action 中 不使用 layout 的話，這邊有兩種作法

1. setting format.html
```ruby
mail subject: 'subject', to: user.email do |format|
  format.html { render layout: false }
end
```

2. call `@_action_has_layout = false` before `mail`
```ruby
@_action_has_layout = false
mail subject: 'subject', to: user.email
```
