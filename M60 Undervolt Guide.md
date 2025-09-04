# 🖥️ Tesla M60 Power & Clock Profiles (Linux, Debian)

This guide shows how to run a Tesla M60 headless server more efficiently using `nvidia-smi` and `systemd`.

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
  balanced : PL=120W, mem=2505 MHz, gfx=873 MHz   (good efficiency, near-full perf bursts)
  eco      : PL=112.5W, mem=2505 MHz, gfx=759 MHz (max efficiency, lower peak perf)
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
  balanced) apply_all 120 2505 873 ;;
  eco)      apply_all 112.5 2505 759 ;;
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

2. Create the systemd service

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
```basg
sudo systemctl daemon-reload
sudo systemctl enable --now nvidia-24x7-tune.service
```

⚡ Profiles
Stock

 - PL: 150 W (default)
 - Clocks: Memory 2505 MHz, Graphics up to 1177 MHz
```bash
sudo gpu-profile stock
```
Balanced (default at boot)

  - PL: 120 W
  - Clocks: Memory 2505 MHz, Graphics 873 MHz
  - ✅ Good 24/7 mode: ~90–95% performance, ~20–25% less power/heat
```bash
sudo gpu-profile balanced
```
Eco

  - PL: 112.5 W (minimum allowed)
  - Clocks: Memory 2505 MHz, Graphics 759 MHz
  - ✅ Maximum efficiency: ~70–80% performance, cooler & quieter
```bash
sudo gpu-profile eco
```

🌀 Switching Profiles Live

  - Eco:
```bash
sudo gpu-profile eco
```
 - Balanced:
```bash
sudo gpu-profile balanced
```
 - Stock:
```bash
sudo gpu-profile stock
```
These changes take effect immediately without reboot.

🔒 Notes
  - Safe: All values use NVIDIA’s supported limits; you can’t “undervolt” below what the card allows.
  - Auto Boost: Leaving it ON is fine; the power limit keeps things safe. Turn it OFF if you want strictly fixed clocks.
  - Service default: Balanced is applied at boot. Use systemctl edit nvidia-24x7-tune.service to change.
