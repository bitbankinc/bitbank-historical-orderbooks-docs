# ビットバンク VIPプログラム ヒストリカルデータについて

[English](README.md)

## 概要
ビットバンク株式会社（以下「当社」といいます。）は、VIPプログラムの一環として、当社暗号資産取引所サービスのヒストリカルデータの提供を行っております。

### VIPプログラムとは  
一定金額以上の取引量があるお客様に対し、手数料の優遇や専用窓口の開設など、様々な特典内容をご用意しております。  
プログラムの適用を希望される際は以下ページよりお申し込みください：  
- [VIPプログラムの申込ページ](https://bitbank.cc/special/vip-program/)


## オーダーブックスナップショット
### データ概要

| 区分                 | 詳細                                                    |
| -------------------- | ------------------------------------------------------- |
| ファイルフォーマット | CSV                                                     |
| 粒度                 | 1分間に2回程度（※ばらつきあり）                         |
| 値幅                 | best bid/ask より上下200本ずつ                          |
| データの更新頻度     | JST（日本標準時）23時前後に前々日のデータが更新されます |

### 提供ペア

| Ticker |         データ期間         |
| :----: | :------------------------: |
|  BTC   | 2019-03-13 ~ 当日より2日前 |
|  XRP   | 2019-03-13 ~ 当日より2日前 |
|  ETH   | 2023-07-24 ~ 当日より2日前 |
|  DOGE  | 2023-07-24 ~ 当日より2日前 |
|  SOL   | 2024-11-21 ~ 当日より2日前 |
|  XLM   | 2023-07-24 ~ 当日より2日前 |
|  ADA   | 2023-07-24 ~ 当日より2日前 |
|  LTC   | 2023-07-24 ~ 当日より2日前 |
|  LINK  | 2023-07-24 ~ 当日より2日前 |
|  AVAX  | 2023-07-24 ~ 当日より2日前 |


## 取得方法

ヒストリカルデータは、当社がご用意するAWS S3バケットにアクセスし、取得いただきます。  
アクセス方法は以下いずれかの方法からお選びいただけます：

1. AWSアカウントIDの登録  
2. IPアドレスの登録  

登録後、当社よりS3バケットへのアクセスを許可します。

S3のデータ構造は以下となります。

- S3バケット名: `564226375708-historical-order-books`
- PATH(ディレクトリ): `orderbook-snapshot/${TICKER}JPY/YYYY_MM/`
  - `${TICKER}`は、提供ペアを参照し大文字で入力ください。(例：`BTC`)
- オブジェクト(gz形式): `order_book_snapshots_btba_spot_${ticker}jpy_YYYY-MM-DD.csv.gz`
  - `-`と`_`の間違いに注意ください。
  - `${ticker}`は小文字となります（例：`xrp`）

## 例

実装サンプルを記載しております。参考程度にご利用ください。

### 1. AWSアカウントIDを利用する場合

#### 初期設定
- Credential等の設定を済ませていること。
  - 参考
    - [公式ドキュメント](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-files.html)
    - [AWSアカウントで最初にやるべきこと](https://dev.classmethod.jp/articles/aws-baseline-setting-202206/)のMUST項目の設定をしておくことをお勧めします。

#### Pythonによるデータの取得
- [AWS SDK for Python](https://github.com/boto/boto3)(boto3)が必要です。


<details>
  <summary>サンプルコード1: 単日での取得例</summary>
  <p>

  ```Python
  import boto3

  def download_s3_object(bucket_name, object_key, local_file_path):
      """
      指定されたS3バケットからオブジェクトファイルをダウンロードします。
      Args:
          bucket_name (str): S3バケットの名前。
          object_key (str): ダウンロードするオブジェクトのキー。
          local_file_path (str): ローカルに保存するファイルパス。
      """
      # boto3を使用してS3クライアントを作成
      s3 = boto3.resource('s3', region_name='us-east-1')

      try:
          # S3からオブジェクトをダウンロードしてローカルに保存
          s3.meta.client.download_file(bucket_name, object_key, local_file_path)
          print(f"Object downloaded successfully to {local_file_path}")
      except Exception as e:
          print(f"Error downloading object: {e}")

  # テスト用のバケット名、オブジェクトキー、ローカルファイルパス
  bucket_name = '564226375708-historical-order-books'
  object_key = 'orderbook-snapshot/BTCJPY/2024-02/order_book_snapshots_btba_spot_btcjpy_2024-02-06.csv.gz' # 必要に応じて値を変更してください
  local_file_path = './hoge.csv.gz' # 必要に応じて値を変更してください

  # サンプルの実行
  download_s3_object(bucket_name, object_key, local_file_path)
  ```
  </p>
</details>

<details>
  <summary>サンプルコード2: 1ヶ月単位での取得例</summary>
  <p>

  ```python
  # BTCJPYの2024年2月のデータをダウンロードする場合
  import boto3

  def download_s3_objects_recursively(bucket_name, prefix='', local_directory='./'):
      """
      バケットから再帰的にオブジェクトをダウンロードする場合の例

      Args:
          bucket_name (str): S3バケットの名前。
          prefix (str): オブジェクトのプレフィックス。デフォルトは空文字列。
          local_directory (str): ローカルに保存するディレクトリパス。デフォルトはカレントディレクトリ。
      """
      # boto3 S3クライアントを作成
      s3 = boto3.client('s3')

      # 指定したプレフィックスでバケット内のオブジェクトを再帰的に取得
      paginator = s3.get_paginator('list_objects_v2')
      for result in paginator.paginate(Bucket=bucket_name, Prefix=prefix):
          if 'Contents' in result:
              for obj in result['Contents']:
                  # オブジェクトのキーを取得
                  object_key = obj['Key']
                  # オブジェクトをダウンロードしてローカルに保存
                  local_file_path = local_directory + object_key[len(prefix):]
                  s3.download_file(bucket_name, object_key, local_file_path)
                  print(f"Downloaded {object_key} to {local_file_path}")
          else:
              print("No objects found in this bucket.")

  # テスト用のバケット名とプレフィックス
  bucket_name = '564226375708-historical-order-books'
  prefix = 'orderbook-snapshot/BTCJPY/2024-02/' #必要に応じて値を変更してください

  # サンプルの実行
  download_s3_objects_recursively(bucket_name, prefix)

  ```
  </p>
</details>

- 参考
  - [公式API Document(S3部分)](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/s3-example-download-file.html)
  - [AWS SDK を使用して、Amazon S3 バケットからオブジェクトを取得する](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/download-objects.html)

#### CLIによるデータの取得
- AWS CLIがインストールされていること

<details>
  <summary>BTCJPYの2024年2月のデータをダウンロードする場合のサンプル</summary>
  <p>

  ```shell
  $ aws s3 sync s3://564226375708-historical-order-books/orderbook-snapshot/BTCJPY/2024-02/ ./
  ```
  - 参考
    - [公式Document](https://docs.aws.amazon.com/cli/latest/reference/s3/)
  </p>
</details>

### 2.IPアドレスを利用する場合

<details>
  <summary>Pythonによる取得</summary>
  <p>

  ```python
  import requests

  def download_gz_file(url, output_file):
      try:
          # APIからgzファイルをダウンロード
          response = requests.get(url)
          response.raise_for_status()

          # ファイルを保存
          with open(output_file, 'wb') as f_out:
              f_out.write(response.content)

          print("ダウンロードが完了しました。")

      except requests.exceptions.RequestException as e:
          print("ダウンロード中にエラーが発生しました:", e)

  # ダウンロードするgzファイルのURLと保存先ファイル名を指定
  url = "https://564226375708-historical-order-books.s3.amazonaws.com/orderbook-snapshot/BTCJPY/2019-03/order_book_snapshots_btba_spot_btcjpy_2019-03-13.csv.gz"
  output_file = "order_book_snapshots_btba_spot_btcjpy_2019-03-13.csv.gz"

  # gzファイルをダウンロード
  download_gz_file(url, output_file)

  ```
  </p>
</details>

<details>
  <summary>ターミナルから直接取得(curl)</summary>
  <p>

  ```bash
  # 2019-03-13のBTCJPYデータを取得したい場合
  $ curl -O https://564226375708-historical-order-books.s3.amazonaws.com/orderbook-snapshot/BTCJPY/2019-03/order_book_snapshots_btba_spot_btcjpy_2019-03-13.csv.gz
  ```
  - 参考
    - [curl tutorial](https://curl.se/docs/manual.html)

  </p>
</details>


## 注意事項
- ご利用に関する事項
  - 弊社ではAWSを利用しているため、AWS側での仕様変更によるAPIやライブラリなどの変更が生じる可能性があります。
  - 当社のS3バケットはバージニアリージョン(us-east-1)です。
  - サンプルコードのご利用は自己責任でお願いいたします。
  - データの取得時にクラウドサーバー等を利用する場合は、サービスに応じてお客様側にコストがかかる可能性がございますので、ご注意ください。
  - リクエスト数が多すぎる場合一時的にアクセスを制限させていただくことがございます。

- データの利用条件
  - 本データはご自身での利用に限り、データの第三者への配布は禁止します。
  - 本データの正確性について当社は一切保証いたしません。
  - 都合によりデータの更新が遅れる場合がございます。
  - 当社は、裁量によって、本サービスの内容を変更したり、それ自体を終了させる場合があります。