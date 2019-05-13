# carrierwaveのcacheについての質問

## 状況

仕様で

new→確認画面→create

edit→確認画面→update

という流れでDBに保存されるのですが、

```
1. 登録(編集)際、モデルのバリデーションに引っかかりnewへrenderされる
2. 確認画面から "戻る"でnewに戻る
```

いずれかをするとcarrierwaveでアップロードした画像が消えてしまう。



1. 登録(編集)でモデルのバリデーションに引っかかりnewへrenderされる

* 以下の様に、画像をアップしプロフィールを書かずに確認画面へ
![スクリーンショット 2019-05-13 19.13.10](https://github.com/ajk0421/questions/blob/master/Q_carrierwave/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-05-13%2019.13.10.png)



* プロフィールは `presence: true ` にしている為バリデーションに引っかかり、newへrennderされるんですが、アップした画像が消えてしまう。

![スクリーンショット 2019-05-13 19.13.36](https://github.com/ajk0421/questions/blob/master/Q_carrierwave/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-05-13%2019.13.36.png)


2. 確認画面から "戻る"でnewに戻る

* 入力後確認画面から"戻る"

![スクリーンショット 2019-05-13 19.23.02](https://github.com/ajk0421/questions/blob/master/Q_carrierwave/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-05-13%2019.23.02.png)

* 画像が消えてしまう

![スクリーンショット 2019-05-13 19.23.16](https://github.com/ajk0421/questions/blob/master/Q_carrierwave/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202019-05-13%2019.23.16.png)

## 実装したいこと

アップした画像を残しておきたいです！



## 試したこと

### 1.viewファイル、ストロングパラメーターにimage_cacheの記述、

#### 参考
[公式github](https://github.com/carrierwaveuploader/carrierwave)

#### コード

new.html.slim

```
＃前略

.form-group
        | ＊
        = f.label :image
        = f.file_field :image, class: 'form-control', id: 'artist_image'
        = f.hidden_field :image_cache
        
＃後略
```



artist_controller.rb

```
＃前略

private

  def artist_params
    params.require(:artist).permit(:name, :furi_gana, :image, :image_cache, :place, :profile, :hp, :twitter, :youtube)
  end
  
 #後略
```



### 2.インスタンス作成時のログをチェック
#### 参考
https://codeday.me/jp/qa/20190414/623098.html


#### ログ

```
Parameters: {"utf8"=>"✓", "authenticity_token"=>"IpK846DB4x2o0v2f9F16IDJ/09qv6BgOGrDFP00jky2F1biFYW91q4DW/q+3usKD2Qh04Nr+wj+X4K96WUt9gw==", "artist"=>{"name"=>"テスト", "furi_gana"=>"テスト", "image"=>#<ActionDispatch::Http::UploadedFile:0x00007fc683a0f270 @tempfile=#<Tempfile:/var/folders/1v/ly7ct_v13wj6c6mtqp2jz6hh0000gn/T/RackMultipart20190513-40625-g1ij92.JPG>, @original_filename="TOP.JPG", @content_type="image/jpeg", @headers="Content-Disposition: form-data; name=\"artist[image]\"; filename=\"TOP.JPG\"\r\nContent-Type: image/jpeg\r\n">,

"image_cache"=>"",

"place"=>"札幌", "profile"=>"", "hp"=>"", "twitter"=>"", "youtube"=>""}, "commit"=>"確認"}
```



`"image_cache"=>""`とある様にそもそもキャッシュが発行されていないのが問題？？



このあと、 `binding.pry`でも確認しましたが、解決には至らずでした。



お力添え頂けると嬉しいです。



よろしくお願い致します！



## ソースコード

<https://github.com/ajk0421/Portfolio_1/tree/modified_upload_image>