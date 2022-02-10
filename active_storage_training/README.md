# [Active Storageの概要](https://railsguides.jp/active_storage_overview.html)

## 1. 要件

## 2. セットアップ
```
$ rails -v
Rails 6.0.4.4
$ rails new active_storage_training --skip-webpack-install
$ rails active_storage:install
Copied migration 20220210052021_create_active_storage_tables.active_storage.rb from active_storage

$ bin/rails db:migrate
== 20220210052021 CreateActiveStorageTables: migrating ========================
-- create_table(:active_storage_blobs, {})
   -> 0.0040s
-- create_table(:active_storage_attachments, {})
   -> 0.0035s
== 20220210052021 CreateActiveStorageTables: migrated (0.0076s) ===============

$ vim config/storage.yml
$ vim Gemfile
$ bundle install
```

config/storage.yml
```
test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>

local:
  service: Disk
  root: <%= Rails.root.join("storage") %>

# Use rails credentials:edit to set the AWS secrets (as aws:access_key_id|secret_access_key)
amazon:
  service: S3
  access_key_id: <%= Rails.application.credentials.dig(:aws, :access_key_id) %>
  secret_access_key: <%= Rails.application.credentials.dig(:aws, :secret_access_key) %>
  region: us-east-1
  bucket: your_own_bucket
```

Gemfile
```
...
gem "aws-sdk-s3", require: false
```

## 3. ファイルをレコードに添付する
### 3.1 `has_one_attached`
レコードとファイルの間に1対1のマッピングを設定する。レコード1件ごとに1個のファイルを添付できる。

```
$ bin/rails generate model User avatar:attachment
Running via Spring preloader in process 11712
      invoke  active_record
      create    db/migrate/20220210055029_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
$ cat app/models/user.rb
class User < ApplicationRecord
  has_one_attached :avatar
end
$ bin/rails migrate
```

```
irb> user.avatar.attach(params[:avatar])
irb> user.avatar.attached?
irb> User.first.avatar
  User Load (0.3ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT ?  [["LIMIT", 1]]
=> #<ActiveStorage::Attached::One:0x000055acbbdbc2f8 @name="avatar", @record=#<User id: 1, created_at: "2022-02-10 06:02:53", updated_at: "2022-02-10 06:02:53">>

```

### 3.2 `has_many_attached`
レコードとファイルの間に1対多の関係を設定する。レコード1件ごとに、多数の添付ファイルを添付できる。

```
$ bin/rails generate model Message
...
$ vim app/models/message.rb
# class Message < ApplicationRecord
#   has_many_attached :images
# end
$ bin/rails db:migrate
```

```
irb> @message.images.attach(params[:images])
irb> @message.images.attached?
irb> Message.first.images
  Message Load (0.2ms)  SELECT "messages".* FROM "messages" ORDER BY "messages"."id" ASC LIMIT ?  [["LIMIT", 1]]
=> #<ActiveStorage::Attached::Many:0x000055acb994f9d8 @name="images", @record=#<Message id: 1, created_at: "2022-02-10 06:16:27", updated_at: "2022-02-10 06:16:27">>
```

