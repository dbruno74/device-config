# Device Config Snap

This is a snap that is used to configure Ubuntu Core based systems. It allows you to create;
* A System User from a System User assertion file dowloaded from an hardcoded http server (192.168.1.100)
* A default network configuration.

## Build
Clone the repo and run the following command.
```
snapcraft --use-lxd
```

## Installation
You can install the snap package that you built with
```
snap install config-snap --dangerous
```

Please do not forget to add required plugs (network-control, snapd-control) in your gadget snap file beforehand. Otherwise, config-snap will not work!
Note that for snapd-control plugin, that's a super-priledged interface, additionally you need to contact support and ask for interface auto-connection.


## Configuration
### Network configuration
Initial configuration could be done by tweaking the `connect-plug-account-control` and `connect-plug-network-setup-control` script.
You can change netplan config at runtime using snap set as described below:

```
snap set config-snap config="content of your netplan.yaml file"
```

Hint: You could use;
```sh
yournetplanyaml=`cat <pathofyourfile>`
snap set config-snap config="$cmd $yournetplanyaml"
```

### System User configuration
To create a System User you need first to create a System User assertion using the script below
https://github.com/dbruno74/toolbox/blob/20/glue/bin/sign-system-user-assertion

Then you need to copy the assertion file on a web server on your LAN and tweaks the `scripts/create_user` script accordingly (HTTP_SERVER_IP, HTTP_SERVER_PORT and ASSERT_NAME)

To test system user creation only with unasserted snap, follow the steps below
  - assure that your device has access to your LAN (e.g. configure with console-conf)
  - create a system user assertion with the script below 
    https://github.com/dbruno74/toolbox/blob/20/glue/bin/sign-system-user-assertion
  - place it in a directory on a linux machine
  - start a web server with the command below     
    sudo python3 -m http.server <port number of your choice> --bind <ip of your linux machine>
  - edit the scripts/create_user and fill in HTTP_SERVER_IP (ip of your linux machine), HTTP_SERVER_PORT (port number of your choice) and ASSERT_NAME (file name of the system user assertion)
  - edit snap/snapcraft.yaml and uncomment the following line:
    install-mode: disable
  - remove the file snap/hooks/configure
  - create the snap
    snapcraft --use-lxd
  - copy the snap on your device (e.g. scp device-config_0.1_arm64.snap <your device ip>:~)
  - on your device, install the unasserted snap
    snap install device-config_0.1_arm64.snap --dangerous
  - on your device, manually connect snapd-control interface
    snap connect device-config:snapd-control
  - on a different terminal, connect to your device and launch journalctl to monitor the user creation
    sudo journalctl --output=short --follow --all
  - launch the daemon
    snap start device-config
  - look at journalctl log for printouts
  Note that further  http tracing can be enabled if needed adding "--trace /tmp/curllog.log" to the curl calls into the scripts/create_user script

## Enjoy

and provide feedback ;)
