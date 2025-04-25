# AWSの基本的なインフラストラクチャ構築

## VPCの構築

VPC（Virtual Private Cloud）は、AWSクラウド内の仮想ネットワークです。VPCを使用することで、AWSリソースを論理的に分離し、セキュアな環境を構築できます。

### VPCの基本構成

```mermaid
graph TD
    A[VPC] --> B[パブリックサブネット]
    A --> C[プライベートサブネット]
    
    B --> D[インターネットゲートウェイ]
    C --> E[NATゲートウェイ]
    
    B --> F[ルートテーブル]
    C --> G[ルートテーブル]
    
    F --> D
    G --> E
    E --> D
```

### VPCの各コンポーネントの役割

1. **VPC（Virtual Private Cloud）**
   - AWSクラウド内の論理的に分離された仮想ネットワーク
   - 独自のIPアドレス範囲（CIDRブロック）を持つ
   - セキュリティグループやネットワークACLでトラフィックを制御
   - 複数のアベイラビリティゾーンにまたがって使用可能

2. **サブネット**
   - VPC内のIPアドレス範囲を分割した論理的なセグメント
   - アベイラビリティゾーンごとに作成する
   - パブリックサブネットとプライベートサブネットの2種類がある

3. **パブリックサブネット**
   - インターネットから直接アクセス可能
   - インターネットゲートウェイへのルートを持つ
   - Webサーバーやロードバランサーなど、外部からアクセスが必要なリソースを配置
   - セキュリティグループで適切なアクセス制御が必要

4. **プライベートサブネット**
   - インターネットから直接アクセス不可
   - NATゲートウェイを経由してインターネットにアクセス
   - データベースサーバーやアプリケーションサーバーなど、内部のみで使用するリソースを配置
   - セキュリティが高い

5. **インターネットゲートウェイ**
   - VPCとインターネットの間のゲートウェイ
   - パブリックサブネットのリソースがインターネットと通信するために必要
   - 双方向の通信を可能にする

6. **NATゲートウェイ**
   - プライベートサブネットのリソースがインターネットと通信するためのゲートウェイ
   - プライベートサブネットからインターネットへの通信のみを許可
   - インターネットからプライベートサブネットへの通信は不可
   - パブリックサブネットに配置する必要がある

7. **ルートテーブル**
   - サブネット内のトラフィックのルーティングを制御
   - パブリックサブネット用とプライベートサブネット用で異なる設定
   - パブリックサブネット：インターネットゲートウェイへのルート
   - プライベートサブネット：NATゲートウェイへのルート

8. **セキュリティグループ**
   - インスタンスレベルでのファイアウォール
   - インバウンド（受信）とアウトバウンド（送信）のトラフィックを制御
   - プロトコル、ポート、ソース/デスティネーションIPアドレスで制御

9. **ネットワークACL**
   - サブネットレベルでのファイアウォール
   - インバウンドとアウトバウンドのトラフィックを制御
   - ステートレス（接続状態を保持しない）
   - 複数のサブネットに適用可能

### VPCの作成

```hcl
# vpc.tf
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "main-vpc"
  }
}

# パブリックサブネット
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-northeast-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet"
  }
}

# プライベートサブネット
resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "ap-northeast-1a"

  tags = {
    Name = "private-subnet"
  }
}

# インターネットゲートウェイ
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}

# NATゲートウェイ
resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public.id

  tags = {
    Name = "main-nat"
  }
}

# Elastic IP for NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
}

# パブリックルートテーブル
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "public-rt"
  }
}

# プライベートルートテーブル
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }

  tags = {
    Name = "private-rt"
  }
}
```

## EC2インスタンスの作成

EC2（Elastic Compute Cloud）は、AWSの仮想サーバーサービスです。

### EC2の基本概念

1. **インスタンスタイプ**
   - インスタンスのCPU、メモリ、ストレージ、ネットワーク性能を定義
   - 主要なファミリー：
     - 汎用（t3, m5）：バランスの取れたリソース配分
     - コンピューティング最適化（c5）：CPU集中型ワークロード
     - メモリ最適化（r5）：メモリ集中型ワークロード
     - ストレージ最適化（i3）：高I/Oワークロード
     - GPUインスタンス（g4）：機械学習やグラフィックス処理

2. **AMI（Amazon Machine Image）**
   - インスタンスの起動に必要な情報を含むテンプレート
   - オペレーティングシステム、アプリケーションサーバー、アプリケーションを含む
   - 種類：
     - Amazon提供のAMI
     - AWS MarketplaceのAMI
     - カスタムAMI
   - リージョンごとに異なるAMI ID

3. **ストレージオプション**
   - インスタンスストア
     - 物理的にインスタンスに接続されたストレージ
     - インスタンス終了時にデータが失われる
   - EBS（Elastic Block Store）
     - ネットワーク接続型のブロックストレージ
     - 永続的なデータ保存
     - スナップショットによるバックアップ可能

4. **セキュリティグループ**
   - インスタンスのファイアウォール
   - インバウンド/アウトバウンドトラフィックの制御
   - プロトコル、ポート、IPアドレス範囲で制御

5. **キーペア**
   - SSH接続の認証に使用
   - 公開鍵と秘密鍵のペア
   - インスタンス起動時に指定
   - 秘密鍵は安全に保管

### EC2インスタンスの基本構成

```mermaid
graph TD
    A[EC2インスタンス] --> B[セキュリティグループ]
    A --> C[キーペア]
    A --> D[AMI]
    A --> E[インスタンスタイプ]
    A --> F[ストレージ]
    
    F --> F1[インスタンスストア]
    F --> F2[EBS]
    
    E --> E1[汎用]
    E --> E2[コンピューティング最適化]
    E --> E3[メモリ最適化]
    E --> E4[ストレージ最適化]
    E --> E5[GPU]
```

