# systemd on WSL 2

[Genie](https://github.com/arkane-systems/genie) is used to bottle systemd in WSL 2.

## Setup

1. Install dotnet 3.x (`yay -S dotnet-runtime-bin`)
2. Setup `DOTNET_RUNTIME` enviroment variable  
```bash
export DOTNET_ROOT=/opt/dotnet
```  
!!! Tip
	Setup the environment variable in `~/.profile` or `/etc/environment` in order to get the environment variable in each new shell.
3. Install genie  
```bash
yay -S genie-systemd
```

## Usage

See [genie usage description](https://github.com/arkane-systems/genie#usage).