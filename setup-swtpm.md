Using `swtpm` inside **WSL2** is a bit of a special case. Since WSL2 runs on a lightweight utility VM managed by Microsoft, it doesn't come with a virtual TPM enabled by default, and it doesn't automatically pass through your Windows host's hardware TPM.

To use `swtpm` in WSL2, you essentially treat the Linux distribution as the "host" and use it to provide TPM services to applications or other nested containers/VMs.

---

## 1. Install swtpm in WSL2

First, ensure your WSL2 distro is up to date and install the package. Most WSL2 users run Ubuntu:

```bash
sudo apt update
sudo apt install swtpm swtpm-tools

```

## 2. Setting up a Software TPM Instance

Since WSL2 doesn't use `systemd` in the traditional sense (though it's available in newer versions), the easiest way to run `swtpm` is as a local character device.

1. **Create a state directory:**
```bash
mkdir -p ~/.swtpm

```


2. **Start the daemon using a Unix socket:**
This allows other Linux applications (like a nested QEMU instance) to interact with it.
```bash
swtpm socket --tpmstate dir=$HOME/.swtpm \
  --ctrl type=unixio,path=$HOME/.swtpm/swtpm-sock \
  --tpm2 \
  --flags not-need-resettable

```



---

## 3. The "CUSE" Method (Advanced)

If you need the TPM to appear as a system device (like `/dev/tpm0`), you have to use the **CUSE** (Character device in Userspace) interface.

> **Note:** This requires the `cuse` kernel module. WSL2's default kernel often has this enabled, but you may need to load it:
> `sudo modprobe cuse`

Run `swtpm` to create a device node:

```bash
sudo swtpm_setup --tpm-state $HOME/.swtpm --tpm2 --createek
sudo swtpm cuse --tpmstate dir=$HOME/.swtpm --tpm2 --name tpm0

```

This will create `/dev/tpm0` inside your WSL2 environment, which tools like `tpm2-tools` can then interact with.

---

## 4. Why use swtpm in WSL2?

The most common use cases for this setup include:

* **Developing TPM-aware software:** Testing `tpm2-tss` or `openssl` TPM engines without risking your hardware's actual TPM.
* **Nested Virtualization:** If you are running QEMU inside WSL2 to boot a secondary VM that requires a TPM.
* **Learning:** Practicing with `tpm2_pcrread` or sealing secrets to a virtualized PCR.

---

## Verifying it Works

Once your `swtpm` is running (via the CUSE method), install the tools to talk to it:

```bash
sudo apt install tpm2-tools
tpm2_pcrread sha256

```

If you see a list of hash values (PCRs), your software TPM is successfully simulating a hardware chip!

---

**Common WSL2 Gotchas:**

* **Persistency:** If you restart WSL (`wsl --shutdown`), you will need to restart the `swtpm` process.
* **Root Access:** Creating nodes in `/dev/` usually requires `sudo`.

Would you like me to show you how to write a small script to auto-start `swtpm` whenever you open your WSL2 terminal?