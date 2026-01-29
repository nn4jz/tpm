# tpm (swtpm)

Since **swtmp** is already installed, you just need a small wrapper script that launches it cleanly and predictably. Below is a solid, idiomatic shell script that handles:

- Ensuring the binary exists  
- Allowing optional arguments  
- Logging (optional but useful)  
- Running swtmp in the foreground or background  

You can trim it down if you want something ultra‑minimal.

---

## 🟦 Minimal launch script

```bash
#!/usr/bin/env bash

# Simple launcher for swtmp
exec swtmp "$@"
```

Save it as `swtmp.sh`, make it executable:

```bash
chmod +x swtmp.sh
```

Run it:

```bash
./swtmp.sh
```

---

## 🟩 More robust version (recommended)

```bash
#!/usr/bin/env bash

set -euo pipefail

# Path to swtmp (adjust if installed elsewhere)
SWTMP_BIN="$(command -v swtmp || true)"

if [[ -z "$SWTMP_BIN" ]]; then
    echo "Error: swtmp not found in PATH" >&2
    exit 1
fi

LOGFILE="${HOME}/.swtmp/swtmp.log"
mkdir -p "$(dirname "$LOGFILE")"

echo "Launching swtmp..."
echo "[$(date)] swtmp launched with args: $*" >> "$LOGFILE"

exec "$SWTMP_BIN" "$@"
```

### Why this version is nicer
- Fails early if `swtmp` isn’t in PATH  
- Logs launches for debugging  
- Accepts arguments (`./launch-swtmp.sh --debug`)  
- Uses `exec` so the script doesn’t leave zombie processes  

---

## 🟧 Optional: systemd user service (if you want it to auto‑start)

If you want swtmp to run automatically in the background:

`~/.config/systemd/user/swtmp.service`:

```ini
[Unit]
Description=swtmp service

[Service]
ExecStart=/usr/bin/swtmp
Restart=on-failure

[Install]
WantedBy=default.target
```

Enable it:

```bash
systemctl --user daemon-reload
systemctl --user enable --now swtmp
```

---

If you want, I can tailor the script to your exact environment — WSL2, Ubuntu, systemd‑enabled WSL, or even Windows Task Scheduler.