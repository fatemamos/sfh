# Design Decisions

## Message Classes
The message-handling system is built around an abstract base class `Message` and three concrete subclasses for the different types: `TEL`, `GPS`, and `SET`.  
This structure keeps validation rules isolated inside each specific message class while sharing common checks—such as sensor ID validation and message type checking—through the base class.  
Using OOP keeps the code more organized, readable, and easy to extend if new message types are needed in the future.

---

## Circular Buffer Structure
The circular buffer is implemented using the classic ring-buffer model with three internal variables:

- **head** – points to the oldest element  
- **tail** – points to the next insertion point  
- **size** – current message count  

This design ensures **O(1)** push and pop operations without shifting any elements in memory.  
It also makes overwrite behavior simple, since advancing the head automatically drops the oldest message.

---

## Flags Design
The buffer uses sticky flags for:

- **overflow**
- **underflow**
- **resize data loss**

A sticky flag stays `True` until it is explicitly read.  
This mimics hardware-style buffers, where events should not be missed even if they occur between operations.

---

## Resize Strategy
The buffer supports increasing and decreasing capacity with clear rules:

- **Increasing size** → always allowed; no data loss.  
- **Decreasing size (safe)** → allowed if the current number of messages fits.  
- **Decreasing size (unsafe)**:
  - If `overwrite=False` → resize is rejected.
  - If `overwrite=True` → oldest messages are dropped, and the data-loss flag is set.

This ensures predictable and controlled behavior when resizing while protecting the user from unexpected data loss.

---

# Edge Cases Handled

- Sensor ID not exactly three digits  
- Sensor ID containing non-numeric characters  
- Unknown message type identifiers  
- Incorrect GPS/SET formats (wrong number of fields, missing commas)  
- Numeric values outside required ranges (battery %, latitude, longitude, etc.)  
- Pushing into a full buffer with overwrite disabled  
- Pushing into a full buffer with overwrite enabled (drops oldest)  
- Popping from an empty buffer (returns `None` or raises exception)  
- Attempting to shrink buffer below current size  
- Resizing rejected when overwrite is off  
- Resizing allowed with overwrite on (keeps newest messages)  
- Flags only clear after being read  

---

# Possible Improvements

- Add a debug/visualization function showing head/tail positions and buffer contents  
- Implement serialization to save and load messages  
- Add thread-safety for multi-threaded or real-time systems  
- Make the buffer iterable (`for msg in buffer`)  
- Add optional logging for buffer events such as overflow and underflow  

