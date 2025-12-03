# Design Decisions

## Message Classes
Messages are modeled using an abstract base class `Message` and three subclasses: `TEL`, `GPS`, and `SET`.  
This structure isolates validation logic for each type while keeping shared functionality—such as sensor ID checks and type validation—inside the base class.  
Using OOP helps maintain clean, modular, and easily extendable code.

---

## Circular Buffer Structure
The circular buffer uses a classic **ring-buffer** design with:

- **head** — index of the oldest message  
- **tail** — index for the next insertion  
- **size** — current number of stored messages  

This ensures that buffer operations (`push`, `pop`) are constant time (O(1)) and prevents unnecessary shifting of elements.  
The structure also makes overwrite mode straightforward to implement.

---

## Flags Design
The buffer maintains three internal flags:

- **overflow flag**  
- **underflow flag**  
- **resize data-loss flag**  

All flags use a **sticky** design: once an event occurs, the flag stays `True` until explicitly read.  
This approach mirrors the behavior of hardware buffers, ensuring important events are not missed.

---

## Resize Strategy
Resizing follows strict and predictable rules:

- **Increasing capacity** → always allowed and never causes data loss.  
- **Decreasing capacity (safe)** → allowed if the current number of messages fits the new size.  
- **Decreasing capacity (unsafe)**:  
  - If `overwrite=False` → resize is rejected.  
  - If `overwrite=True` → the buffer drops the oldest messages and sets the data-loss flag.

This approach ensures controlled behavior and prevents accidental data loss.

---

# Edge Cases Handled

- Sensor ID with non-digit characters or incorrect length  
- Unknown message type identifiers  
- Incorrect GPS/SET formats (missing comma, too many fields, etc.)  
- Numeric values outside valid ranges  
- Pushing into a full buffer (overwrite on/off)  
- Popping from an empty buffer  
- Attempting to shrink the buffer while it contains more data than allowed  
- Rejecting unsafe resize operations (overwrite disabled)  
- Correctly keeping the newest messages when overwrite is enabled during resize  
- Sticky flags only clearing when explicitly read  

---

# Possible Improvements

- Add a visual debugging tool showing head/tail positions and internal layout  
- Add serialization support to save/load messages  
- Add thread-safety for concurrent applications  
- Make the buffer iterable to allow:  
  ```python
  for message in buffer:
      ...
