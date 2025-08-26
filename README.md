# proxmox-jellyfin-gpu-passthrough
How to pass a GPU through to Jellyfin on a proxmox VM

### Credit
https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/
https://www.reddit.com/r/Proxmox/comments/lcnn5w/proxmox_pcie_passthrough_in_2_minutes/
https://old.reddit.com/r/Proxmox/comments/vvzryq/how_to_disable_simplefb_for_gpu_passthrough_with/ifnh77v/

## Proxmox Prep

```vim /etc/modules```

### Add this
```vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd```
