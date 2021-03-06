
# アカウントとアクセスキー

## クラウドストレージアカウント

ABCI利用者には、ABCIのグループ毎にABCIクラウドストレージ用のアカウントが発行されます。一方、ABCIグループの利用管理者には、利用者用とは別に、管理者用のクラウドストレージアカウントが1つ発行されます。

### 利用者用のクラウドストレージアカウント

ABCIクラウドストレージの一般的な利用（データのアップロードやダウンロード等）に用いてるアカウントです。例えば、aaa00000aa.1 のアカウント名が付与されます。

複数のグループに所属し、他のグループでもABCIクラウドストレージを利用する場合には、aaa00000aa.2 といった別のクラウドストレージアカウントを持つことになります。

多くの場合、１つのグループにおいて１つのクラウドストレージアカウントがあれば十分ですが、必要に応じて、同じグループでも複数のクラウドストレージアカウントを持つこともできます。例えば、開発中のアプリケーション用に aaa00000aa.3 を追加で持つことができます。ただし、名前の指定はできません。

各利用者が持つことのできるアカウントは、所属する1グループあたり最大10個までです。もし、2つのグループに所属していればそれぞれ10個、合計20個のクラウドストレージアカウントを持つことができます。

### 管理者用のクラウドストレージアカウント

管理者用のクラウドストレージアカウントには、一般用のクラウドストレージアカウントでは実行できないアクセス制御を行う権限が付与されています。具体的には、[アクセス制御(2)](policy.md)をご参照ください。　

管理者用のクラウドストレージアカウントでも、データのアップロードやダウンロードなど、利用者用のクラウドストレージアカウントで行える操作が可能ですが、原則、利用者用のアカウントを用いるようにしてください。

管理者用のクラウドストレージアカウントは、利用管理者に１人１つ発行されます。複数のグループの利用管理者となっている場合は、それぞれのグループにおいて１つ発行されます。

## アクセスキー

各クラウドストレージアカウントにはアクセスキーが設定されます。アクセスキーは、アクセスキー ID とシークレットアクセスキーのペアから成ります。シークレットアクセスキーは、第三者に見せたり、第三者から見えるところに置いてはいけません。

１つのクラウドストレージアカウントにはアクセスキーを2つまで設定することができます。複数の異なるクライアントを用いる場合には、個別のアクセスキーを設定することを推奨します。

## クラウドストレージアカウントの削除

ABCI利用者ポータルは、現在のところクラウドストレージアカウントの削除を行えません。アクセスキーを削除することで、そのクラウドストレージアカウントをアクセスできない状態にできますが、削除を希望される場合は [お問い合わせ](../contact.md) の内容をご確認の上、<qa@abci.ai> までクラウドストレージアカウント名をお知らせください。

!!! note
    - 削除したアカウントは元に戻せません。アカウント名に間違いがないか確認してください。
    - 削除したアカウントの番号は欠番になります。1グループあたり10アカウント作成できますので、1グループのみに所属していてアカウントを1つ削除した場合、最後に作られるアカウントの番号は 10 ではなく 11 (例: "aaa00000aa.11") です。
