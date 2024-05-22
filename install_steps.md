apt update && apt upgrade -y
reboot

apt install build-essential dkms pve-headers-$(uname -r)

wget https://us.download.nvidia.com/tesla/550.54.15/NVIDIA-Linux-x86_64-550.54.15.run

chmod +x NVIDIA-Linux-x86_64-550.54.15.run

./NVIDIA-Linux-x86_64-550.54.15.run  --dkms

let it make changes and disable nodemon, install will fail, abort install, reboot

rerun installer
./NVIDIA-Linux-x86_64-550.54.15.run  --dkms

reboot

nvidia-smi


# Ollama

we should not edit the ollama daemon service file directly. what we should do is creating a extra config file like what docker did in its doc. in summry, you do the following stuff:

sudo mkdir /etc/systemd/system/ollama.service.d
sudo vim /etc/systemd/system/ollama.service.d/http-host.conf
sudo systemctl daemon-reload
sudo systemctl restart ollama
the content of the http-host.conf file should be:

[Service]
Environment="OLLAMA_HOST=0.0.0.0"
after all this, you can tell ollama is indeed serving on all interfaces by sudo systemctl status ollama, there will be logs like Listening on [::]:11434

No need for alarm; This already happens when you run systemctl edit ollama.service
