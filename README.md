# Bitbank Historical Data

[日本語](README_JP.md)

## Overview
Bitbank, Inc. (hereinafter "the Company") provides historical data from its crypto-asset exchange service.

### Volume-Based Benefits
We offer various benefits for users who meet certain trading volume thresholds, including discounted trading fees, access to a dedicated support channel, and more.


## Order-Book Snapshots

### Data Overview

| Category           | Details                                                            |
| ------------------ | ------------------------------------------------------------------ |
| File Format        | CSV                                                                |
| Sampling Frequency | ≈ 2 times per minute (Note: Actual acquisition frequency may vary) |
| Depth              | 200 levels above and 200 levels below the best bid/ask             |
| Update Timing      | Data as of two days ago is posted or updated around 11:00 PM JST   |

#### Supported Pairs and Data Range

| Ticker |   Available Data Period   |
| :----: | :-----------------------: |
|  BTC   | 2019-03-13 – two days ago |
|  XRP   | 2019-03-13 – two days ago |
|  ETH   | 2023-07-24 – two days ago |
|  DOGE  | 2023-07-24 – two days ago |
|  SOL   | 2024-11-21 – two days ago |
|  XLM   | 2023-07-24 – two days ago |
|  ADA   | 2023-07-24 – two days ago |
|  LTC   | 2023-07-24 – two days ago |
|  LINK  | 2023-07-24 – two days ago |
|  AVAX  | 2023-07-24 – two days ago |

## How to Access the Data
Historical data can be accessed through an AWS S3 bucket provided by the Company.
You can choose one of the following methods to access the data:

1. AWS account ID registration
2. IP address registration

After registration, the Company will grant you access to the S3 bucket.

### S3 Bucket and Folder Structure

- S3 Bucket name: `564226375708-historical-order-books`
- Path: `orderbook-snapshot/${TICKER}JPY/YYYY_MM/`
  - `${TICKER}` in the path is uppercase (e.g., BTC).
- Object name(.gz): `order_book_snapshots_btba_spot_${ticker}jpy_YYYY-MM-DD.csv.gz`
  - `${ticker}` in the file name is lowercase (e.g., xrp).
  - Note: Pay attention to hyphen and underscore placement.

## Example Usage
Below are usage examples for reference.

