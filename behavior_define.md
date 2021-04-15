# User behavior define

## Normal use

1. Users click power button to lock the phone.
2. Users touch fingerprint sensor to wake up the phone.
3. If finger matched, login to home screen.
4. If not, stay in login screen, users could retry the finger or swtich to use password.

## Wake up before touch

1.Users click power button to wakeup the phone. Or phone waked by some other ways.
2.Users touch the sensor to unlock. State is the same to 'Normal use 4'

## Power on

1. Users power up or reboot the phone to login screen.
2. Android phones always need users to use password in this situation.

## Retry

5-10 times retry then force users to use password.

## 3rd party Use

1.Most time when host is running, the fignerprint modudle has nothing to do, so it should be in low power.
2.When 3rd party application calls fprintd, the module should response normally to finish the task then back to standby mode.
