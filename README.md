# FluxGrip Cynic commands

Commands for interacting with FluxGrip using the `cn` CLI, adapted from
[Fluxgrip: Cynic commands on the Zubax Forum](https://forum.zubax.com/t/fluxgrip-cynic-commands/2884).

Run commands from `~/fluxgrip_cynic` so Cynic reads this project's `cynic.toml`.
Use the SocketCAN interface connected to the device (configured as `slcan0`).
The feedback and management DSDL files are under
`~/fluxgrip_cynic/lib/zubax_dsdl/zubax/fluxgrip/`.

## Discover topics

List available topics and find the device's management topic:

```sh
cn ls
```

In every management command below, replace the example UID `2cb553eaeab5ec95`
with the UID reported by `cn ls`.

## Receive feedback

Subscribe to device feedback, including magnetization state, cycle counts, and errors:

```sh
cn sub feedback
```

## Control the magnet

Demagnetize:

```sh
cn pub command.i8 -- '\x00'
```

Magnetize:

```sh
cn pub command.i8 -- '\x01'
```

## Manage the device

Read device identity and software information:

```sh
cn req com.zubax.fluxgrip/2cb553eaeab5ec95/zubax.manage -- '{get_info: {}}'
```

Read the current configuration:

```sh
cn req com.zubax.fluxgrip/2cb553eaeab5ec95/zubax.manage -- '{configure: []}'
```

Write a complete configuration enabling the second CAN interface. Read the
device's configuration first and preserve its other settings when preparing an
update. Configuration changes require a restart to save and apply.

```sh
cn req com.zubax.fluxgrip/2cb553eaeab5ec95/zubax.manage -- '{configure: [{
  "version": 2,
  "prio_feedback": 4,
  "enable_second_can_interface": true,
  "can_bitrate": 1000000,
  "flyback_period_ns": 5000,
  "target_duty_cycle": [320,210,156,123,100,85,73,64,56],
  "min_duty_cycle": [150,150,100,80,20,20,20,20,20],
  "ramp_cap_voltage": [150,150,150,150,150,150,150,150,150],
  "ds": [-100,100,-100,100,-99,-92,-88,80,74,-67,-61,56,51,-46,-42,38,35,-32,-29,27,25,-22,-20,18,-16,14,-11,10,-8,7,-6,5,-4,4,-3,3,-2,2,-1,2,-1,2,-1,1,-1,1,-1,1,-1,1,-1,0,0,0,0,0,0,0,0,0,0,0,0,0]
}]}'
```

Clear topic remapping:

```sh
cn req com.zubax.fluxgrip/2cb553eaeab5ec95/zubax.manage -- '{remap: ""}'
```

Set the node namespace to `my.namespace` and remap `feedback` to `custom-feedback`:

```sh
cn req com.zubax.fluxgrip/2cb553eaeab5ec95/zubax.manage -- '{remap: "=my.namespace feedback=custom-feedback"}'
```

Restart the device:

```sh
cn req com.zubax.fluxgrip/2cb553eaeab5ec95/zubax.manage -- '{restart: false}'
```

Reset to factory defaults and restart:

```sh
cn req com.zubax.fluxgrip/2cb553eaeab5ec95/zubax.manage -- '{restart: true}'
```

## Update firmware

In one terminal, change to the directory containing `firmware.bin` and serve it
on `firmware-topic`:

```sh
cn fs firmware-topic
```

Keep that server running. In a second terminal, from `~/fluxgrip_cynic`, request
the update using the firmware filename and server topic:

```sh
cn req com.zubax.fluxgrip/2cb553eaeab5ec95/zubax.manage -- '{command: "upd firmware.bin firmware-topic"}'
```
