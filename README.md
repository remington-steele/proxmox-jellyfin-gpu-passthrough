# proxmox-jellyfin-gpu-passthrough
How to pass a GPU through to Jellyfin on a proxmox VM

### Credit
https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/
https://www.reddit.com/r/Proxmox/comments/lcnn5w/proxmox_pcie_passthrough_in_2_minutes/
https://old.reddit.com/r/Proxmox/comments/vvzryq/how_to_disable_simplefb_for_gpu_passthrough_with/ifnh77v/

## Proxmox Prep
```vim /etc/default/grub```

### Change to this
```GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"```

```vim /etc/modules```

### Add this
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

### Reboot
```reboot```

## Create VM
Create a new VM with OVMF BIOS and as a q35 machine. DO NOT SKIP setting processor type!

# Use whatever other settings you want, but the following are required
```
SYSTEM
  BIOS: OVMF(UEFI)
  Machine: q35

PROCESSOR
  TYPE: IvyBridge        # Your CPU may not be an IvyBridge
```

Add your graphics card as a new PCI device

Set device to use All functions and PCI Express. DO NOT set as primary GPU!

## Install nvidia drivers and nvidia-smi on the host VM
```
sudo apt update
ubuntu-drivers devices
sudo apt install -y nvidia-driver-535    # or the recommended version shown
sudo reboot
```

This should output your GPU status information
```nvidia-smi```

## Install nvidia container toolkit on the host VM
```
# Repo + toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list > /dev/null

sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure
sudo systemctl restart docker
```

## Add nvidia GPU options to docker-compose Jellyfin section in the host VM
```
jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1001
      - PGID=1001
      - TZ=America/Los_Angeles
      - JELLYFIN_PublishedServerUrl=https://media.unsavory.com #optional
      - NVIDIA_VISIBLE_DEVICES=all      # Expose all GPUs (or specify GPU IDs)
      - NVIDIA_DRIVER_CAPABILITIES=all
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]       # Pass GPU capability to the container
    runtime: nvidia
    volumes:
      - /home/unsavory/containers/jellyfin/config:/config
      - /data/media/tv:/data/tvshows
      - /data/media/movies:/data/movies
      - /archive/media/tv:/archive/media/tv
      - /archive/media/movies:/archive/media/movies
    networks:
      - docker-network
    ports:
      - "0.0.0.0:8096:8096"
      - "0.0.0.0:8920:8920" #optional
      - "0.0.0.0:7359:7359/udp" #optional
      - "0.0.0.0:1900:1900/udp" #optional
    restart: unless-stopped
```
