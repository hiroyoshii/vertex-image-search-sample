# Overview
- 動物の写真を検索する。

# Architecture
- gcs に画像を保存
- vertex embedding で画像を embedding??
  - https://cloud.google.com/vertex-ai/docs/generative-ai/embeddings/get-multimodal-embeddings?hl=ja
- vertex search 
  - https://console.cloud.google.com/gen-app-builder/engines/create?project=hiroyoshii-sandbox
  - どうやって embedding と連携するんだろ。。
  - これだ
    - https://cloud.google.com/generative-ai-app-builder/docs/bring-embeddings
- 改善系
  - https://cloud.google.com/generative-ai-app-builder/docs/tune-search

- これかもな、、
  - https://cloud.google.com/vertex-ai/docs/vector-search/overview

# 手順
1. これで embedding を作って
https://cloud.google.com/vertex-ai/docs/generative-ai/embeddings/get-multimodal-embeddings?hl=ja#python

2. これで非構造 embedding のメタデータを作って(embedding を追加)
https://cloud.google.com/generative-ai-app-builder/docs/prepare-data#unstructured

3. これで embedding をキープロパティに指定する
https://cloud.google.com/generative-ai-app-builder/docs/bring-embeddings#byb

4. 最後にアプリ
https://cloud.google.com/generative-ai-app-builder/docs/create-engine-es

# メモ
だめっぽいな。
- vertex search のサポートする embedding の大きさとあわない
- mime type が pdf か html だけ
-> 構造化データでやればよいかも？検索対象の image はただの url にして、 embedding に情報こめちゃえば。resource は image で
  -> vector size があわないのだけ問題。。
  -> BQ にバイト文字列として画像入れとけばよい？あー、検索 resp の制限にかかりそうだけど。。ｚｘ
    - '<img src="data:image/' + file_type + ';base64,' + img_base64 + '" />'


https://hiroyoshii-sandbox.web.app/


# vector search
https://cloud.google.com/vertex-ai/docs/vector-search/overview