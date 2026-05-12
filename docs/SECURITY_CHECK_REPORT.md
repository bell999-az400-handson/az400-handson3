# 🔒 セキュリティチェックレポート

**実施日**: 2026年5月12日  
**リポジトリ**: az400-handson3  
**スキャン範囲**: 全ファイル、コミット履歴、設定ファイル

---

## 📊 総合評価

| カテゴリ | ステータス | 評価 |
|---------|-----------|------|
| シークレット管理 | ⚠️ 要対応 | B |
| コード品質 | ✅ 良好 | A |
| .gitignore 設定 | ✅ 良好 | A |
| ドキュメント | ✅ 良好 | A |
| 依存関係 | ✅ 問題なし | A |
| **総合評価** | **⚠️ 要対応** | **B** |

---

## 🚨 重要な発見事項

### 1. ⚠️ **ローカル環境に実際の PAT が存在**（緊急度: 高）

**ファイル**: `.env`  
**問題**: 実際の Azure DevOps Personal Access Token (PAT) が平文で保存されています

```
AZURE_DEVOPS_EXT_PAT=何ちゃら
```

**影響範囲**:
- ✅ **Git リポジトリには追跡されていません**（.gitignore が正常に機能）
- ⚠️ ローカルファイルシステムに平文で存在
- ⚠️ バックアップやクラウド同期（OneDrive、Dropbox など）に含まれる可能性
- ⚠️ マルウェアや不正アクセスでローカルから漏洩する可能性

**推奨対応**:

1. **即座に PAT を無効化**
   ```powershell
   # Azure DevOps で PAT を無効化
   # https://dev.azure.com/bell999/_usersSettings/tokens
   ```

2. **新しい PAT を生成**（必要最小限のスコープで）

3. **Azure Key Vault に保存**（推奨）

   **⚠️ 注意**: 以下のコマンドは Key Vault が既に存在する前提です。Key Vault がまだ存在しない場合は、先に作成する必要があります。

   **Step 1: リソースグループと Key Vault を作成**
   ```powershell
   # 1. リソースグループを作成（存在しない場合）
   az group create `
     --name "rg-az400-handson3" `
     --location "japaneast"

   # 2. 一意な Key Vault 名を生成
   # Key Vault 名はグローバルに一意である必要があるため、ユーザー名を含める
   $kvName = "kv-az400-$($env:USERNAME)"
   Write-Host "Key Vault name: $kvName"

   # 3. Key Vault を作成
   az keyvault create `
     --name $kvName `
     --resource-group "rg-az400-handson3" `
     --location "japaneast" `
     --enable-rbac-authorization true

   # 4. 自分に Key Vault Secrets Officer 権限を付与
   $userId = az ad signed-in-user show --query id -o tsv
   $subscriptionId = az account show --query id -o tsv

   az role assignment create `
     --role "Key Vault Secrets Officer" `
     --assignee $userId `
     --scope "/subscriptions/$subscriptionId/resourceGroups/rg-az400-handson3/providers/Microsoft.KeyVault/vaults/$kvName"

   # 権限の伝播を待つ（30秒程度）
   Write-Host "Waiting for RBAC propagation..."
   Start-Sleep -Seconds 30
   ```

   **Step 2: Azure DevOps PAT を Key Vault に保存**
   ```powershell
   # 新しい PAT を環境変数に設定
   $env:AZURE_DEVOPS_EXT_PAT = "あなたの新しいPAT"

   # Key Vault に保存
   az keyvault secret set `
     --vault-name $kvName `
     --name "AZURE-DEVOPS-PAT" `
     --value $env:AZURE_DEVOPS_EXT_PAT

   Write-Host "✅ Azure DevOps PAT saved to Key Vault: $kvName"
   ```

   **Step 3: Key Vault から PAT を取得して使用**
   ```powershell
   # Key Vault から PAT を取得して環境変数にセット
   $env:AZURE_DEVOPS_EXT_PAT = az keyvault secret show `
     --vault-name $kvName `
     --name "AZURE-DEVOPS-PAT" `
     --query value -o tsv

   # Azure DevOps CLI で使用
   az devops project list --org https://dev.azure.com/bell999

   # セッション終了時にクリア（オプション）
   Remove-Item Env:\AZURE_DEVOPS_EXT_PAT
   ```

   **💡 Key Vault を使うメリット:**
   - 🔒 PAT を平文ファイル（.env）に保存する必要がない
   - 🔒 Azure RBAC でアクセス制御が可能
   - 🔒 監査ログで誰がいつアクセスしたか記録される
   - 🔒 自動ローテーション（有効期限管理）が可能
   - 🔒 チームメンバーと安全に共有できる

   **📝 Key Vault 名の確認**
   ```powershell
   # 作成した Key Vault 名を確認
   az keyvault list `
     --resource-group "rg-az400-handson3" `
     --query "[].name" -o tsv

   # シークレット一覧を確認
   az keyvault secret list `
     --vault-name $kvName `
     --query "[].name" -o tsv
   ```

4. **.env ファイルの削除または安全な管理**
   ```powershell
   # .env ファイルを削除（Key Vault に移行した場合）
   Remove-Item .env -Force

   # または .env.example を作成してコミット（値は空または例示）
   # .env ファイルは .gitignore で除外されていることを確認
   ```

---

## ✅ 良好な点

### 1. **.gitignore が適切に設定されている**

以下の機密ファイルが正しく除外されています：

```gitignore
# 環境変数ファイル
.env
.env.local
*.env

