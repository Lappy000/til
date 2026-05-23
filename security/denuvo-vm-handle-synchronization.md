# Denuvo Anti-Tamper VM - Handle-Based Synchronization

When reversing Denuvo's protection VM, the obfuscated code uses Windows kernel objects for inter-thread synchronization rather than simple spin locks.

## Observed Pattern

```
NtWaitForSingleObject(handle, FALSE, NULL)  // VM thread blocks here
```

The handle could be:
- **Thread** — waiting for another thread to finish computation
- **IoCompletionPort** — waiting for async I/O result
- **KeyedEvent** — lightweight semaphore used by NTDLL internally
- **ALPC Port** — inter-process communication

## Diagnosis via NtQueryObject

```c
OBJECT_BASIC_INFORMATION info;
NtQueryObject(handle, ObjectBasicInformation, &info, sizeof(info), NULL);
// Check GrantedAccess bits to determine object type

// Or query type name directly:
NtQueryObject(handle, ObjectTypeInformation, &type_info, size, NULL);
// type_info.TypeName.Buffer == L"Thread" / L"Event" / L"Semaphore"
```

## Key Insight

If `SetEvent()` and `ReleaseSemaphore()` both return ERROR_INVALID_HANDLE/STATUS_OBJECT_TYPE_MISMATCH on the handle, it's likely a **Thread handle** — the VM waits for a worker thread that performs the actual decryption/integrity check.

## Bypassing

Patch the `NtWaitForSingleObject` call to return immediately (`mov eax, 0; ret`) — but only after confirming the waited thread's result isn't consumed downstream.