### EC2インスタンスの作成

```hcl
# ec2.tf
resource "aws_instance" "web" {
  ami           = "ami-0c3fd0f5d33134a76"  # Amazon Linux 2
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public.id

  vpc_security_group_ids = [aws_security_group.web.id]
  key_name              = aws_key_pair.main.key_name

  tags = {
    Name = "web-server"
  }
}

# セキュリティグループ
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Security group for web server"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}

# キーペア
resource "aws_key_pair" "main" {
  key_name   = "main-key"
  public_key = file("~/.ssh/id_rsa.pub")
}
```

## RDSの作成

RDS（Relational Database Service）は、AWSのマネージドデータベースサービスです。

### RDSの基本概念

1. **データベースエンジン**
   - MySQL
   - PostgreSQL
   - MariaDB
   - Oracle
   - SQL Server
   - Amazon Aurora

2. **インスタンスクラス**
   - 汎用（db.t3, db.m5）
   - メモリ最適化（db.r5）
   - バースト可能（db.t3）
   - 用途に応じて適切なサイズを選択

3. **ストレージタイプ**
   - 汎用SSD（gp2）
   - プロビジョンドIOPS（io1）
   - マグネティック
   - ストレージの自動拡張設定可能

4. **マルチAZ配置**
   - 高可用性のための機能
   - プライマリとスタンバイの2つのインスタンス
   - 自動フェイルオーバー
   - データの同期レプリケーション

5. **バックアップとスナップショット**
   - 自動バックアップ
   - 手動スナップショット
   - バックアップの保持期間設定
   - クロスリージョンコピー

6. **セキュリティ機能**
   - 暗号化（保存時と転送時）
   - VPC内での実行
   - セキュリティグループによるアクセス制御
   - IAM認証

### RDSの基本構成

```mermaid
graph TD
    A[RDSインスタンス] --> B[サブネットグループ]
    A --> C[パラメータグループ]
    A --> D[セキュリティグループ]
    A --> E[バックアップ設定]
    A --> F[マルチAZ]
    
    B --> B1[プライベートサブネット]
    B --> B2[プライベートサブネット]
    
    C --> C1[データベースパラメータ]
    C --> C2[クラスターパラメータ]
    
    D --> D1[インバウンドルール]
    D --> D2[アウトバウンドルール]
    
    E --> E1[自動バックアップ]
    E --> E2[手動スナップショット]
    
    F --> F1[プライマリ]
    F --> F2[スタンバイ]
```

### セキュリティグループの詳細

1. **セキュリティグループの基本**
   - インスタンスレベルでのファイアウォール
   - ステートフル（接続状態を保持）
   - インバウンドとアウトバウンドのルール
   - 複数のセキュリティグループを適用可能

2. **ルールの設定**
   - プロトコル（TCP, UDP, ICMP）
   - ポート範囲
   - ソース/デスティネーション
   - カスタムIPアドレス範囲
   - 他のセキュリティグループ

3. **ベストプラクティス**
   - 最小権限の原則
   - 特定のIPアドレス範囲からのアクセス制限
   - 必要なポートのみ開放
   - 定期的な見直しと更新

4. **セキュリティグループの種類**
   - Webサーバー用
   - アプリケーションサーバー用
   - データベース用
   - 管理用

### セキュリティグループの構成

```mermaid
graph TD
    A[セキュリティグループ] --> B[インバウンドルール]
    A --> C[アウトバウンドルール]
    
    B --> B1[HTTP/HTTPS]
    B --> B2[SSH]
    B --> B3[カスタムTCP]
    
    C --> C1[すべてのトラフィック]
    C --> C2[特定のポート]
    
    B1 --> D1[ポート80/443]
    B2 --> D2[ポート22]
    B3 --> D3[カスタムポート]
```

### RDSインスタンスの作成

```hcl
# rds.tf
resource "aws_db_subnet_group" "main" {
  name       = "main-db-subnet-group"
  subnet_ids = [aws_subnet.private.id]

  tags = {
    Name = "main-db-subnet-group"
  }
}

resource "aws_security_group" "db" {
  name        = "db-sg"
  description = "Security group for RDS"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.web.id]
  }

  tags = {
    Name = "db-sg"
  }
}

resource "aws_db_instance" "main" {
  identifier           = "main-db"
  engine              = "mysql"
  engine_version      = "8.0"
  instance_class      = "db.t3.micro"
  allocated_storage   = 20
  storage_type        = "gp2"
  
  db_name             = "mydb"
  username            = "admin"
  password            = "your-password"  # 本番環境では変数を使用
  
  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  skip_final_snapshot    = true

  tags = {
    Name = "main-db"
  }
}
```

## インフラストラクチャのデプロイ

1. **Terraformの初期化**

```bash
terraform init
```

2. **実行計画の確認**

```bash
terraform plan
```

3. **インフラの作成**

```bash
terraform apply
```

4. **インフラの削除**

```bash
terraform destroy
```

## セキュリティのベストプラクティス

1. **ネットワークセキュリティ**
   - 最小限のポート開放
   - セキュリティグループの適切な設定
   - プライベートサブネットの活用

2. **データベースセキュリティ**
   - 強力なパスワードポリシー
   - 暗号化の有効化
   - バックアップの設定

3. **インスタンスセキュリティ**
   - 最新のAMIの使用
   - セキュリティパッチの適用
   - キーペアの適切な管理

## 次のステップ

次の章では、より高度なAWSサービスの使用方法について学びます。ロードバランサー、Auto Scaling、CloudFrontなどのサービスを使用して、スケーラブルで高可用性なインフラストラクチャを構築する方法を解説します。
