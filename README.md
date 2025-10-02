# Watch Your Temps

> **Why this exists / Зачем это нужно**
>
> **EN:** I needed a quick, readable temperature view for a home Proxmox server. The raw outputs of `sensors` and `smartctl` were noisy and hard to scan in `watch`, so I made a one‑liner that renders a compact, colorized table that’s easy to scan at a glance.
>
> **RU:** Мне нужен был быстрый и наглядный мониторинг температур на домашнем сервере Proxmox. Штатные выводы `sensors` и `smartctl` перегружены и плохо читаются в `watch`, поэтому я сделал однострочник, который выводит компактную цветную таблицу, понятную с первого взгляда.

Colorized CPU & SSD temperatures in your terminal (`lm-sensors` + `smartctl`) via `watch`. Zero install, one‑liner friendly.

Цветной и удобочитаемый вывод температур CPU и SSD в терминале (`lm-sensors` + `smartctl`) через `watch`. Не требует установки скриптов — один однострочник.

<img width="728" height="409" alt="изображение" src="https://github.com/user-attachments/assets/37487a68-128d-4c89-b249-25ed5b164760" />

---

## English

### Features

* Compact, aligned table for CPU (Package & per‑core) and ACPI temp.
* SSD current temperature (SCT), min/max lines preserved.
* Color thresholds: green < warn, yellow ≥ warn, red ≥ crit.
* Pure one‑liner (no files, no functions), works well in `tmux` / Proxmox shell.

### Requirements

* Debian/Ubuntu/Proxmox‑like systems: `lm-sensors`, `smartmontools`, `bash`, `perl`.

Install dependencies on Debian/Ubuntu/Proxmox with:
```bash
sudo apt update && sudo apt install -y lm-sensors smartmontools
```

### Quick start (one‑liner, with degree symbol)

> Requires UTF‑8 locale in your terminal (see Troubleshooting below).

```bash
watch -c -n1 --no-title bash -lc 'date "+%F %T"; echo "=== CPU ==="; sensors 2>/dev/null | perl -Mstrict -Mwarnings -ne '\''BEGIN{ printf "%-16s %8s %8s %8s\n","Sensor","Temp","High","Crit" } our $in_acpi=0; if(/^Adapter:\s*ACPI interface/){ $in_acpi=1; } elsif(/^$/){ $in_acpi=0; } if( /^(Package id \d+|Core \d+):.*?\+([\d.]+)/ ){ my($n,$t)=($1,$2); my($hi)=/high\s*=\s*\+([\d.]+)/? $1:100; my($cr)=/crit\s*=\s*\+([\d.]+)/? $1:100; my $col = ($t>=95) ? "\e[31m" : ($t>=80) ? "\e[33m" : "\e[32m"; if($n =~ /^Package id/){ $n="CPU Package"; printf "%-16s ${col}\e[1m%6.1f°C\e[0m %7s°C %7s°C\n",$n,$t,$hi,$cr; print "\e[2m──────────────────────────────────────────────\e[0m\n"; } else { printf "%-16s ${col}%6.1f°C\e[0m %7s°C %7s°C\n",$n,$t,$hi,$cr; } } elsif( $in_acpi && /^temp\d+:\s*\+?([\d.]+)/ ){ my $t=$1; my $col = ($t>=75) ? "\e[31m" : ($t>=60) ? "\e[33m" : "\e[32m"; printf "%-16s ${col}%6.1f°C\e[0m %7s %7s\n","ACPI temp1",$t,"--","--"; }'\''; echo; echo "=== SSD /dev/sda ==="; smartctl -l scttemp /dev/sda 2>/dev/null | perl -Mstrict -Mwarnings -ne '\''if(/Current Temperature:\s+(\d+)/){ my $t=$1; my $col = ($t>=70) ? "\e[31m" : ($t>=60) ? "\e[33m" : "\e[32m"; printf "%-24s ${col}%5d°C\e[0m\n","Current Temperature:",$t; } elsif(/Power Cycle Min\/Max Temperature:/ || /Lifetime Min\/Max Temperature:/){ print; }'\'''
```

### ASCII fallback (no `°` symbol)

If your locale/font doesn’t render `°`, use the ASCII variant:

