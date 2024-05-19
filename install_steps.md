Install steps


apt update && apt upgrade -y
reboot

apt install build-essential dkms pve-headers-$(uname -r)

download driver...

chmod +x NVIDIA-Linux-x86_64-460.39.run

./NVIDIA-Linux-x86_64-460.39.run

reboot

nvidia-smi
