`apt update && apt upgrade -y`
reboot

other -  https://us.download.nvidia.com/XFree86/Linux-x86_64/470.199.02/NVIDIA-Linux-x86_64-470.199.02.run

`apt install build-essential dkms pve-headers-$(uname -r)`

`wget https://us.download.nvidia.com/tesla/550.54.15/NVIDIA-Linux-x86_64-550.54.15.run`
or `wget https://download.nvidia.com/XFree86/Linux-x86_64/555.42.02/NVIDIA-Linux-x86_64-555.42.02.run`

support for steam-headless only with `wget https://download.nvidia.com/XFree86/Linux-x86_64/535.183.01/NVIDIA-Linux-x86_64-535.183.01.run`

`chmod +x NVIDIA-Linux-x86_64-535.183.01.run`

`./NVIDIA-Linux-x86_64-535.183.01.run  --dkms`

let it make changes and disable nodemon, install will fail, abort install, reboot

rerun installer
`./NVIDIA-Linux-x86_64-535.183.01.run  --dkms`

reboot

`nvidia-smi`


# Ollama

we should not edit the ollama daemon service file directly. what we should do is creating a extra config file like what docker did in its doc. in summry, you do the following stuff:

sudo mkdir /etc/systemd/system/ollama.service.d
sudo vim /etc/systemd/system/ollama.service.d/http-host.conf
sudo systemctl daemon-reload
sudo systemctl restart ollama
the content of the http-host.conf file should be:

```
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```
after all this, you can tell ollama is indeed serving on all interfaces by sudo systemctl status ollama, there will be logs like Listening on [::]:11434

No need for alarm; This already happens when you run systemctl edit ollama.service

# Proxmox passthrough

`sudo usermod -aG video,render,input,xrdp,sudo,syslog usernamehere`

`sudo apt install pkg-config libglvnd-dev`

```
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 65536
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 534:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia1 dev/nvidia1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,c


```
