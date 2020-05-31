# Tesla pre-AP EPAS steering firmware patch
Normally on pre-AP Tesla vehicles the gateway sends a message indicating that steering over CAN is disabled.

This tool patches the firmware to enable steering over CAN.

## usage notes
* flashing firmware can fail and brick your EPAS, while unlikely,  
  **do not flash something that you are not willing to pay to replace**
* [comma.ai panda](https://comma.ai/shop/products/panda-obd-ii-dongle)
  is used to communicate with EPAS over CAN
* requires direct CAN bus communication line to EPAS (EPAS is not flashed through gateway)
* requires an EPAS firmware update file from Tesla because a secondary bootloader
  is needed to flash the EPAS which is only available in the firmware update
* before flashing, a backup of the firmware is taken from the EPAS to ensure that
  the firmware on your EPAS is the expected firmware and compatible

## setup
```sh
pip3 install -r requirements.txt
```

## patch the firmware
connect [comma.ai panda](https://comma.ai/shop/products/panda-obd-ii-dongle) to EPAS and run:

```sh
./tesla-epas-patcher.py /path/to/epas_combined.hex
```

where `/path/to/epas_combined.hex` is a EPAS firmware update file from Tesla

## restore original firmware
connect [comma.ai panda](https://comma.ai/shop/products/panda-obd-ii-dongle) to EPAS and run:

```sh
./tesla-epas-patcher.py /path/to/epas_combined.hex --restore
```

where `/path/to/epas_combined.hex` is a EPAS firmware update file from Tesla

## how it works
The gateway in pre-AP vehicles constantly sends an EPAS CONTROL message (CAN address 0x101) with two signals we care about:

* LDW ENABLE (1 bit)
  * 0 = DISABLE
  * 1 = ENABLE
* CONTROL TYPE (3 bits)
  * 0 = INHIBIT
  * 1 = ANGLE
  * 2 = TORQUE
  * 3 = BOTH

The gateway in pre-AP vehicles sets these signals to have a value of 0 (disabled).

The firmware patch applied by this tool changes the EPAS CONTROL (CAN address 0x101) message parsing in the EPAS to extract the least significant bit of the POWER MODE signal in the same message instead (which is 1 while driving) for the above signals.  This enables angle based steering control by setting LDW ENABLE and CONTROL TYPE to have a value of 1 while driving.
