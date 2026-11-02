# DGX Spark GB10 templates for Zabbix

[English](README.md) | 日本語

NVIDIA DGX Spark（GB10）を Zabbix で監視するためのテンプレートです。GPU・温度・ユニファイドメモリをまとめて見る `DGX Spark GB10 by Zabbix agent 2` と、温度だけを軽く監視する `DGX Spark GB10 Thermal` の 2 つがあります。

DGX Spark では、nvidia-smi が報告する GPU 温度が ACPI の thermal zone より 8〜15℃ 低く出るという報告が NVIDIA のフォーラムで複数あり、nvidia-smi 上は余裕があるように見えたまま熱で電源断に至った例もあります。どちらのテンプレートも、熱のアラートは ACPI 側の値で出すようにしています。

## テンプレート

`DGX Spark GB10 by Zabbix agent 2`（`template/dgx_spark_gb10.yaml`）は、公式の「Nvidia by Zabbix agent 2」の GPU メトリクスに、ACPI thermal zone 7 か所の温度と、`/proc/meminfo` から取るユニファイドメモリの使用量を加えたものです。GB10 では値が返らない FB メモリと PCIe スループットは除いてあります。6 ページのダッシュボード「DGX Spark: Overview」が付いていて、GPU 使用率、温度、電力、メモリ、クロック、エンコーダーなどを一通り確認できます。

`DGX Spark GB10 Thermal`（`template/dgx_spark_gb10_thermal.yaml`）は温度だけのテンプレートです。エージェント標準の `vfs.file.contents` キーで `/sys/class/thermal/thermal_zone*/temp` を読むだけなので、プラグインや追加のスクリプト、root 権限は必要ありません。

## 必要なもの

`DGX Spark GB10 by Zabbix agent 2` には、Zabbix server 7.2 以降と、NVIDIA GPU プラグイン（`zabbix-agent2-plugin-nvidia-gpu`）を入れた Zabbix agent 2 が必要です。このプラグインは 7.2 で追加されたもので、Zabbix 7.0 のリポジトリにはありません。エージェントはサーバーと同じバージョン系列のリポジトリから入れてください。`DGX Spark GB10 Thermal` は Zabbix 7.0 以降なら、agent と agent 2 のどちらでも動きます。

## 導入

データ収集 → テンプレート → インポートから YAML を読み込み、DGX Spark のホストにリンクしてください。どちらか片方だけをリンクします。2 つのテンプレート、および公式の「Nvidia by Zabbix agent 2」は同じキーのアイテムを持っているため、同時にはリンクできません。すでにどれかをリンクしている場合は、「リンクと保存データを削除」で外してから付け替えてください。

GPU の自動検出は「Get devices」アイテムをきっかけに動き、その監視間隔は公式テンプレートと同じ 1 時間です。リンクした直後は GPU のアイテムがまだ作られていないので、「Get devices」を選んで「今すぐ実行」すると、すぐに検出されます。

## 公式 Nvidia テンプレートとの違い

AI 用途では GPU 使用率が常に高いのが普通なので、GPU 使用率とメモリ使用率のトリガーは、マクロを 101 にして既定で無効にしています。NVML の温度トリガーも、ACPI の温度トリガーと重複するため同じく無効にしています。必要ならマクロを下げてください。ユニファイドメモリの使用率が 5 分間 95% を超えたときの Warning トリガーを追加しています。

## しきい値

温度のしきい値は `{$SPARK.TEMP.WARN}`（既定 85）、`{$SPARK.TEMP.HIGH}`（既定 92）、`{$SPARK.TEMP.HYST}`（既定 5）で変更できます。`{$SPARK.TEMP.WARN:"zone5"}` のようにコンテキストを付ければゾーンごとに上書きできます。Warning はしきい値を 1 分間超え続けたときに、High は 1 回でも超えたときに発報し、どちらも 5℃ 下がってから復旧します。既定値はフォーラムでの熱落ち報告から逆算した暫定値で、NVIDIA の公式な仕様ではありません。数日運用して平常時の値を確認してから調整することをおすすめします。ユニファイドメモリのしきい値は `{$SPARK.MEM.PUSED.WARN}`（既定 95）です。

## 既知の制約

zone 番号と ACPI 名の対応は、TSOC（SoC）、TS0E、TS0P、TS1E、TS1P、TGPU（GPU）、TUNC の順という想定で名前を付けています。TSOC と TGPU 以外の意味は分かっていません。個体やファームウェア、他社製の GB10 搭載機では対応が異なる可能性があるため、`sudo dmesg | grep "Thermal Zone"` で確認してください。また、ECC エラー、BAR1 メモリ、ファン速度などは GB10 で値が返らない可能性があり、その場合はダッシュボードの該当グラフが空になります。

トリガー名とアイテムの説明の一部は日本語です。

## ライセンス

MIT
