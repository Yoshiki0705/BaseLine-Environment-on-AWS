# BLEA Guest: FSx for ONTAP Modernization Platform

[日本語で読む](#日本語)

> A modular CDK sample that deploys Amazon FSx for ONTAP as shared storage with selectable compute patterns (EC2, Lambda, ECS, EKS, AWS Batch). Includes monitoring, auto-capacity-expansion, S3 Access Point integration, and AWS Backup data protection.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  VPC (Multi-AZ)                                                 │
│                                                                 │
│  ┌──────────────┐   NFS / S3 AP   ┌────────────────────────┐  │
│  │ FSx for ONTAP│◄────────────────►│  Compute Patterns      │  │
│  │  (SVM + Vol) │                  │  ┌─────┐ ┌──────────┐ │  │
│  └──────┬───────┘                  │  │ EC2 │ │ Lambda   │ │  │
│         │                          │  │ ASG │ │(S3 AP)   │ │  │
│         │                          │  └─────┘ └──────────┘ │  │
│         │                          │  ┌─────┐ ┌──────────┐ │  │
│         │                          │  │ ECS │ │ AWS Batch│ │  │
│         │                          │  │Farg.│ │ (NFS)    │ │  │
│         │                          │  └─────┘ └──────────┘ │  │
│         │                          │  ┌─────┐              │  │
│         │                          │  │ EKS │              │  │
│         │                          │  │Trid.│              │  │
│         │                          │  └─────┘              │  │
│         │                          └────────────────────────┘  │
│         │                                                       │
│  ┌──────▼───────┐  ┌─────────────┐  ┌───────────────────────┐ │
│  │  Monitoring  │  │Serverless   │  │   Data Protection     │ │
│  │  (CW+SNS+   │  │Ops (Lambda  │  │   (AWS Backup)        │ │
│  │   Chatbot)   │  │auto-expand) │  │                       │ │
│  └──────────────┘  └─────────────┘  └───────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Features

| Component | Description |
| --- | --- |
| **FSx for ONTAP** | Shared file system with NFS volume + S3 Access Point |
| **Compute (toggle-based)** | EC2 ASG (NFS), Lambda (S3 AP), ECS Fargate (S3 AP), EKS (Trident CSI), AWS Batch (NFS) |
| **Monitoring** | CloudWatch alarms on capacity utilization → SNS → Slack/Email |
| **Serverless Ops** | Lambda-based auto-capacity-expansion triggered by alarm |
| **Data Protection** | AWS Backup with configurable retention |
| **Encryption** | KMS CMK with key rotation enabled |

## Prerequisites

- BLEA governance base deployed (see [HowTo](../../doc/HowTo.md#deployment-overview))
- Node.js >= 18.0.0, npm >= 8.1.0

## Deploy

1. Edit `parameter.ts` — configure storage, compute toggles, and monitoring settings
2. Deploy:

```sh
cd usecases/blea-guest-fsxn-modernization-sample
npx aws-cdk bootstrap --profile prof_dev   # first time only
npx aws-cdk deploy --all --profile prof_dev
```

## Parameters

| Parameter | Description | Default (dev) |
| --- | --- | --- |
| `fsxnStorageCapacityGiB` | Total SSD storage capacity | 1024 |
| `fsxnThroughputCapacityMBps` | Throughput capacity (128/256/512/1024/2048/4096) | 128 |
| `fsxnDeploymentType` | `MULTI_AZ_1` or `SINGLE_AZ_1` | `SINGLE_AZ_1` |
| `enableEc2Pattern` | Deploy EC2 Auto Scaling Group with NFS mount | `true` |
| `enableLambdaPattern` | Deploy Lambda function with S3 AP access | `true` |
| `enableEcsPattern` | Deploy ECS Fargate service with S3 AP access | `false` |
| `enableEksPattern` | Deploy EKS cluster (Trident CSI ready) | `false` |
| `enableBatchPattern` | Deploy AWS Batch with NFS mount | `false` |
| `backupRetentionDays` | AWS Backup retention | 7 |
| `capacityAlarmThresholdPercent` | CloudWatch alarm threshold for auto-expand | 80 |
| `maxCapacityGiB` | Maximum capacity the auto-expand function will scale to | 2048 |

## License

MIT-0. See [LICENSE](../../LICENSE).

---

<a id="日本語"></a>

## 日本語

# BLEA ゲスト: FSx for ONTAP モダナイゼーションプラットフォーム

> Amazon FSx for ONTAP を共有ストレージとして、選択可能なコンピュートパターン（EC2、Lambda、ECS、EKS、AWS Batch）とともにデプロイするモジュラー CDK サンプル。監視、自動容量拡張、S3 Access Point 統合、AWS Backup によるデータ保護を含みます。

### 機能

| コンポーネント | 概要 |
| --- | --- |
| **FSx for ONTAP** | NFS ボリューム + S3 Access Point による共有ファイルシステム |
| **コンピュート（トグル式）** | EC2 ASG (NFS)、Lambda (S3 AP)、ECS Fargate (S3 AP)、EKS (Trident CSI)、AWS Batch (NFS) |
| **監視** | CloudWatch アラーム（容量使用率）→ SNS → Slack/Email |
| **Serverless Ops** | アラームトリガーの Lambda による自動容量拡張 |
| **データ保護** | AWS Backup（保持期間設定可能） |
| **暗号化** | KMS CMK（キーローテーション有効） |

### 前提条件

- BLEA ガバナンスベースがデプロイ済み（[HowTo](../../doc/HowTo_ja.md#デプロイ概要) 参照）
- Node.js >= 18.0.0、npm >= 8.1.0

### デプロイ

1. `parameter.ts` を編集 — ストレージ、コンピュートトグル、監視設定を構成
2. デプロイ:

```sh
cd usecases/blea-guest-fsxn-modernization-sample
npx aws-cdk bootstrap --profile prof_dev   # 初回のみ
npx aws-cdk deploy --all --profile prof_dev
```

### パラメータ

| パラメータ | 説明 | デフォルト (dev) |
| --- | --- | --- |
| `fsxnStorageCapacityGiB` | 合計 SSD ストレージ容量 | 1024 |
| `fsxnThroughputCapacityMBps` | スループット容量 (128/256/512/1024/2048/4096) | 128 |
| `fsxnDeploymentType` | `MULTI_AZ_1` or `SINGLE_AZ_1` | `SINGLE_AZ_1` |
| `enableEc2Pattern` | NFS マウント付き EC2 Auto Scaling Group をデプロイ | `true` |
| `enableLambdaPattern` | S3 AP アクセス付き Lambda をデプロイ | `true` |
| `enableEcsPattern` | S3 AP アクセス付き ECS Fargate をデプロイ | `false` |
| `enableEksPattern` | EKS クラスター（Trident CSI 対応）をデプロイ | `false` |
| `enableBatchPattern` | NFS マウント付き AWS Batch をデプロイ | `false` |
| `backupRetentionDays` | AWS Backup 保持期間 | 7 |
| `capacityAlarmThresholdPercent` | 自動拡張の CloudWatch アラームしきい値 | 80 |
| `maxCapacityGiB` | 自動拡張がスケールする最大容量 | 2048 |

### License

MIT-0. [LICENSE](../../LICENSE) を参照。
