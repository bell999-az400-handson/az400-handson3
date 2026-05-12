# Lab 3: Azure Artifacts パッケージ管理

## 🎯 目的
このLabでは、Azure Artifacts を使用したパッケージ管理を学び、Feed、Views、NuGet パッケージの発行と利用方法を習得します。

## ⏱️ 所要時間
約60分

## 📋 前提条件
- Lab 0 の環境準備が完了していること
- .NET SDK 8.0 がインストールされていること
- Azure DevOps プロジェクト

## 🎓 学習内容

### 重要ポイント（試験頻出）
✅ **Feed の概念**
- パッケージを保存・管理するコンテナ
- NuGet、npm、Maven、Python などに対応

✅ **Views の種類**
- **@Local**: Feed 内のすべてのパッケージ
- **@Prerelease**: プレリリース版
- **@Release**: 正式リリース版

✅ **BACPAC vs DACPAC**
- **BACPAC**: schema + data（データ移行用）
- **DACPAC**: schema only（スキーマのみ）

## 📝 演習内容

### Exercise 1: Azure Artifacts Feed の作成

#### 1.1 Feed の作成

1. Azure DevOps → Artifacts → 「+ Create Feed」をクリック
2. 以下を入力：
   - Name: `az400-package-feed`
   - Visibility: 
     - ✅ Members of bell999（組織内のみ）
   - Upstream sources:
     - ✅ Include packages from common public sources
     - nuget.org, npmjs.com, PyPI などを含める
3. 「Create」をクリック

#### 1.2 Feed の確認

- Feed が作成されたことを確認
- Views タブで以下が存在することを確認：
  - `@Local`
  - `@Prerelease`  
  - `@Release`

#### 1.3 Azure CLI での確認（オプション）

```powershell
# Azure DevOps 拡張機能で Feed を確認
az artifacts universal package list --feed az400-package-feed --organization https://dev.azure.com/bell999 --project az400-handson3
```

### Exercise 2: NuGet パッケージの作成

#### 2.1 クラスライブラリプロジェクトの作成

```powershell
# 新しいディレクトリを作成
Set-Location ~\Documents
New-Item -ItemType Directory -Name Az400.Utils
Set-Location Az400.Utils

# クラスライブラリを作成
dotnet new classlib -n Az400.Utils

# プロジェクトディレクトリに移動
Set-Location Az400.Utils
```

#### 2.2 サンプルコードの追加

`StringHelper.cs` を作成：

```csharp
namespace Az400.Utils;

public static class StringHelper
{
    /// <summary>
    /// 文字列を逆順にします
    /// </summary>
    public static string Reverse(string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;
        
        char[] charArray = input.ToCharArray();
        Array.Reverse(charArray);
        return new string(charArray);
    }
    
    /// <summary>
    /// 文字列が回文かどうかを判定します
    /// </summary>
    public static bool IsPalindrome(string input)
    {
        if (string.IsNullOrEmpty(input))
            return false;
        
        string normalized = input.ToLower().Replace(" ", "");
        return normalized == Reverse(normalized);
    }
}
```

#### 2.3 プロジェクトファイルの更新

`Az400.Utils.csproj` を編集：

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    
    <!-- パッケージ情報 -->
    <PackageId>Az400.Utils</PackageId>
    <Version>1.0.0</Version>
    <Authors>Your Name</Authors>
    <Company>AZ400 Training</Company>
    <Description>AZ-400 ハンズオン用のユーティリティライブラリ</Description>
    <PackageTags>az400;utils;training</PackageTags>
  </PropertyGroup>

</Project>
```

#### 2.4 パッケージのビルド

```powershell
# パッケージを作成
dotnet pack --configuration Release

# 出力先を確認
Get-ChildItem bin\Release\

# 結果: Az400.Utils.1.0.0.nupkg が作成される
```

### Exercise 3: Azure Artifacts への発行

#### 3.1 認証情報の設定

```powershell
# Azure Artifacts Credential Provider のインストール（Windows）
Invoke-WebRequest -Uri https://aka.ms/install-artifacts-credprovider.ps1 -OutFile install-artifacts-credprovider.ps1
.\install-artifacts-credprovider.ps1

# または環境変数で認証
$env:VSS_NUGET_EXTERNAL_FEED_ENDPOINTS = @"
{
  "endpointCredentials": [
    {
      "endpoint": "https://pkgs.dev.azure.com/bell999/_packaging/az400-package-feed/nuget/v3/index.json",
      "password": "your-personal-access-token"
    }
  ]
}
"@
```

#### 3.2 NuGet ソースの追加

1. Azure DevOps → Artifacts → Feed を開く
2. 「Connect to Feed」をクリック
3. 「NuGet.exe」を選択
4. ソース URL をコピー（例）：
   ```
   https://pkgs.dev.azure.com/bell999/_packaging/az400-package-feed/nuget/v3/index.json
   ```

```powershell
# NuGet ソースを追加
dotnet nuget add source `
  https://pkgs.dev.azure.com/bell999/_packaging/az400-package-feed/nuget/v3/index.json `
  --name Az400Feed `
  --username any `
  --password your-PAT `
  --store-password-in-clear-text

# ソースが追加されたことを確認
dotnet nuget list source
```

