# iOS CI/CDワークフロードキュメント

## 概要

このドキュメントは、GitHub ActionsとFastlaneを使用したiOSアプリケーションの自動化された継続的インテグレーション・継続的デプロイメント（CI/CD）ワークフローについて説明します。

---

## インフラストラクチャ

### CI/CDプラットフォーム
- **プラットフォーム**: GitHub Actions
- **ランナー**: 標準GitHub提供ランナー（macOS）
- **オペレーティングシステム**: macOS 26.0.1 (25A362)
- **Xcodeバージョン**: 26.0.1 Build 17A400

### コード署名
- **証明書管理**: Fastlane Match
- **ストレージ**: プライベートGitリポジトリ
- **対応プロファイル**: 
  - Ad-Hoc（最大100台のiPhone登録可能）
  - App Store配信用

---

## ワークフロー

異なるデプロイメントターゲットと環境に対応する、2つの独立したCI/CDワークフローを維持しています。

### 1. Firebase App Distributionワークフロー

**目的**: 自動テストと社内配信

**設定**:
- **トリガー**: 手動実行 (git操作でもいける。例え、gitタグ)
- **プロビジョニングプロファイル**: Ad-Hoc
- **ビルド構成 (Xcode Build Configuration)**: CI_AdHoc
- **配信プラットフォーム**: Firebase App Distribution
- **ターゲット環境**: テスト環境
- **デバイス制限**: 最大100台の登録済みデバイス(iPhone、iPod Touch、iPadなどで100台ずつ)
- **典型的な使用例**:
  - QAテスト
  - 社内チームレビュー
  - 特定のテスターによるベータテスト

**ワークフローファイル**: `.github/workflows/firebase.yml`

**主な機能**:
- ビルド番号の自動インクリメント
- クラッシュ解析用のデバッグシンボルを含む
- 特定のテスターグループへの配信
- 開発のための高速イテレーションサイクル

---

### 2. TestFlight配信ワークフロー

**目的**: Appleの公式プラットフォームを通じた本番環境対応のベータテスト

**設定**:
- **トリガー**: 手動実行 (git操作でもいける。例え、gitタグ)
- **プロビジョニングプロファイル**: App Store
- **ビルド構成 (Xcode Build Configuration)**: CI_AppStore
- **配信プラットフォーム**: TestFlight（App Store Connect）
- **ターゲット環境**: 本番環境
- **デバイス制限**: 無制限（TestFlight経由）
- **典型的な使用例**:
  - リリース前テスト
  - 外部ベータテスト
  - App Store申請前の最終検証
  - 段階的ロールアウトの準備

**ワークフローファイル**: `.github/workflows/testflight.yml`

**主な機能**:
- 本番レベルの最適化
- App Store Connect完全統合
- App Storeガイドラインへの準拠
- TestFlightへの自動送信
- 段階的ロールアウトテストのサポート

---

## ワークフロー比較

| 機能 | Firebase（Ad-Hoc） | TestFlight（App Store） |
|------|-------------------|------------------------|
| **ブランチ** | `feature`, `bugfix` | `release` |
| **ビルドモード** | Debug | Release |
| **プロビジョニング** | Ad-Hoc | App Store |
| **環境** | テスト/ステージング | 本番 |
| **デバイス制限** | 100台 | 無制限 |
| **テスター管理** | Firebaseコンソール | App Store Connect |

---

## デプロイメントフロー

### Firebaseワークフロー（テスト環境）

```
未定（例：開発者からタグのプッシュ）
    ↓
GitHub Actionsがトリガー
    ↓
1. コードのチェックアウト
2. Rubyと依存関係のセットアップ
3. Ad-Hoc証明書の同期（Match）
4. アプリのビルド（Debugモード）
5. Firebase App Distributionへアップロード
    ↓
テスターが通知を受信
    ↓
登録済みデバイスにインストール
```

### TestFlightワークフロー（本番環境）

```
未定（例：開発者からタグのプッシュ）
    ↓
GitHub Actionsがトリガー
    ↓
1. コードのチェックアウト
2. Rubyと依存関係のセットアップ
3. App Store証明書の同期（Match）
4. アプリのビルド（Releaseモード）
5. App Store Connectへアップロード
6. TestFlightに送信
    ↓
Appleがビルドを処理（約5-15分）
    ↓
TestFlightでビルドが利用可能に
    ↓
テスターがTestFlightアプリ経由でインストール
```

---

## ワークフローのトリガー

### Firebase配信

**自動トリガー**:
```bash
# developブランチにプッシュ
git push origin develop
```

**手動トリガー**:
- GitHubのActionsタブに移動
- "Firebase Distribution"ワークフローを選択
- "Run workflow"をクリック
- `develop`ブランチを選択

### TestFlight配信

**自動トリガー**:
```bash
# mainブランチにプッシュ
git push origin main

# またはバージョンタグを作成
git tag v1.2.3
git push origin v1.2.3
```

**手動トリガー**:
- GitHubのActionsタブに移動
- "TestFlight Distribution"ワークフローを選択
- "Run workflow"をクリック
- `main`ブランチを選択

