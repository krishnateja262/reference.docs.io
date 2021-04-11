## Write Ahead Logging

Fundamental block for all databases. It provides durability guarantee even if the storage engine does not flush the data to the disk.

Command which changes the state of the database will be recorded in the log before applying to the database.\
In case of failures in the database the state can be restored from the latest snapshot + replaying all the commands applied to the database from the time of snapshotting

Why is it append only
1. We only need to add commands to the end of the file
2. Sequential access is much faster than random access

 A single log is maintained for each server process. Each log entry is given a unique identifier which helps in
 1. Segmented Log
 2. Cleaning the log with Low-Water mark

### Record Format

Let each record be represented as byte array
```
type Record struct {
  len uint32
  data []byte
  crc uint32
}
```
`len` occupies 4 bytes and `data` occupies `len` bytes. It is straight forward to deserialize record to the above struct.
1. Read first 4 bytes to get the length
2. Read len bytes of data
3. Verify len and data together yeild same checksum as crc

The program need to either create/open a log file first in append only mode. When we send the record to append method the data will be written to the file
But there are some caveats
1. Applications generally have buffer to batch the log append records and flush them to file when enough have accumulated
2. Operation Systems often cache data in memory before witing to disk
3. Disk drivers can have their own layer of cache who dont usually give control access to the application

With the above scenarios there is never 100% data durability guarantee\
In general to avoid those situations
1. Flush writes for every record. Pay the price of throughput
2. Call fsync for every record. Pay the price of throughput
3. No solution


`fsync` is called N times for a certain time window to maintain the balance between durability guarantee and throuhgput.
