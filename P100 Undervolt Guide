# 🖥️ Tesla P100 Power & Clock Profiles (Linux, Debian)

This guide shows how to run a **Tesla P100** headless server more efficiently using `nvidia-smi` and `systemd`.

---

## 🔧 Installation

### 1. Create the profile script
Save as `/usr/local/sbin/gpu-profile`:

```bash
#!/usr/bin/env bash
set -euo pipefail

PROFILE="${1:-}"

if [[ -z "${PROFILE}" || "${PROFILE}" == "-h" || "${PROFILE}" == "--help" ]]; then
  cat <<'USAGE'
Usage: gpu-profile <balanced|eco|stock|status>

Profiles:
  balanced : PL=200W, mem=715 MHz, gfx=1180 MHz   (good efficiency, near-full perf)
  eco      : PL=175W, mem=715 MHz, gfx=1013 MHz   (max efficiency, lower peak perf)
  stock    : reset to vendor defaults
Commands:
  status   : show current power limits and clocks
USAGE
  exit 0
fi

apply_all() {
  local pl="$1" mem="$2" gfx="$3"
  nvidia-smi -pm 1 >/dev/null

  while IFS=',' read -r idx minlim maxlim; do
    req_pl=$(awk -v req="$pl" -v min="$minlim" -v max="$maxlim" 'BEGIN{
      v=req; if (v<min) v=min; if (v>max) v=max; printf("%.2f", v)
    }')

    echo "[GPU $idx] PL=${req_pl}W mem=${mem} gfx=${gfx}"
    nvidia-smi -i "$idx" -pm 1
    nvidia-smi -i "$idx" -pl "$req_pl"
    nvidia-smi -i "$idx" -ac "${mem},${gfx}"
  done < <(nvidia-smi --query-gpu=index,power.min_limit,power.max_limit --format=csv,noheader,nounits)
}

case "${PROFILE}" in
  balanced) apply_all 200 715 1180 ;;
  eco)      apply_all 175 715 1013 ;;
  stock)
    echo "Resetting clocks and power limits..."
    nvidia-smi -rac
    while read -r idx defpl; do
      echo "[GPU $idx] Restoring PL=${defpl}"
      nvidia-smi -i "$idx" -pl "${defpl% *}"
    done < <(nvidia-smi --query-gpu=index,power.default_limit --format=csv,noheader)
    ;;
  status)   nvidia-smi -q -d POWER,CLOCK; exit 0 ;;
  *)        echo "Unknown profile: ${PROFILE}" >&2; exit 1 ;;
esac

echo
nvidia-smi -q -d POWER,CLOCK | sed -n '1,200p'
```

Make it executable:
```bash
sudo chmod 755 /usr/local/sbin/gpu-profile
```

---

### 2. Create the systemd service

File: `/etc/systemd/system/nvidia-24x7-tune.service`
```ini
[Unit]
Description=Apply NVIDIA 24/7 efficiency tuning at boot
After=multi-user.target
ConditionPathExists=/proc/driver/nvidia

[Service]
Type=oneshot
# Default profile (Balanced). Override with drop-ins.
ExecStart=/usr/local/sbin/gpu-profile balanced
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

Enable it:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now nvidia-24x7-tune.service
```

---

## ⚡ Profiles

**Stock (default)**  
- PL: 250 W (default)  
- Clocks: Memory 715 MHz, Graphics up to ~1328–1480 MHz

```bash
sudo gpu-profile stock
```

**Balanced (default at boot)**  
- PL: 200 W  
- Clocks: Memory 715 MHz, Graphics 1180 MHz  
- ✅ Good 24/7 mode: ~90–95% performance, ~20–25% less power/heat

```bash
sudo gpu-profile balanced
```

**Eco**  
- PL: 175 W  
- Clocks: Memory 715 MHz, Graphics 1013 MHz  
- ✅ Maximum efficiency: ~80–85% performance, cooler & quieter

```bash
sudo gpu-profile eco
```

---

## 🌀 Switching Profiles Live

All changes take effect immediately without reboot:

```bash
sudo gpu-profile eco
sudo gpu-profile balanced
sudo gpu-profile stock
```

---

## 🔒 Notes

- **Safe**: All values use NVIDIA’s supported limits; you can’t undervolt below the card’s firmware min.
- **Auto Boost**: Leaving it ON is fine; power limits keep things safe. Turn it OFF if you want strictly fixed clocks.
- **Service default**: Balanced is applied at boot. Use `systemctl edit nvidia-24x7-tune.service` to change.

---

## 🧪 Validation Commands

After applying a profile, verify:

```bash
sudo gpu-profile status
```

Expected output should show:
```
Power Draw: ~175–200 W
Clocks: Graphics 1013/1180 MHz, Memory 715 MHz
```

To check available ranges:

```bash
nvidia-smi --query-gpu=index,power.min_limit,power.max_limit,power.default_limit --format=csv
nvidia-smi -q -d SUPPORTED_CLOCKS
```

---

✅ **Summary**
- Balanced = 200 W @ 1180 MHz (best 24/7 efficiency)
- Eco = 175 W @ 1013 MHz (max efficiency)
- Safe, reversible, and systemd‑automated.