#### 3.3 パッケージの発行

```powershell
# パッケージを Azure Artifacts に発行
dotnet nuget push `
  bin\Release\Az400.Utils.1.0.0.nupkg `
  --source Az400Feed `
  --api-key az
```

#### 3.4 発行の確認

1. Azure DevOps → Artifacts → Feed を開く
2. `Az400.Utils` パッケージが表示されることを確認
3. Version: `1.0.0`
4. View: `@Local` に配置されていることを確認

### Exercise 4: Views の管理

#### 4.1 Prerelease View への昇格

1. パッケージ `Az400.Utils` をクリック
2. 「Promote」ボタンをクリック
3. View: `@Prerelease` を選択
4. 「Promote」をクリック

#### 4.2 Release View への昇格

1. 再度「Promote」ボタンをクリック
2. View: `@Release` を選択
3. 「Promote」をクリック

#### 4.3 Views の使い分け（試験重要）

| View | 用途 | 例 |
|------|------|-----|
| @Local | すべてのパッケージ | 開発中のすべてのバージョン |
| @Prerelease | プレリリース版 | ベータテスト用（1.0.0-beta） |
| @Release | 正式リリース版 | 本番環境で使用（1.0.0） |

### Exercise 5: パッケージの利用

#### 5.1 新しいコンソールアプリの作成

```powershell
# 新しいディレクトリを作成
Set-Location ~\Documents
New-Item -ItemType Directory -Name Az400.Consumer
Set-Location Az400.Consumer

# コンソールアプリを作成
dotnet new console -n ConsumerApp
Set-Location ConsumerApp
```

#### 5.2 パッケージの追加

```powershell
# Azure Artifacts から パッケージをインストール
dotnet add package Az400.Utils --version 1.0.0 --source Az400Feed
```

#### 5.3 パッケージの使用

`Program.cs` を編集：

```csharp
using Az400.Utils;

Console.WriteLine("=== Az400.Utils パッケージのテスト ===");

// 文字列の逆順
string original = "Hello, AZ-400!";
string reversed = StringHelper.Reverse(original);
Console.WriteLine($"Original: {original}");
Console.WriteLine($"Reversed: {reversed}");

// 回文チェック
string[] words = { "racecar", "hello", "level", "world" };
foreach (var word in words)
{
    bool isPalindrome = StringHelper.IsPalindrome(word);
    Console.WriteLine($"{word} is palindrome: {isPalindrome}");
}
```

#### 5.4 実行

```powershell
# ビルドして実行
dotnet build
dotnet run

# 期待される出力:
# === Az400.Utils パッケージのテスト ===
# Original: Hello, AZ-400!
# Reversed: !004-ZA ,olleH
# racecar is palindrome: True
# hello is palindrome: False
# level is palindrome: True
# world is palindrome: False
```

### Exercise 6: パイプラインでのパッケージ発行

#### 6.1 パイプライン YAML の作成

`azure-pipelines-package.yml` を作成：

```yaml
# azure-pipelines-package.yml
trigger:
  branches:
    include:
    - main
  tags:
    include:
    - v*

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  packageVersion: '1.0.$(Build.BuildId)'

stages:
- stage: Build
  displayName: 'Build and Pack'
  jobs:
  - job: BuildJob
    steps:
    - task: UseDotNet@2
      displayName: 'Use .NET 8.0'
      inputs:
        version: '8.0.x'
    
    - task: DotNetCoreCLI@2
      displayName: 'Restore'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
    
    - task: DotNetCoreCLI@2
      displayName: 'Build'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
        arguments: '--configuration $(buildConfiguration)'
    
    # パッケージを作成
    - task: DotNetCoreCLI@2
      displayName: 'Pack NuGet Package'
      inputs:
        command: 'pack'
        packagesToPack: '**/Az400.Utils.csproj'
        versioningScheme: 'byEnvVar'
        versionEnvVar: 'packageVersion'
        configuration: '$(buildConfiguration)'
    
    # Azure Artifacts に発行
    - task: NuGetCommand@2
      displayName: 'Push to Azure Artifacts'
      inputs:
        command: 'push'
        packagesToPush: '$(Build.ArtifactStagingDirectory)/**/*.nupkg'
        publishVstsFeed: 'az400-handson3/az400-package-feed'
        allowPackageConflicts: false
