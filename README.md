# Design Choices

## Message Representation
The message system is built using an object-oriented hierarchy.  
A base `Message` class contains shared fields (sensor ID, message type), and each specific message type (`TEL`, `GPS`, `SET`) is implemented as its own subclass.  
Each subclass handles its own parsing logic and validation rules.  

This design avoids large `if/else` blocks and keeps the code modular, clear, and easy to extend if new message types are added in the future.

---

## Buffer Implementation
The circular buffer uses a fixed-size Python list together with two pointers:

- **head** → points to the oldest message  
- **tail** → points to the next free slot  

This structure guarantees **O(1)** push/pop operations and avoids shifting data like a regular list would.  
The buffer is memory-efficient and behaves like a proper FIFO queue.

---

## Event Flags
The buffer supports three internal flags:

- **overflow flag** – set when pushing into a full buffer  
- **underflow flag** – set when popping from an empty buffer  
- **data-loss-resize flag** – set when resizing causes message loss  

All flags use a **sticky** design:  
they remain `True` until the user explicitly checks them.  
This makes it easy to detect rare events that may happen between operations.

---

## Resizing Rules
The buffer supports dynamic resizing with predictable behavior:

- **Increasing capacity**: always allowed; all messages preserved.
- **Decreasing capacity (safe)**: allowed when current size ≤ new capacity.
- **Decreasing with data loss**: allowed only if overwrite mode is enabled.
- **Decreasing with data loss but overwrite=False**: resize is rejected.

If a resize operation causes message loss, the buffer sets the `data_loss_resize_flag` so the user knows the buffer dropped old data.

---

# Edge Cases Considered

- Messages with incorrect length or non-digit sensor IDs  
- Unknown message type strings  
- Missing commas or extra arguments in GPS/SET messages  
- Out-of-range values (battery %, latitude, longitude, msgs_per_second)  
- Pushing to a full buffer with overwrite on/off  
- Popping from an empty buffer with/without raising an exception  
- Resizing smaller than the current message count  
- Rejecting unsafe resize attempts  
- Ensuring flags are set and cleared consistently  
- Ensuring FIFO order remains valid after resizing  

---

# Future Enhancements

- Add buffer visualization or debugging tools  
- Add thread-safety for multi-threaded applications  
- Add serialization (export/import of messages)  
- Make the buffer iterable (`for msg in buffer:`)  
- Add optional logging to monitor push/pop/resize events  

