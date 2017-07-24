# Booting up a VHDX
Since Windows 7, there has been the ability to boot your PC directly into a virtual hard disc, giving you performance close to an otherwise unvirtualised operating system.

Windows 8 or above is required to boot into a VHDX and also Windows 10.

## Creating the VHDX
I first create a VHDX and install Windows using Hyper-V. Hyper-V can be enabled via "Turn Windows features on or off". I typically store the VHDX in an easily assessable location such as `C:\VMs`.

## Configuring the boot manager
You use `bcdedit` from an administrator command prompt to modify your boot configuration. This can be done in the host OS and also while booted into a VHDX.

You can view all your boot records by running:
```bat
bcdedit
```

To create a new boot record for your VHDX, follow these steps:

1. Copy an existing boot record such as your currently loaded one.
```bat
bcdedit /copy {current} /d "My VHDX"
```

2. Set the device and osdevice parameter to your VHDX path. Use the GUID of your newly created record.
```bat
bcdedit /set {cd67a0a4-b586-11e6-bf4e-bc8385086e7d} device vhd=[C:]\VMs\donkey.vhdx
bcdedit /set {cd67a0a4-b586-11e6-bf4e-bc8385086e7d} osdevice vhd=[C:]\VMs\donkey.vhdx
```

Don't worry about the drive letter, that is always set and displayed relative to your current boot. For example it will show up as `[D:]` when you are booted into the VHDX.

3. Some other parameters you might like to set:
```bat
REM User friendly description that shows on boot screen
bcdedit /set {cd67a0a4-b586-11e6-bf4e-bc8385086e7d} description "Windows 10 VHDX"

REM Required for some x86-based systems to detect certain hardware information.
bcdedit /set {cd67a0a4-b586-11e6-bf4e-bc8385086e7d} detecthal on

REM enable Hyper-V (this is what windows features does for you)
bcdedit /set {cd67a0a4-b586-11e6-bf4e-bc8385086e7d} hypervisorlaunchtype Auto
```

After you have set up your new boot record for the VHDX, you can restart your computer and the boot screen should show up. Moving the mouse when the boot screen is displayed will stop the automatic timer. From the boot screen you can change the default boot record and the automatic time limit.
