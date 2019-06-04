# フッシーさんコードリファクタリング

## ソースコード

``` 
# Musicクラスの定義
#→クラスの中に処理のかたまり(defメソッド)を定義する
class Book
  attr_accessor :title
  attr_accessor :year
  attr_accessor :other


  # インスタンスを生成し、インスタンス変数を代入
  #→initializeメソッドを使うことで、@title,@year,@otherが毎回初期化されるので毎処理ごとに設定すなくてする（同じコードを書かなくて済む）

  def initialize(title:, year:, other:)
    @title = title
    @year = year
    @other = other
  end

#title,year,otherを表示する処理
  def booklist
    return "#{@title}　/(#{@year})"
  end

# 選択中（最初）の本情報を出力する処理
  def book_date
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end


  # 次に選択本の情報を出力する処理
  def book_selected
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end

  # 次に選択中の情報を出力する処理
  def book_selected
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end

end

# 本リスト（class Bookからインスタンスメソッド（newメソッド）を生成し、
#それぞれに情報（title,year,other)を与える

book1 = Book.new(title: "ワンピース",  year: 1997, other: "海賊王を夢見る少年モンキー・D・ルフィを主人公とする、「ひとつなぎの大秘宝（ワンピース）」を巡る海賊漫画")
book2 = Book.new(title: "ドラゴンボール",year: 1984, other: "世界中に散らばった七つの球をすべて集めると、どんな願いも一つだけ叶えられるという秘宝・ドラゴンボールと、主人公・孫悟空を中心に展開する「冒険」「夢」「バトル」「友情」などを描いた漫画")
book3 = Book.new(title: "スラムダンク", year: 1990, other: "主人公の不良少年桜木花道の挑戦と成長を軸にしたバスケットボール漫画")

# 変数booksを定義して配列を代入

books = [book1, book2, book3]

# プレイリスト表示
puts "Booklist --------------------------------------------------------"

#each文を使い、本を表示（繰り返し処理）

books.each.with_index do |book, i|
  puts "#{i}. #{book.booklist}"
end
puts "-----------------------------------------------------------------"
puts ""

# １冊目を選んで番号を入力
#getメソッド（キーボード入力された文字列を取得する）を使い、.chompで改行を外し、
#.to_iで数値オブジェクトに変換することでターミナルに入力できるようにする

puts "1冊目を選んでください。"
book_first = gets.chomp.to_i

# １冊目が選択される

puts books[book_first].book_date


# 次の本を選択

puts "次の本を選んでください。"
book_next = gets.chomp.to_i
#book_nextという新しく変数に代入することで1冊目に選んだものは選択できないようにする

puts books[book_next].book_selected

# 次の本を選択

puts "次の本を選んでください。"
book_next1 = gets.chomp.to_i

puts books[book_next1].book_selected
```



## リファクタリング



```
# Musicクラスの定義
```
 `Bookクラス` ですね！笑



```
attr_accessor :title
attr_accessor :year
attr_accessor :other
```
↓
```
attr_accessor :title, :year, :other
```

ここは１行で書いちゃいましょう！



で、ここからです！

```
def initialize(title:, year:, other:)
  @title = title
  @year = year
  @other = other
end
```
この部分ですが `オプショナル引数` という渡し方があります！

実際書くと

```
def initialize(**params)
  @title = params[:title]
  @year = params[:year]
  @other = params[:other]
end
```
このように記述になります！

こうすることで、例えば、`作者や一言コメントも追加したい` となった時にも簡潔に書くことができます！

```
def initialize(**params)
  @title = params[:title]
  @year = params[:year]
  @other = params[:other]
  @author= params[:author]
  @comments = params[:comments]
end
```
こんな感じですね！是非活用して下さい！

参考：https://qiita.com/rtoya/items/33617078501776fdcad7




```
#title,year,otherを表示する処理
  def booklist
    return "#{@title}　/(#{@year})"
  end

# 選択中（最初）の本情報を出力する処理
  def book_date
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end


  # 次に選択本の情報を出力する処理
  def book_selected
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end

  # 次に選択中の情報を出力する処理
  def book_selected
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end

end
```



