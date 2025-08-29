# [Handoff] "OpenTelemetryの調査" — 2025-08-29 16:52 (branch: (no-git))

## Goal / Scope
**やること**：
- OpenTelemetry Java + Log4j Appender統合の完全理解
- ライブラリ計装（ゼロコード計装ではない）
- Log4j AppenderベースでのログテレメトリーLog API直接使用禁止
- Javaインスタンス関係性・データフロー重点調査
- 詳細コメント付きサンプル作成
- 技術解説資料・図表作成

**やらないこと**：
- OpenTelemetry Collector調査
- ゼロコード計装（Javaエージェント）

## Key decisions
- ✅ **最新バージョン採用**: opentelemetry-instrumentation 2.18.1-alpha使用（2025年8月最新）
- ✅ **Log4j Appender使用**: Logs API直接呼び出し回避、既存コード無変更
- ✅ **詳細コメント**: Java初心者向けに1行ずつ解説
- ✅ **Podman対応**: Docker代替としてpodman-compose.yml作成
- ✅ **英語図表**: 文字化け回避のため英語ラベル使用

## Done / Pending
**完了済み**：
- ✅ 最新バージョン調査（2.18.1-alpha, SDK 1.53.0）
- ✅ Maven設定（pom.xml）
- ✅ Log4j設定（log4j2.xml）
- ✅ サンプルプログラム（LoggingExample.java）500行超の詳細コメント
- ✅ 技術解説資料（OpenTelemetry-Log4j-説明資料.md）
- ✅ アーキテクチャ図2種（データフロー・インスタンス関係性）
- ✅ Podman設定（Dockerfile, podman-compose.yml）
- ✅ プロジェクト説明書（CLAUDE.md）
- ✅ リソースコンテキスト付与メカニズム調査・追加

**未完了**：なし

## Next actions
1. **実行確認**: `mvn exec:java` でサンプル動作確認（140字以内）
2. **Podman実行**: `podman-compose up --build` でコンテナ実行テスト
3. **Collector連携**: localhost:4317でCollector起動後の統合テスト
4. **カスタマイズ**: 自分のアプリケーションへの設定適用
5. **パフォーマンス**: バッチサイズ・送信間隔の最適化テスト

## Affected files
- **`pom.xml`** - Maven依存関係定義
- **`src/main/java/com/example/otel/LoggingExample.java`** - メインサンプル
- **`src/main/resources/log4j2.xml`** - Log4j設定
- **`Dockerfile`** - Podman/Dockerコンテナ定義  
- **`podman-compose.yml`** - Compose設定
- **`OpenTelemetry-Log4j-説明資料.md`** - 技術解説
- **`CLAUDE.md`** - プロジェクト説明
- **`generated-diagrams/*.png`** - アーキテクチャ図

## Affected symbols
- **`LoggingExample#main()`** - エントリーポイント
- **`LoggingExample#initializeOpenTelemetry()`** - SDK初期化
- **`OpenTelemetryAppender.install()`** - Log4j統合
- **`SdkLoggerProvider.builder().setResource()`** - リソース関連付け
- **`BatchLogRecordProcessor`** - バッチ処理
- **`OtlpGrpcLogRecordExporter`** - OTLP送信

## Repro / Commands
```bash
# 依存関係インストール
mvn clean compile

# サンプル実行  
mvn exec:java

# Podman実行
podman-compose up --build

# デバッグ実行
mvn exec:java -Dlog4j2.debug=true
```

## Risks / Unknowns
- **alphaバージョン使用**: 2.18.1-alphaの本番利用リスク→安定版確認必要
- **Podman互換性**: host.containers.internal サポート状況→network_mode: host で回避済み
- **Collector互換性**: OTLP 1.53.0形式との互換性→既存環境での検証必要
- **メモリ使用量**: バッチ処理設定の本番環境最適化→負荷テスト推奨

## Links
- [OpenTelemetry Java Log4j Appender](https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/main/instrumentation/log4j/log4j-appender-2.17/library/README.md)
- [Maven Repository](https://mvnrepository.com/artifact/io.opentelemetry.instrumentation/opentelemetry-log4j-appender-2.17)
- [OpenTelemetry Java Examples](https://github.com/open-telemetry/opentelemetry-java-examples/blob/main/log-appender/README.md)