---

## ビルド成果物

### Firebaseビルド（Debug）
- **形式**: .ipaファイル（Ad-Hoc署名）
- **シンボル**: dSYMを含む
- **サイズ**: 大きい（デバッグシンボルを含む）
- **最適化**: なし
- **ログ**: 詳細

### TestFlightビルド（Release）
- **形式**: .ipaファイル（App Store署名）
- **シンボル**: dSYMは別途アップロード
- **サイズ**: 最適化済み
- **最適化**: 完全なコンパイラ最適化
- **ログ**: 本番レベル

---

## 環境変数とシークレット

両方のワークフローには以下のGitHub Secretsが必要です：

### 認証
- `APP_STORE_CONNECT_API_KEY_ID`: APIキー識別子
- `APP_STORE_CONNECT_ISSUER_ID`: 発行者識別子
- `APP_STORE_CONNECT_API_KEY`: Base64エンコードされた.p8キーファイル

### コード署名
- `MATCH_PASSWORD`: 証明書リポジトリのパスフレーズ
- `MATCH_GIT_BASIC_AUTHORIZATION`: Base64エンコードされたGit認証情報

### Firebase（Firebaseワークフローのみ）
- `FIREBASE_TOKEN`: Firebase CLI認証トークン
- `FIREBASE_APP_ID`: Firebaseアプリ識別子

---

## モニタリングとログ

### ワークフローステータスの確認
1. GitHubのリポジトリに移動
2. **Actions**タブをクリック
3. 特定のワークフロー実行を選択
4. 各ステップのログを表示

### ビルド成果物
失敗したビルドは自動的に以下をアップロード：
- ビルドログ
- Fastlaneレポート
- Gymログ（ビルド失敗時）

### 通知
- GitHubはワークフロー失敗時に自動通知
- オプション: Slack/メール通知の設定

---

## ベストプラクティス

### 開発ワークフロー
1. **機能開発**: 機能ブランチで作業
2. **テスト**: `develop`にマージ → Firebase配信をトリガー
3. **QA承認**: Firebase経由で登録済みデバイスでテスト
4. **本番**: `main`にマージ → TestFlight配信をトリガー
5. **最終テスト**: TestFlightベータテスト
6. **リリース**: App Store ConnectからApp Storeに申請

### ブランチ戦略
```
feature/* → develop → main → App Store
              ↓          ↓
           Firebase  TestFlight
           (Debug)   (Release)
```

### バージョン管理
- **ビルド番号**: Fastlaneによって自動インクリメント
- **バージョン番号**: Xcodeプロジェクトで手動更新
- **タグ付け**: リリースにはセマンティックバージョニング（v1.2.3）を使用

---

## トラブルシューティング

### よくある問題

**1. コード署名エラー**
```bash
# ローカルで証明書を再生成
bundle exec fastlane match appstore --force
bundle exec fastlane match adhoc --force
```

**2. Firebaseアップロード失敗**
- `FIREBASE_TOKEN`が有効か確認（トークンは30日後に期限切れ）
- `FIREBASE_APP_ID`が正しいか確認

**3. TestFlightアップロード失敗**
- App Store ConnectでAPIキーのアクセス権限を確認
- Bundle Identifierが一致しているか確認

**4. ビルドタイムアウト**
- 標準GitHubランナーには6時間のタイムアウトがある
- ビルドステップの最適化またはキャッシングを検討

### サポート
1. GitHub Actionsのワークフローログを確認
2. 詳細ログでFastlane出力をレビュー
3. すべてのGitHub Secretsが正しく設定されているか確認
4. ローカルでテスト: `bundle exec fastlane firebase` または `bundle exec fastlane beta`

---

## メンテナンス

### 定期タスク
- **月次**: 必要に応じてFirebaseトークンをローテーション
- **四半期**: テスターリストをレビューして更新
- **必要時**: ワークフローのXcodeバージョンを更新
- **必要時**: `bundle update`でRuby gemを更新

### 更新
XcodeまたはmacOSバージョンを更新する場合：
1. ワークフローYAMLファイルを新しいバージョン番号で更新
2. 新しいXcodeバージョンでローカルテスト
3. このドキュメントを更新

---

## セキュリティ上の考慮事項

- すべてのシークレットはGitHub Secretsに保存（暗号化済み）
- 証明書リポジトリはプライベート
- APIキーは必要最小限のアクセス権限
- Matchリポジトリへのアクセスは制限
- .p8ファイルはリポジトリにコミットしない
- ワークフローログは機密データを露出しない

---

## 追加リソース

- **GitHub Actionsドキュメント**: https://docs.github.com/ja/actions
- **Fastlaneドキュメント**: https://docs.fastlane.tools
- **Firebase App Distribution**: https://firebase.google.com/docs/app-distribution?hl=ja
- **TestFlight**: https://developer.apple.com/jp/testflight/

---

## ドキュメント情報

**最終更新日**: 2024年12月  
**管理者**: iOSデベロップメントチーム  
**バージョン**: 1.0