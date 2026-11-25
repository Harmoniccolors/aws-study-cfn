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

# AWS Infrastructure Practice (CloudFormation)

現場の業務課題をテクノロジーで解決するクラウドエンジニア・ソリューションアーキテクト候補を目指し、AWS CloudFormation（YAML）を用いたInfrastructure as Code（IaC）の構築・検証記録をまとめています。

## 🎯 目的・学習方針
- **コードによる再現性の確保:** 手作業（AWSマネジメントコンソール）での構築にとどまらず、CloudFormationテンプレート化による環境の再現・変更管理を実践
- **セキュリティ・運用の意識:** 最小権限の原則に基づくアクセス制御や、ログ出力・エラーハンドリングなど実務を意識した設定の検証
- **自走力とナレッジの言語化:** 構築時に発生したエラーやロールバックの原因と対策を `notes.md` に記録し、再現可能な知見として蓄積

## 🛠 主な使用技術・サービス
- **IaC:** AWS CloudFormation (YAML)
- **Compute / Network / Storage:** Amazon VPC, Subnet, Route Table, Security Group, EC2, ALB, Amazon S3, RDS
- **Security / Management:** AWS IAM, CloudWatch Logs, AWS CLI / CloudShell

## 📐 構築・検証テーマ一覧

### 1. ALBアクセスログのS3出力とバケットポリシー設計
- **概要:** Application Load Balancer（ALB）のアクセスログをAmazon S3バケットへ自動出力する構成をCloudFormationで構築
- **設計・実装のポイント:** 最小権限の原則に基づき、ALBからの書き込みのみを許可する `AWS::S3::BucketPolicy` を設計
- **ハマりポイント・学び:** リージョン固有のプリンシパル／ELBアカウント指定やリソースARN（`/*`）の記述不備によるデプロイエラーを経験し、CloudFormationイベントログからの原因切り分けと権限設計を検証
- **関連ファイル:**
  - 📄 テンプレート: [`alb-s3-logging/alb-s3-logging.yaml`](./alb-s3-logging/alb-s3-logging.yaml)
  - 📝 検証メモ・トラブルシューティング記録: [`alb-s3-logging/notes.md`](./alb-s3-logging/notes.md)

### 2. 基本ネットワーク・Webサーバ基盤の構築
- **概要:** VPC、パブリック／プライベートサブネット、インターネットゲートウェイ、ルートテーブル、セキュリティグループ、EC2・RDSの基盤構成をコード化
- **設計・実装のポイント:** `Parameters` と `Outputs` を活用し、環境ごとの再利用性とスタック間のリソース参照を意識した設計を実装
- **関連ファイル:**
  - 📄 テンプレート: [`network/vpc-base.yaml`](./network/vpc-base.yaml) / [`compute-db/web-app-stack.yaml`](./compute-db/web-app-stack.yaml)
  - 📝 検証メモ: [`network/notes.md`](./network/notes.md) *(※順次追加)*

### 3. スタック更新・エラーハンドリング・ロールバック検証
- **概要:** パラメータ変更時の挙動確認や、デプロイ失敗時のロールバック（`ROLLBACK_COMPLETE` / `UPDATE_ROLLBACK_COMPLETE`）の発生条件と復旧手順を検証
- **設計・実装のポイント:** 依存関係（`DependsOn`）やリソース削除時の順序性を考慮したテンプレート記述を徹底

## 📂 リポジトリ構成
```text
.
├── README.md                     # 本ドキュメント（全体概要・検証テーマ一覧）
├── alb-s3-logging/
│   ├── alb-s3-logging.yaml       # ALB + S3アクセスログ出力用テンプレート
│   └── notes.md                  # 設計意図・検証手順・エラー解決記録
├── network/
│   └── vpc-base.yaml             # VPC・サブネット・ルーティング基盤
└── compute-db/
    └── web-app-stack.yaml        # Web・DBサーバ構築用テンプレート
```
*(※実際のフォルダ名・ファイル名に合わせてパス部分を書き換えてください)*

## 💡 学習を通じて得た気づき（現場経験との接続）
これまでの小売・製造・物流現場での業務改善やプロジェクト推進の経験から、「仕組みやルールを属人化させず、誰でも再現できる形に落とし込むこと」の重要性を実感してきました。
CloudFormationによるインフラのコード化は、現場改善における「標準化・仕組み化」と本質的に同じ考え方だと捉えており、今後はビジネス要件と技術構成をつなぐ設計力・提案力をさらに高めていきます。