```bash
watch -c -n1 --no-title bash -lc 'date "+%F %T"; echo "=== CPU ==="; sensors 2>/dev/null | perl -Mstrict -Mwarnings -ne '\''BEGIN{ printf "%-16s %8s %8s %8s\n","Sensor","Temp","High","Crit" } our $in_acpi=0; if(/^Adapter:\s*ACPI interface/){ $in_acpi=1; } elsif(/^$/){ $in_acpi=0; } if( /^(Package id \d+|Core \d+):.*?\+([\d.]+)/ ){ my($n,$t)=($1,$2); my($hi)=/high\s*=\s*\+([\d.]+)/? $1:100; my($cr)=/crit\s*=\s*\+([\d.]+)/? $1:100; my $col = ($t>=95) ? "\e[31m" : ($t>=80) ? "\e[33m" : "\e[32m"; if($n =~ /^Package id/){ $n="CPU Package"; printf "%-16s ${col}\e[1m%6.1fC\e[0m %7sC %7sC\n",$n,$t,$hi,$cr; print "\e[2m──────────────────────────────────────────────\e[0m\n"; } else { printf "%-16s ${col}%6.1fC\e[0m %7sC %7sC\n",$n,$t,$hi,$cr; } } elsif( $in_acpi && /^temp\d+:\s*\+?([\d.]+)/ ){ my $t=$1; my $col = ($t>=75) ? "\e[31m" : ($t>=60) ? "\e[33m" : "\e[32m"; printf "%-16s ${col}%6.1fC\e[0m %7s %7s\n","ACPI temp1",$t,"--","--"; }'\''; echo; echo "=== SSD /dev/sda ==="; smartctl -l scttemp /dev/sda 2>/dev/null | perl -Mstrict -Mwarnings -ne '\''if(/Current Temperature:\s+(\d+)/){ my $t=$1; my $col = ($t>=70) ? "\e[31m" : ($t>=60) ? "\e[33m" : "\e[32m"; printf "%-24s ${col}%5dC\e[0m\n","Current Temperature:",$t; } elsif(/Power Cycle Min\/Max Temperature:/ || /Lifetime Min\/Max Temperature:/){ print; }'\'''
```

### Default thresholds

* CPU: warn **80°C**, crit **95°C** (color applies to current temp only).
* ACPI: warn **60°C**, crit **75°C**.
* SSD (SATA): warn **60°C**, crit **70°C**.

### Customize

* Change thresholds inside the one‑liner (the numbers `80/95` and `60/75/70`).
* Add NVMe drive: replace the SSD part with `smartctl -l scttemp,nvme /dev/nvme0` (some models expose different logs).

### Troubleshooting

* **Degree symbol not shown / weird characters**: ensure UTF‑8 locale and a font with `°`.

  ```bash
  locale | grep -E 'LANG|LC_CTYPE'
  export LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8   # or ru_RU.UTF-8
  ```
* **No ACPI temp**: some platforms don’t expose `acpitz`; the ACPI line will simply be omitted.
* **`smartctl` shows no SCT**: some SSDs lack SCT temperature log; use the device‑specific log or NVMe variant.


### Light future plans

If enough people star/use it, I’ll add an optional "full" script with flags (`--nvme`, `--disk`, `--cpu-warn`, etc.), `-h` help and better autodetection.

---

## Русский

### Возможности

* Компактная таблица CPU (пакет и ядра) + ACPI‑температура.
* Текущая температура SSD (SCT), строки Min/Max сохраняются.
* Цвета: зелёный < warn, жёлтый ≥ warn, красный ≥ crit.
* Чистый однострочник — без файлов и функций, удобно для Proxmox/`tmux`.

### Требования

`lm-sensors`, `smartmontools`, `bash`, `perl`.

Установите зависимости в Debian/Ubuntu/Proxmox:
```bash
sudo apt update && sudo apt install -y lm-sensors smartmontools
```

### Быстрый старт (с символом градуса)

