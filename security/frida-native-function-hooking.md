# Hooking Native Functions at Runtime with Frida Interceptor

When reversing a binary, static analysis tells you *where* a function is, but Frida tells you *what flows through it* live. `Interceptor.attach` lets you read arguments, tamper with return values, and bypass checks without patching the binary on disk.

## Hook by Export Name

```javascript
// frida -f ./target -l hook.js   (or: frida -n target -l hook.js)
const strcmp = Module.getExportByName(null, "strcmp"); // null = search all modules

Interceptor.attach(strcmp, {
  onEnter(args) {
    this.a = args[0].readUtf8String();
    this.b = args[1].readUtf8String();
  },
  onLeave(retval) {
    // Leak every comparison — great for finding the expected password/serial
    console.log(`strcmp("${this.a}", "${this.b}") = ${retval}`);
    retval.replace(0); // force "equal" → bypass the check entirely
  }
});
```

## Hook by Offset (stripped binaries, no symbols)

```javascript
const base = Module.getBaseAddress("target");
const checkLicense = base.add(0x11a40); // RVA from Ghidra/IDA

Interceptor.attach(checkLicense, {
  onLeave(retval) {
    console.log(`check_license() -> ${retval}, forcing 1`);
    retval.replace(1);
  }
});
```

## Why It Beats Patching

- **No file modification** — defeats checksum/anti-tamper that scans the on-disk binary.
- **ASLR-safe** — `Module.getBaseAddress` resolves the live load address every run.
- **Reversible & scriptable** — iterate on logic without rebuilding.

## Gotcha

`onEnter`/`onLeave` share `this` per-call, but reading pointers in `onLeave` that were valid only in `onEnter` will crash — stash them on `this` during `onEnter`. For 64-bit return values, use `retval.replace(ptr("0x1"))`, not a JS number.
