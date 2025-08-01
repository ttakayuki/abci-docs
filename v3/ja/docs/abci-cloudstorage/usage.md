# クラウドストレージの使い方

## AWS Command Line Interfaceセットアップ

AWS Command Line Interface (AWS CLI)はインタラクティブノードにインストール済みです。
クラウドストレージにアクセスするために以下の手順を参考にAWS CLIの設定を行ってください。

### 認証情報の設定

クラウドストレージへのアクセスは、ABCIアカウントとは別のクラウドストレージアカウントを用います。
アカウントは複数持つことができ、それぞれにアクセスキー(アクセスキー ID とシークレットアクセスキーのペア)が対応しています。
複数のABCIグループに所属しており、それぞれクラウドストレージを利用している場合は、それぞれのクラウドストレージアカウントが発行されます。

AWS CLIの利用を始めるには次のように、`aws configure`コマンドを使用して、アクセスキーを設定する必要があります。

`Access Key ID`、`Secret Access Key`にはそれぞれ発行されたアクセスキーID、シークレットアクセスキーを設定してください。
また、`region name`には、`ap-northeast-1`を設定してください。`output format`は空欄でかまいません。

```
[username@login1 ~] $ aws configure
AWS Access Key ID [None]: 発行されたACCESS-KEY-ID
AWS Secret Access Key [None]: 発行されたSECRET-ACCESS-KEY
Default region name [None]: ap-northeast-1
Default output format [None]: (入力不要)
```

複数のクラウドストレージアカウントを所持している場合は、`--profile`オプションを使用することでアカウントを使い分けることができます。
例えば、クラウドストレージアカウント `gaa00000.2` を別に登録する場合は、以下のように設定します。

```
[username@login1 ~]$ aws configure --profile gaa00000.2
AWS Access Key ID [None]: gaa00000.2's ACCESS-KEY-ID
AWS Secret Access Key [None]: gaa00000.2's SECRET-ACCESS-KEY
Default region name [None]: ap-northeast-1
Default output format [None]: (入力不要)
```

クラウドストレージアカウント`gaa00000.2`で`aws`コマンドを実行するときは、以下のように`--profile`オプションで設定したプロファイルを指定します。

```
[username@login1 ~]$ aws --profile gaa00000.2 --endpoint-url https://s3.v3.abci.ai s3api list-buckets
```

設定はホームディレクトリ下(`~/.aws`)に保存されるため、インタラクティブノードで設定していれば、計算ノードで改めて行う必要はありません。

アクセスキーの再発行や削除は、ABCI利用者ポータルから行います。

### エンドポイントURLの設定

クラウドストレージのエンドポイントURLは`https://s3.v3.abci.ai`です。
設定ファイル `~/.aws/config` にエンドポイントURLを設定することで、コマンドラインで毎回`--endpoint-url`オプションを指定する手間が省けます。

エンドポイントURLは、`config`ファイルの各プロファイルに`endpoint_url = {エンドポイントURL}` を設定します。

defaultプロファイルにエンドポイントURLを指定する例は次のとおりです。

```
[default]
region = ap-northeast-1
endpoint_url = https://s3.v3.abci.ai
```

gaa00000.2プロファイルにエンドポイントURLを指定する例は次のとおりです。

```
[profile gaa00000.2]
region = ap-northeast-1
endpoint_url = https://s3.v3.abci.ai
```

!!! warning
    エンドポイントURLの指定がない場合は、Amazon S3にアクセスしますのでご注意ください。

## 各種操作

バケットの作成やデータのアップロードなどの基本操作について説明します。

### 基本事項

各種操作を行うにあたって、以下の基本事項を理解しておく必要があります。

AWS CLIの構文は次のとおりです。

```
aws [options] <command> <subcommand> [parameters]
```

例えば `aws s3 ls` は、`s3`が`<command>`で、`ls`は`<subcommand>` (s3 コマンドの ls コマンド、あるいは s3 ls コマンド)です。

s3 コマンドでは、オブジェクトのパスをS3 URIで表します。

例えば次のURLは、`bucket-1` がバケット名、`project-1/docs/fig1.png` がオブジェクトキー(またはキー名)です。`project-1/docs/` の部分はプレフィックスと呼ばれ、オブジェクトキーを疑似階層的に(いわゆるファイルシステムにおける"フォルダ"の下に入っているかのように)表現するために使われます。

