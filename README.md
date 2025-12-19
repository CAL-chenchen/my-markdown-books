# Markdown Playgorund
<!-- ===================================================== -->
<!-- 🌈 Project README — Powered by GitBook -->
<!-- ===================================================== -->

<p align="center">
  <img src="https://images.unsplash.com/photo-1519389950473-47ba0277781c"
       alt="cover"
       width="100%" />
</p>

<h1 align="center">🚀 iOS CI/CD Knowledge Base</h1>

<p align="center">
  <strong>
    Fastlane × GitHub Actions × Firebase / TestFlight<br/>
    ― 実践ベースでまとめた iOS 自動化ナレッジ集 ―
  </strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/iOS-CI%2FCD-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Fastlane-Ready-success?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/GitHub_Actions-Automated-black?style=for-the-badge"/>
</p>

---

## ✨ このリポジトリについて

このリポジトリは、**iOS アプリの CI/CD 構築・運用に関する知見を体系的にまとめた GitBook 用ナレッジベース**です。

- 実際のプロダクト運用で得られたノウハウ
- 証明書・プロビジョニングの落とし穴
- Fastlane / GitHub Actions の設計思想
- チームで安全に回すためのベストプラクティス

を中心に、「**あとから来た人が読んで理解できる**」ことを最優先に整理しています。

---

## 🧭 対象読者

- iOS CI/CD をこれから整備したい方
- Fastlane / GitHub Actions を運用している開発者
- 証明書・App Store Connect 周りで苦しんだ経験のある方
- 属人化しない iOS 運用を目指すチーム

---

## 🏗 全体構成イメージ

```mermaid
flowchart LR
    Dev[Developer] -->|Push / Tag| GHA[GitHub Actions]
    GHA --> Fastlane
    Fastlane --> Firebase[Firebase App Distribution]
    Fastlane --> TestFlight[TestFlight]
    Fastlane --> ASC[App Store Connect]

