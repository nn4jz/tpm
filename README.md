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


If you want your launch script to explicitly use the socket directory **`~/tpm/swtpm-sock`**, you can wrap `swtpm` so it always starts with that path and creates the socket there.

Below is a clean, reliable script that does exactly that.

---

## 🟦 `launch-swtpm.sh` (recommended)

```bash
#!/usr/bin/env bash
set -euo pipefail

# Directory where the swtpm socket will live
SOCK_DIR="$HOME/tpm/swtpm-sock"

# Ensure the directory exists
mkdir -p "$SOCK_DIR"

# Path to swtpm binary
SWTMP_BIN="$(command -v swtpm || true)"

if [[ -z "$SWTMP_BIN" ]]; then
    echo "Error: swtpm not found in PATH" >&2
    exit 1
fi

# Launch swtpm with a UNIX socket in the specified directory
exec "$SWTMP_BIN" socket \
    --tpmstate dir="$SOCK_DIR" \
    --ctrl type=unixio,path="$SOCK_DIR/swtpm-control.sock" \
    --server type=unixio,path="$SOCK_DIR/swtpm.sock" \
    --flags not-need-init \
    "$@"
```

---

## 🟩 Make it executable

```bash
chmod +x launch-swtpm.sh
```

Run it:

```bash
./launch-swtpm.sh
```

---

## 🟧 What this script gives you
- Ensures `~/tpm/swtpm-sock` exists  
- Creates both control and TPM sockets inside that directory  
- Uses `exec` so the script doesn’t leave a wrapper process  
- Accepts extra arguments (`./launch-swtpm.sh --log-level=20`)  

---
