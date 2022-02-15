# [Railsのjbuilderの書き方と便利なイディオムやメソッド - Qiita](https://qiita.com/ryouzi/items/06cb0d4aa7b6527b3645)

### １．単純な書き方
``` index.json.jbuilder
json.text "テキスト"

# {"text": "テキスト"}
```

### ２．set!メソッド
``` index.json.jbuilder
json.set! :tweet do
  json.set! text, "テキスト"
end

# {"tweet": {"text": "テキスト"} }
```

### ３．インスタンス変数を使ったjbuidlerの書き方

### ３．１インスタンス変数の中身が１つの時

``` tweets_controller.rb
class Api::TweetsController < ApplicationController
  def show
    @tweet = Tweet.find(params[:id])
  end
end
```

``` index.json.jbuilder
json.tweet do |tweet|
  tweet.title @tweet.title
  tweet.text  @tweet.text
end
# or
json.tweet do
  json.text  @tweet.text
  json.title @tweet.title
end

# {"tweet": {"text": "テキスト1", "title": "タイトル1"} }
```

### extract!メソッド
引数のオブジェクトの中身を指定して返す
``` index.json.jbuilder
json.tweet do
  json.extract! @tweet, :text, :title
end

# {"tweet": {"text": "テキスト1", "title": "タイトル1"} }
```

### merge!メソッド
ハッシュで値を追加する
``` index.json.jbuilder
text_hash = { text: "テキスト" }

json.tweet do
  json.title "タイトル"
  json.merge! text_hash
end

# {"tweet": {"title": "タイトル", "text": "テキスト"} }
```

### ３．２インスタンス変数の中身が１つの時
``` tweets_controller.rb
class Api::TweetsController < ApplicationController
  def index
    @tweets = Tweet.all
  end
end
```

### array!メソッド
配列を回して値を返す
``` index.json.jbuilder
json.array! @tweets, :title, :text
# or
json.array! @tweets do |tweet|
  json.title tweet.title
  json.text  tweet.text
end
# [{"title": "タイトル1", "text": "テキスト1"}, {"title": "タイトル2", "text": "テキスト2"}]
```

# [Jbuilderのpartialを使ってDRYにjsonを書く - Qiita](https://qiita.com/upinetree/items/22e2442d99e5a78d680f)

### partial!メソッド
``` _article.json.jbuilder
json.extract! article, :id, :amount
json.url article_url(article)
```

``` index.json.jbuilder
json.articles @articles do |article|
  json.partial! article
end
```
