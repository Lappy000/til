# Parsing Binary Protocols with struct.unpack

Using Python's `struct` module to parse network packets and binary file formats.

## Basic Unpacking

```python
import struct

# Format chars: < little-endian, > big-endian
# B=uint8, H=uint16, I=uint32, Q=uint64
# s=char[], f=float, d=double

# Parse a TCP header (big-endian)
tcp_header = struct.unpack('>HHIIBBHHH',
    raw_header)  # 20 bytes
src_port, dst_port, seq, ack, offset, flags, window, checksum, urgent = tcp_header
```

## Custom Binary Parser

```python
import struct

class BinaryReader:
    def __init__(self, data, endian='<'):
        self.data = data
        self.pos = 0
        self.endian = endian

    def read(self, fmt):
        size = struct.calcsize(self.endian + fmt)
        result = struct.unpack_from(self.endian + fmt, self.data, self.pos)
        self.pos += size
        return result

    def uint32(self): return self.read('I')[0]
    def uint16(self): return self.read('H')[0]
    def uint8(self):  return self.read('B')[0]
    def string(self, n): return self.read(f'{n}s')[0].decode('utf-8', errors='replace')
    def float(self):  return self.read('f')[0]

# Parse a custom protocol header
reader = BinaryReader(raw_data, endian='<')
magic = reader.uint32()
version = reader.uint16()
flags = reader.uint8()
name = reader.string(32)
payload_size = reader.uint32()
```

## C struct Mapping

```c
struct header {
    uint32_t magic;      // reader.uint32()
    uint16_t version;    // reader.uint16()
    uint8_t  flags;      // reader.uint8()
    char     name[32];   // reader.string(32)
    uint32_t payload_size;
};
```
