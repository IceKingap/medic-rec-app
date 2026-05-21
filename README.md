[README.md](https://github.com/user-attachments/files/28081073/README.md)
# 薬服用記録 (MedicRec)

アプリを開いてボタン1回で、薬を飲んだ時間を記録できる Android アプリ。

## 主な機能

- 薬ごとに「名前 / 効果時間 / 1日の回数」を設定
- ホーム画面に薬ごとのボタンが並び、タップ1回で服用時刻を記録
- 前回服用から効果時間が経っていない状態でボタンを押すと **「まだ早い」警告** を表示
  - 正しい間隔ならボタン押下時に警告は出ない
  - 警告ダイアログから強制記録も可能
- 効果時間 + 30 分経過しても次の記録がない場合、**「飲み忘れていませんか?」通知**
- 服用履歴タブで過去の記録を一覧
- ボタンは「同じタブ」に並ぶので、タブ移動なしで「アプリ開いてボタン1回」が成立

## 技術スタック

- Kotlin 2.0.21 / Jetpack Compose (Material 3) / Navigation Compose
- Room (KSP) でローカル DB に永続化
- AlarmManager + BroadcastReceiver で飲み忘れ通知
- minSdk 26 / compileSdk 36 / targetSdk 36
- Android Gradle Plugin 8.13.0 / Gradle 9.2.1

## ディレクトリ構成

```
medic-rec-app/
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── gradle/wrapper/gradle-wrapper.properties
└── app/
    ├── build.gradle.kts
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/com/medicrec/app/
        │   ├── MedicRecApplication.kt
        │   ├── MainActivity.kt
        │   ├── data/                  # Room エンティティ・DAO・Repository
        │   ├── notification/          # アラーム & 通知
        │   └── ui/
        │       ├── theme/             # Material 3 テーマ
        │       ├── home/              # 記録タブ (メイン)
        │       ├── history/           # 履歴タブ
        │       └── settings/          # 薬の設定タブ
        └── res/                       # アイコン・文字列・テーマ
```

## ビルド方法

### Android Studio を使う (推奨)

1. Android Studio を起動し **Open** → このプロジェクトを開く
2. 初回オープン時に Gradle Wrapper の jar が自動でダウンロードされ、依存関係が解決される
3. ツールバーの **Run** ボタンで実機/エミュレータに実行、または **Build → Build Bundle(s)/APK(s) → Build APK(s)** で APK を生成

### コマンドラインから

Gradle Wrapper は同梱済みなので、ターミナルから直接ビルドできます:

```bash
cd "/Users/sato/AI projects/medic-rec-app"
./gradlew :app:assembleDebug    # デバッグ APK
./gradlew :app:assembleRelease  # リリース APK (デバッグ署名)
```

生成物:
- デバッグ: `app/build/outputs/apk/debug/app-debug.apk` (約 16MB)
- リリース: `app/build/outputs/apk/release/app-release.apk`

リリース署名を別途設定する場合は `app/build.gradle.kts` の `signingConfig` を編集してください。実機 / エミュレータへのインストールは:

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## 使い方

1. アプリ初回起動時に通知権限を許可
2. 下部ナビ「薬の設定」タブで **+** ボタンから薬を追加
   - 例: 「ロキソニン」効果時間 6 時間 / 1日3回
3. 「記録」タブに戻ると、登録した薬のボタンが表示される
4. 飲んだらボタンを1回タップで時刻が記録される
5. まだ早いタイミングで押すと警告ダイアログ。問題なければそのまま閉じる、本当に飲むなら「それでも記録する」

## 制限事項

- リリース署名は debug キーで仮設定されています。Play Store などに公開する場合は別途設定が必要です
- 通知は端末スリープ中も発火させるため `USE_EXACT_ALARM` を使用しています (API 33+ では医薬リマインダー用途として許可されます)
- バックアップは Auto Backup で `medic_rec.db` を対象に含めています
