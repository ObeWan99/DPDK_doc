# Настройка hugepages
```sh
sudo mkdir -p /mnt/huge
sudo mount -t hugetlbfs nodev /mnt/huge
echo 1024 | sudo tee /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

# Количество свободных hugepages можно проверить следующим образом
```sh
sudo ./usertools/dpdk-hugepages.py --show
```
Приведенная выше команда показывает, сколько огромных страниц свободно для поддержки запроса QEMU на выделение памяти.

# Привязка сетевого интерфейса к драйверу  uio_pci_generic DPDK
```sh
sudo ./usertools/dpdk-devbind.py --bind=uio_pci_generic <PCI_ADDRESS>
```

# Отвзяка сетевого интерфейса от драйвера DPDK
```sh
sudo ./usertools/dpdk-devbind.py -u <PCI_ADDRESS>
```

# Статус привязки сетевого интерфейса к драйевру DPDK
```sh
sudo ./usertools/dpdk-devbind.py --status
```

# Если вдруг интерфейс вообще не привязан ни к одному драйверу, то привяжите его к драйверу virtio-pci(если интерфейс virtio) Linux
```sh
sudo ./usertools/dpdk-devbind.py -b virtio-pci <interface_name>
```

Так же узнать список всех интерфейсов всегда можно узнать след командной:
```sh
sudo ./usertools/dpdk-devbind.py --status
```

```sh
sudo apt-get install ethtool
```