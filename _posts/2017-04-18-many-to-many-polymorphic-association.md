---
layout: single
title: many to many polymorphic association
excerpt: 紀錄 polymorphic 多對多作法。
published: false
---
# 建立 models
```bash
rails g model Book title:string
rails g model Movie title:string
rails g model Tag name:string
rails g model Tagging tag:references, taggable:references{polymorphic}
```
上面的最後一行會建立這樣的 migration 檔, 使用 type `references` ，去自動建立 `tag_id`, `taggable_type`, `taggable_id`, 並且自動設定好 `belongs_to`
```ruby
class CreateTaggings < ActiveRecord::Migration[5.0]
  def change
    create_table :taggings do |t|
      t.references :tag, foreign_key: true
      # t.integer  "tag_id"
      t.references :taggable, polymorphic: true
      #t.string   "taggable_type"
      #t.integer  "taggable_id"
      t.timestamps
    end
  end
end
```
# 設定 association
```ruby
# Book.rb
class Book < ApplicationRecord
  has_many :taggings, as: :taggable
  has_many :tags, through: :taggings
end
```
```ruby
# Movie.rb
class Movie < ApplicationRecord
  has_many :taggings, as: :taggable
  has_many :tags, through: :taggings
end
```
```ruby
# Tagging.rb
class Tagging < ApplicationRecord
  belongs_to :tag
  belongs_to :taggable, polymorphic: true
end
```
```ruby
# Tag.rb
class Tag < ApplicationRecord
  has_many :taggings, dependent: :destroy
  has_many :books, through: :taggings, source_type: 'Book', source: :taggable
  has_many :movies, through: :taggings, source_type: 'Movie', source: :taggable
end
```

# 使用
## create
```ruby
book = Book.create(title: 'book-1')
movie = Movie.create(title: 'Movie-1')
tag_1 = Tag.create(name: 'tag-1')
tag_2 = Tag.create(name: 'tag-1')
```
## add tag
```ruby
book.tags << tag_1
# add multiple tags
book.tags << [tag_1, tag_2]
```
## find
```ruby
# find books from tag
Tag.first.books
# find tags from book
book.tags
```

# validate uniqueness tag
```ruby
class Tagging < ApplicationRecord
  belongs_to :tag
  belongs_to :taggable, polymorphic: true
+ validates_uniqueness_of :tag_id, scope: [:taggable]
end
```
