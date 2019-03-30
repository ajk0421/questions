# Rspecのテストコードの質問

## 状況

 `artist.rb`に記載されている バリデーション `presense: true` が機能しているか確認したのですが、予想通りテストを通過してくれない。

artist.rb

```ruby
class Artist < ApplicationRecord
  before_save :set_data
  mount_uploader :image, ImageUploader

  attr_accessor :image_cache

  validates :name, presence: true
  validates :furi_gana, presence: true
  validate :katanaka_valid
  validates :image, presence: true
  validates :place, presence: true
  validates :profile, presence: true

  belongs_to :user
  
  省略
```





### 1.テストコードを走らせた

走らせたテストコード

```ruby
require 'rails_helper'

RSpec.describe Artist, type: :model do
  # pending "add some examples to (or delete) #{__FILE__}"


  context "name を指定してるとき" do
    it "アーティストが作成される" do
      artist = Artist.new(name: "テスト", furi_gana: "テスト", image: Rails.root.join("public/test.jpg").open, place: "札幌", profile: "テスト", user_id: 1)
      expect(artist.valid?).to eq true

    end
  end
end

```

結果

```
ajikikoutanoAir% bundle exec rspec          

Artist
  name を指定してるとき
    アーティストが作成される (FAILED - 1)

Failures:

  1) Artist name を指定してるとき アーティストが作成される
     Failure/Error: expect(artist.valid?).to eq true
     
       expected: true
            got: false
     
       (compared using ==)
     # ./spec/models/artist_spec.rb:10:in `block (3 levels) in <main>'

Finished in 0.33537 seconds (files took 1.47 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/models/artist_spec.rb:8 # Artist name を指定してるとき アーティストが作成される
```



### 2.binding.pryでエラー内容を確認

走らせたテストコード

```　ruby
require 'rails_helper'

RSpec.describe Artist, type: :model do
  # pending "add some examples to (or delete) #{__FILE__}"


  context "name を指定してるとき" do
    it "アーティストが作成される" do
      artist = Artist.new(name: "テスト", furi_gana: "テスト", image: Rails.root.join("public/test.jpg").open, place: "札幌", profile: "テスト", user_id: 1)
      artist.valid?
      binding.pry
    end
  end
end
```

結果

```
ajikikoutanoAir% bundle exec rspec          

Artist
  name を指定してるとき
    アーティストが作成される (FAILED - 1)

Failures:

  1) Artist name を指定してるとき アーティストが作成される
     Failure/Error: expect(artist.valid?).to eq true
     
       expected: true
            got: false
     
       (compared using ==)
     # ./spec/models/artist_spec.rb:10:in `block (3 levels) in <main>'

