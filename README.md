# TR

TR or Terminal Record is a binary format though to store terminal session in a
simple and stream able way.

## File structure

### Header

The header contains a magic number used to identify the file, a version field, a
timestamp indicating when the session started, an initial number of column and
rows. The header should have **16 bytes**.

| Field        | Size (bytes) | Description                           |
|--------------|--------------|---------------------------------------|
| Magic Number | 2            | Identifies the file format            |
| Version      | 2            | Version of the terminal record format |
| Timestamp    | 8            | Start time of the session (UNIX time) |
| Cols         | 2            | Initial number of terminal columns    |
| Rows         | 2            | Initial number of terminal rows       |
#### Example

| magic number | version | timestamp       | cols | rows |
| ------------ | ------- | --------------- | ---- | ---- |
| 84 82        | 0 0     | 0 0 0 0 0 0 0 0 | 0 0  | 0 0  |

### Event

After the header, the file will contain N data segments representing events in
the session with a timestamp related to the initial one, the events type,
*input*, *output* or a *resize*, a size, presenting the length of the data that
the events produced and the data itself. The event's size is variable, but the
minimum value should be **12 bytes**.

| Field      | Size (bytes) | Description                           |
| ---------- | ------------ | ------------------------------------- |
| Timestamp  | 8            | Time relative to the start timestamp  |
| Event Type | 1            | Type of event (input, output, resize) |
| Size       | 2            | Length of the data segment            |
| Data       | Variable     | Actual data of the event              |

#### Example

| timestamp       | event | size | data |
| --------------- | ----- | ---- | ---- |
| 0 0 0 0 0 0 0 0 | 0     | 0 0  | ...  |

## Questions

- Why don't use a JSON format?
	Comparing the same file structure, considering each letter as a byte, the same structure in a JSON file could consume at minimum **52 bytes** only on the header.
	
	Example JSON:
	```json
	 {
		"version": 1, 
		"timestamp": 0, 
		"cols": 80, 
		"rows": 24,
		"data": [
			{
				"timestamp": 0, 
				"event": 0,
				"size": 0,
				"data": "",
			},
		],
	 }
	```
	 
	In General, the advantages are:
	- Few bytes
	- Faster parsing
	- Stream able
