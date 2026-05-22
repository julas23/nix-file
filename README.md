# NixOS

Configuração completa do NixOS mantida em repositório dedicado:

**`~/Vault/GIT/nixos/`**

## Estrutura do repo nixos

```
nixos/
├── configuration.nix       # entry point
├── installer.py            # instalador interativo (TUI)
├── install.sh              # script de instalação
├── modules/
│   ├── core/               # boot, network, system, nix
│   ├── desktop/            # xfce, hyprland, gnome, plasma, i3...
│   ├── hardware/           # audio, bluetooth, gpu
│   ├── locale/             # i18n, timezone, xkb
│   ├── packages.nix        # lista de pacotes
│   ├── services/           # docker, ssh, lsyncd, ollama...
│   └── storage/            # zfs, lvm, volumes
├── profiles/
│   ├── desktop.nix
│   ├── developer.nix
│   ├── minimal.nix
│   └── server.nix
└── dotfiles/
    └── awesome/
```
