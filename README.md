# TR

TR (Terminal Record) is a binary format used to store terminal sessions in a
simple and streamable way. It consists of a header followed by a series of
events. Each event records the terminal size at the time, along with the event
data (whether it's input, output, or a resize).

## File Structure

### Header

The header contains a magic number to identify the file, a version field, and a
timestamp indicating when the session started.

| Field        | Size (bytes) | Description                           |
|--------------|--------------|---------------------------------------|
| Magic Number | 2            | Identifies the file format            |
| Version      | 2            | Version of the terminal record format |
| Timestamp    | 8            | Start time of the session (UNIX time) |

#### Example

| Magic Number | Version | Timestamp            |
|--------------|---------|----------------------|
| 84 82        | 0 0     | 0 0 0 0 0 0 0 0      |

### Event

After the header, the file contains a series of events. Each event includes:

- A **timestamp** relative to the session start.
- An **event type** (input, output, or resize).
- A **size** field representing the length of the data in the event.
- The **terminal size** at the time of the event (number of columns and rows).
- The actual **data** for the event (this varies based on event type).

#### Event Types

- **Input (0)**: Data typed into the terminal by the user.
- **Output (1)**: Data output by the terminal.
- **Resize (2)**: Data representing a change in terminal size (number of columns
    and rows).

| Field         | Size (bytes) | Description                                  |
|---------------|--------------|----------------------------------------------|
| Timestamp     | 8            | Time relative to the start timestamp         |
| Event Type    | 1            | Type of event (0 = input, 1 = output, 2 = resize) |
| Size          | 2            | Length of the data segment                   |
| Cols          | 4            | Number of columns at the time of the event   |
| Rows          | 4            | Number of rows at the time of the event      |
| Data          | Variable     | Actual data for the event (depends on the event type) |

#### Resize Event Data

- **Resize Event**: Even for resize events, the terminal size is recorded in the
    `Cols` and `Rows` fields, representing the size of the terminal **after the
    resize**.

#### Example (Resize Event)

| Timestamp       | Event Type | Size | Cols | Rows | Data           |
|-----------------|------------|------|------|------|----------------|
| 0 0 0 0 0 0 0 0 | 2          | 0    | 100  | 40   | (empty data)   |

In this example, the `resize` event records the new terminal size (100 columns
and 40 rows). The `Size` field is 0 because no actual data is associated with
this event (it's just a resize).

#### Example (Input Event)

| Timestamp       | Event Type | Size | Cols | Rows | Data          |
|-----------------|------------|------|------|------|---------------|
| 0 0 0 0 0 0 0 0 | 0          | 5    | 80   | 24   | "Hello"       |

This represents an input event where the user typed "Hello" into the terminal.
The terminal size at the time was 80 columns and 24 rows.

#### Example (Output Event)

| Timestamp       | Event Type | Size | Cols | Rows | Data          |
|-----------------|------------|------|------|------|---------------|
| 0 0 0 0 0 0 0 0 | 1          | 4    | 80   | 24   | "Goodbye"     |

This represents an output event where the terminal outputs "Goodbye". The
terminal size at the time was 80 columns and 24 rows.

### Example File Flow

1. **Header**: Contains magic number, version, and start timestamp.
2. **Event 1** (Resize): The terminal is resized to 100 columns and 40 rows.
3. **Event 2** (Input): The user types "Hello" with the terminal at 80 columns
   and 24 rows.
4. **Event 3** (Output): The terminal outputs "Goodbye" while still at 80
   columns and 24 rows.

---

## Why Not Use JSON?

Compared to using a JSON format, TR is much more compact. For the same file
structure, a JSON file could consume at least **52 bytes** just for the header,
while the binary TR format is more efficient.

Example JSON:
```json
{
  "version": 1,
  "timestamp": 0,
  "data": [
    {
      "timestamp": 0,
      "event": 0,
      "size": 5,
      "cols": 80,
      "rows": 24,
      "data": "Hello"
    }
  ]
}
```

**Advantages of TR Format:**
- **Compact**: The binary format uses fewer bytes, making it more efficient.
- **Faster Parsing**: Binary data can be parsed much faster than text-based
    formats like JSON.
- **Streamable**: The data can be processed on the fly, which is useful for
    long-running terminal sessions.