ここはいくつか疑問点があるので書き出します！



```
- 処理の内容とメソッド名が一致しない
- 同じメソッドが2つ定義されている
- メソッド内の処理が同じのものがある
```

### 処理の内容とメソッド名が一致しない
  これは
```
def booklist
  return "#{@title}　/(#{@year})"
end
```
ここですね！ 

`booklist` という名前だと、 `いくつかの本のリスト` というイメージに僕はなるのですが、
処理内容は `各本の概要を出力` になっています！
なので、この処理だと、 `book_data` みたいな名前が良いと思います！
それか！

```
puts "Booklist --------------------------------------------------------"

#each文を使い、本を表示（繰り返し処理）

books.each.with_index do |book, i|
  puts "#{i}. #{book.booklist}"
end
puts "-----------------------------------------------------------------"
puts ""
```

この部分丸っとメソッドにするのもありですね！
```
class Book
  
  略

#title,year,otherを表示する処理
  def self.list(books)
    puts "Booklist --------------------------------------------------------"

    #each文を使い、本を表示（繰り返し処理）

    books.each.with_index do |book, i|
      puts "#{i}. #{book.title} /(#{book.year})"
    end
    puts "-----------------------------------------------------------------"
    puts ""
  end
end
```



### 同じメソッドが2つ定義されている

### メソッド内の処理が同じのものがある

今回の一番疑問点です！

```
def book_date
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end


  # 次に選択本の情報を出力する処理
  def book_selected
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end

  # 次に選択中の情報を出力する処理
  def book_selected
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end
```

* 3つのメソッドは全て処理内容が同じ
* 下2つは名前も同じ
* returnの使い方がおかしい



ここが一番改善すべきとこかと思います！



Railsの設計理念に `DRY(Don’t Repeat Yourself)` という考えがあります。

何回も同じ事を書くな！ってことですね！つまり、同じ処理を何個も書いた時は、 `ここには無駄が潜んでいるかも、、、` と考えると良いです！



それで、今回ふっしーさんが実装したかたった内容をコードのコメントから推測すると

```
- Booklistから一つ本を選ぶ
- 次に選択す時は1冊目に選んだ本は選べない
```

という事を実装したかったのか？？と思っています！あくまで予想ですが、、

そんなこんなで、僕なりにリファクタリングしつつコードを書いてみたので、参考にしてみて下さい！

※実装かなーーーーーり強引です！笑



```
class Book
  attr_accessor :title, :year, :other

  def initialize(**params)
    @title = params[:title]
    @year = params[:year]
    @other = params[:other]
  end

  def self.list(books)
    puts "Booklist --------------------------------------------------------"

    books.each.with_index(1) do |book, i|
      puts "#{i}. #{book.title} /(#{book.year})"
    end
    puts "-----------------------------------------------------------------"
    puts ""
  end

  def self.select(books)
    select_nums = []
    roop_times = books.size

    roop_times.times do |i|
      puts "#{i + 1}冊目の本を選んでください"
      select_num = gets.chomp.to_i - 1

      redo if select_nums.include?(select_num) || select_num > roop_times

      select_nums << select_num

      puts books[select_num].book_data
     end
  end

  def book_data
    return "選択中の本は【 #{@title}　/ 連載開始は#{@year}~  】です \n #{@other}"
  end

end

book1 = Book.new(title: "ワンピース",  year: 1997, other: "海賊王を夢見る少年モンキー・D・ルフィを主人公とする、「ひとつなぎの大秘宝（ワンピース）」を巡る海賊漫画")
book2 = Book.new(title: "ドラゴンボール",year: 1984, other: "世界中に散らばった七つの球をすべて集めると、どんな願いも一つだけ叶えられるという秘宝・ドラゴンボールと、主人公・孫悟空を中心に展開する「冒険」「夢」「バトル」「友情」などを描いた漫画")
book3 = Book.new(title: "スラムダンク", year: 1990, other: "主人公の不良少年桜木花道の挑戦と成長を軸にしたバスケットボール漫画")

books = [book1, book2, book3]

# プレイリスト表示
Book.list(books)

# booksの中から選択
Book.select(books)
```