```

### Exercise 7: Azure SQL の BACPAC と DACPAC（試験重要）

#### 7.1 BACPAC と DACPAC の違い

| 項目 | BACPAC | DACPAC |
|------|--------|--------|
| 含まれる内容 | **schema + data** | **schema のみ** |
| 用途 | データベース移行 | スキーマのデプロイ |
| ファイル拡張子 | `.bacpac` | `.dacpac` |
| 作成コマンド | `SqlPackage.exe /a:Export` | `SqlPackage.exe /a:Extract` |
| 復元コマンド | `SqlPackage.exe /a:Import` | `SqlPackage.exe /a:Publish` |

#### 7.2 BACPAC のエクスポート（Azure CLI）

```powershell
# Azure SQL Database から BACPAC をエクスポート
az sql db export `
  --resource-group myResourceGroup `
  --server myserver `
  --name mydb `
  --admin-user myadmin `
  --admin-password mypassword `
  --storage-key-type StorageAccessKey `
  --storage-key mystoragekey `
  --storage-uri https://mystorageaccount.blob.core.windows.net/bacpac/mydb.bacpac
```

#### 7.3 DACPAC の作成（SqlPackage.exe）

```powershell
# DACPAC をエクスポート（スキーマのみ）
SqlPackage.exe /a:Extract `
  /ssn:myserver.database.windows.net `
  /sdn:mydb `
  /su:myadmin `
  /sp:mypassword `
  /tf:mydb.dacpac
```

#### 7.4 ユースケース

**BACPAC を使用する場合:**
- 開発環境から本番環境へのデータ移行
- データベースのバックアップ
- データを含むテスト環境の構築

**DACPAC を使用する場合:**
- CI/CD パイプラインでのスキーマデプロイ
- データベーススキーマのバージョン管理
- スキーマの変更を追跡

### Exercise 8: Upstream Sources の活用

#### 8.1 Upstream Source の確認

1. Azure DevOps → Artifacts → Feed Settings
2. 「Upstream sources」タブを開く
3. デフォルトで以下が有効:
   - nuget.org
   - npmjs.com (npm の場合)
   - PyPI (Python の場合)

#### 8.2 動作確認

```powershell
# 公開パッケージをインストール（Upstream 経由）
dotnet add package Newtonsoft.Json --source Az400Feed

# Azure Artifacts にキャッシュされることを確認
# Feed → Packages → Newtonsoft.Json が表示される
```

#### 8.3 Upstream の利点

- パッケージの高速ダウンロード（キャッシュ）
- 外部パッケージの可用性向上
- パッケージのバージョン固定

## 📊 演習のまとめ

### Feed 作成から利用までのフロー

```
1. Feed 作成
   ↓
2. パッケージをビルド (dotnet pack)
   ↓
3. パッケージを発行 (dotnet nuget push)
   ↓
4. Views で管理 (@Local → @Prerelease → @Release)
   ↓
5. 他のプロジェクトで利用 (dotnet add package)
```

### Views の昇格パス

```
@Local (すべて)
  ↓ Promote
@Prerelease (プレリリース)
  ↓ Promote
@Release (正式版)
```

## ✅ 確認問題

### Q1: データとスキーマの両方を含むSQL Server移行ファイル形式は？
- [ ] A. DACPAC
- [ ] B. BACPAC
- [ ] C. MDF
- [ ] D. LDF

<details>
<summary>解答</summary>

**正解: B**

説明:
- BACPAC: schema + data（データベース移行用）
- DACPAC: schema only（スキーマのみ）
- MDF/LDF: SQL Server のデータファイル
</details>

### Q2: Azure Artifacts で正式リリース版を管理するViewは？
- [ ] A. @Local
- [ ] B. @Prerelease
- [ ] C. @Release
- [ ] D. @Production

<details>
<summary>解答</summary>

**正解: C**

説明:
- @Local: すべてのパッケージ
- @Prerelease: プレリリース版
- @Release: 正式リリース版
</details>

### Q3: NuGet パッケージを Azure Artifacts に発行するコマンドは？
- [ ] A. dotnet publish
- [ ] B. dotnet nuget push
- [ ] C. dotnet pack
- [ ] D. dotnet deploy

<details>
<summary>解答</summary>

**正解: B**

説明:
- `dotnet pack`: パッケージを作成（.nupkg）
- `dotnet nuget push`: パッケージを Feed に発行
</details>

## 🔍 トラブルシューティング

### パッケージの発行に失敗
```powershell
# 認証エラーの場合
# PAT を再作成して環境変数を設定

# Feed の権限を確認
# Feed Settings → Permissions → 自分が Contributor 以上か確認
```

### パッケージが見つからない
```powershell
# NuGet ソースを確認
dotnet nuget list source

# ソースを削除して再追加
dotnet nuget remove source Az400Feed
dotnet nuget add source {feed-url} --name Az400Feed
```

## 📚 参考リンク
- [Azure Artifacts ドキュメント](https://learn.microsoft.com/azure/devops/artifacts/)
- [NuGet パッケージの作成](https://learn.microsoft.com/nuget/create-packages/creating-a-package)
- [SqlPackage.exe](https://learn.microsoft.com/sql/tools/sqlpackage/)
- [BACPAC と DACPAC](https://learn.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications)

## ➡️ 次のステップ
Lab 3 が完了したら、[Lab 4: セキュリティとコンプライアンス](./04-セキュリティとコンプライアンス.md) に進んでください。

---

**Fantastic! You've mastered Azure Artifacts! 📦**