### 3.3 File/IO Objects
```
irb:021:0> Message.first.images.attach(io: File.open('app/assets/images/sample.jpg'), filename: 'sample.jpg')
  Message Load (0.4ms)  SELECT "messages".* FROM "messages" ORDER BY "messages"."id" ASC LIMIT ?  [["LIMIT", 1]]
  ActiveStorage::Blob Load (0.3ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" INNER JOIN "active_storage_attachments" ON "active_storage_blobs"."id" = "active_storage_attachments"."blob_id" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ?  [["record_id", 1], ["record_type", "Message"], ["name", "images"]]
   (0.2ms)  begin transaction
  ActiveStorage::Attachment Load (0.5ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ?  [["record_id", 1], ["record_type", "Message"], ["name", "images"]]
  ActiveStorage::Blob Create (1.2ms)  INSERT INTO "active_storage_blobs" ("key", "filename", "content_type", "metadata", "byte_size", "checksum", "created_at") VALUES (?, ?, ?, ?, ?, ?, ?)  [["key", "uka468s2cl2wji6tutsozc27ewwq"], ["filename", "sample.jpg"], ["content_type", "image/png"], ["metadata", "{\"identified\":true}"], ["byte_size", 8475], ["checksum", "Qgb407lo+rcLI3RMM2hZaw=="], ["created_at", "2022-02-10 06:38:27.501173"]]
  ActiveStorage::Attachment Create (1.0ms)  INSERT INTO "active_storage_attachments" ("name", "record_type", "record_id", "blob_id", "created_at") VALUES (?, ?, ?, ?, ?)  [["name", "images"], ["record_type", "Message"], ["record_id", 1], ["blob_id", 3], ["created_at", "2022-02-10 06:38:27.507291"]]
  Message Update (0.6ms)  UPDATE "messages" SET "updated_at" = ? WHERE "messages"."id" = ?  [["updated_at", "2022-02-10 06:38:27.511337"], ["id", 1]]
   (14.4ms)  commit transaction
  Disk Storage (2.8ms) Uploaded file to key: uka468s2cl2wji6tutsozc27ewwq (checksum: Qgb407lo+rcLI3RMM2hZaw==)
Enqueued ActiveStorage::AnalyzeJob (Job ID: ce4659da-b1ca-4f7b-9a66-270e7ac26dd3) to Async(active_storage_analysis) with arguments: #<GlobalID:0x000055acbb422710 @uri=#<URI::GID gid://active-storage-training/ActiveStorage::Blob/3>>
=> true
irb(main):022:0>   ActiveStorage::Blob Load (0.1ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = ? LIMIT ?  [["id", 3], ["LIMIT", 1]]
Performing ActiveStorage::AnalyzeJob (Job ID: ce4659da-b1ca-4f7b-9a66-270e7ac26dd3) from Async(active_storage_analysis) enqueued at 2022-02-10T06:38:27Z with arguments: #<GlobalID:0x000055acbb417130 @uri=#<URI::GID gid://active-storage-training/ActiveStorage::Blob/3>>
  Disk Storage (0.1ms) Downloaded file from key: uka468s2cl2wji6tutsozc27ewwq
Skipping image analysis because the mini_magick gem isn't installed
   (0.1ms)  begin transaction
  ActiveStorage::Blob Update (0.3ms)  UPDATE "active_storage_blobs" SET "metadata" = ? WHERE "active_storage_blobs"."id" = ?  [["metadata", "{\"identified\":true,\"analyzed\":true}"], ["id", 3]]
   (8.0ms)  commit transaction
Performed ActiveStorage::AnalyzeJob (Job ID: ce4659da-b1ca-4f7b-9a66-270e7ac26dd3) from Async(active_storage_analysis) in 20.16ms

irb(main):023:0> Message.first.images.attached?
  Message Load (0.4ms)  SELECT "messages".* FROM "messages" ORDER BY "messages"."id" ASC LIMIT ?  [["LIMIT", 1]]
  ActiveStorage::Attachment Exists? (0.6ms)  SELECT 1 AS one FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ? LIMIT ?  [["record_id", 1], ["record_type", "Message"], ["name", "images"], ["LIMIT", 1]]
=> true
```

## 4. ファイルを削除する
```
irb(main):007:0> Message.first.images.find_by_id(1).purge
  Message Load (0.4ms)  SELECT "messages".* FROM "messages" ORDER BY "messages"."id" ASC LIMIT ?  [["LIMIT", 1]]
  ActiveStorage::Attachment Load (0.3ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ? AND "active_storage_attachments"."id" = ? LIMIT ?  [["record_id", 1], ["record_type", "Message"], ["name", "images"], ["id", 1], ["LIMIT", 1]]
  ActiveStorage::Attachment Destroy (16.5ms)  DELETE FROM "active_storage_attachments" WHERE "active_storage_attachments"."id" = ?  [["id", 1]]
  ActiveStorage::Blob Load (0.2ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
   (0.1ms)  begin transaction
  ActiveStorage::Attachment Exists? (0.2ms)  SELECT 1 AS one FROM "active_storage_attachments" WHERE "active_storage_attachments"."blob_id" = ? LIMIT ?  [["blob_id", 1], ["LIMIT", 1]]
  ActiveStorage::Attachment Load (0.1ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ? LIMIT ?  [["record_id", 1], ["record_type", "ActiveStorage::Blob"], ["name", "preview_image"], ["LIMIT", 1]]
  ActiveStorage::Blob Destroy (0.2ms)  DELETE FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = ?  [["id", 1]]
   (8.2ms)  commit transaction
  Disk Storage (0.2ms) Deleted file from key: dh9ymk33ww4wafjedtnuojfb1htj
  Disk Storage (0.1ms) Deleted files by key prefix: variants/dh9ymk33ww4wafjedtnuojfb1htj/
=> []
irb(main):008:0>
```