### 1. Using AWS Account ID
#### Initial Setup
- Ensure that you have configured your AWS credentials.
  - For reference:
    - [Official Documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
    - It is recommended to set up the MUST items in [AWS Baseline Setting](https://dev.classmethod.jp/articles/aws-baseline-setting-202206/).   

### Python Example for Data Retrieval
<details>
  <summary>Sample Code 1: Single Day Retrieval Example</summary>
  <p>     
  
  ```Python
  import boto3

  def download_s3_object(bucket_name, object_key, local_file_path):
      """
      # Downloads a single object from the specified S3 bucket.
      Args:
          bucket_name (str): Name of the S3 bucket.  
          object_key (str): Key of the object to be downloaded.  
          local_file_path (str): Path to save the file locally.  
      """
      # Create a boto3 S3 client
      s3 = boto3.resource('s3', region_name='us-east-1')

      try:
          # Download the object from S3
          s3.meta.client.download_file(bucket_name, object_key, local_file_path)
          print(f"Object downloaded successfully to {local_file_path}")
      except Exception as e:
          print(f"Error downloading object: {e}")

  # Test bucket name, object key, and local file path
  bucket_name = '564226375708-historical-order-books'
  object_key = 'orderbook-snapshot/BTCJPY/2024-02/order_book_snapshots_btba_spot_btcjpy_2024-02-06.csv.gz' # Change values as needed
  local_file_path = './hoge.csv.gz' # Change values as needed

  # Sample execution
  download_s3_object(bucket_name, object_key, local_file_path)
  ```
  </p>
  </details>

<details>
  <summary>Sample Code 2: Monthly Retrieval Example</summary>
  <p> 

  ```python
  # Example for downloading BTC/JPY data for February 2024
  import boto3

  def download_s3_objects_recursively(bucket_name, prefix='', local_directory='./'):
      """
      This function downloads objects from an S3 bucket recursively.
      It retrieves objects based on the specified prefix and saves them to a local directory.
      This function downloads all objects that match the specified prefix.

      Args:
          bucket_name (str): Name of the S3 bucket.  
          prefix (str): The prefix of the objects. Default is an empty string.
          local_directory (str): Path to the local directory where files will be saved. Default is the current directory.
      """
      # Create a boto3 S3 client
      s3 = boto3.client('s3')

      # Retrieve objects from the bucket recursively based on the specified prefix
      paginator = s3.get_paginator('list_objects_v2')
      for result in paginator.paginate(Bucket=bucket_name, Prefix=prefix):
          if 'Contents' in result:
              for obj in result['Contents']:
                  # Get the object key
                  object_key = obj['Key']
                  # Download the object and save it locally
                  local_file_path = local_directory + object_key[len(prefix):]
                  s3.download_file(bucket_name, object_key, local_file_path)
                  print(f"Downloaded {object_key} to {local_file_path}")
          else:
              print("No objects found in this bucket.")

  # Test Bucket Name and Prefix
  bucket_name = '564226375708-historical-order-books'
  prefix = 'orderbook-snapshot/BTCJPY/2024-02/'   # Change the values as needed

  # Sample execution
  download_s3_objects_recursively(bucket_name, prefix)
  ```
  </p>
</details>

- References:
  - [Official API Document (S3 section)](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/s3-example-download-file.html)
  - [Using AWS SDK to Retrieve Objects from Amazon S3 Buckets](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/download-objects.html)

## CLI Method for Data Retrieval
- Ensure that AWS CLI is installed.

<details>
  <summary>Sample: Downloading BTC/JPY Data for February 2024</summary>
  <p>

  ```shell
  $ aws s3 sync s3://564226375708-historical-order-books/orderbook-snapshot/BTCJPY/2024-02/ ./
  ```
  - References:
    - [Official Document](https://docs.aws.amazon.com/cli/latest/reference/s3/)
  
  </p>
</details>

### 2. Using IP Address Registration

<details>
  <summary>Python Example for Downloading .gz File</summary>
  <p>

  ```python
  import requests

  def download_gz_file(url, output_file):
      try:
          ### Downloading gz File
          response = requests.get(url)
          response.raise_for_status()

          # Save File
          with open(output_file, 'wb') as f_out:
              f_out.write(response.content)

          print("Download completed successfully.")

      except requests.exceptions.RequestException as e:
          print("Error occurred during download:", e)

  # Download URL and output file name
  url = "https://564226375708-historical-order-books.s3.amazonaws.com/orderbook-snapshot/BTCJPY/2019-03/order_book_snapshots_btba_spot_btcjpy_2019-03-13.csv.gz"
  output_file = "order_book_snapshots_btba_spot_btcjpy_2019-03-13.csv.gz"

  # Download gz file
  download_gz_file(url, output_file)

  ```
  </p>
</details>

<details>
  <summary>Terminal Example for Downloading .gz File</summary>
  <p>

  ```bash
  # If you want to retrieve BTCJPY data for 2019-03-13
  $ curl -O https://564226375708-historical-order-books.s3.amazonaws.com/orderbook-snapshot/BTCJPY/2019-03/order_book_snapshots_btba_spot_btcjpy_2019-03-13.csv.gz
  ```

  - References:
    - [curl Manual - Official Documentation](https://curl.se/docs/manual.html)
  </p>
</details>



## Notes
### Usage Considerations
- These code samples use AWS and may be affected by changes to AWS specifications.
- The data bucket is located in the `us-east-1` (Virginia) region.  
- The sample code is provided “as is”, without any express or implied warranties. Use at your own risk.
- Retrieving data through cloud infrastructure may incur charges from your cloud service provider.  
- High-frequency access may result in temporary rate limiting or throttling.

### Data Usage Policy
- This data is provided for internal use only. Redistribution to third parties is strictly prohibited.
- No warranty is provided regarding the accuracy of the data.
- Data updates may be delayed without notice.  
- We reserve the right to modify, suspend, or discontinue this service at any time, with or without notice.
