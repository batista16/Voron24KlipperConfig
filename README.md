# configuring can0
sudo nano /etc/network/interfaces.d/can0

allow-hotplug can0
iface can0 can static
    bitrate 1000000
    up ifconfig $IFACE txqueuelen 256
    pre-up ip link set can0 type can bitrate 1000000
    pre-up ip link set can0 txqueuelen 256

# to flash katapult to ebb
cd katapult
make menuconfig
make
sudo dfu-util -l
sudo dfu-util -a 0 -D out/katapult.bin -s 0x08000000:mass-erase:force --device xxxxx


# to flash klipper to ebb
cd ~/klipper
make menuconfig
  Enable extra low-level configuration options: check
  Micro-controller Architecture: STMicroelectronics STM32
  Processor model: STM32G0B1
  Bootloader offset: 8KiB bootloader
  Clock Reference: 8 MHz crystal
  Communication interface: CAN bus (on PB0/PB1)
  CAN bus speed: 1000000
make clean
make

sudo service klipper stop
python3 ~/katapult/scripts/flashtool.py -i can0 -q
python3 ~/katapult/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u aa975c1b5e99