```
s3://bucket-1/project-1/docs/fig1.png
```

バケット名は以下の規約があります。

* クラウドストレージ全体で一意である必要があります
* 3～63文字で定義する必要があります
* '\_'(アンダースコア)を含めることはできません
* 先頭は小文字の英数字を指定する必要があります
* IPアドレス(例えば192.168.0.1)のような形式は使用できません
* '.'(ピリオド)は問題になる場合があります。使用しないことを推奨します

オブジェクトキーには、UTF-8文字も使うことはできますが、アスキーコードの範囲であっても、避けた方が良い特殊文字があります。

* `-` (ハイフンマイナス)、`_` (アンダースコア)、`.`(ドット)は問題なく扱えます
* `!` `*` `'` `(` `)` (エクスクラメーションマーク、アスタリスク、アポストロフィ(0x27)、左括弧、右括弧)、これら5つは、例えばシェルにおいてはエスケープしたりクォートするなど正しく扱っているならば、問題ありません

上記以外の特殊文字(例えば `<` や `#` など)は、避けた方が安全です。

### バケットの作成

`aws s3 mb`コマンドでバケットを作成します。例として、`dataset-summer-2012` というバケットを作るには、以下のように aws コマンドを実行します。

```
[username@login1 ~]$ aws s3 mb s3://dataset-summer-2012
make_bucket: dataset-summer-2012
```

### バケットの一覧表示

所属するABCIグループ内で作成されたバケットの一覧をリストするには、`aws s3 ls` を実行します。

実行例
```
[username@login1 ~]$ aws s3 ls
2025-06-15 10:47:37 testbucket1
2025-06-15 18:10:37 testbucket2
```


### オブジェクトのリスト

バケット入っているオブジェクトをリストするには、 `aws s3 ls s3://bucket-name` を実行します。

```
[username@login1 ~]$ aws s3 ls s3://mybucket
                           PRE pics/
2025-06-05 17:33:05          4 test1.txt
2025-06-05 21:12:47          4 test2.txt
```

例えば `pics/` というプレフィックスを持つデータをリストするには、バケット名の後ろにプレフィックスをつけます。

```
[username@login1 ~]$ aws s3 ls s3://mybucket/pics/
2025-06-07 21:55:57    1048576 test3.png
2025-06-07 21:55:59    1048576 test4.png
```

`--recursive` オプションを使い、バケット内の全オブジェクトをリストすることもできます。

```
[username@login1 ~]$ aws s3 ls s3://mybucket --recursive
2025-06-05 17:33:05          4 test1.txt
2025-06-05 21:12:47          4 test2.txt
2025-06-07 21:55:57    1048576 pics/test3.png
2025-06-07 21:55:59    1048576 pics/test4.png
```

### データのコピー(アップロード、ダウンロード、コピー)

ファイルシステム上からクラウドストレージ上のバケットへ、クラウドストレージ上のバケットからファイルシステム上へ、またはクラウドストレージ上のバケットからクラウドストレージ上のバケットへデータをコピーできます。

`0001.jpg`ファイルを`dataset-c0541`バケットにコピー

```
[username@login1 ~]$ ls images
0001.jpg    0002.jpg    0003.jpg    0004.jpg
[username@login1 ~]$ aws s3 cp ./images/0001.jpg s3://dataset-c0541/
upload: images/0001.jpg to s3://dataset-c0541/0001.jpg
[username@login1 ~]$
```

`images`ディレクトリの中身を`dataset-c0542`バケットにコピー

```
[username@login1 ~]$ ls images
0001.jpg    0002.jpg    0003.jpg    0004.jpg
[username@login1 ~]$ aws s3 cp images s3://dataset-c0542/ --recursive
upload: images/0001.jpg to s3://dataset-c0542/0001.jpg
upload: images/0002.jpg to s3://dataset-c0542/0002.jpg
upload: images/0003.jpg to s3://dataset-c0542/0003.jpg
upload: images/0004.jpg to s3://dataset-c0542/0004.jpg
[username@login1 ~]$ aws s3 ls s3://dataet-c0542/
2025-06-10 19:03:19    1048576 0001.jpg
2025-06-10 19:03:19    1048576 0002.jpg
2025-06-10 19:03:19    1048576 0003.jpg
2025-06-10 19:03:19    1048576 0004.jpg
[username@login1 ~]$
```

