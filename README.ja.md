# DGX Spark GB10 Thermal for Zabbix

[English](README.md) | 日本語

NVIDIA DGX Spark（GB10）の ACPI thermal zone 7 か所の温度を Zabbix で監視するためのテンプレートです。Zabbix エージェント標準の `vfs.file.contents` キーで `/sys/class/thermal/thermal_zone*/temp` を読むだけなので、追加のスクリプトや root 権限、常駐プロセスは必要ありません。

DGX Spark では、nvidia-smi が報告する GPU 温度が ACPI の thermal zone より 8〜15℃ 低く出るという報告が NVIDIA のフォーラムで複数あり、nvidia-smi 上は余裕があるように見えたまま熱で電源断に至った例もあります。このテンプレートは、熱の監視とアラートを ACPI 側の値で行うために作りました。GPU 使用率や消費電力が必要な場合は、公式の「Nvidia by Zabbix agent 2」テンプレートと併用してください。

## 内容

thermal zone 0〜6 のアイテム（15 秒間隔、℃換算済み）、各ゾーンの Warning と High のトリガー、全ゾーンを重ねたグラフが含まれています。Warning はしきい値を 1 分間超え続けたときに、High は 1 回でもしきい値を超えたときに発報し、どちらも 5℃ 下がってから復旧します。High の発報中は Warning を抑制します。

## 導入

Zabbix 7.0 以降で、データ収集 → テンプレート → インポートから `template/dgx_spark_gb10_thermal.yaml` を読み込み、DGX Spark のホストにリンクしてください。エージェントは Zabbix agent でも agent 2 でも動きます。同じキーのアイテムをホストに手作業で作っている場合は、リンクの前に削除してください。

## しきい値

しきい値はマクロで変更できます。`{$SPARK.TEMP.WARN}`（既定 85）、`{$SPARK.TEMP.HIGH}`（既定 92）、`{$SPARK.TEMP.HYST}`（既定 5）の 3 つがあり、`{$SPARK.TEMP.WARN:"zone5"}` のようにコンテキストを付ければゾーンごとに上書きできます。既定値はフォーラムでの熱落ち報告から逆算した暫定値で、NVIDIA の公式な仕様ではありません。数日運用して平常時の値を確認してから調整することをおすすめします。

## 既知の制約

zone 番号と ACPI 名の対応は、TSOC（SoC）、TS0E、TS0P、TS1E、TS1P、TGPU（GPU）、TUNC の順という想定で名前を付けています。TSOC と TGPU 以外の意味は分かっていません。また、個体やファームウェア、他社製の GB10 搭載機では対応が異なる可能性があるため、`sudo dmesg | grep "Thermal Zone"` で確認し、違っていればアイテム名を直してください。

## ライセンス

MIT
