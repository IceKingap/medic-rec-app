[README.md](https://github.com/user-attachments/files/28088315/README.md)
# 薬服用記録 (MedicRec)

アプリを開いてボタン1回で、薬を飲んだ時間を記録できる Android アプリ。

## 主な機能

### 記録
- 薬ごとに「名前 / 効果時間 / 1日の回数」を登録
- ホーム画面に薬ごとのボタンが並び、**タップ1回で服用時刻を記録**
- 「同じタブ」に複数の薬が並ぶので、タブ移動なしで「アプリ開いてボタン1回」が成立
- 記録後は Snackbar に **「取り消す」アクション** が表示され、誤タップを即座に巻き戻せる
- カード右上の × アイコンからも直前の記録を取り消し可能

### 警告 / 通知
- 前回服用から効果時間が経っていない状態でボタンを押すと **「まだ早い」警告ダイアログ**
  - 正しい間隔ならボタン押下時に警告は出ず、Snackbar に記録完了のみ表示
  - 警告ダイアログから「それでも記録する」で強制記録も可能
- 効果時間 + 30 分経過しても次の記録がない場合、**「飲み忘れていませんか?」通知**
- 通知の **「飲んだ」アクション** をタップすると、アプリを開かずに記録 + 次回リマインダーの再スケジュール

### ホーム画面
- 上部に **本日の服用サマリーカード** (全薬合計の進捗バー、`X/Y回(Z%)`)
- 「まだ早い」状態のカードは **amber 警告色** (エラーではないので赤は使わない)
- 薬未登録時は薬アイコン + 詳細説明文付きのオンボーディング表示

### カレンダー (履歴)
- 月単位で服用回数を可視化
- 各日のセルに **服用回数分のドット** (4回以上は `+N` 表記)
- 今日の日付に primary 枠線
- 月切替で **当月の達成率サマリーカード** が更新 (`服用回数 / 予定回数 (% )`)
  - 予定回数は各薬の登録日以降の日数のみ加算
- 日付をタップするとその日の服用記録一覧、各記録に削除ボタン

### 設定
- 薬の追加 / 編集 / 削除
- **飲み忘れ通知の ON/OFF トグル** (OFF にすると全アラームをキャンセル、ON で再スケジュール)
- **起動時に常に「記録」タブを開く** のトグル (OFF の場合は最後に開いていたタブを復元)

## 技術スタック

- Kotlin 2.0.21 / Jetpack Compose (Material 3) / Navigation Compose
- Room (KSP) でローカル DB に永続化
- SharedPreferences (`AppPreferences`) でアプリ設定を保持
- AlarmManager + BroadcastReceiver で飲み忘れ通知
- `QuickRecordReceiver` で通知からのクイック記録
- minSdk 26 / compileSdk 36 / targetSdk 36
- Android Gradle Plugin 8.13.0 / Gradle 9.2.1

## ディレクトリ構成

```
medic-rec-app/
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── gradlew / gradlew.bat
├── gradle/wrapper/                       # Gradle Wrapper (jar 同梱)
└── app/
    ├── build.gradle.kts
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/com/medicrec/app/
        │   ├── MedicRecApplication.kt    # DI ルート (DB / Repository / Preferences)
        │   ├── MainActivity.kt           # 通知権限リクエスト + Compose ルート
        │   ├── data/                     # Room エンティティ・DAO・Repository・AppPreferences
        │   ├── notification/             # アラーム / 通知 / ブート再スケジュール / クイック記録
        │   └── ui/
        │       ├── MedicRecApp.kt        # ボトムナビ + NavHost (フェード遷移)
        │       ├── theme/                # Material 3 テーマ + WarningColors (amber)
        │       ├── home/                 # 記録タブ (メイン)
        │       ├── history/              # カレンダータブ
        │       ├── settings/             # 薬・アプリ設定タブ
        │       └── util/Format.kt        # formatHours / formatRemaining 共通関数
        └── res/                          # アイコン・文字列・テーマ
```

### 主要クラスの責務

| ファイル | 役割 |
|---|---|
| `data/AppDatabase.kt` | Room データベース (`medic_rec.db`) |
| `data/Medication.kt` / `DoseRecord.kt` | エンティティ |
| `data/MedicationRepository.kt` | 薬・服用記録の CRUD と「本日の服用回数」集計 |
| `data/AppPreferences.kt` | 通知 ON/OFF・起動時タブ・最後に開いたタブの永続化 |
| `notification/ReminderScheduler.kt` | AlarmManager 用 PendingIntent 構築・過去時刻ガード |
| `notification/ReminderRefresh.kt` | 最新の服用記録に基づくリマインダー再設定 (3 ViewModel で共通化) |
| `notification/ReminderReceiver.kt` | 通知発火 (本文 + 「飲んだ」アクション) |
| `notification/QuickRecordReceiver.kt` | 通知からの即時記録 → 通知消去 → 次回リマインダー設定 |
| `notification/BootReceiver.kt` | 端末再起動・アプリ更新時の全アラーム再スケジュール |
| `ui/home/HomeScreen.kt` + `HomeViewModel.kt` | 記録タブ、本日サマリーカード、警告ダイアログ、取り消し |
| `ui/history/HistoryScreen.kt` | カレンダー表示、月間達成率、日別記録、削除 |
| `ui/settings/SettingsScreen.kt` | 薬編集ダイアログ、通知 ON/OFF、起動時タブ設定 |

## ビルド方法

### Android Studio を使う (推奨)

1. Android Studio を起動し **Open** → このプロジェクトを開く
2. 初回オープン時に依存関係が解決される
3. ツールバーの **Run** ボタンで実機/エミュレータに実行、または **Build → Build Bundle(s)/APK(s) → Build APK(s)** で APK を生成

### コマンドラインから

Gradle Wrapper は同梱済みなので、ターミナルから直接ビルドできます:

```bash
cd "/Users/sato/AI projects/medic-rec-app"
./gradlew :app:assembleDebug    # デバッグ APK
./gradlew :app:assembleRelease  # リリース APK (デバッグ署名)
```

生成物:
- デバッグ: `app/build/outputs/apk/debug/app-debug.apk` (約 17MB)
- リリース: `app/build/outputs/apk/release/app-release.apk`

実機 / エミュレータへのインストール:

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

リリース署名を別途設定する場合は `app/build.gradle.kts` の `signingConfig` を編集してください。

## 使い方

1. アプリ初回起動時に通知権限を許可
2. 下部ナビ「薬の設定」タブで **+** ボタンから薬を追加
   - 例: 「ロキソニン」効果時間 6 時間 / 1日3回
3. 「記録」タブに戻ると、登録した薬のボタンと本日サマリーが表示される
4. 飲んだらボタンを1回タップで時刻が記録される
   - 誤タップは Snackbar の「取り消す」かカード右上の × で即取り消し
5. まだ早いタイミングで押すと amber の警告ダイアログ。問題なければキャンセル、本当に飲むなら「それでも記録する」
6. 飲み忘れ通知が来たら、通知の「飲んだ」ボタンでアプリを開かずに記録可能

設定で必要に応じて:
- 「飲み忘れ防止通知」OFF にすると全アラームが停止
- 「起動時は記録タブを開く」OFF にすると、最後に開いていたタブで再開

## データの取り扱い

- すべて端末ローカル (`medic_rec.db`) に保存
- Auto Backup で `medic_rec.db` をバックアップ対象に含める
- データ移行を伴うスキーマ変更は現状なし

## 制限事項

- リリース署名は debug キーで仮設定されています。Play Store などに公開する場合は別途設定が必要です
- 通知は端末スリープ中も発火させるため `USE_EXACT_ALARM` を使用しています (API 33+ では医薬リマインダー用途として許可されます)
- 薬の「曜日指定」「朝・昼・夜の時間帯指定」「CSV エクスポート」「ホームウィジェット」「薬ごとの色/アイコン」は未実装