`dataset-tmpl-c0000`バケットから`dataset-c0541`バケットへ`logo.png`ファイルをコピー

```
[username@login1 ~]$ aws s3 cp s3://dataset-tmpl-c0000/logo.png s3://dataset-c0541/logo.png
copy: s3://dataset-tmpl-c0000/logo.png to s3://dataset-c0541/logo.png
```

### データの移動

オブジェクトを移動するには`aws s3 mv`コマンドを使用します。バケット間の移動の他、ローカルからバケット、バケットからローカルへ移動を行えます。
タイムスタンプは保持されません。`--recursive` オプションの付与で特定のプレフィックスを持つオブジェクト、または特定のディレクトリに入っているファイルを対象として扱えます。

次の例では、カレントディレクトリにある`annotations.zip`ファイルをクラウドストレージ上の`dataset-c0541`バケットに移動を行っています。

```
[username@login1 ~]$ aws s3 mv annotations.zip s3://dataset-c0541/
move: ./annotations.zip to s3://dataset-c0541/annotations.zip
```

次の例は、`dataset-c0541`バケットの`sensor-1`プレフィックスをもつオブジェクトをまとめて`dataset-c0542`バケットに移動させています。

```
[username@login1 ~]$ aws s3 mv s3://dataset-c0541/sensor-1/ s3://dataset-c0542/sensor-1/ --recursive
move: s3://dataset-c0541/sensor-1/0001.dat to s3://dataset-c0542/sensor-1/0001.dat
move: s3://dataset-c0541/sensor-1/0003.dat to s3://dataset-c0542/sensor-1/0003.dat
move: s3://dataset-c0541/sensor-1/0004.dat to s3://dataset-c0542/sensor-1/0004.dat
move: s3://dataset-c0541/sensor-1/0002.dat to s3://dataset-c0542/sensor-1/0002.dat
```

### ローカルディレクトリとクラウドストレージの同期

以下の例では、カレントディレクトリにある`sensor2`というディレクトリと`mybucket`というバケットを同期させています。
`--delete`オプションをつけていなければバケットにあった既存のオブジェクトは削除されませんが、同名のものは上書きされます。
次に同じコマンドラインを実行すると、更新されたファイルのみ送ります。

```
[username@login1 ~]$ aws s3 sync ./sensor2 s3://mybucket/
upload: sensor2/0002.dat to s3://mybucket/0002.dat
upload: sensor2/0004.dat to s3://mybucket/0004.dat
upload: sensor2/0001.dat to s3://mybucket/0001.dat
upload: sensor2/0003.dat to s3://mybucket/0003.dat
```

`sensor3`バケットの`rev1`プレフィックスをもつオブジェクトを`testdata`ディレクトリに同期する例です。

```
[username@login1 ~]$ aws s3 sync s3://sensor3/rev1/ testdata
download: s3://sensor3/rev1/0001.zip to testdata/0001.zip
download: s3://sensor3/rev1/0004.zip to testdata/0004.zip
download: s3://sensor3/rev1/0003.zip to testdata/0003.zip
download: s3://sensor3/rev1/0002.zip to testdata/0002.zip
```

!!! note
    次回にこのコマンドラインを実行した際、バケットに入っているデータが更新されていても、サイズが全く同じ場合は再取得の対象となりません。その場合、`--exact-timestamps`オプションをつけることで対象とすることができますが、ABCI環境の場合はこのオプションを指定すると、すべてのオブジェクトが再取得の対象となります。


### オブジェクトの削除

オブジェクトの削除は`aws s3 rm <S3Uri> [parameters]`コマンドで行います。

実行例

```
[username@login1 ~]$ aws s3 rm s3://mybucket/readme.txt
delete: s3://mybucket/readme.txt
```

`--recursive`パラメータを使うことで、指定のプレフィックス以下のオブジェクトを削除できます。

