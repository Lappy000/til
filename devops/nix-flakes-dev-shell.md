# Nix Flakes for Dev Environments

## Minimal flake.nix

```nix
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  outputs = { self, nixpkgs }: {
    devShells.x86_64-linux.default = nixpkgs.legacyPackages.x86_64-linux.mkShell {
      packages = with nixpkgs.legacyPackages.x86_64-linux; [
        python312 nodejs_20 go_1_22 postgresql_16
      ];
    };
  };
}
```

## Usage
```bash
nix develop              # enter shell
nix develop --command python3 -c "print('hi')"
nix flake update         # update deps
```

## vs Docker for Dev

| Aspect | Nix | Docker |
|--------|-----|--------|
| File access | Native | Bind mounts (slow macOS) |
| GPU | Native | Requires config |
| Startup | Instant | Seconds |
| Reproducibility | Exact (hash) | Approximate |
