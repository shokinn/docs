# PlayOnLinux

## Install

```shell
# sudo pacman -S playonlinux
```

## Register Browser protocol in Firefox

1. open Confoguration with `about:config`
2. Right Click -> New -> Boolean:
  1. `network.protocol-handler.expose.playonlinux`
  2. Value: `false`
3. Click on the "Thy this update" button
4. Choose the following application:
  1. `/usr/share/playonlinux/bash/playonlinux-url_handler`