```
[username@login1 ~]$ aws s3 ls s3://mybucket --recursive
2025-06-30 20:46:53         32 a.txt
2025-06-30 20:46:53         32 b.txt
2025-06-31 14:51:50        512 xml/c.xml
2025-06-31 14:51:54        512 xml/d.xml

[username@login1 ~]$ aws s3 rm s3://mybucket/xml --recursive
delete: s3://mybucket/xml/c.xml
delete: s3://mybucket/xml/d.xml

[username@login1 ~]$ aws s3 ls s3://mybucket --recursive
2025-06-30 20:46:53         32 a.txt
2025-06-30 20:46:53         32 b.txt
```


### バケットの削除

バケットの削除は`aws s3 rb <S3Uri> [parameters]`コマンドで行います。

`dataset-c0541` バケットを削除する例です。

```
[username@login1 ~]$ aws s3 rb s3://dataset-c0541
remove_bucket: dataset-c0541
```

空でないバケットを削除しようとするとエラーが返されますが、中身も全部消してしまって良ければ`--force`をつけると中身を消した上でバケットを削除してくれます。

```
[username@login1 ~]$ aws rb s3://dataset-c0542 --force
delete: s3://dataset-c0542/0001.jpg
delete: s3://dataset-c0542/0002.jpg
delete: s3://dataset-c0542/0003.jpg
delete: s3://dataset-c0542/0004.jpg
remove_bucket: dataset-c0542
```

### オブジェクトの所有者の確認

オブジェクトの所有者は、`aws s3api get-object-acl`コマンドで確認することができます。

以下の例のようにオブジェクトを格納するバケットを`mybucket`、オブジェクトの名前を`testdata`として、`Owner.ID`で示されるUUID(`3ce596167d2cc6ecc36c8beedea92029fa3642c9`)がオブジェクトを所有していることがわかります。

```
[username@login1 ~]$ aws s3api get-object-acl --bucket mybucket --key testdata
{
    "Owner": {
        "DisplayName": "3ce596167d2cc6ecc36c8beedea92029fa3642c9",
        "ID": "3ce596167d2cc6ecc36c8beedea92029fa3642c9"
    },
    "Grants": [
        {
            "Grantee": {
                "DisplayName": "3ce596167d2cc6ecc36c8beedea92029fa3642c9",
                "ID": "3ce596167d2cc6ecc36c8beedea92029fa3642c9",
                "Type": "CanonicalUser"
            },
            "Permission": "FULL_CONTROL"
        }
    ]
}
```

### マルチパートアップロード(MPU)について

AWS CLIは、データを自動的に分割して並行に送信することで、効率良くデータをアップロードする機能を備えています。
これはマルチパートアップロード (MPU) と呼ばれています。MPU が有効であるかどうかは、クライアントツールやその設定に依存します。

AWS CLIでは、MPU が有効となるデータサイズは、デフォルトで8MBと定義されています。

### マルチパートアップロードを手動で有効にする方法

AWS CLIでは、データサイズに応じてMPUが自動的に有効になりますが、以下の方法を用いることでMPUを有効にすることもできます。

まず、`split` コマンドなどでアプロードしたいデータを分割します。以下の例では、`15M_test.dat`を3つに分割しています。

```
[username@login1 ~]$ split -n 3 -d 15M_test.dat mpu_part-
total 3199056
-rw-r----- 1 username group   15728640 Nov 30 15:42 15M_test.dat
-rw-r----- 1 username group    5242880 Nov 30 15:51 mpu_part-02
-rw-r----- 1 username group    5242880 Nov 30 15:51 mpu_part-01
-rw-r----- 1 username group    5242880 Nov 30 15:51 mpu_part-00
[username@login1 ~]$
```

次に、`aws s3api create-multipart-upload`コマンドでアップロード先のバケットとパスを指定して、 MPU を開始します。
アップロード先は、バケットを`--bucket`オプション、パスを`--key`オプションで指定します。

以下の例は、バケット`testbucket-00`に`mpu-sample`というオブジェクトを作成します。
コマンドが成功すると、以下のように `UploadId` が発行されます。

```
[username@login1 ~]$ aws s3api create-multipart-upload --bucket testbucket-00 --key mpu-sample
{
    "Bucket": "testbucket-00",
    "Key": "mpu-sample",
    "UploadId": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
}
```

