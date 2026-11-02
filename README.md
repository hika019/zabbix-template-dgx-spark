# DGX Spark GB10 templates for Zabbix

English | [日本語](README.ja.md)

Zabbix templates for monitoring the NVIDIA DGX Spark (GB10). There are two: `DGX Spark GB10 by Zabbix agent 2`, which covers the GPU, temperatures and unified memory together, and `DGX Spark GB10 Thermal`, a lightweight template for temperatures only.

On the DGX Spark, several reports on the NVIDIA developer forums show the GPU temperature from nvidia-smi reading 8–15 °C lower than the ACPI thermal zones, including cases where the unit powered off from heat while nvidia-smi still looked comfortable. Both templates therefore base their thermal alerts on the ACPI values.

## Templates

`DGX Spark GB10 by Zabbix agent 2` (`template/dgx_spark_gb10.yaml`) takes the GPU metrics from the official "Nvidia by Zabbix agent 2" template and adds the seven ACPI thermal zones and unified memory usage read from `/proc/meminfo`. FB memory and PCIe throughput, which GB10 does not report, are left out. It includes a six-page dashboard, "DGX Spark: Overview", covering GPU utilization, temperatures, power, memory, clocks, encoders and more.

`DGX Spark GB10 Thermal` (`template/dgx_spark_gb10_thermal.yaml`) monitors temperatures only. It reads `/sys/class/thermal/thermal_zone*/temp` with the agent's built-in `vfs.file.contents` key, so it needs no plugin, no extra scripts and no root privileges.

## Requirements

`DGX Spark GB10 by Zabbix agent 2` requires Zabbix server 7.2 or later and Zabbix agent 2 with the NVIDIA GPU plugin (`zabbix-agent2-plugin-nvidia-gpu`). The plugin was added in 7.2 and is not available in the Zabbix 7.0 repository. Install the agent from the repository matching your server's version series. `DGX Spark GB10 Thermal` works on Zabbix 7.0 or later with either agent or agent 2.

## Installation

Import the YAML from Data collection → Templates → Import and link it to your DGX Spark host. Link only one of them: the two templates and the official "Nvidia by Zabbix agent 2" share item keys, so they cannot be linked at the same time. If one is already linked, remove it with "Unlink and clear" before switching.

GPU discovery is driven by the "Get devices" item, which polls every hour as in the official template. Right after linking, the GPU items don't exist yet; select "Get devices" and click "Execute now" to discover them immediately.

## Differences from the official Nvidia template

GPU utilization is expected to stay high on an AI workload, so the GPU and memory utilization triggers are disabled by default by setting their macros to 101. The NVML temperature triggers are disabled the same way because they duplicate the ACPI temperature triggers. Lower the macros if you want them back. A Warning trigger is added for unified memory utilization staying above 95% for five minutes.

## Thresholds

Temperature thresholds are set with `{$SPARK.TEMP.WARN}` (default 85), `{$SPARK.TEMP.HIGH}` (default 92) and `{$SPARK.TEMP.HYST}` (default 5). Add a context such as `{$SPARK.TEMP.WARN:"zone5"}` to override a single zone. Warning fires when the threshold is exceeded continuously for one minute, High fires as soon as a single value exceeds it, and both recover once the temperature drops 5 °C below the threshold. The defaults are provisional values worked back from overheating shutdown reports on the forums, not NVIDIA specifications. It's best to watch your normal readings for a few days before tuning them. The unified memory threshold is `{$SPARK.MEM.PUSED.WARN}` (default 95).

## Known limitations

Item names assume the zones map in order to TSOC (SoC), TS0E, TS0P, TS1E, TS1P, TGPU (GPU) and TUNC. The meaning of the zones other than TSOC and TGPU is unknown. The mapping may differ between units, firmware versions or other vendors' GB10 systems, so check it with `sudo dmesg | grep "Thermal Zone"`. ECC errors, BAR1 memory and fan speed may not be reported on GB10, in which case those dashboard graphs stay empty.

Some trigger names and item descriptions are in Japanese.

## License

MIT