```bash
watch -c -n1 --no-title bash -lc 'date "+%F %T"; echo "=== CPU ==="; sensors 2>/dev/null | perl -Mstrict -Mwarnings -ne '\''BEGIN{ printf "%-16s %8s %8s %8s\n","Sensor","Temp","High","Crit" } our $in_acpi=0; if(/^Adapter:\s*ACPI interface/){ $in_acpi=1; } elsif(/^$/){ $in_acpi=0; } if( /^(Package id \d+|Core \d+):.*?\+([\d.]+)/ ){ my($n,$t)=($1,$2); my($hi)=/high\s*=\s*\+([\d.]+)/? $1:100; my($cr)=/crit\s*=\s*\+([\d.]+)/? $1:100; my $col = ($t>=95) ? "\e[31m" : ($t>=80) ? "\e[33m" : "\e[32m"; if($n =~ /^Package id/){ $n="CPU Package"; printf "%-16s ${col}\e[1m%6.1f°C\e[0m %7s°C %7s°C\n",$n,$t,$hi,$cr; print "\e[2m──────────────────────────────────────────────\e[0m\n"; } else { printf "%-16s ${col}%6.1f°C\e[0m %7s°C %7s°C\n",$n,$t,$hi,$cr; } } elsif( $in_acpi && /^temp\d+:\s*\+?([\d.]+)/ ){ my $t=$1; my $col = ($t>=75) ? "\e[31m" : ($t>=60) ? "\e[33m" : "\e[32m"; printf "%-16s ${col}%6.1f°C\e[0m %7s %7s\n","ACPI temp1",$t,"--","--"; }'\''; echo; echo "=== SSD /dev/sda ==="; smartctl -l scttemp /dev/sda 2>/dev/null | perl -Mstrict -Mwarnings -ne '\''if(/Current Temperature:\s+(\d+)/){ my $t=$1; my $col = ($t>=70) ? "\e[31m" : ($t>=60) ? "\e[33m" : "\e[32m"; printf "%-24s ${col}%5d°C\e[0m\n","Current Temperature:",$t; } elsif(/Power Cycle Min\/Max Temperature:/ || /Lifetime Min\/Max Temperature:/){ print; }'\'''
```

### ASCII‑вариант (без символа «°»)

```bash
watch -c -n1 --no-title bash -lc 'date "+%F %T"; echo "=== CPU ==="; sensors 2>/dev/null | perl -Mstrict -Mwarnings -ne '\''BEGIN{ printf "%-16s %8s %8s %8s\n","Sensor","Temp","High","Crit" } our $in_acpi=0; if(/^Adapter:\s*ACPI interface/){ $in_acpi=1; } elsif(/^$/){ $in_acpi=0; } if( /^(Package id \d+|Core \d+):.*?\+([\d.]+)/ ){ my($n,$t)=($1,$2); my($hi)=/high\s*=\s*\+([\d.]+)/? $1:100; my($cr)=/crit\s*=\s*\+([\d.]+)/? $1:100; my $col = ($t>=95) ? "\e[31m" : ($t>=80) ? "\e[33m" : "\e[32m"; if($n =~ /^Package id/){ $n="CPU Package"; printf "%-16s ${col}\e[1m%6.1fC\e[0m %7sC %7sC\n",$n,$t,$hi,$cr; print "\e[2m──────────────────────────────────────────────\e[0m\n"; } else { printf "%-16s ${col}%6.1fC\e[0m %7sC %7sC\n",$n,$t,$hi,$cr; } } elsif( $in_acpi && /^temp\d+:\s*\+?([\d.]+)/ ){ my $t=$1; my $col = ($t>=75) ? "\e[31m" : ($t>=60) ? "\e[33m" : "\e[32m"; printf "%-16s ${col}%6.1fC\e[0m %7s %7s\n","ACPI temp1",$t,"--","--"; }'\''; echo; echo "=== SSD /dev/sda ==="; smartctl -l scttemp /dev/sda 2>/dev/null | perl -Mstrict -Mwarnings -ne '\''if(/Current Temperature:\s+(\d+)/){ my $t=$1; my $col = ($t>=70) ? "\e[31m" : ($t>=60) ? "\e[33m" : "\e[32m"; printf "%-24s ${col}%5dC\e[0m\n","Current Temperature:",$t; } elsif(/Power Cycle Min\/Max Temperature:/ || /Lifetime Min\/Max Temperature:/){ print; }'\'''
```

### Пороговые значения по умолчанию

* CPU: предупреждение **80°C**, критично **95°C**.
* ACPI: предупреждение **60°C**, критично **75°C**.
* SSD (SATA): предупреждение **60°C**, критично **70°C**.

### Настройка

Правьте числа в однострочнике; для NVMe используйте `smartctl -l scttemp,nvme /dev/nvme0`.

### Неполадки

* **Не отображается символ «°» / кракозябры** — включите UTF‑8‑локаль и шрифт с символом градуса.

  ```bash
  locale | grep -E 'LANG|LC_CTYPE'
  export LANG=ru_RU.UTF-8 LC_CTYPE=ru_RU.UTF-8  # или en_US.UTF-8
  ```
* **Нет ACPI‑температуры** — платформа может не экспонировать `acpitz`.
* **Нет SCT у SSD** — устройство может не поддерживать этот лог; используйте NVMe/вендорные логи.

### Небольшие планы

Если инструмент окажется полезным, добавлю опциональную «расширенную» версию с флагами и автодетектом; создайте issue и поставьте ⭐, чтобы это ускорить.

**Keywords:** lm-sensors, smartctl, terminal, watch, temperatures, Proxmox, Debian, Linux, one-liner
