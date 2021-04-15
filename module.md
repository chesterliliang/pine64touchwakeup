# Module Design for touch wakeup

## Current firmware

```
Host      Module
----------------
verify--->
        <------ touch
        do verify
<-------INT
<-------response
```

## For touch wakeup

### Basically

1. fprintd is framework we work with, try not to involve any kernel level changes for this task.
2. Module must be powered when Pine64 got locked and in sleep mode.
3. Pine64 should be able to wake up by the INT pin.

### Timing

1. As former experience, users finger stay on sensor about 100-200ms  by a natural touch which is the time window for image capture.
2. We are not sure if Pine64 could wake up so quickly to launch fprintd to do the verify job.
3. So maybe we could play some tricks to simple the task.

### Stand by mode

We bring a 'stand by mode' which module stay in low power mode, could dectect the finger touch and trigered by spi CS pin to normal mode.

In stand by mode, when users touch the sensor, the module back to normal mode and does the verify function. No matter it matched or not, module will generate the INT signal. Then module will stay in normal mode for 1 second wait the host the send verify comand to get the result.

If the host is not in login status, fprintd will not work, so no one will send any command, the module will turn back to stand by mode after 1 second.

If the host waked by the INT and get into login status, fprintd will send the verify command and get the verify result to do the login action. The module will remains in normal mode unless receives 'Set Stand By' command. fprintd could decide when to send this command by its judgement but just follow by every verify process is suggested since that could ensure the module right in stand by mode.

Let's bring the behavior define to see all state changes.

#### Power on

Module powered up and get into stand by mode automatically. Host may not use fingerprint when power on or reboot due to its policy.

#### Normal use

SBM(stand by mode)
NM(normal mode)

```

Host      Module
----------------
          SBM
          <------ touch
          NM
          do verify
  <-------INT
          .
          .
          1s (could be more to fit system wake time)
          .
verify------>
<-------response
SetSBM----->
          SBM
```

#### Wake up before touch

```
Host      Module
----------------
          SBM
wakeup
verify------>
          NM
          <------ touch
          do verify
<-------INT
<-------response
SetSBM----->
          SBM
```

#### 3rd party Use

```
Host      Module
----------------
          SBM
....IDLE....
         
verify------>
           NM
          <------ touch
          do verify
<-------INT
<-------response
SetSBM----->
          SBM
```


## Host Changes

The following changes should be made if we add standby mode

1. Add set standby mode command after doing verify, enroll when needed.
2. Add 10-20ms delay to start spi transfer after CS goes low to wait the module wake up from stand by mode.

## Why not automatically stand by

1. To avoid unstable status when wake up from stand by mode. The MCU goes very deep sleep at stand by mode. There is concern that if we switch to much by every command will make some unstable issues.
2. To make it compatible with current firmware. If we can not make touch to wake up stable when we have to release the firmware. The changes won't effect anything. We could just stay the current mode to use the module.