# Secrets and credentials
secrets.json
appsettings.*.json
local.settings.json

# SSH Keys
*.pem
*.key
id_rsa*

# Personal Access Tokens
*.pat
```

**検証結果**:
- ✅ .env ファイルは Git で追跡されていません
- ✅ secrets.json などの機密ファイルは存在しません
- ✅ SSH 秘密鍵ファイルは存在しません

### 2. **コード内にハードコーディングされたシークレットなし**

**スキャン実施**:
```powershell
# 実際の GitHub PAT パターン（ghp_*, gho_*, github_pat_*）
# 実際の AWS キー（AKIA*）
# 実際の OpenAI キー（sk-*）
```

**結果**: ❌ 実際のシークレットは検出されませんでした

### 3. **ドキュメント内の例示は適切**

以下のファイルで使用されているシークレットはすべて**プレースホルダー**です：

| ファイル | 例示内容 | ステータス |
|---------|---------|-----------|
| `01-GitHub-AzureBoards連携.md` | `ghp_YourPersonalAccessTokenHere` | ✅ プレースホルダー |
| `SECURITY_AUDIT.md` | `"password": "abc123~secret~xyz789"` | ✅ ダミー値 |
| `01-GitHub-AzureBoards連携.md` | Slack Webhook URL 例 | ✅ プレースホルダー |

### 4. **SECURITY.md が包括的**

既存の `SECURITY.md` には以下が含まれています：

- ✅ セキュリティポリシー
- ✅ 脆弱性報告手順
- ✅ セキュリティベストプラクティス
- ✅ 禁止事項リスト
- ✅ 自動スキャンツールの推奨
- ✅ セキュリティチェックリスト

### 5. **Pull Request テンプレートにセキュリティチェック**

`.github/pull_request_template.md` に以下が含まれています：

```markdown
- [ ] No secrets in code
```

### 6. **GitHub Copilot Instructions にセキュリティ方針**

`.github/copilot-instructions.md` に以下が含まれています：

```markdown
- Never put secrets in code
- Use Azure Key Vault for secrets
```

---

## 📋 追加の推奨事項

### 1. **GitHub Secret Scanning の有効化**（優先度: 高）

リポジトリ設定で有効化：
1. Settings → Code security and analysis
2. "Secret scanning" を有効化
3. "Push protection" を有効化（推奨）

### 2. **Dependabot Alerts の有効化**（優先度: 中）

依存関係の脆弱性を自動検出：
1. Settings → Code security and analysis
2. "Dependabot alerts" を有効化
3. "Dependabot security updates" を有効化

### 3. **Branch Protection Rules の設定**（優先度: 中）

main ブランチを保護：
1. Settings → Branches → Add rule
2. Branch name pattern: `main`
3. 以下を有効化：
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require conversation resolution before merging
   - ✅ Do not allow bypassing the above settings

### 4. **.env.example の作成**（優先度: 低）

開発者向けにテンプレートを提供：

```env
# .env.example（リポジトリにコミット可能）
AZURE_DEVOPS_EXT_PAT=your-pat-here
AZURE_DEVOPS_ORG=https://dev.azure.com/your-org
AZURE_DEVOPS_PROJECT=your-project
```

### 5. **定期的なセキュリティスキャン**（優先度: 中）

月次または四半期ごとに実施：

```powershell
# シークレットスキャン
git log -p | Select-String -Pattern "password|secret|token|api[_-]?key" -CaseSensitive:$false

