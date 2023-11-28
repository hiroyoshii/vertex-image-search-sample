
- 画像検索における Vector Search と Vertex Search の違い
  - サマリ
    - ユースケース
    - custom embedding で
  - vector search の立ち上げ手順
    - https://cloud.google.com/vertex-ai/docs/vector-search/overview
  - vertex search の立ち上げ手順


vector search
- https://zenn.dev/google_cloud_jp/articles/getting-started-matching-engine
- stream で update, query
- index の作成

# タイトル
画像検索における Vector Search と Vertex Search の違い

# サマリ
画像の類似検索に利用するときのポイント。

|  |vertex AI Search  | Vector Search  |
|---|---|---|
|概要  |2  |3  |
|検索のインプット  |テキスト  |ベクトル  |
|embeddingの制約 |768上限  |6  |
|課金方式  |API呼び出しごと  |サーバ課金  |
|ユースケース  |5  |6  |

モデルの学習をする？
vertex ai search で、multimordal embedding によって生成したものは使えない。

# Vertex AI Search
- 書いてある通り。
- 構造化のパターンで
  - embedding の作成
  - json を gcs に配置
  - スキーマの編集
  - 取り込み
  - サービスの設定
  - アプリ化


# Vertex Vector Search
- 書いてある通り。
  - 
  - https://github.com/GoogleCloudPlatform/vertex-ai-samples/blob/main/notebooks/official/matching_engine/sdk_matching_engine_create_stack_overflow_embeddings_vertex.ipynb
- 手順
  - embedding の作成
  - gcs に配置

  - index 作成
  - index endpoint の作成

## embedding の作成
- env
```
GCS_DIR=gs://animal-search-us/
```

- multi mordal embedding
```
echo "{\"instances\": [{\"image\": {\"bytesBase64Encoded\": \"$(base64 ./multimodalembedding/img/bull-dog.png -w 0)\"}}]}" | \
 curl -X POST \
 -H "Authorization: Bearer $TOKEN"\
 -H "Content-Type: application/json; charset=utf-8" \
 -d @-  \
 https://us-central1-aiplatform.googleapis.com/v1/projects/hiroyoshii-sandbox/locations/us-central1/publishers/google/models/multimodalembedding@001:predict
```

- embedding.json
```
{"id": "1", "uri":"https://storage.cloud.google.com/animal-search-public/cats.png", "embedding": [-0.0232418478,0.0302745607,...]}
{"id": "2", "uri":"https://storage.cloud.google.com/animal-search-public/american-cat.png", "embedding": [-0.00336091942,0.0222148206,...]}
```

- upload
```
gsutil embedding.json gs://animal-search-us/embedding.json
```

## deploy index
```
INDEX_NAME=animal
PROJECT_NAME=hiroyoshii-sandbox
REGION=us-central1
METADATA_FILE_NAME=./vectorembedding/index.json
```

```
{
    "contentsDeltaUri": "$GCS_DIR",
    "config": {
      "dimensions": 1408,
      "approximateNeighborsCount": 150,
      "distanceMeasureType": "DOT_PRODUCT_DISTANCE",
      "shardSize": "SHARD_SIZE_SMALL",
      "algorithm_config": {
        "treeAhConfig": {
          "leafNodeEmbeddingCount": 5000,
          "leafNodesToSearchPercent": 3
        }
      }
    }
  }
```

```
 gcloud ai indexes create --metadata-file=$METADATA_FILE_NAME   --display-name=$INDEX_NAME --project=$PROJECT_NAME   --region=$REGION
```

- deploy index endpoint
```
gcloud ai index-endpoints create \
  --display-name=${INDEX_NAME}_endpoint \
  --public-endpoint-enabled \ 
  --project=$PROJECT_NAME \
  --region=$REGION
```

`Created Vertex AI index endpoint: projects/226144098216/locations/us-central1/indexEndpoints/4866003057931976704.`

- deploy index
```
INDEX_ENDPOINT_ID=4866003057931976704
INDEX_ID=3794709296571219968
gcloud ai index-endpoints deploy-index ${INDEX_ENDPOINT_ID} \
  --deployed-index-id=${INDEX_NAME}_id \
  --display-name=${INDEX_NAME} \
  --index=${INDEX_ID} \
  --enable-access-logging \
  --machine-type=e2-standard-2 \
  --max-replica-count=1 --min-replica-count=1 \
  --project=$PROJECT_NAME \
  --region=$REGION
  ```
autoscale を1にしないと quota に引っかかるよ。
 'The following quotas are exceeded: MatchingEngineDeployedIndexNodes'


  - アプリ化
```
 python .\predict_request_gapic.py --image_file .\img\bull-dog.png --project hiroyoshii-sandbox

 POST https://us-central1-aiplatform.googleapis.com/v1/projects/hiroyoshii-sandbox/locations/us-central1/publishers/google/models/multimodalembedding@001:predict
```

json
```
echo "{\"instances\": [{\"image\": {\"bytesBase64Encoded\": \"$(base64 ./multimodalembedding/img/cats.png -w 0)\"}}]}" \
 | curl -X POST \
 -H "Authorization: Bearer $TOKEN" \
 -H "Content-Type: application/json; charset=utf-8" \
 -d @- \
 https://us-central1-aiplatform.googleapis.com/v1/projects/hiroyoshii-sandbox/locations/us-central1/publishers/google/models/multimodalembedding@001:predict
```

```
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN"  https://142351449.us-central1-226144098216.vdb.vertexai.goog/v1/projects/226144098216/locations/us-central1/indexEndpoints/1245108957526097920:findNeighbors -d '{deployed_index_id: "animal_1700983711903", queries: [{datapoint: { feature_vector: [1, 1, 1]}, neighbor_count: 3}]}'
```


https://ex-ture.com/blog/2023/11/24/vertex-ai_vector_search/
https://zenn.dev/google_cloud_jp/articles/getting-started-matching-engine
