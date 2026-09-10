# 動作確認ビルド: アクティブモード ローカルポート範囲指定

社内AWS環境でFFFTPをアクティブモードのFTPクライアントとして使う際、データ接続の
送信元ポートがOS任せのランダムポートになってしまい、AWSセキュリティグループ側で
送信元ポートによる絞り込みができないために通信できない問題への対応検証用ビルドです。
FileZillaの「Limit local ports used by FileZilla」と同等の機能を追加しています。

## 変更点

- `common.h` : 設定値 `PortRangeEnabled` / `PortRangeMin` / `PortRangeMax` を追加
- `registry.cpp` : 上記3値をレジストリ(`HKEY_CURRENT_USER\Software\Sota\FFFTP`)に
  読み書きするテーブルに追加
- `connect.cpp` の `GetFTPListenSocket()` : `PortRangeEnabled` が有効な場合、
  `PortRangeMin`〜`PortRangeMax` の範囲内でbind可能なポートを探して
  アクティブモードのデータ接続用リスニングソケットを作成するよう分岐を追加

## この検証ビルドの制約(重要)

- **設定画面(GUI)はまだ実装していません。** 値はレジストリで直接設定します。
  同梱の `portrange_verify.reg` をダブルクリックしてインポートしてから
  FFFTPを起動してください(デフォルトは社内実績のある `50000-50100` )。
- 別の範囲で試したい場合は `portrange_verify.reg` の
  `PortRangeMin`(16進) / `PortRangeMax`(16進) を書き換えてから
  再度インポートしてください。
- 無効化するには `PortRangeEnabled` を `0` にしてインポートし直すか、
  `regedit` で直接書き換えてください。
- あくまで動作確認用の一時的な改造ビルドであり、正式なリリース版ではありません。
  本採用する場合は、GUI追加やアップストリームへのフィードバックを別途検討します。

## ビルド方法

本リポジトリの `HowToBuild`(GitHub Wiki)の手順どおりです。
Visual Studio 2022 + Windows 11 SDK + サブモジュール(Boost.Regex, GSL)を
用意した上でソリューションをビルドしてください。GitHub Actions
(`.github/workflows/main.yml`, `windows-2022` ランナー)でも同様にビルドできます。