## 5. ファイルを配信する
### 5.1 リダイレクトモード
`RedirectController`は、サービスの実際のエンドポイントにリダイレクトします。
この間接参照によってサービスURLと実際のURLが切り離され、たとえば添付ファイルを別サービスにミラーリングして可用性を高めることが可能になります。
リダイレクトのHTTP有効期限は5分です。

HOSTは適当なので違うかもしれない
```
irb(main):013:0> url_for(Message.first.images.find_by_id(2))
  Message Load (0.2ms)  SELECT "messages".* FROM "messages" ORDER BY "messages"."id" ASC LIMIT ?  [["LIMIT", 1]]
  ActiveStorage::Attachment Load (0.1ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ? AND "active_storage_attachments"."id" = ? LIMIT ?  [["record_id", 1], ["record_type", "Message"], ["name", "images"], ["id", 2], ["LIMIT", 1]]
  ActiveStorage::Blob Load (0.1ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
=> "http://localhost:3000/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBCdz09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--b76e9ce3b96ccd7a8a66e6add90eaef5128cb2cf/sample.jpg"
```

ダウンロードリンクを作成するには、`rails_blob_path`や`rails_blob_url`ヘルパーを使います。
```
irb(main):015:0> rails_blob_path(Message.first.images.find_by_id(2))
  Message Load (0.4ms)  SELECT "messages".* FROM "messages" ORDER BY "messages"."id" ASC LIMIT ?  [["LIMIT", 1]]
  ActiveStorage::Attachment Load (0.4ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ? AND "active_storage_attachments"."id" = ? LIMIT ?  [["record_id", 1], ["record_type", "Message"], ["name", "images"], ["id", 2], ["LIMIT", 1]]
  ActiveStorage::Blob Load (0.4ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
=> "/rails/active_storage/blobs/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBCdz09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--b76e9ce3b96ccd7a8a66e6add90eaef5128cb2cf/sample.jpg"
```

### 5.2 プロキシモード

### 5.3 認証済みコントローラ

## 6. ファイルをダウンロードする
保存する方法はわからんかった
```
irb(main):022:0> binary = Message.first.images.find_by_id(2).download
```
## 7. ファイルを解析する

## 8. 画像、動画、PDFを表示する
```
irb(main):001:0> Message.first.images.find_by_id(2).representable?
   (1.5ms)  SELECT sqlite_version(*)
  Message Load (0.2ms)  SELECT "messages".* FROM "messages" ORDER BY "messages"."id" ASC LIMIT ?  [["LIMIT", 1]]
  ActiveStorage::Attachment Load (0.2ms)  SELECT "active_storage_attachments".* FROM "active_storage_attachments" WHERE "active_storage_attachments"."record_id" = ? AND "active_storage_attachments"."record_type" = ? AND "active_storage_attachments"."name" = ? AND "active_storage_attachments"."id" = ? LIMIT ?  [["record_id", 1], ["record_type", "Message"], ["name", "images"], ["id", 2], ["LIMIT", 1]]
  ActiveStorage::Blob Load (0.1ms)  SELECT "active_storage_blobs".* FROM "active_storage_blobs" WHERE "active_storage_blobs"."id" = ? LIMIT ?  [["id", 2], ["LIMIT", 1]]
=> true
```
## 9. 画像を変形する

## 10. ファイルをプレビューする

## 11. ダイレクトアップロード

## 12. テスト

## 13. テスト中に作成したファイルを破棄する

## 14. その他のクラウドサービスのサポートを実装する

## 15. あタッチされなかったアップロードを破棄する
