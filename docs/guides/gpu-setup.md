# GPU Setup

## Check GPU Availability

```bash
nvidia-smi -L
# Expected: GPU 0: NVIDIA GeForce RTX 4060 (UUID: ...)
```

## Install CUDA PyTorch

Python 3.12 is recommended (3.14 does not have CUDA wheels yet):

```bash
# Create venv with Python 3.12
python3.12 -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Install PyTorch with CUDA 12.4
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu124
```

## Verify

```python
import torch
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"Device: {torch.cuda.get_device_name(0)}")
# Expected: CUDA available: True
# Expected: Device: NVIDIA GeForce RTX 4060
```

## Training Impact

c4nav-core's model (~180K parameters) is small. GPU acceleration primarily helps with batch operations in the replay buffer:

| Hardware | 2000 Episodes Training Time |
|----------|:--------------------------:|
| CPU only (i7) | ~40 min |
| RTX 4060 GPU | ~25 min |

For this model size, the speedup is moderate (~1.6x). The benefit grows with larger networks or batch sizes.
