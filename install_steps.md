apt update && apt upgrade -y
reboot

apt install build-essential dkms pve-headers-$(uname -r)

download driver... (NVIDIA-Linux-x86_64-550.54.15)

chmod +x NVIDIA-Linux-x86_64-550.54.15.run

./NVIDIA-Linux-x86_64-550.54.15.run

reboot

nvidia-smi
