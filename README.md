# AWS Study Infrastructure (CloudFormation)

VPC、ALB、EC2（Amazon Linux 2023）、RDS（MySQL 8.0）で構成される3層Webアプリケーション基盤のCloudFormationテンプレートです。

## 前提条件 (Prerequisites)

スタックをデプロイする前に、対象リージョン（`ap-northeast-1`）で以下のリソースが作成済みである必要があります。

1. **EC2 キーペアの作成**
   - EC2コンソール ＞「キーペア」から事前に作成してください（例: `awsstudyv2`）。
   - **注意:** 存在しないキーペア名を指定すると、CloudFormationのスタック作成は `ROLLBACK`（失敗）します。

## パラメータ一覧

| パラメータ名 | デフォルト値 | 説明 |
| :--- | :--- | :--- |
| `KeyName` | *(必須選択)* | 事前作成したEC2キーペア名。コンソール上ではプルダウンから選択可能。 |
| `EC2InstanceType` | `t2.micro` | EC2のインスタンスタイプ（`t2.micro` / `t3.micro` / `t3.small`）。 |
| `DBMasterUsername` | `admin` | RDS MySQLのマスターユーザー名。 |
| `DBMasterUserPassword` | *(必須入力)* | RDS MySQLのマスターパスワード（8〜41文字）。 |

## 出力値 (Outputs)

デプロイ完了後、CloudFormationの「出力」タブから以下の値を取得できます。次課題のアプリケーション設定や動作確認に使用します。

- **`ALBEndpointURL`**: ブラウザからアプリへアクセスするためのURL（`http://<ALB-DNS>`）
- **`ALBDNSName`**: ALBのDNS名（Route 53のエイリアスレコード設定等に利用）
- **`RDSEndpointAddress`**: アプリからDBへ接続するためのホスト名
- **`RDSEndpointPort`**: DBの接続ポート番号（`3306`）
- **`EC2PublicIP`**: サーバー作業用のEC2パブリックIP

## デプロイ後の確認手順

1. **スタック完了の確認**: ステータスが `CREATE_COMPLETE` になることを確認（RDS構築のため約5〜10分かかります）。
2. **EC2へのログインとアプリ起動**: `EC2PublicIP` へ接続し、8080番ポートでWebアプリケーションを起動します。（※セキュリティグループで22番ポートを閉じているため、作業時は一時的に自分のグローバルIPのみ22番を許可するか、SSM Session Managerを利用してください）
3. **ヘルスチェック確認**: EC2ターゲットグループのステータスが `Healthy` に変わった後、`ALBEndpointURL` にアクセスして画面表示を確認します。
