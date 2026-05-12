# Lab 1: GitHub ↔ Azure Boards 連携

## 🎯 目的
このLabでは、GitHub と Azure Boards の連携方法を学び、特に AB# による Work Item リンクの動作仕様を理解します。

## ⏱️ 所要時間
約60分

## 📋 前提条件
- Lab 0 の環境準備が完了していること
- GitHub アカウント
- Azure DevOps Organization とプロジェクト

## 🎓 学習内容

### 重要ポイント（試験頻出）
✅ **AB# が認識される場所**
- ✔️ Pull Request の **title**
- ✔️ Pull Request の **description**
- ❌ Pull Request の **comment**（認識されない）
- ❌ Issue の **label**（認識されない）

この違いを実際に試して理解することが重要です！

## 📝 演習内容

### Exercise 1: GitHub リポジトリの作成

#### 1.1 GitHub Organization にリポジトリを作成

**方法1: Web ブラウザで作成**

1. [GitHub](https://github.com/) にログイン
2. 左上の組織メニューから「bell999-az400-handson」を選択
3. 「Repositories」タブをクリック
4. 「New repository」をクリック
5. 以下を入力：
   - Repository name: `az400-handson3`
   - Description: `AZ-400 Lab 1: Azure Boards Integration`
   - Visibility: Public または Private（どちらでも可）
   - ⚠️ **Initialize with README のチェックは外す**（既存のローカルリポジトリを使用するため）
6. 「Create repository」をクリック
7. リポジトリ URL をメモ（例: `https://github.com/bell999-az400-handson/az400-handson3.git`）

**方法2: GitHub CLI (gh) で作成（推奨）**

```powershell
# GitHub CLIでログイン（初回のみ）
gh auth login

# Organization配下にパブリックリポジトリを作成
gh repo create bell999-az400-handson/az400-handson3 `
  --description "AZ-400 Lab 1: Azure Boards Integration" `
  --public

# または、プライベートリポジトリとして作成する場合
gh repo create bell999-az400-handson/az400-handson3 `
  --description "AZ-400 Lab 1: Azure Boards Integration" `
  --private

# リポジトリが作成されたことを確認
gh repo view bell999-az400-handson/az400-handson3
```

**GitHub CLI のインストール（未インストールの場合）**
```powershell
# Windows (winget を使用)
winget install --id GitHub.cli

# または Chocolatey を使用
choco install gh

# インストール確認
gh --version
```

#### 1.2 ローカルリポジトリをリモートにプッシュ

```powershell
# 既存のローカルリポジトリに移動
Set-Location c:\Users\bell9\github\az400-handson3

# リモートリポジトリを追加
git remote add origin https://github.com/bell999-az400-handson/az400-handson3.git

# リモートリポジトリの確認
git remote -v

# 既存の変更をコミット（必要に応じて）
git add .
git commit -m "Initial commit for AZ-400 Lab 1"

# リモートリポジトリにプッシュ
git push -u origin main

# プッシュが成功したことを確認
# ブラウザで https://github.com/bell999-az400-handson/az400-handson3 にアクセス
```

#### 1.3 デフォルトブランチをmainに変更（重要）

GitHubのベストプラクティスとして、デフォルトブランチ名を `master` から `main` に変更します。

**ローカルとリモートのブランチをmainに統一:**

```powershell
# 現在のブランチ状態を確認
git branch -a

# ローカルのmasterブランチをmainにリネーム
git branch -m master main

# リモートにmainブランチをプッシュ
git push -u origin main

# GitHubのデフォルトブランチをmainに変更（GitHub CLI使用）
gh repo edit bell999-az400-handson/az400-handson3 --default-branch main

# リモートのmasterブランチを削除
git push origin --delete master

# リモートHEADポインタを更新
git remote set-head origin -a

# 変更を確認
git branch -a
# 出力: * main
#       remotes/origin/main
```

**💡 ポイント:**
- `git branch -m` でローカルブランチ名を変更
- `gh repo edit --default-branch` でGitHubのデフォルトブランチを変更
- 古いmasterブランチは削除して混乱を避ける
- すべてのコマンドはローカルとリモート両方を同期する

**⚠️ 注意:**
既にリモートリポジトリにmainブランチが存在する場合は、この手順は不要です。新規作成時や既存のmasterブランチからの移行時のみ実行してください。

### Exercise 2: Azure Boards の準備

#### 2.1 Work Item の作成

1. [Azure DevOps](https://dev.azure.com/) にアクセス
2. Organization → `az400-handson3` プロジェクトを開く
3. 左メニュー「Boards」→「Work items」をクリック
4. 「+ New Work Item」→「User Story」を選択
5. 以下を入力：
   - Title: `Implement login feature`
   - Description: `ユーザー認証機能を実装する`
   - State: New
6. 「Save & Close」をクリック
7. **Work Item ID をメモ**（例: #123）

#### 2.2 追加の Work Item を作成

同様に以下の Work Item を作成：
- User Story: `Add unit tests` (#124)
- Bug: `Fix login button alignment` (#125)

### Exercise 3: Azure Boards と GitHub の連携設定

#### 3.1 GitHub 接続の追加

**方法1: Web UI で接続（基本）**

1. Azure DevOps プロジェクトで「Project Settings」（左下の歯車アイコン）をクリック
2. 左メニュー「Boards」→「GitHub connections」を選択
3. 「Connect your GitHub account」をクリック
4. GitHub にリダイレクトされたら「Authorize」をクリック
5. 接続が成功したことを確認

**方法2: Azure DevOps CLI で接続（推奨）**

```powershell
# 前提: GitHub Personal Access Token (PAT) の作成
# 1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
# 2. "Generate new token (classic)" をクリック
# 3. Scopes を選択:
#    - repo (Full control of private repositories)
#    - admin:repo_hook (Full control of repository hooks)
# 4. トークンをコピーして安全に保存

# 環境変数にトークンを設定（セッション内で一時的に保存）
$env:GITHUB_PAT = "ghp_YourPersonalAccessTokenHere"

# Azure DevOps にログイン
az login
az devops configure --defaults organization=https://dev.azure.com/bell999 project=az400-handson3

# GitHub Service Endpoint を作成
az devops service-endpoint github create `
  --name "GitHub-Connection" `
  --github-url "https://github.com/bell999-az400-handson/az400-handson3" `
  --github-token $env:GITHUB_PAT `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3

# 作成された Service Endpoint を確認
az devops service-endpoint list `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3 `
  --output table

# Azure Boards 用の GitHub 接続を有効化
# 注意: Azure Boards と GitHub の連携は Web UI での初回承認が必要な場合があります
```

**💡 ポイント:**
- GitHub PAT には `repo` と `admin:repo_hook` スコープが必要
- PAT は安全に保管し、環境変数や Azure Key Vault に保存
- Service Endpoint は CI/CD パイプラインでも使用可能
- Azure Boards の GitHub 連携は、初回のみ Web UI での OAuth 承認が必要な場合があります

**⚠️ セキュリティベストプラクティス:**
```powershell
# PAT をセッション後にクリア
Remove-Item Env:\GITHUB_PAT

# または Azure Key Vault に保存して使用
az keyvault secret set `
  --vault-name "your-keyvault" `
  --name "github-pat" `
  --value $env:GITHUB_PAT

# Key Vault から取得して使用
$env:GITHUB_PAT = az keyvault secret show `
  --vault-name "your-keyvault" `
  --name "github-pat" `
  --query value -o tsv
```

#### 3.2 リポジトリの追加

**方法1: Web UI で追加**

1. 「+ Add GitHub repositories」をクリック
2. `az400-handson3` リポジトリを選択
3. 「Save」をクリック

**方法2: Azure DevOps REST API で追加（コマンドライン）**

```powershell
# 前提: Service Endpoint ID を取得
$serviceEndpointId = az devops service-endpoint list `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3 `
  --query "[?name=='GitHub-Connection'].id" -o tsv

# GitHub リポジトリを Azure Boards に接続
# 注意: Azure Boards 専用の GitHub 接続は REST API での自動化が制限されています
# Web UI での初回承認後、以下のコマンドで確認可能

# GitHub 接続済みリポジトリを確認
az repos list `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3 `
  --output table
```

**💡 Azure Boards と GitHub の統合に関する重要な注意:**

Azure Boards の GitHub 接続は、**Service Endpoint** とは異なる専用の接続方式を使用します：

1. **Service Endpoint** (上記 3.1 で作成):
   - Azure Pipelines で GitHub リポジトリにアクセスするために使用
   - `az devops service-endpoint` コマンドで管理可能
   - CI/CD パイプラインで参照

2. **Azure Boards GitHub Connection**:
   - Work Item と Pull Request をリンクするために使用
   - **初回は Web UI での OAuth 承認が必須**
   - Project Settings → Boards → GitHub connections で管理

**推奨アプローチ:**
- **初回セットアップ**: Web UI で OAuth 承認を完了（3.1 の方法1）
- **以降の管理**: CLI や REST API でリポジトリを追加・削除

#### 確認方法
```powershell
# Azure DevOps CLI で Service Endpoint を確認
az devops service-endpoint list `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3 `
  --output table

# GitHub CLI でリポジトリの接続状態を確認
gh repo view bell999-az400-handson/az400-handson3 --json url,name

# Azure Boards の Work Item から GitHub リンクをテスト（後のExerciseで実施）
```

### Exercise 4: AB# の動作確認（title での認識）

#### 4.1 新しいブランチを作成

```powershell
Set-Location c:\Users\bell9\github\az400-handson3

# feature ブランチを作成
git checkout -b feature/login-implementation

# コードを変更
"function login() { return true; }" | Add-Content src/app.js
git add app.js
git commit -m "Implement basic login function"
git push origin feature/login-implementation
```

#### 4.2 Pull Request を作成（title に AB# を含める）

1. GitHub リポジトリのページにアクセス
2. 「Compare & pull request」ボタンをクリック
3. **重要: Title に AB# を含める**
   ```
   Title: AB#574: Implement login feature
   Description: 
   This PR implements the basic login functionality.
   
   Changes:
   - Add login function
   - Update app.js
   ```
4. 「Create pull request」をクリック

#### 4.3 Azure Boards で確認

1. Azure DevOps → Boards → Work items
2. Work Item #574 を開く
3. 「Development」セクションを確認
4. ✅ **Pull Request へのリンクが表示されることを確認**

### Exercise 5: AB# の動作確認（description での認識）

#### 5.1 新しい Pull Request を作成

```powershell
# 別のブランチを作成
git checkout main
git checkout -b feature/add-tests

# ファイルを追加
"// Test placeholder" | Out-File src/app.test.js -Encoding utf8
git add src/app.test.js
git commit -m "Add test file"
git push origin feature/add-tests
```

#### 5.2 Pull Request を作成（description に AB# を含める）

1. GitHub で「Compare & pull request」をクリック
2. **重要: Description に AB# を含める**
   ```
   Title: Add unit tests
   Description:
   This PR adds unit tests for the login feature.
   Related Work Item: AB#575
   ```
3. 「Create pull request」をクリック

#### 5.3 確認

1. Azure Boards で Work Item #575 を開く
2. ✅ **Pull Request へのリンクが表示されることを確認**

### Exercise 6: AB# が認識されない場所の確認

#### 6.1 Comment に AB# を追加（認識されない例）

1. 既存の Pull Request を開く
2. Comment に以下を追加：
   ```
   AB#125 にも関連する変更です。
   ```
3. Comment を投稿

#### 6.2 確認

1. Azure Boards で Work Item #125 を開く
2. ❌ **Comment からのリンクは作成されないことを確認**

#### 結論
- ✅ Title に AB# → リンクされる
- ✅ Description に AB# → リンクされる
- ❌ Comment に AB# → リンクされない

### Exercise 7: GitHub Notifications 設定

#### 7.1 通知設定の確認

**方法1: Web UI でアクセス**

1. GitHub画面右上の**プロフィールアイコン**（自分のアバター）をクリック
2. ドロップダウンメニューから「**Settings**」を選択
3. 左サイドバーから「**Notifications**」を選択
4. 通知設定ページが表示されます

**方法2: 直接URLでアクセス**

ブラウザで以下のURLにアクセス：
```
https://github.com/settings/notifications
```

**💡 ポイント:**
- **リポジトリ設定**（Repository Settings）ではなく、**個人アカウント設定**（User Settings）から行います
- プロフィールアイコンは画面右上にあります
- 直接URLでアクセスするのが最も早い方法です

**📋 GitHub通知設定ページの構成（2026年5月時点）**

通知設定ページには主に以下のセクションがあります：
1. **Default notification email**: 通知を受け取るメールアドレス
2. **Subscriptions**: 通知の管理
   - **Watching**: Watch中のリポジトリからの通知設定
   - **Participating, @mentions and custom**: 自分が参加・メンションされた際の通知
3. **Customize email updates**: メール通知の詳細設定
4. **Ignored repositories**: 無視するリポジトリの管理
5. **System**: システム通知（Actions、Dependabotなど）

#### 7.2 最適な設定（試験重要ポイント）

**❗ 重要な変更（2025年5月23日以降）**

GitHub は **2025年5月23日** に以下の機能を**正式に廃止**しました：
- ❌ **Automatically watch repositories**（リポジトリの自動ウォッチ）
- ❌ **Automatically watch teams**（チームの自動ウォッチ）

**廃止の理由（GitHub公式）:**
- 通知ノイズを減らすため
- 大規模Organization参加時に大量の不要なウォッチが発生する問題を解消
- ユーザーが「自分で選んでウォッチする」モデルへ移行

参考: [GitHub Blog - Sunset notice for automatic watching](https://github.blog/changelog/2025-04-14-sunset-notice-for-automatic-watching-of-repositories-and-teams/)

**⚠️ 重要:**
- 廃止前に自動ウォッチされていたリポジトリはそのまま維持されます
- しかし、**新規に自動ウォッチされることはもうありません**
- 今後はリポジトリを**手動でウォッチ**する必要があります

#### 現在の推奨設定（2026年5月時点）

**自動ウォッチ機能が廃止されたため、以下の方法で通知を管理します：**

**1. 既にウォッチしているリポジトリを確認・解除**

```
手順:
1. https://github.com/watching にアクセス
2. Watch中のリポジトリ一覧が表示されます
3. 不要なリポジトリの「Unwatch」ボタンをクリック
4. または「Custom」を選択して通知レベルを調整
```

**通知レベルの種類:**
- **All Activity**: すべてのアクティビティを通知（通知が多い）
- **Ignore**: 通知を受け取らない
- **Participating and @mentions**: 自分が参加・メンションされた場合のみ通知（推奨）
- **Custom**: Issue、PR、Releaseなどを個別に設定

**2. 通知方法を調整**

https://github.com/settings/notifications で以下を確認：

```
Subscriptions セクション:
- Watching: 「Notify me on GitHub, Email」（推奨: GitHub のみ）
- Participating: 「Notify me on GitHub, Email」（推奨: 両方 ON）
```

**💡 実務での推奨アプローチ:**
- 重要なリポジトリのみ手動でWatch
- CODEOWNERS + Branch Protection Rulesでレビュー依頼を自動化
- GitHub → Teams/Slack 連携で通知を集約

**📌 AZ-400 試験対策のポイント:**
- 2025年5月以前の試験問題では「Automatically watch」の設定が出題される可能性がありますが、**現在は廃止済み**です
- 試験では「廃止された」という選択肢があるかもしれません
- 実務では**手動ウォッチ管理**が標準になっています

### Exercise 8: Azure Repos への通知設定

#### 8.1 Service Hooks の設定

1. Azure DevOps → Project Settings → Service hooks
2. 「+ Create subscription」をクリック
3. サービスを選択: **Web Hooks** または **Teams**

#### 8.2 Teams 通知の設定（オプション）

1. Service: **Microsoft Teams** を選択
2. Trigger: **Pull request created** を選択
3. Filters:
   - Repository: `All`
   - Target branch: `main`
4. Action: **Post a message to a channel**
5. Teams webhook URL を入力
6. 「Finish」をクリック

### Exercise 9: Pull Request のマージと確認

#### 9.1 Pull Request をマージ

1. GitHub で Pull Request を開く
2. 「Merge pull request」をクリック
3. 「Confirm merge」をクリック

#### 9.2 Azure Boards で Work Item を更新

1. Azure Boards で Work Item #123 を開く
2. State を「Active」→「Resolved」に変更
3. 「Save」をクリック

#### 9.3 リンクの確認

- Development セクションに Merged PR が表示されることを確認
- Commit へのリンクも表示されることを確認

## 📊 演習のまとめ

### AB# の動作仕様（重要！）

| 場所 | AB# 認識 | 例 |
|------|----------|-----|
| PR Title | ✅ 認識される | `AB#123: Implement feature` |
| PR Description | ✅ 認識される | `Related: AB#123` |
| PR Comment | ❌ 認識されない | Comment に AB# を書いても無効 |
| Issue Label | ❌ 認識されない | Label には使用できない |
| Commit Message | ✅ 認識される | `git commit -m "Fix AB#123"` |

### GitHub 通知設定の推奨（試験重要！）

**❗ 2025年5月23日の変更点:**

| 項目 | 状態 | 説明 |
|------|------|------|
| **Automatically watch repositories** | ❌ **廃止** | GitHub公式が2025年5月23日に機能を廃止 |
| **Automatically watch teams** | ❌ **廃止** | 同上 |

**現在の推奨設定（2026年5月時点）:**

| 設定項目 | 推奨値 | 設定場所 | 説明 |
|----------|--------|----------|------|
| **Watch管理** | 手動で選択 | https://github.com/watching | 必要なリポジトリのみWatch |
| **Watching通知** | GitHub only | settings/notifications | メール通知を減らす |
| **Participating通知** | GitHub + Email | settings/notifications | 重要な通知は両方で受け取る |

**💡 実務での通知管理:**
- **自動ウォッチは廃止**されたため、リポジトリごとに手動で設定
- **Participating**（自分が参加・メンション）は自動で有効
- https://github.com/watching で一括管理が便利

### Azure DevOps と GitHub の接続方式

| 接続方式 | 用途 | 作成方法 | 認証 |
|----------|------|----------|------|
| **Service Endpoint** | CI/CD パイプライン | `az devops service-endpoint` | GitHub PAT |
| **Azure Boards GitHub Connection** | Work Item 連携（AB#） | Web UI（初回OAuth必須） | OAuth + PAT |

### コマンドラインでの主要操作

```powershell
# GitHub PAT の作成（GitHub Web UI で実施）
# Scopes: repo, admin:repo_hook

# Service Endpoint の作成
az devops service-endpoint github create `
  --name "GitHub-Connection" `
  --github-url "https://github.com/ORG/REPO" `
  --github-token $env:GITHUB_PAT

# Service Endpoint の確認
az devops service-endpoint list --output table

# Azure Boards と GitHub の接続確認（Web UI）
# Project Settings → Boards → GitHub connections
```

## ✅ 確認問題

### Q1: AB# が認識される場所を2つ選んでください
- [ ] A. Pull Request の comment
- [ ] B. Pull Request の title
- [ ] C. Issue の label
- [ ] D. Pull Request の description

<details>
<summary>解答</summary>

**正解: B と D**

説明:
- AB# が認識されるのは Pull Request の **title** と **description** のみ
- Comment や Label では Work Item にリンクされない
</details>

### Q2: GitHub 通知設定に関する正しい説明を選んでください（2026年5月時点）
- [ ] A. Automatically watch repositories 設定を OFF にすることで通知を減らせる
- [ ] B. 通知を減らすには https://github.com/watching で不要なリポジトリをUnwatchする
- [ ] C. Automatically watch teams は Organization メンバーのみ設定可能
- [ ] D. すべての通知を停止するには Email notifications を OFF にする

<details>
<summary>解答</summary>

**正解: B のみ**

説明:
- **A. 誤り**: Automatically watch repositories は **2025年5月23日に GitHub が廃止**しました。現在この設定は存在しません
- **B. 正しい**: https://github.com/watching で Watch 中のリポジトリを確認・解除することで通知を減らせます（現在の推奨方法）
- **C. 誤り**: Automatically watch teams も **廃止されました**。Organization メンバーでも設定できません
- **D. 誤り**: Email notifications を完全に OFF にすると、重要な通知（Participating, @mentions）も受け取れなくなります

**💡 試験対策ポイント:**
- **2025年5月23日以降、自動ウォッチ機能は完全に廃止**
- 現在は **手動でリポジトリをウォッチ/アンウォッチ** する方式
- 古い試験問題では廃止前の設定が出題される可能性があります
- 実務では https://github.com/watching での一括管理が標準

**参考:**
[GitHub Blog - Sunset notice for automatic watching](https://github.blog/changelog/2025-04-14-sunset-notice-for-automatic-watching-of-repositories-and-teams/)
</details>

### Q3: Azure Boards と GitHub を連携するために必要な手順を正しい順序で並べてください
- [ ] A. Pull Request を作成
- [ ] B. GitHub 接続を追加
- [ ] C. Work Item を作成
- [ ] D. リポジトリを選択

<details>
<summary>解答</summary>

**正解: C → B → D → A**

説明:
1. Work Item を作成（AB# に使用するIDを取得）
2. GitHub 接続を追加（Azure DevOps と GitHub を認証）
3. リポジトリを選択（連携対象を指定）
4. Pull Request を作成（AB# でリンク）
</details>

### Q4: Azure DevOps CLI で GitHub Service Endpoint を作成する際に必要な GitHub PAT のスコープを2つ選んでください
- [ ] A. repo
- [ ] B. user
- [ ] C. admin:repo_hook
- [ ] D. workflow

<details>
<summary>解答</summary>

**正解: A と C**

説明:
- **repo**: リポジトリへのフルアクセス（プライベートリポジトリ含む）
- **admin:repo_hook**: リポジトリフックの管理（Azure Boards との連携に必要）
- user や workflow は Azure Boards 連携には不要
</details>

### Q5: Azure Boards の GitHub 連携で正しい説明を選んでください
- [ ] A. Service Endpoint と Azure Boards GitHub Connection は同じもの
- [ ] B. Azure Boards GitHub Connection の初回承認は Web UI が必須
- [ ] C. すべての操作を Azure DevOps CLI で自動化できる
- [ ] D. GitHub PAT は不要で OAuth のみで接続できる

<details>
<summary>解答</summary>

**正解: B**

説明:
- A: **誤り** - Service Endpoint（CI/CD用）と Azure Boards GitHub Connection（Work Item連携用）は異なる
- B: **正しい** - Azure Boards の GitHub 接続は初回 OAuth 承認が Web UI で必須
- C: **誤り** - Azure Boards GitHub Connection の初回承認は Web UI が必要
- D: **誤り** - Service Endpoint 作成には GitHub PAT が必要（OAuth は Web UI での承認用）
</details>

## 🔍 トラブルシューティング

### AB# が認識されない
- Title または Description に AB# が含まれているか確認
- Azure Boards と GitHub の接続が有効か確認
- Work Item ID が正しいか確認

### GitHub 接続エラー
```powershell
# Azure DevOps CLI で接続を確認
az devops project show --org https://dev.azure.com/bell999 --project az400-handson3

# Service Endpoint の状態を確認
az devops service-endpoint list `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3 `
  --output table

# GitHub PAT の有効性を確認
gh auth status

# GitHub 接続を再認証（Web UI）
# Project Settings → GitHub connections → Re-authorize
```

### Service Endpoint のトラブルシューティング
```powershell
# Service Endpoint の詳細を確認
$endpointId = az devops service-endpoint list `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3 `
  --query "[0].id" -o tsv

az devops service-endpoint show `
  --id $endpointId `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3

# Service Endpoint の削除（再作成が必要な場合）
az devops service-endpoint delete `
  --id $endpointId `
  --org https://dev.azure.com/bell999 `
  --project az400-handson3 `
  --yes

# GitHub PAT の権限を確認（GitHub CLI）
gh auth status
gh api user -q .login

# 必要なスコープの確認
# repo, admin:repo_hook が含まれているか確認
```

### GitHub通知設定のトラブルシューティング

**問題: 「Automatically watch repositories」や「Automatically watch teams」が見つからない**

<details>
<summary>解決方法</summary>

**✅ これは正常です！**

GitHub は **2025年5月23日** にこれらの機能を**正式に廃止**しました。

**廃止された機能:**
- ❌ Automatically watch repositories
- ❌ Automatically watch teams

**理由:**
- 通知ノイズを減らすため
- ユーザーが自分で選んでウォッチするモデルへ移行

**現在の対処法:**

1. **Watch中のリポジトリを管理**
   - https://github.com/watching にアクセス
   - 不要なリポジトリを「Unwatch」
   - 必要なリポジトリのみ「Watch」を維持

2. **通知設定を調整**
   - https://github.com/settings/notifications
   - 「Watching」を「Notify me on GitHub」のみに変更（メール通知を減らす）
   - 「Participating」は「GitHub + Email」を維持（重要な通知）

3. **リポジトリごとに個別設定**
   - リポジトリページで「Watch」ボタンをクリック
   - 「Participating and @mentions」を選択（推奨）

**参考:**
[GitHub Blog - Sunset notice](https://github.blog/changelog/2025-04-14-sunset-notice-for-automatic-watching-of-repositories-and-teams/)

</details>

## 📚 参考リンク
- [Azure Boards と GitHub の統合](https://learn.microsoft.com/azure/devops/boards/github/)
- [AB# 参照の使用](https://learn.microsoft.com/azure/devops/boards/github/link-to-from-github)
- [GitHub 通知設定](https://docs.github.com/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications)
- [Azure DevOps CLI - Service Endpoint](https://learn.microsoft.com/cli/azure/devops/service-endpoint)
- [GitHub Personal Access Token の作成](https://docs.github.com/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

## ➡️ 次のステップ
Lab 1 が完了したら、[Lab 2: Azure Pipelines 基礎](./02-Azure-Pipelines-基礎.md) に進んでください。

---

**Great job! You've completed Lab 1! 🎉**
