# Layout files for tataouine boards

This repository host all files defining the memory layout and device listing for the various
boards.
These files handle layouting and can be used to modify the way devices are handled (mapping,
permissions, and device partitioning if needed).

## About memory layout

The general memory layout is defined in layout.def. It defines the global memory mapping and base
addresses that will be used mostly by firmaware generation tools.

layout.apps.def define the userspace application specific layout. It is used by each userspace
ldscripts to support efficient application slotting.

## About devices mapping

Device mapping is defined in json files. One file is used at a time, depending on the board name
choosen. For example, when building a firmware for the 'wookey' board, the soc-devmap-wookey.json is used.

The json file list each device using the following json dictionary structure:

```
  "devname": {
      "address":"0x40026440",
      "size":"0",
      "memory_subregion_mask":"0",
      "irqs": [
          "DEV_IRQ", "DEV_IRQ2", 0, 0
      ],
      "read_only": "false",
      "permission": "PERM_RES_DEV_DMA",
      "enable_register":"r_CORTEX_M_RCC_AHB1ENR",
      "enable_register_bits": ["RCC_AHB1ENR_DMA2EN"]
  },
```

The device address and size is the one defined in the SoC datasheet. The memory subregion mask permit to
use the MPU specific properties to mask a part of the device if needed (according to what the MPU is capable of).
The IRQs (up to 4 per device) are interrupt number (use IRQn + 0x10) of the device. See arch/socs/stm32f439/soc-intterupt.h to get the interrupt list.
The permission is the permission associated to the device, as defined in the EwoK permissions API. Only one permission
per device can be set here by now.
The enable register and enable register bit is the RCC clock enable bit defined in the SoC datasheet to activate
the device clock.

You can add as many devices as you wish. You can add delete unwanted devices if you wish. Only devices declared here
(including DMA) can be used by userspace tasks at runtime.

## About generated files

The devices mapping generates the kernel/generated/devmap.h header file.

## About JSON Schema validation

The JSON dictionary used here is associated to a dedicated JSON schema named 'soc-devmap.schema.json.
The conformity of the JSON file to the JSON schema can be verified using any JSON schema validation tool.

A typical check could be, using python3-jsonschema package :

```
   jsonschema -i soc-devmap-wookeyv1.json soc-devmap.schema.json
```

If you which to add some new peripherals, don't hesitate to validate the updated JSON file against the JSON Schema to avoid invalid JSON syntax that may generate invalid header files.
