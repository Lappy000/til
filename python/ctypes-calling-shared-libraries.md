# Calling C Shared Libraries with ctypes

Using Python's `ctypes` to call functions from C shared libraries (.so/.dll).

## Load a Shared Library

```python
import ctypes
import os

# Linux .so
lib = ctypes.CDLL('./libcrypto.so')

# Windows .dll
lib = ctypes.WinDLL('kernel32.dll')    # stdcall
lib = ctypes.CDLL('user32.dll')         # cdecl

# With explicit path
lib = ctypes.CDLL('/usr/lib/libcustom.so')
```

## Define Function Signatures

```python
# C: int add(int a, int b);
lib.add.argtypes = [ctypes.c_int, ctypes.c_int]
lib.add.restype = ctypes.c_int
result = lib.add(3, 5)  # 8

# C: void process_buffer(char* buf, int size);
lib.process_buffer.argtypes = [ctypes.c_char_p, ctypes.c_int]
lib.process_buffer.restype = None

buf = ctypes.create_string_buffer(b"hello", 10)
lib.process_buffer(buf, 10)
```

## Structures

```python
class Point(ctypes.Structure):
    _fields_ = [
        ("x", ctypes.c_int),
        ("y", ctypes.c_int),
        ("label", ctypes.c_char * 32),
    ]

class Rect(ctypes.Structure):
    _fields_ = [
        ("top_left", Point),
        ("bottom_right", Point),
    ]

# C: void draw_rect(Rect* r);
lib.draw_rect.argtypes = [ctypes.POINTER(Rect)]
lib.draw_rect.restype = None

rect = Rect()
rect.top_left.x = 0
rect.top_left.y = 0
rect.bottom_right.x = 100
rect.bottom_right.y = 100
lib.draw_rect(ctypes.byref(rect))
```

## Callbacks

```python
# C: void register_callback(void (*cb)(int));
CALLBACK_TYPE = ctypes.CFUNCTYPE(None, ctypes.c_int)
lib.register_callback.argtypes = [CALLBACK_TYPE]
lib.register_callback.restype = None

def my_callback(value):
    print(f"Callback called with: {value}")

cb = CALLBACK_TYPE(my_callback)
lib.register_callback(cb)
# Keep reference to cb alive! Otherwise GC frees it -> crash
```

## Pointers and Arrays

```python
# Allocate array
arr_type = ctypes.c_int * 10
arr = arr_type(*range(10))

# Pass as pointer
lib.process_array.argtypes = [ctypes.POINTER(ctypes.c_int), ctypes.c_size_t]
lib.process_array(arr, 10)

# Get pointer to existing data
import array
py_arr = array.array('i', range(10))
ptr = (ctypes.c_int * len(py_arr)).from_buffer(py_arr)
lib.process_array(ptr, len(py_arr))

# Read back
print(list(ptr))
```

## Error Handling

```python
# errno support
lib.some_func.argtypes = [ctypes.c_int]
lib.some_func.restype = ctypes.c_int

result = lib.some_func(-1)
if result == -1:
    err = ctypes.get_errno()
    print(f"Error: {os.strerror(err)}")

# Windows GetLastError
lib.GetLastError.restype = ctypes.c_uint32
err = lib.GetLastError()
```

## Practical Example: Call libc functions

```python
libc = ctypes.CDLL('libc.so.6')

# strlen
libc.strlen.argtypes = [ctypes.c_char_p]
libc.strlen.restype = ctypes.c_size_t
print(libc.strlen(b"hello world"))  # 11

# getpid
libc.getpid.restype = ctypes.c_int
print(libc.getpid())

# printf (variadic)
libc.printf(b"Hello %s! Number: %d\n", b"World", 42)
```

## Practical Example: Windows API

```python
kernel32 = ctypes.WinDLL('kernel32.dll')

# CreateProcessA
class STARTUPINFO(ctypes.Structure):
    _fields_ = [
        ("cb", ctypes.c_uint32),
        ("lpReserved", ctypes.c_char_p),
        # ... 16+ fields
    ]

class PROCESS_INFORMATION(ctypes.Structure):
    _fields_ = [
        ("hProcess", ctypes.c_void_p),
        ("hThread", ctypes.c_void_p),
        ("dwProcessId", ctypes.c_uint32),
        ("dwThreadId", ctypes.c_uint32),
    ]

# kernel32.CreateProcessA(None, "cmd.exe", ...)
```
