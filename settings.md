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
sudo apt-get install -y build-essential linux-headers-$(uname -r) gcc make cmake pkg-config libpcap-dev libnuma-dev libelf-dev libdwarf-dev python3-pyelftools meson ninja-build libssl-dev
```

```sh
tar xJf dpdk-<version>.tar.xz
```

```sh
cd dpdk-<version>
```

Создавайте библиотеки, драйверы и тестовые приложения:
```sh
meson setup <options> build
```

Чтобы включить примеры в сборку, замените команду meson на:
```sh
meson setup -Dexamples=all build
```

```sh
ninja -C build
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

### Привязка сетевой карты

Узнать PCI адрес вашей сетевой карты Virtio:
```sh
lspci | grep Eth
```

Отключение сетевого интерфейса:
```sh
sudo ifconfig eth1 down
```

Теперь привяжем сетевую карту к одному из этих драйверов. Используйте утилиту dpdk-devbind для этого. В комплекте с DPDK идет эта утилита:
```sh
sudo ./usertools/dpdk-devbind.py --bind=uio_pci_generic <PCI_ADDRESS>
```

Для vfio-pci:
```sh
sudo dpdk-devbind --bind=vfio-pci <PCI_ADDRESS>
```

Проверьте статус привязки:
```sh
sudo ./usertools/dpdk-devbind.py --status
```

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

# Проверка работы (Дима, не делай, положишь отдел)

Запустите testpmd:
```sh
sudo ./build/app/dpdk-testpmd -l 0-3 -n 4 -- --portmask=0x1 --nb-cores=2 --auto-start
```

Параметры:

- -l 0-3: Использовать CPU ядра 0, 1, 2, 3.

- -n 4: Использовать 4 канала памяти.

- --portmask=0x1: Задействовать первый порт (0000:00:13.0).

- --nb-cores=2: Использовать 2 ядра.

- --auto-start: Автоматически запускать пакеты.

С другого хоста или из другого терминала на той же машине отправьте ping:
```sh
ping <IP-адрес интерфейса>
```

В консоли testpmd используйте команду для отображения статистики портов:
```sh
testpmd> show port stats all
```

# А тут может и не упадёт 

```sh
sudo ./build/app/dpdk-testpmd -l 0-1 -n 4 -- --portmask=0x1 --nb-cores=1 --forward-mode=io
```

Объяснение параметров:

    -l 0-1: Использование ядер 0 и 1.
    -n 4: Количество каналов памяти (memory channels).
    --portmask=0x1: Маска портов для использования (порт 0).
    --nb-cores=1: Количество ядер для обработки пакетов.
    --forward-mode=io: Режим форвардинга пакетов (IO mode).

Запуск форвардинга:
```sh
start
```

Проверка статистики:
```sh
show port stats all
```

Остановка форвардинга:
```sh
stop
```

Выход:
```sh
quit
```


### После привязки интерфейса к DPDK, он больше не виден стандартными сетевыми утилитами. Вы должны использовать DPDK-приложения и утилиты для проверки состояния и работы интерфейса. Для назначения IP-адресов и работы с ними, вам нужно настроить соответствующие структуры данных и логику в ваших DPDK-приложениях.

