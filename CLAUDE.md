# OpenTelemetry Java Log4j Appender 統合プロジェクト

## プロジェクト概要

このプロジェクトは、**OpenTelemetry Java SDK と Log4j Appender の統合**について学習・理解するためのサンプルプロジェクトです。

### 調査対象
- 最新のOpenTelemetry Javaライブラリ（バージョン2.18.1-alpha）
- ライブラリ計装（ゼロコード計装ではなく）
- Log4j Appender機能を使ったログテレメトリー
- Java内のインスタンス関係性とデータフロー

### 技術仕様
- **Java**: 17（Java 8以上対応）
- **OpenTelemetry SDK**: 1.53.0
- **OpenTelemetry Instrumentation**: 2.18.1-alpha
- **Log4j**: 2.21.1
- **Maven**: 3.x

---

## ファイル構成

```
/home/wsl-user/projects/otel-java/
├── pom.xml                              # Maven設定（依存関係定義）
├── Dockerfile                           # Podman/Docker用コンテナ定義
├── podman-compose.yml                   # Podman Compose設定
├── src/main/
│   ├── java/com/example/otel/
│   │   └── LoggingExample.java          # メインサンプルプログラム
│   └── resources/
│       └── log4j2.xml                   # Log4j設定（OpenTelemetry Appender含む）
├── OpenTelemetry-Log4j-説明資料.md       # 詳細技術解説
├── generated-diagrams/                  # アーキテクチャ図
│   ├── otel-java-data-flow-en.png.png              # データフロー図
│   └── otel-java-instance-relationships-en.png.png # インスタンス関係性図
└── CLAUDE.md                           # このファイル
```

---

## クイックスタート

### 1. 依存関係のインストール
```bash
mvn clean compile
```

### 2. サンプルプログラムの実行
```bash
mvn exec:java
```

### 3. OpenTelemetry Collector（オプション）
ログデータを実際に受信したい場合は、別途OpenTelemetry Collectorを起動してください：
```bash
# docker-compose.yml を作成して Collector を起動
docker-compose up
```

---

## Podman での実行方法

### 前提条件
- **既存のOpenTelemetry Collector**がlocalhostで稼働中
- Collectorのポート4317でOTLPレシーバーが待機
- podman-composeがインストール済み

### Podman実行手順

#### 1. コンテナイメージのビルドと実行
```bash
# プロジェクトディレクトリで実行
cd /home/wsl-user/projects/otel-java

# podman-composeでビルド・実行
podman-compose up --build
```

#### 2. バックグラウンド実行
```bash
# デタッチモードで実行（バックグラウンド）
podman-compose up -d --build
```

#### 3. ログの確認
```bash
# リアルタイムログ表示
podman-compose logs -f otel-java-app

# 特定時刻以降のログのみ表示
podman-compose logs --since="10m" otel-java-app
```

#### 4. コンテナの停止・削除
```bash
# 停止
podman-compose down

# ボリュームも含めて完全削除
podman-compose down -v
```

### Podman固有の設定

#### ネットワーク設定オプション

**オプション1: host ネットワーク使用（推奨）**
```yaml
# podman-compose.yml での設定
network_mode: host
```
- 既存のCollectorに直接localhost経由でアクセス
- 最もシンプルで確実な方法

**オプション2: カスタムネットワーク**
```yaml
networks:
  otel-network:
    driver: bridge
```
- より分離されたネットワーク環境
- `host.containers.internal` でホストにアクセス

#### 環境変数のカスタマイズ
```bash
# 環境変数ファイルを使用
echo "OTEL_EXPORTER_OTLP_LOGS_ENDPOINT=http://localhost:4317" > .env
podman-compose --env-file .env up
```

### Podman デバッグコマンド

```bash
# コンテナ内部に入ってデバッグ
podman-compose exec otel-java-app bash

# コンテナの詳細情報確認
podman inspect otel-java-log4j-example

# Podmanネットワークの確認
podman network ls
podman network inspect bridge
```

---

## 学習のポイント

### 🔍 重要な学習項目

