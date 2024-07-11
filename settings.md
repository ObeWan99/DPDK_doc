# The DPDK is composed of several directories, including:

- doc: DPDK Documentation
- license: DPDK license information
- lib: Source code of DPDK libraries
- drivers: Source code of DPDK poll-mode drivers
- app: Source code of DPDK applications (automatic tests)
- examples: Source code of DPDK application examples
- config, buildtools: Framework-related scripts and configuration
- usertools: Utility scripts for end-users of DPDK applications
- devtools: Scripts for use by DPDK developers
- kernel: Kernel modules needed for some operating systems

# Установка dpdk:

```sh
sudo apt update
```

```sh
# Установка основных зависимостей DPDK
sudo apt-get install -y build-essential linux-headers-$(uname -r) gcc make cmake pkg-config libpcap-dev libnuma-dev libelf-dev libdwarf-dev python3-pyelftools meson ninja-build libssl-dev libnl-3-dev libudev-dev
```

```sh
tar xJf dpdk-<version>.tar.xz
cd dpdk-<version>
```

Для настройки сборки DPDK используйте:
```sh
meson setup build
```

или, чтобы включить примеры в сборку, замените команду meson на:
```sh
meson setup -Dexamples=all build
```

После настройки для сборки и установки DPDK в масштабах всей системы используйте:
```sh
cd build
ninja
sudo meson install
sudo ldconfig
```

Последние две команды, указанные выше, обычно необходимо запускать от имени пользователя root, при этом шаг установки meson копирует созданные объекты в их конечные системные расположения, а последний шаг заставляет динамический загрузчик ld.so обновить свой кэш для учета новых объектов.

***В некоторых дистрибутивах Linux, таких как Fedora или Redhat, пути в /usr/local не входят в пути по умолчанию для загрузчика. Поэтому в этих дистрибутивах /usr/local/lib и /usr/local/lib64 следует добавить в файл в /etc/ld.so.conf.d/ перед запуском ldconfig .***

Чтобы решить эту проблему, вам необходимо добавить директории /usr/local/lib и /usr/local/lib64 в конфигурацию ld.so. Это делается следующим образом:

Создайте файл с любым именем (например, local.conf) в директории /etc/ld.so.conf.d/:
```sh
sudo touch /etc/ld.so.conf.d/local.conf
```

Откройте файл local.conf в текстовом редакторе и добавьте следующие строки:
```sh
/usr/local/lib
/usr/local/lib64
```
Эти строки указывают ld.so, что он должен искать библиотеки в этих директориях.

Сохраните файл и выполните команду ldconfig:
```sh
sudo ldconfig
```

# Настройка dpdk:

### Установка драйверов

Убедитесь, что драйверы установлены:
```sh
sudo modprobe uio
sudo modprobe uio_pci_generic
```

Если используете vfio-pci:
```sh
sudo modprobe vfio-pci
```

Режим VFIO без IOMMU:
```sh
sudo modprobe vfio enable_unsafe_noiommu_mode=1
```

### Привязка сетевой карты

Узнать PCI адрес вашей сетевой карты Virtio:
```sh
lspci | grep Eth
```

Отключение сетевого интерфейса:
```sh
sudo ifconfig <interface> down
```

Теперь привяжем сетевую карту к одному из этих драйверов. Используйте утилиту dpdk-devbind для этого. В комплекте с DPDK идет эта утилита:
```sh
sudo ./usertools/dpdk-devbind.py --bind=uio_pci_generic <PCI_ADDRESS>
```

Для vfio-pci:
```sh
sudo ./usertools/dpdk-devbind.py --bind=vfio-pci <PCI_ADDRESS>
```

Проверьте статус привязки:
```sh
sudo ./usertools/dpdk-devbind.py --status
```

Отвязать:
```sh
sudo ./usertools/dpdk-devbind.py --unbind <PCI_ADDRESS>
```

***После привязки интерфейса к DPDK, он больше не виден стандартными сетевыми утилитами. Вы должны использовать DPDK-приложения и утилиты для проверки состояния и работы интерфейса. Для назначения IP-адресов и работы с ними, вам нужно настроить соответствующие структуры данных и логику в ваших DPDK-приложениях.***

### Обеспечение достаточного выделения hugepages

```sh
sudo -i
```

Создание точки монтирования для HugePages:
```sh
mkdir -p /mnt/huge
```

Монтирование hugetlbfs:
```sh
mount -t hugetlbfs nodev /mnt/huge
```

Выделение HugePages:
```sh
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```
Эта команда выделяет 1024 hugepages размером 2MB каждая.

Проверка настроек:
```sh
grep Huge /proc/meminfo
```

```sh
mount | grep hugetlbfs
```

```sh
exit
```

### Если вы хотите, чтобы эти настройки сохранялись при перезагрузке, добавьте соответствующие записи в /etc/fstab и /etc/sysctl.conf.

Добавление в /etc/fstab:
```sh
echo "nodev /mnt/huge hugetlbfs defaults 0 0" >> /etc/fstab
```

Добавление в /etc/sysctl.conf:
```sh
echo "vm.nr_hugepages=1024" >> /etc/sysctl.conf
sysctl -p
```

