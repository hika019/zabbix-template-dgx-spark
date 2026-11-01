# DGX Spark GB10 Thermal for Zabbix

English | [日本語](README.ja.md)

A Zabbix template for monitoring the seven ACPI thermal zones of the NVIDIA DGX Spark (GB10). It reads `/sys/class/thermal/thermal_zone*/temp` with the agent's built-in `vfs.file.contents` key, so it needs no extra scripts, no root privileges and no background processes.

On the DGX Spark, several reports on the NVIDIA developer forums show the GPU temperature from nvidia-smi reading 8–15 °C lower than the ACPI thermal zones, including cases where the unit powered off from heat while nvidia-smi still looked comfortable. This template was made so that thermal monitoring and alerting are based on the ACPI values instead. For GPU utilization and power draw, use it alongside the official "Nvidia by Zabbix agent 2" template.

## What's included

Items for thermal zones 0–6 (15-second interval, converted to °C), a Warning and a High trigger for each zone, and one graph overlaying all zones. Warning fires when the threshold is exceeded continuously for one minute, High fires as soon as a single value exceeds its threshold, and both recover once the temperature drops 5 °C below the threshold. Warning is suppressed while High is active.

## Installation

On Zabbix 7.0 or later, import `template/dgx_spark_gb10_thermal.yaml` from Data collection → Templates → Import, then link it to your DGX Spark host. It works with both Zabbix agent and agent 2. If you have already created items with the same keys directly on the host, delete them before linking.

## Thresholds

Thresholds are set with three macros: `{$SPARK.TEMP.WARN}` (default 85), `{$SPARK.TEMP.HIGH}` (default 92) and `{$SPARK.TEMP.HYST}` (default 5). Add a context such as `{$SPARK.TEMP.WARN:"zone5"}` to override a single zone. The defaults are provisional values worked back from overheating shutdown reports on the forums, not NVIDIA specifications. It's best to watch your normal readings for a few days before tuning them.

## Known limitations

Item names assume the zones map in order to TSOC (SoC), TS0E, TS0P, TS1E, TS1P, TGPU (GPU) and TUNC. The meaning of the zones other than TSOC and TGPU is unknown. The mapping may also differ between units, firmware versions or other vendors' GB10 systems, so check it with `sudo dmesg | grep "Thermal Zone"` and rename the items if it doesn't match.

## License

MIT
