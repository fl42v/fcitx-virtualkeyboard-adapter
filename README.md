# fcitx-virtual-keyboard-adapter

(Note: not a real `fcitx::VirtualKeyboardUserInterface`)

A simple addon that calls commands when a text field is focused/unfocused

A stopgap until I find a good enough on-screen-keyboard that auto shows/hides...

## Usage

Build and install addon with

```
mkdir build
cd build
cmake ..
cmake --build .
make install
```

Specify activate/deactivate commands with:

```
# ~/.config/fcitx5/conf/virtualkeyboardadapter.conf

ActivateCmd="pkill -SIGUSR2 wvkbd"
DeactivateCmd="pkill -SIGUSR1 wvkbd"
```

### NixOS (flakes)

Add to flake inputs:
```(nix)
# flake.nix

{
  inputs = {
    nixpkgs = "github:NixOS/nixpkgs/nixos-unstable";
    fcitx-virtualkeyboard-adapter = {
      url = "github:horriblename/fcitx-virtualkeyboard-adapter";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };
# ... the rest of the flake
}
```
Then use either the package
```(nix)
# flake.nix

{
  # ... inputs
  outputs = { self, nixpkgs, ... }@inputs: {
    nixosConfigurations.<your_hostname> = nixpkgs.lib.nixosSystem {
      modules = [
        ./configuration.nix
        {
          config = {
            i18n.inputMethod = {
              enable = true;
              type = "fcitx5";
              fcitx5.addons = [
                inputs.fcitx-virtualkeyboard-adapter.${pkgs.stdenv.hostPlatform.system}.default
              ];
            };
          };
        }
      ];
    };
  };
}
```
> Note: if you want to configure fcitx outside of `flake.nix` (e.g. in `configuration.nix` or via home-manager), make sure to pass inputs via [`config.nixpkgs.specialArgs`](https://nixos.org/manual/nixpkgs/stable/#module-system-lib-evalModules-param-specialArgs)

or the overlay:
```(nix)
# flake.nix

{
  # ... inputs
  outputs = { self, nixpkgs, ... }@inputs: {
    nixosConfigurations.<your_hostname> = nixpkgs.lib.nixosSystem {
      modules = [
        ./configuration.nix
        {
          config = {
            nixpkgs.overlays = [
              inputs.fcitx-virtualkeyboard-adapter.overlays.default
            ];
            i18n.inputMethod = {
              enable = true;
              type = "fcitx5";
              fcitx5.addons = [
                virtualkeyboard-adapter
              ];
            };
          };
        }
      ];
    };
  };
}
```
Then either follow the configuration step from Usage or create `virtualkeyboardadapter.conf` with home-manager.

## Troubleshooting

1. Make sure fcitx is running: `ps aux | grep fcitx`. If it's not, start it with `fcitx5 -d` and try again;
2. Make sure the activate/deactivate commands work from terminal. 
