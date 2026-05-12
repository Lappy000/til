# Goldberg Steam Emulator - Offline Authentication

When cracking games that use Steam DRM, Goldberg EMU replaces `steam_api64.dll` to emulate Steam's authentication locally.

## Key Setup

```
game_root/
├── steam_api64.dll          # Goldberg EMU (replaces original ~11MB)
├── steam_appid.txt          # Contains AppID (e.g., "2054970")
├── steam_settings/
│   ├── force_account_name.txt
│   ├── force_language.txt
│   └── force_steamid.txt
└── encrypt_app_ticket.bin   # Forged ticket for games that verify ownership
```

## Forging `encrypt_app_ticket.bin`

Games using `ISteamAppTicket::GetAppOwnershipTicketData()` need a valid ticket structure:

```c
struct AppTicket {
    uint32_t ticket_size;
    uint32_t app_id;
    uint64_t steam_id;
    uint32_t timestamp;
    // ... HMAC signature (Goldberg uses hardcoded key)
};
```

## Gotcha

If the game validates tickets against a remote server (e.g., via `runtime_il2cpp.exe` wrapper), you need the wrapper to skip that check — not just the emulator.

## References

- [Goldberg EMU GitLab](https://gitlab.com/Mr_Goldberg/goldberg_emulator)
- `gbe_fork` for latest maintained version
