KVM启动时提示

> Requested operation is not valid: network ‘default’ is not active

解决方案

```bash
sudo virsh net-start default
sudo virsh net-autostart default
```

