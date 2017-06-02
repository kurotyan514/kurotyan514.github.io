---
layout: single
title: Test file download with capybara
categories:
  - IntegrationTest
tags:
  - Capybara
excerpt: 在 capybara 測試檔案下載
header:
  image: /assets/images/capybara.jpg
---
# 概要
這篇主要是紀錄一下如何在 rails 下透過 capybara + Selenium 跑檔案下載的 integation test。

# 實作
## 1. Add Download_helpers
Download_helpers 這隻有幾個功能
1. 設定檔案下載路徑
2. 讀取檔案
3. 刪除檔案
4. 檔案下載狀態以及檔案下載完成前的sleep設定

```ruby
module DownloadHelpers
  TIMEOUT = 5
  PATH    = Rails.root.join('tmp/downloads')

  extend self

  def downloads
    Dir[PATH.join('*')]
  end

  def download
    downloads.first
  end

  def download_content
    wait_for_download
    File.read(download)
  end

  def wait_for_download
    TIMEOUT.times do
      break if downloaded?
      sleep 1
    end
  end

  def downloaded?
    !downloading? && downloads.any?
  end

  def downloading?
    downloads.grep(/\.crdownload$/).any?
  end

  def clear_downloads
    FileUtils.rm_f(downloads)
  end
end
```
## 2.Set spec_helper
### 2.1 set Selenium
設定WebDriver開出來的broswer的預設檔案下載路徑
```ruby
require 'support/download_helper'
Capybara.register_driver :selenium do |app|
  profile = Selenium::WebDriver::Chrome::Profile.new
  profile['download.default_directory'] = DownloadHelpers::PATH.to_s

  Capybara::Selenium::Driver.new(app, browser: :chrome, profile: profile)
end
```
### 2.2 set DatabaseCleaner
設定DatabaseCleaner在test結束的時候把檔案刪除
```ruby
config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, type: :feature) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
    DownloadHelpers.clear_downloads
  end

  config.append_after(:each) do
    DownloadHelpers.clear_downloads
    DatabaseCleaner.clean
  end
```
## 3. Integration test
下面的 code 是簡單化過的，我實際上的 scenario 是點了 link 後會轉跳到另一個頁面並且開新 tab 然後自動下載檔案。有時候下載或是頁面轉跳的動作過快會導致 test 失敗
如果有出現類似的情況的話可能就適時的在某些點使用 sleep 讓動作跟動作的銜接有些緩衝的時間。
```ruby
given(:file_name) { DownloadHelpers.download.split('/').last }
scenario 'automatically download file after sign in', js: true do
  click_link('Download file')

  DownloadHelpers.wait_for_download

  expect(DownloadHelpers.downloaded?).to be_truthy
  expect(file_name).to eql "#{document.name}.pdf"
end
```
