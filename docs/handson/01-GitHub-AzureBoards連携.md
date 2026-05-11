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

### Exercise 2: Azure Boards の準備

#### 2.1 Work Item の作成

1. [Azure DevOps](https://dev.azure.com/) にアクセス
2. Organization → `AZ400-HandsOn` プロジェクトを開く
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

1. Azure DevOps プロジェクトで「Project Settings」（左下の歯車アイコン）をクリック
2. 左メニュー「Boards」→「GitHub connections」を選択
3. 「Connect your GitHub account」をクリック
4. GitHub にリダイレクトされたら「Authorize」をクリック
5. 接続が成功したことを確認

#### 3.2 リポジトリの追加

1. 「+ Add GitHub repositories」をクリック
2. `az400-handson3` リポジトリを選択
3. 「Save」をクリック

#### 確認方法
```powershell
# Azure DevOps CLI で確認
az boards github repo list --org https://dev.azure.com/your-org --project AZ400-HandsOn
```

### Exercise 4: AB# の動作確認（title での認識）

#### 4.1 新しいブランチを作成

```powershell
Set-Location c:\Users\bell9\github\az400-handson3

# feature ブランチを作成
git checkout -b feature/login-implementation

# コードを変更
"function login() { return true; }" | Add-Content app.js
git add app.js
git commit -m "Implement basic login function"
git push origin feature/login-implementation
```

#### 4.2 Pull Request を作成（title に AB# を含める）

1. GitHub リポジトリのページにアクセス
2. 「Compare & pull request」ボタンをクリック
3. **重要: Title に AB# を含める**
   ```
   Title: AB#123: Implement login feature
   Description: 
   This PR implements the basic login functionality.
   
   Changes:
   - Add login function
   - Update app.js
   ```
4. 「Create pull request」をクリック

#### 4.3 Azure Boards で確認

1. Azure DevOps → Boards → Work items
2. Work Item #123 を開く
3. 「Development」セクションを確認
4. ✅ **Pull Request へのリンクが表示されることを確認**

### Exercise 5: AB# の動作確認（description での認識）

#### 5.1 新しい Pull Request を作成

```powershell
# 別のブランチを作成
git checkout main
git checkout -b feature/add-tests

# ファイルを追加
"// Test placeholder" | Out-File app.test.js -Encoding utf8
git add app.test.js
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
   Related Work Item: AB#124
   ```
3. 「Create pull request」をクリック

#### 5.3 確認

1. Azure Boards で Work Item #124 を開く
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

1. GitHub で Settings → Notifications にアクセス
2. 現在の設定を確認：
   - **Watching**: フォロー中のリポジトリの全通知
   - **Participating**: 自分が関与した通知のみ
   - **Automatically watch repositories**: リポジトリへのアクセス時に自動Watch
   - **Automatically watch teams**: チームに参加時に自動Watch

#### 7.2 最適な設定（試験重要ポイント）

通知を減らすための推奨設定：
- ✅ **Participating**: ON のまま（重要な通知は受け取る）
- ❌ **Automatically watch repositories**: OFF（不要な通知を減らす）
- ❌ **Automatically watch teams**: OFF（不要な通知を減らす）

#### 設定方法
1. Settings → Notifications
2. 「Automatically watch repositories」のチェックを外す
3. 「Automatically watch teams」のチェックを外す
4. 「Update preferences」をクリック

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

### GitHub 通知設定の推奨

| 設定項目 | 推奨値 | 理由 |
|----------|--------|------|
| Participating | ON | 自分が関与する通知は必要 |
| Watching | 必要に応じて | 特定のリポジトリのみ |
| Automatically watch repositories | **OFF** | 不要な通知を減らす |
| Automatically watch teams | **OFF** | 不要な通知を減らす |

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

### Q2: GitHub で通知を減らすために OFF にすべき設定を2つ選んでください
- [ ] A. Participating
- [ ] B. Automatically watch repositories
- [ ] C. Watching
- [ ] D. Automatically watch teams

<details>
<summary>解答</summary>

**正解: B と D**

説明:
- **Automatically watch repositories**: OFF（リポジトリアクセス時に自動Watchしない）
- **Automatically watch teams**: OFF（チーム参加時に自動Watchしない）
- Participating は OFF にすると必要な通知も来なくなるため、ON のままにする
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

## 🔍 トラブルシューティング

### AB# が認識されない
- Title または Description に AB# が含まれているか確認
- Azure Boards と GitHub の接続が有効か確認
- Work Item ID が正しいか確認

### GitHub 接続エラー
```powershell
# Azure DevOps CLI で接続を確認
az devops project show --org https://dev.azure.com/your-org --project AZ400-HandsOn

# GitHub 接続を再認証
# Project Settings → GitHub connections → Re-authorize
```

## 📚 参考リンク
- [Azure Boards と GitHub の統合](https://learn.microsoft.com/azure/devops/boards/github/)
- [AB# 参照の使用](https://learn.microsoft.com/azure/devops/boards/github/link-to-from-github)
- [GitHub 通知設定](https://docs.github.com/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications)

## ➡️ 次のステップ
Lab 1 が完了したら、[Lab 2: Azure Pipelines 基礎](./02-Azure-Pipelines-基礎.md) に進んでください。

---

**Great job! You've completed Lab 1! 🎉**
