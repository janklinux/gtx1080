# GPU Setup — GTX 1080 + PyTorch

## Status

| Component | State |
|---|---|
| Host driver | NVIDIA 560.28.03 (CUDA 12.6 capable) |
| GPU | GeForce GTX 1080 @ PCI 0000:07:00.0 |
| PyTorch | 2.6.0+cu124 installed in `/home/dev/torch-env` |
| CUDA in container | **NOT YET ACTIVE** — device files not mounted |

## What's missing

The container is missing the NVIDIA device files (`/dev/nvidia0`, `/dev/nvidiactl`, `/dev/nvidia-uvm`).
The host driver is loaded and working — the container just needs them passed through.

## Fix (on the host machine)

### Option A — nvidia-container-toolkit + CDI (recommended)

```bash
# Install nvidia-container-toolkit on the HOST
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update && sudo apt install -y nvidia-container-toolkit

# Generate CDI spec
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

# Then restart the terok container with:
# --device nvidia.com/gpu=all
```

### Option B — explicit device pass-through (simpler)

Add these flags when starting the container via terok/Podman:

```
--device /dev/nvidia0
--device /dev/nvidiactl
--device /dev/nvidia-uvm
```

## Verifying it works (inside the container)

```python
source /home/dev/torch-env/bin/activate
python3 -c "import torch; print(torch.cuda.get_device_name(0))"
# Expected: NVIDIA GeForce GTX 1080
```

## Notes

- GTX 1080 is Pascal (compute capability 6.1) — fully supported by CUDA 12.4
- The `torch-env` venv is auto-activated in `.bashrc`
- PyTorch includes its own CUDA runtime — no system CUDA toolkit needed