これらのデータを`aws s3api part-upload`コマンドでアップロードします。この時、上記の`UpLoadId`を指定してください。コマンドが成功すると表示される`ETag`は後で使用するため控えておいてください。

```
[username@login1 ~]$ aws s3api upload-part --bucket testbucket-00 --key mpu-sample --part-number 1 --body mpu_part-00 --upload-id aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
{
    "ETag": "\"sample1d8560e70ca076c897e0715024\""
}
```

同様に、順次`--part-number`の値と対応するデータを指定して残りのデータもアップロードします。

```
[username@login1 ~]$ aws s3api upload-part --bucket testbucket-00 --key mpu-sample --part-number 2 --body mpu_part-01 --upload-id aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
{
    "ETag": "\"samplee36a6ef6ae8f2c0ea3328c5e7c\""
}
[username@login1 ~]$ aws s3api upload-part --bucket testbucket-00 --key mpu-sample --part-number 3 --body mpu_part-02 --upload-id aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
{
    "ETag": "\"sample9e391d5673d2bfd8951367eb01\""
}
[username@login1 ~]$
```

全てのデータをアップロードしたら、`ETag`の値を以下のような`JSON`形式のファイルにまとめます。

```
[username@login1 ~]$ cat mpu_fileparts.json
{
    "Parts":[{
        "ETag": "sample1d8560e70ca076c897e0715024",
        "PartNumber": 1
    },
    {
        "ETag": "samplee36a6ef6ae8f2c0ea3328c5e7c",
        "PartNumber": 2
    },
    {
        "ETag": "sample9e391d5673d2bfd8951367eb01",
        "PartNumber": 3
    }]
}
```

最後に、`aws s3api complete-multipart-upload`コマンドを使用してMPUを完了させます。
`complete-multipart-upload`コマンドが成功すると、`--key`で指定したオブジェクトが作成されます。

```
[username@login1 ~]$ aws s3api complete-multipart-upload --multipart-upload file://mpu_fileparts.json --bucket testbucket-00 --key mpu-sample --upload-id aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
{
    "Location": "http://testbucket-00.s3.v3.abci.ai/mpu-sample",
    "Bucket": "testbucket-00",
    "Key": "mpu-sample",
    "ETag": "\"6203f5cdbecbe0556e2313691861cb99-3\""
}
```

オブジェクトが作成されていることを確認します。

```
[username@login1 ~]$ aws s3 ls s3://testbucket-00/
2020-12-01 09:28:03   15728640 mpu-sample
```

### MPU の中止 {#abort-mpu}

実行中の MPU を中止するには、まず MPU の一覧を表示し、`UploadId` と `Key` を確認します。MPU の一覧は、`aws s3api list-multipart-uploads`コマンドにアップロード時のバケットを指定して、確認することができます。MPUが残っていない場合は何も表示されません。

以下の例では、オブジェクト`data_10gib-1.dat`を `s3://mybucket/testdata/` にアップロードする途中であることが確認できます。`Key` にはバケットより下のパスとオブジェクト名が表示されています。

```
[username@login1 ~]$ aws s3api list-multipart-uploads --bucket mybucket
{
    "Uploads": [
        {
            "UploadId": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
            "Key": "testdata/data_10gib-1.dat",
            "Initiated": "2025-11-12T09:58:16.000Z",
            "StorageClass": "STANDARD",
            "Owner": {
                "DisplayName": "3ce596167d2cc6ecc36c8beedea92029fa3642c9",
                "ID": "3ce596167d2cc6ecc36c8beedea92029fa3642c9"
            },
            "Initiator": {
                "ID": "3ce596167d2cc6ecc36c8beedea92029fa3642c9",
                "DisplayName": "3ce596167d2cc6ecc36c8beedea92029fa3642c9"
            }
        }
    ]
}
```

次に、MPU の中止を実行します。以下に示すように MPU の中止は、`aws s3api abort-multipart-upload`コマンドの引数に、中止したい MPU の `UploadId` と `Key` を指定します。コマンドが成功すると、プロンプトが返ります。

```
[username@login1 ~]$ aws s3api abort-multipart-upload --bucket mybucket --key testdata/data_10gib-1.dat --upload-id aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
[username@login1 ~]$
```

これで MPU の中止は完了です。MPU の実行中にサーバ側に作成されたデータも削除されます。