Finished in 0.33537 seconds (files took 1.47 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/models/artist_spec.rb:8 # Artist name を指定してるとき アーティストが作成される

ajikikoutanoAir% bundle exec rspec

Artist
  name を指定してるとき

From: /Users/ajk/Desktop/Portfolio_1/spec/models/artist_spec.rb @ line 13 :

     8:     it "アーティストが作成される" do
     9:       artist = Artist.new(name: "テスト", furi_gana: "テスト", image: Rails.root.join("public/test.jpg").open, place: "札幌", profile: "テスト", user_id: 1)
    10:       artist.valid?
    11:       binding.pry
    12: 
 => 13:     end
    14:   end
    15: end

```

からの
```
[1] pry(#<RSpec::ExampleGroups::Artist::Name>)> artist.errors

=> #<ActiveModel::Errors:0x00007f9c3d3d3d68

 @base=

  \#<Artist:0x00007f9c3d5d6e30

   id: nil,

   name: "テスト",

   furi_gana: "テスト",

   image: nil,

   place: "札幌",

   profile: "テスト",

   hp: nil,

   twitter: nil,

   youtube: nil,

   created_at: nil,

   updated_at: nil,

   user_id: 1>,

 @details={:user=>[{:error=>:blank}]},

 @messages=

  {:user=>["translation missing: ja.activerecord.errors.models.artist.attributes.user.required"]}>
```

### ここで仮説

```
@details={:user=>[{:error=>:blank}]},

 @messages=

  {:user=>["translation missing: ja.activerecord.errors.models.artist.attributes.user.required"]}>
```

を見る限り、 `user`が読み込まれてないよ！空だよ！というエラー？？？

というのも、まず、 `devise`でログイン機能をつけており、かつ



application_controller.rb

```ruby
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  private

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
  end

  def login_required
    redirect_to new_user_session_path, alert: "ログインして下さい" unless current_user
  end
end

```

ここで `login_required`を定義して



artist_controler.rb

```ruby
class ArtistsController < ApplicationController
  before_action :login_required, except: [:index, :show, :gozyu_on]

  FURI_GANA = [/^[ア-オ]+/,/^[カ-コ]+/,/^[サ-ソ]+/,/^[タ-ト]+/,/^[ナ-ノ]+/,/^[ハ-ホ]+/,
              /^[マ-モ]+/,/^[ヤ-ヨ]+/,/^[ラ-ロ]+/,/^[ワ-ン]+/]

  def index
    @artists = Artist.all
  end

  def gozyu_on
    @name = params[:name]
    id = params[:id].to_i
    @gozyu_on_artists = Artist.gozyu_on(FURI_GANA[id])
  end

  def new
    @artist = Artist.new
  end
  
  省略
```

ここで、 `ログインユーザー以外がArtistインスタンスを作るのを制限してる`ので、 `テストコードもログイン状態でないとダメなのでは？？`と考えました！



### 参考URL

<https://github.com/plataformatec/devise/wiki/How-To:-sign-in-and-out-a-user-in-Request-type-specs-(specs-tagged-with-type:-:request)>



### 3.コード追記

rails_helper.rb

```ruby
# This file is copied to spec/ when you run 'rails generate rspec:install'
require 'spec_helper'
ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../../config/environment', __FILE__)
# Prevent database truncation if the environment is production
abort("The Rails environment is running in production mode!") if Rails.env.production?
require 'rspec/rails'

中略

config.filter_rails_from_backtrace!
  # arbitrary gems may also be filtered via:
  # config.filter_gems_from_backtrace("gem name")

 + RSpec.configure do |config|
 + # ...
 +  config.include Devise::Test::IntegrationHelpers, type: :request
 + end
end

```



artist_spec.rb

```ruby
require 'rails_helper'

RSpec.describe Artist, type: :model do
  # pending "add some examples to (or delete) #{__FILE__}"


  context "name を指定してるとき" do
    it "アーティストが作成される" do
    + sign_in
      artist = Artist.new(name: "テスト", furi_gana: "テスト", image: Rails.root.join("public/test.jpg").open, place: "札幌", profile: "テスト", user_id: 1)
      expect(artist.valid?).to eq tru

    end
  end
end

```

### 4.再度テスト

結果

```
ajikikoutanoAir% bundle exec rspec

Artist
  name を指定してるとき
    アーティストが作成される (FAILED - 1)

Failures:

  1) Artist name を指定してるとき アーティストが作成される
     Failure/Error: sign_in
     
     NameError:
       undefined local variable or method `sign_in' for #<RSpec::ExampleGroups::Artist::Name:0x00007f809ad973b8>
     # ./spec/models/artist_spec.rb:9:in `block (3 levels) in <main>'

Finished in 0.00466 seconds (files took 5.39 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/models/artist_spec.rb:8 # Artist name を指定してるとき アーティストが作成される
```



んー、メソッド無いのか、、、



という流れで質問させて頂きました！

最後にソースコードになります。 `add_rspec`というブランチで切っております！

何卒よろしくお願い致します。m(_ _)m

<https://github.com/ajk0421/Portfolio_1/tree/add_rspec>