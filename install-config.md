# Initial setup for kmonad

Config file goes in `$HOME/.config/kmonad/config.kbd`

For development & testing, run:

```sh
kmonad ~/.config/kmonad/config.kbd
```

Once config is set, enable as service

## Configure to run as a user service

1. Ensure executable is in correct location

    ```sh
    sudo cp ./kmonad /usr/bin/kmonad
    ```

1. Create a service file for KMonad in $HOME/.config/systemd/user/kmonad.service with below contents:

    ```sh
    [Unit]
    Description=KMonad keyboard service
    After=network.target

    [Service]
    Type=simple
    ExecStart=/usr/bin/kmonad $HOME/.config/kmonad/config.kbd
    Restart=always
    RestartSec=5
    StandardOutput=null
    StandardError=null
    User=jeremy
    Group=jeremy

    [Install]
    WantedBy=default.target
    ```

1. Reload the systemd daemon: `systemctl --user daemon-reload`
1. Enable the service: `systemctl --user enable kmonad.service`
1. Start the service: `systemctl --user start kmonad.service`
1. Check the status: `systemctl --user status kmonad.service`

## Add user to groups to prevent needing `sudo`

1. Check for `input` and `uinput` groups:
   - Open a terminal and run: `groups`
   - If you don't see input or uinput listed, you need to create them:
        - `sudo groupadd input`
        - `sudo groupadd uinput`
1. Add User to Groups:
    - Add user to the `input` and `uinput` groups:
      - `sudo usermod -aG input,uinput $USER`
    - Replace `$USER` with your actual username if needed
1. Verify by doing ONE of the following:
    - Log out and log back in
    - Restart system
    - Run `su - $USER`
1. Run groups again to confirm that your user is now a member of both groups.
1. `udev` Rule (Optional but Recommended):
    1. Create a udev rule to ensure the correct permissions for the `uinput` device file:
        - Create a file (e.g., `kmonad.rules` or `90-kmonad.rules`) in `/etc/udev/rules.d/`: `sudo nano /etc/udev/rules.d/kmonad.rules`
        - Add the following content:

            ```sh
            KERNEL=="uinput", MODE="0660", GROUP="input"
            SUBSYSTEM=="input", ACTION=="add", ENV{ID_INPUT_DEVICE_IGNORE}="1"
            SUBSYSTEM=="input", ACTION=="add", ENV{ID_INPUT_DRIVER}!="uinput", ENV{ID_INPUT_DRIVER}!="evdev", ENV{ID_INPUT_DRIVER}!="hid", ENV{ID_INPUT_DRIVER}!="serdev", ENV{ID_INPUT_DEVICE_IGNORE}="1"
            ```

    1. Save the file and reload udev rules: `sudo udevadm control --reload-rules`
    1. Check uinput Permissions:
        - Ensure the uinput device file has the correct permissions: `sudo ls -l /dev/uinput`
        - It should be owned by root and have read/write permissions for the input and uinput groups.

1. After Permissions:
You should now be able to run `kmonad` without `sudo`

ref [codebenchers.com](https://codebenchers.com/blog/kmonad-on-ubuntu-program-your-keyboard#installing-as-a-service)