1. **Log4j Appenderの仕組み**
   - `log4j2.xml` での設定方法（91-102行目）
   - `OpenTelemetryAppender.install()` による統合（LoggingExample.java:98行目）

2. **OpenTelemetry SDK の初期化**
   - `Resource` による サービス識別（LoggingExample.java:125-130行目）
   - `BatchLogRecordProcessor` によるバッチ処理（LoggingExample.java:156-161行目）
   - `OtlpGrpcLogRecordExporter` による外部送信（LoggingExample.java:145-149行目）

3. **トレースとログの関連付け**
   - `Span.makeCurrent()` によるコンテキスト設定（LoggingExample.java:226行目）
   - 自動的な trace_id, span_id の付与仕組み

4. **構造化ログ**
   - `MapMessage` による属性付きログ（LoggingExample.java:174-188行目）
   - `ThreadContext` による スレッド固有情報（LoggingExample.java:156-158行目）

### 📚 参考ドキュメント

- **`OpenTelemetry-Log4j-説明資料.md`**: 技術的詳細とインスタンス関係性の完全解説
- **`LoggingExample.java`**: 実装例とベストプラクティス
- **図表**: データフローとアーキテクチャの視覚的理解

---

## 開発コマンド

### Maven コマンド
```bash
# 依存関係の確認
mvn dependency:tree

# コンパイルのみ
mvn compile

# テスト実行（テストがある場合）
mvn test

# プロジェクトのクリーンアップ
mvn clean
```

### デバッグ実行
```bash
# デバッグ情報を有効にして実行
mvn exec:java -Dlog4j2.debug=true
```

---

## トラブルシューティング

### よくある問題

#### 1. ログがOpenTelemetryに送信されない
**チェックポイント**:
- `OpenTelemetryAppender.install(sdk)` が実行されているか
- `log4j2.xml` で `AppenderRef` が設定されているか
- OpenTelemetry SDK が適切に初期化されているか

#### 2. 文字化けが発生する
**対処法**:
```bash
# UTF-8エンコーディングで実行
export JAVA_TOOL_OPTIONS="-Dfile.encoding=UTF-8"
mvn exec:java
```

#### 3. 外部システムへの送信エラー
**確認事項**:
- エンドポイントURL（`http://localhost:4317`）が正しいか
- OpenTelemetry Collector が起動しているか
- ネットワーク接続に問題がないか

---

## 学習進行の推奨順序

1. **📖 概念理解**: `OpenTelemetry-Log4j-説明資料.md` を読む
2. **🎯 コード理解**: `LoggingExample.java` のコメントを追いながらコードを読む
3. **🔧 設定理解**: `log4j2.xml` と `pom.xml` の設定を確認
4. **▶️ 実行確認**: `mvn exec:java` で実際に動作させる
5. **🎨 アーキテクチャ**: 図表でデータフローとインスタンス関係性を理解

---

## 関連リソース

### 公式ドキュメント
- [OpenTelemetry Java Documentation](https://opentelemetry.io/docs/languages/java/)
- [OpenTelemetry Java Instrumentation](https://github.com/open-telemetry/opentelemetry-java-instrumentation)
- [Log4j Appender README](https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/main/instrumentation/log4j/log4j-appender-2.17/library/README.md)

### Maven Repository
- [OpenTelemetry Log4j Appender](https://mvnrepository.com/artifact/io.opentelemetry.instrumentation/opentelemetry-log4j-appender-2.17)
- [OpenTelemetry SDK](https://mvnrepository.com/artifact/io.opentelemetry/opentelemetry-sdk)

---

## プロジェクトの特徴

✅ **完全なサンプル**: 実際に動作する完全なコード例  
✅ **詳細コメント**: 初心者向けの丁寧な説明  
✅ **最新バージョン**: 2025年最新の安定版を使用  
✅ **ベストプラクティス**: 推奨される実装パターンに準拠  
✅ **視覚的理解**: アーキテクチャ図でのデータフロー可視化  

このプロジェクトを通して、OpenTelemetryとLog4jの統合による**可観測性の実装**を実践的に学習できます。