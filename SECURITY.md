# セキュリティポリシー

## 🔒 サポート対象バージョン

このリポジトリは **教育目的** のAZ-400試験ハンズオン教材です。

| ブランチ | サポート状況 |
| ------- | ---------- |
| main    | ✅ 最新版 |
| その他   | ❌ サポート外 |

---

## 🚨 脆弱性の報告

### セキュリティ上の問題を発見した場合

**公開のIssueを作成しないでください。** 代わりに以下の手順に従ってください：

1. **プライベート報告**
   - GitHubの[Security Advisories](../../security/advisories)を使用
   - または、リポジトリ管理者に直接連絡

2. **報告に含めるべき情報**
   - 脆弱性の種類（例: ハードコードされたシークレット、XSS、SQLインジェクションなど）
   - 影響を受けるファイルとコード行
   - 再現手順
   - 潜在的な影響範囲
   - 可能であれば、修正案

3. **報告例**
   ```
   タイトル: [SECURITY] ハードコードされたAPIキーの検出
   
   説明:
   ファイル: docs/example.md
   行: 42
   問題: GitHub Personal Access Tokenが平文で記載されています
   影響: 認証情報の漏洩リスク
   推奨修正: プレースホルダーに置き換え
   ```

---

## 🛡️ セキュリティベストプラクティス

### このリポジトリで実施している対策

#### 1. シークレット管理
- ✅ `.gitignore` で環境変数ファイルを除外
- ✅ Azure Key Vault の使用を推奨
- ✅ 環境変数による認証情報管理
- ✅ Pull Requestテンプレートでシークレットチェック

#### 2. アクセス制御
- ✅ Service Principal による最小権限アクセス
- ✅ Branch Protection Rules（推奨）
- ✅ Required reviewers for Pull Requests
- ✅ Azure RBAC による権限管理

#### 3. コードレビュー
- ✅ Pull Request必須
- ✅ AB# による Work Item トラッキング
- ✅ セキュリティチェックリスト

#### 4. 監視
- ✅ Application Insights によるテレメトリ
- ✅ Azure Monitor によるアラート
- ✅ ログの集約と分析

---

## 📋 セキュリティチェックリスト

### コントリビューター向け

Pull Requestを作成する前に、以下を確認してください：

- [ ] コード内にシークレット（パスワード、APIキー、トークンなど）が含まれていない
- [ ] 環境変数を適切に使用している
- [ ] `.gitignore` に機密ファイルが追加されている
- [ ] ハードコードされた認証情報がない
- [ ] Azure Key Vault の使用を推奨している（該当する場合）
- [ ] 最小権限の原則に従っている
- [ ] セキュリティ上のベストプラクティスに準拠している

### レビュワー向け

Pull Requestをレビューする際は、以下を確認してください：

- [ ] `.env` ファイルや機密情報がコミットされていない
- [ ] サンプルコードがダミー値を使用している
- [ ] 認証情報が環境変数または Key Vault から取得されている
- [ ] エラーメッセージに機密情報が含まれていない
- [ ] ログ出力に認証情報が含まれていない
- [ ] 外部依存関係に既知の脆弱性がない

---

## 🚫 禁止事項

以下の情報を **絶対にコミットしないでください**：

❌ GitHub Personal Access Tokens (PAT)  
❌ Azure Service Principal の Client Secret  
❌ データベース接続文字列  
❌ API Keys / Secret Keys  
❌ SSH 秘密鍵  
❌ SSL/TLS 証明書の秘密鍵  
❌ `.env` ファイル  
❌ `local.settings.json` （Azure Functions）  
❌ `.publishsettings` （Azure）  
❌ 個人を特定できる情報 (PII)  

---

## 🔍 自動セキュリティスキャン

### 有効化を推奨するツール

1. **GitHub Secret Scanning**
   - リポジトリ設定で有効化
   - 自動的にシークレットを検出

2. **Dependabot Alerts**
   - 依存関係の脆弱性を検出
   - 自動的にセキュリティアップデートを提案

3. **Code Scanning (CodeQL)**
   - GitHub Advanced Security（有料）
   - コード内の脆弱性を検出

### 手動スキャン

```powershell
# シークレットスキャン（正規表現）
git log -p | Select-String -Pattern "password|secret|token|api[_-]?key" -CaseSensitive:$false

# .gitignore の検証
git check-ignore -v .env
git check-ignore -v local.settings.json

# コミット履歴のスキャン
git log --all --full-history --source --find-copies-harder -S "password"
```

---

## 📞 連絡先

セキュリティに関する質問や懸念事項がある場合：

- **GitHub Security Advisories:** [こちら](../../security/advisories)
- **Issue（非機密情報のみ）:** [こちら](../../issues)

---

## 📚 参考資料

- [Azure Key Vault のベストプラクティス](https://learn.microsoft.com/azure/key-vault/general/best-practices)
- [GitHub のセキュリティベストプラクティス](https://docs.github.com/code-security/getting-started/github-security-features)
- [Azure DevOps のセキュリティ](https://learn.microsoft.com/azure/devops/organizations/security/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)

---

## 📅 更新履歴

| 日付 | 変更内容 |
|------|---------|
| 2026-05-11 | セキュリティポリシーの初版作成 |
| 2026-05-11 | `.gitignore` の包括的な設定を追加 |

---

**このポリシーは定期的に見直され、更新されます。**

最終更新: 2026年5月11日
