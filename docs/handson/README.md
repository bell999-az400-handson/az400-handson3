# AZ-400 試験対策 1日ハンズオンコース

## 📋 概要
このハンズオンコースは、AZ-400（Designing and Implementing Microsoft DevOps Solutions）試験対策として、模擬試験の弱点分析に基づいて設計された実践的な学習プログラムです。

## 🎯 学習目標
- GitHub と Azure DevOps の連携を理解する
- Azure Pipelines のビルド・リリース戦略を実装できる
- Azure Artifacts でパッケージ管理を実践する
- セキュリティとコンプライアンスの実装方法を習得する
- 監視とフィードバックループを構築できる

## 📚 AZ-400 試験範囲（最新シラバス）
1. **Configure processes and communications (10-15%)**
2. **Design and implement source control (15-20%)**
3. **Design and implement build and release pipelines (40-45%)**
4. **Develop a security and compliance plan (10-15%)**
5. **Implement an instrumentation strategy (10-15%)**

## ⏱️ タイムテーブル（1日コース）

### 午前の部（9:00-12:00）
| 時間 | 内容 | 所要時間 |
|------|------|----------|
| 9:00-9:30 | [環境準備](./00-環境準備.md) | 30分 |
| 9:30-10:30 | [Lab 1: GitHub ↔ Azure Boards 連携](./01-GitHub-AzureBoards連携.md) | 60分 |
| 10:30-10:45 | 休憩 | 15分 |
| 10:45-12:00 | [Lab 2: Azure Pipelines 基礎](./02-Azure-Pipelines-基礎.md) | 75分 |

**ランチ休憩（12:00-13:00）**

### 午後の部（13:00-17:00）
| 時間 | 内容 | 所要時間 |
|------|------|----------|
| 13:00-14:00 | [Lab 3: Azure Artifacts パッケージ管理](./03-Azure-Artifacts-パッケージ管理.md) | 60分 |
| 14:00-15:00 | [Lab 4: セキュリティとコンプライアンス](./04-セキュリティとコンプライアンス.md) | 60分 |
| 15:00-15:15 | 休憩 | 15分 |
| 15:15-16:15 | [Lab 5: 監視とフィードバック](./05-監視とフィードバック.md) | 60分 |
| 16:15-17:00 | [Lab 6: 総合演習](./06-総合演習.md) | 45分 |

## 🛠️ 必要な環境
- Azure サブスクリプション（無料試用版可）
- Azure DevOps Organization（無料）
- GitHub アカウント（無料）
- Visual Studio Code
- Git クライアント
- .NET SDK 8.0 以上（Lab 3用）
- Azure CLI

## 📖 各Labの概要

### Lab 1: GitHub ↔ Azure Boards 連携
- AB# の動作仕様（title/description での認識）
- Pull Request と Work Item の連携
- GitHub Notifications 設定

### Lab 2: Azure Pipelines 基礎
- YAML パイプラインの作成
- Cache タスクと Artifacts の違い
- Branch Policy の設定
- System.Debug による詳細ログ

### Lab 3: Azure Artifacts パッケージ管理
- Feed の作成と管理
- Views（dev/prerelease/release）の活用
- NuGet パッケージの発行と利用

### Lab 4: セキュリティとコンプライアンス
- Service Principal 認証
- Azure AD PIM（Privileged Identity Management）
- Branch Policy による品質ゲート
- SonarCloud 連携

### Lab 5: 監視とフィードバック
- Azure Monitor の設定
- Logic App による Teams 通知
- Application Insights の実装
- ダッシュボード（Lead Time/Velocity/Cycle Time）

### Lab 6: 総合演習
- エンドツーエンドのCI/CDパイプライン構築
- 学習内容の統合と確認

## 📝 模擬試験からの重要ポイント

### 頻出・重要トピック
1. **GitHub 連携**
   - AB# が認識されるのは title と description のみ
   - comment や label では Work Item にリンクされない

2. **Azure Pipelines**
   - Cache タスクは依存関係のキャッシュに使用
   - Pipeline Artifacts はビルド成果物の保存に使用
   - System.Debug = true で詳細ログを有効化

3. **Azure Artifacts**
   - BACPAC: schema + data
   - DACPAC: schema only

4. **セキュリティ**
   - Service Principal 認証に必要な3つの値:
     - Application ID
     - Client Secret
     - Tenant ID
   - Azure AD PIM は Premium P2 ライセンスが必要

5. **監視**
   - Lead Time: 完了までの総時間（復旧時間・MTTR）
   - Cycle Time: 作業開始から完了まで
   - Velocity: 完了したバックログ数

## 🎓 学習のコツ
1. 各Labを順番に実施してください
2. 実際に手を動かして操作することが重要です
3. エラーが発生した場合は、ログを確認して原因を特定してください
4. 各Labの最後にある「確認問題」で理解度をチェックしてください
5. 分からないことは公式ドキュメントで確認してください

## 📚 参考リンク
- [AZ-400 試験公式ページ](https://learn.microsoft.com/certifications/exams/az-400)
- [Azure DevOps ドキュメント](https://learn.microsoft.com/azure/devops/)
- [GitHub Actions ドキュメント](https://docs.github.com/actions)
- [Azure Pipelines ドキュメント](https://learn.microsoft.com/azure/devops/pipelines/)

## ✅ 完了チェックリスト
- [ ] 環境準備完了
- [ ] Lab 1: GitHub ↔ Azure Boards 連携 完了
- [ ] Lab 2: Azure Pipelines 基礎 完了
- [ ] Lab 3: Azure Artifacts パッケージ管理 完了
- [ ] Lab 4: セキュリティとコンプライアンス 完了
- [ ] Lab 5: 監視とフィードバック 完了
- [ ] Lab 6: 総合演習 完了

---

**Good luck with your AZ-400 preparation! 🚀**