# .gitignore の検証
git check-ignore -v .env
git check-ignore -v secrets.json

# コミット履歴のスキャン
git log --all --full-history --source --find-copies-harder -S "password"
```

---

## 🔍 スキャン実施内容

### スキャンパターン

| パターン | 説明 | 結果 |
|---------|------|------|
| `password\|secret\|api[_-]?key\|access[_-]?token` | 一般的なシークレット | ドキュメント内のプレースホルダーのみ |
| `ghp_[a-zA-Z0-9]{36}` | GitHub PAT (classic) | 検出なし ✅ |
| `gho_[a-zA-Z0-9]{36}` | GitHub OAuth token | 検出なし ✅ |
| `github_pat_[a-zA-Z0-9]{22}_[a-zA-Z0-9]{59}` | GitHub PAT (fine-grained) | 検出なし ✅ |
| `AKIA[0-9A-Z]{16}` | AWS Access Key ID | 検出なし ✅ |
| `sk-[a-zA-Z0-9]{48}` | OpenAI API Key | 検出なし ✅ |

### ファイルスキャン

| ファイルパターン | 検索対象 | 結果 |
|----------------|---------|------|
| `**/*.{env,pem,key,pfx,p12,jks}` | 機密ファイル | 検出なし ✅ |
| `**/secrets.json` | シークレット設定 | 検出なし ✅ |
| `.env` | 環境変数ファイル | ⚠️ ローカルに存在（Git追跡なし） |

### Git 履歴スキャン

| チェック項目 | 結果 |
|-------------|------|
| 最近20件のコミットメッセージ | 問題なし ✅ |
| Git 追跡ファイルリスト | .env は追跡されていない ✅ |
| ステージングエリア | 機密ファイルなし ✅ |

---

## 🎯 アクションプラン

### 緊急（今すぐ実施）

- [ ] **Azure DevOps PAT を無効化**（.env に保存されているもの）
- [ ] **新しい PAT を生成**（必要最小限のスコープ）
- [ ] **Azure Key Vault に保存**または .env を削除

### 短期（1週間以内）

- [ ] **GitHub Secret Scanning を有効化**
- [ ] **Dependabot Alerts を有効化**
- [ ] **.env.example を作成**してコミット

### 中期（1ヶ月以内）

- [ ] **Branch Protection Rules を設定**
- [ ] **定期的なセキュリティスキャンをスケジュール**
- [ ] **セキュリティ監査のドキュメント更新**

---

## 📞 参考リンク

- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning)
- [Azure Key Vault ベストプラクティス](https://learn.microsoft.com/azure/key-vault/general/best-practices)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Azure DevOps PAT 管理](https://learn.microsoft.com/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)

---

## ✅ 結論

**総合評価**: ⚠️ **要対応（B評価）**

**良い点**:
- .gitignore が適切に設定されている
- コード内にハードコーディングされたシークレットがない
- セキュリティドキュメントが充実している
- Pull Request テンプレートにセキュリティチェックがある

**改善が必要な点**:
- ⚠️ ローカルの .env ファイルに実際の PAT が存在
- GitHub Secret Scanning が未有効
- Branch Protection Rules が未設定

**次のステップ**:
1. 即座に Azure DevOps PAT を無効化・再生成
2. Azure Key Vault に移行または .env を削除
3. GitHub のセキュリティ機能を有効化
4. 定期的なセキュリティスキャンを実施

---

**スキャン実施者**: GitHub Copilot  
**レポート生成日**: 2026年5月12日
