I attempted to use flatbuffers to send lots of data between Python and Rust programs.  This was my first time using flatbuffers.  

##  Challenges
- Compared to, say, protobuf, the flatbuffers documentation is minimal, and finding examples relevant to my particular use case wasn't easy.
- It's not a streaming format in itself. Like protobuf, flatbuffers _can_ be saved to disk or streamed; however, out-of-the-box they don't have streaming support.

## Send length-prefixed chunks
Here's a summary of the solution I used: Wrap an arbitrary number of core messages in a flatbuffer vector within a flatbuffer table.  Flatten the table, measure the size of the flattened buffer, and stream that length prefix followed by the body of the buffer. 

### The Writer / Sender 
On the writing / sending end:
- Create a basic message type (this could be the location of a player in a game, or the kinematics of a robot, or 6DOF of a drone)
- Wrap an arbitrary number of these messages in a flatbuffer vector, as a field in a flatbuffer table
- Push messages into the vector (and thus into the table)
- Finish the vector, finalize the table, measure the flattened size of the table
- Write the flattened size to disk/stream as a little-endian encoding uint32
- Write the flattened buffer to disk

Here's a python sample:
```
import flatbuffers
import struct
import Simulator.Event
import Simulator.FrameData

def generate_events(rising_locations, falling_locations, event_file):
  fbb = flatbuffers.Builder(0)

  total_events = len(rising_locations) + len(falling_locations)
  Simulator.FrameData.FrameDataStartEventsVector(fbb, total_events)

  # create a series of flatbuf events and insert into vector
  for item in rising_locations:
    x = item[0][0]
    y = item[0][1]
    event = Simulator.Event.CreateEvent(fbb, x, y, 1)
    fbb.PrependUOffsetTRelative(event)

  for item in falling_locations:
    x = item[0][0]
    y = item[0][1]
    event = Simulator.Event.CreateEvent(fbb, x, y, 0)
    fbb.PrependUOffsetTRelative(event)

  # done creating the vector of events
  all_events = fbb.EndVector(total_events)

  # now start the wrapping table
  Simulator.FrameData.FrameDataStart(fbb)
  # add the event vector
  Simulator.FrameData.FrameDataAddEvents(fbb, all_events)
  framed_data = Simulator.FrameData.FrameDataEnd(fbb)
  fbb.Finish(framed_data)
  # grab the flattened data buffer 
  buf = fbb.Output()
  flatbuf_len = len(buf)
  print("total events: %d flatbuf size: %d" % (total_events, flatbuf_len))
  # encode the chunk with a LE-encoded length prefix
  event_file.write(struct.pack('<L', flatbuf_len))
  event_file.write(buf)
```

### The Reader / Receiver
On the reading / receiving end:
- Read the LE-encoded length prefix
- Read exactly length bytes into a buffer
- Use the `get_root` methods to obtain the table root
- Dig into the vector which contains an arbitrary number of messages

Here's a rust sample:

```
use flatbuffers;
use std::io::Read;
use std::io::BufReader;
use byteorder::{LittleEndian, ReadBytesExt};

#[allow(dead_code, unused_imports)]
mod simulator_flatbuf_generated;
use crate::simulator_flatbuf_generated::simulator;
use crate::simulator_flatbuf_generated::simulator::FrameData;
use crate::simulator_flatbuf_generated::simulator::Event;
...

  let  f = std::fs::File::open("./out/events.dat").unwrap();
  let mut buf_reader = BufReader::new(f);

  let mut chunk_len:u32 = 0;
  while {
    chunk_len = buf_reader.read_u32::<LittleEndian>().unwrap_or(0);
    println!("chunk_len: {}", chunk_len);
    chunk_len > 0 } {
    let mut buf:Vec<u8> = vec![0; chunk_len as usize];

    buf_reader.read_exact(&mut buf).expect("couldn't read_exact");
    let rooty:FrameData = flatbuffers::get_root::<simulator::FrameData>(&buf);
    let evts = rooty.events().expect("no events");
    println!("events: {}", evts.len());
    // TODO actually use the events
  }
```

### The Schema 
Here's roughly the schema file (`.fbs` file) I used:

```
namespace Simulator;

struct Event {
    x: uint16;
    y: uint16;
    rising:uint8;
}

table FrameData {
    events: [Event];
}
```



## Notes and open questions

- There are some methods in the flatbuffers API related to `Prefix` -- it sounds like these might be intended to automatically send length prefix, but there's zero documentation. 
- For any flatbuffers samples/examples, the minimal set of things needed to form a useful example is:
  - The schema file (`.fbs` file) used
  - The writer code (the code writing the flatbuffer)
  - The reader code (the code reading the flatbuffer)
  
