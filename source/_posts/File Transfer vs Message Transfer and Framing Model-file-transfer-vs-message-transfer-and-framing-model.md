---
title: File Transfer vs Message Transfer and Framing Model
date:  2019-07-16 12:00:00
---
> In the last post, I have illustrated a simple way to address sticky & lost package during socket transfer, which works for small messages. In this post, I would like to draw out attention to File transfer, which shares a lot of things in common with normal message transfer but also has its own particularities which could lead to new problems. 

## Difference between File Transfer and Message Transfer
-  The data size of a file could be far larger than normal message 
......
Well, one is enough for us to be preoccupied since it is indeed the most important difference that cannot be underestimated if we come to see its problems.
1.  As there is great amount of data to transfer, if we transfer them all at once (let's say it takes 10 minutes to transfer all), the next message cannot be sent until 10 minutes afterwards as the socket has been fully occupied by this big file
2.  If we get frustrated and want to cancel the transfer, it will become worse as the server is waiting for the whole file being completely transferred, and hence blocked the whole socket transfer.
3.  Even if we only have one big file to transfer, with the solution introduced in my last post, there will be a big amount of buffer being used, which could obviously leads to out of memory if we are talking about many gegabytes files.  
4. If during the transfer, the network crash, then we dont know at all which part has been trasferred and which part has not been transferred, which forces us to retransfer the whole file again. 

How could we address all of them in an elegant way?  Framing come to rescue. 

## What is Frame
A frame is a simple container for a single network packet. In our case, it could simply be a fix sized container that could transfer a part of the whole file or message with supplementary informations for us to be able to recombine them during reception. 

Imagine a following frame data structure:

```
<Frame: capacity(64KB)><{
  Length: 2 Bytes [0~65535], // size of payload
  Type: 1 Byte [-128~127],   // type id
  Flags: 1 Byte [00000000],  // encryption id
  Identifier: 1 Byte[1~255], // parent packet id
  Other: 1 Byte [00000000],  // other things
  Payload: X Bytes [X,X,X...]// payload data 
}>
```

In this data structure, we have a fixed header part which represents 6 bytes. 

The Type attributes could be 
 - TYPE_PACKET_HEADER 
   It could be used as the first frame for a big packet (file or message) to send, transporting the information of the whole packet for later combination usage. 
   Typically, we could have packet length [5 bytes] and packet type[one byte]. It is interesting to mention that through the type of packet, we could then determine whether we are receving a file or a message. 
  
 - TYPE_PACKET_ENTITY
   This will be the entity frame containing frame header part and the frame entity part containing part of the binary data (file or message). 
 - TYPE_COMMAND_SEND_CANCEL
   Special type of frame which indicates that a packet transfer has been canceled.
 - ...

## File Transfer Packet and Framing
##### Sending Frame logic
Whenever we want to send a file, we could define a FileSendPacket like the following. Note that we have used InputStream to enable streaming to prevent unnecessary buffer.

```
public class FileSendPacket extends SendPacket<FileInputStream> {
    private final File file;

    public FileSendPacket(File file) {
        this.file = file;
        this.length = file.length();
    }

    @Override
    public byte type() {
        return TYPE_STREAM_FILE;
    }

    /**
     * use file to build readable FileInputStream
     *
     * @return file readable stream
     */
    @Override
    protected FileInputStream createStream() {
        try {
            return new FileInputStream(file);
        } catch (FileNotFoundException e) {
            return null;
        }
    }
}

```

With the FileSendPacket, we could firstly build a SendHeaderFrame.  
 
```
public boolean requestTakePacket() {
        synchronized (this) {
            if (nodeSize >= 1) {
                return true;
            }
        }
        SendPacket packet = provider.takePacket();
        if (packet != null) {
            // am identifier will be generated and put in each frame header at the 5th byte position
            short identifier = generateIdentifier();
            SendHeaderFrame frame = new SendHeaderFrame(identifier, packet);
            // append the first sendHeaderFrame
            appendNewFrame(frame);
        }
        synchronized (this) {
            return nodeSize != 0;
        }
}

...

public class SendHeaderFrame extends AbsSendPacketFrame {
    static final int PACKET_HEADER_FRAME_MIN_LENGTH = 6;
    private final byte[] body;

    public SendHeaderFrame(short identifier, SendPacket packet) {
        super(PACKET_HEADER_FRAME_MIN_LENGTH,
                Frame.TYPE_PACKET_HEADER,
                Frame.FLAG_NONE,
                identifier,
                packet);

        final long packetLength = packet.length();
        final byte packetType = packet.type();
        final byte[] packetHeaderInfo = packet.headerInfo();

        body = new byte[bodyRemaining];

        body[0] = (byte) (packetLength >> 32);
        body[1] = (byte) (packetLength >> 24);
        body[2] = (byte) (packetLength >> 16);
        body[3] = (byte) (packetLength >> 8);
        body[4] = (byte) (packetLength);

        body[5] = packetType;

        if (packetHeaderInfo != null) {
            System.arraycopy(packetHeaderInfo, 0, body, PACKET_HEADER_FRAME_MIN_LENGTH, packetHeaderInfo.length);
        }
    }

    @Override
    protected int consumeBody(IoArgs args) {
        int count = bodyRemaining;
        int offset = body.length - count;
        return args.readFrom(body, offset, count);
    }

    @Override
    public Frame buildNextFrame() {
        InputStream stream = packet.open();
        ReadableByteChannel channel = Channels.newChannel(stream);
        return new SendEntityFrame(getBodyIdentifier(), packet.length(), channel, packet);
    }
}


```

Then we could write sendHeaderFrame to a intermediate buffer IoArgs. 

```
public IoArgs fillData() {
    Frame currentFrame = getCurrentFrame();
    if (currentFrame == null) {
        return null;
    }

    try {
        // if sendHeaderFrame, then put its header and body to args
        // if sendEntityFrame, then put its header and body from inputstream to args
        if (currentFrame.handle(args)) {
            // currentFrame has been consumed
            Frame nextFrame = currentFrame.nextFrame(); // if headerFrame => nextFrame is entityFrame if not then nextFrame is still entityFrame
            if (nextFrame != null) {
                appendNewFrame(nextFrame);
            } else if (currentFrame instanceof SendEntityFrame) {
                // the last sendEntityFrame
                provider.completedPacket(((SendEntityFrame) currentFrame).getPacket(), true);
            }
            // popCurrentFrame
            // if nodeSize == 0 then requestTakePacket
            popCurrentFrame();
        }
        return args;
    } catch (IOException e) {
        e.printStackTrace();
    }

    return null;
}
```

Then we would write the intermediate buffer to socket channel. 

``` 
try {
    // writeTo operation to consume args
    if (args == null) {
        processor.onConsumeFailed(null, new IOException("ProvideIoArgs is null."));
    } else {
        int count = args.writeTo(socketChannel);
        if (count == 0) {
            System.out.println("Current write zero data!");
        }

        if (args.remained()) {
            // attach unconsumed args
            attach = args;
            // complete writeTo callback
            ioProvider.registerOutput(socketChannel, this);
        } else {
            attach = null;
            // write finished callback
            processor.onConsumeCompleted(args);
        }
    }
 ...

public void onConsumeCompleted(IoArgs args) {
    synchronized (isSending) {
        isSending.set(false);
    }
    requestSend();
}

```

The requestSend method will register for writable event again, once it became again writable, it will once again call fillData() to construct the next SendEntityFrame to fill data to the IoArgs until there all packet has been separated to frames. 

In SendEntityFrame we have a unConsumeEntityLength to mark the rest packet length to read from the FileInputStream so that we know when to stop building next frame.

Here is the model of SendEntityFrame.

```
public class SendEntityFrame extends AbsSendPacketFrame {
    private final ReadableByteChannel channel;
    // the rest packet length to send
    private final long unConsumeEntityLength;

    SendEntityFrame(short identifier,
                    long entityLength, // the whole packet length
                    ReadableByteChannel channel,
                    SendPacket packet) {
        super((int) Math.min(entityLength, Frame.MAX_CAPACITY),
                Frame.TYPE_PACKET_ENTITY,
                Frame.FLAG_NONE,
                identifier,
                packet);
        this.unConsumeEntityLength = entityLength - bodyRemaining;
        this.channel = channel;
    }

    @Override
    protected int consumeBody(IoArgs args) throws IOException {
        if (packet == null) {
            // current frame has been stopped, fill fake data
            return args.fillEmpty(bodyRemaining);
        }
        return args.readFrom(channel);
    }

@Override
public Frame buildNextFrame() {
    if (unConsumeEntityLength == 0) {
        return null;
    }
    return new SendEntityFrame(getBodyIdentifier(),
            unConsumeEntityLength, channel, packet);
}
} 
```
##### Receiving Frame logic

As we receive from the socket channel, we just need to receive the binary data frame by frame. By reading the frame type, we could know if it is a header frame or entity frame. 

```
public static AbsReceiveFrame createInstance(IoArgs args) {
        byte[] buffer = new byte[Frame.FRAME_HEADER_LENGTH];
        args.writeTo(buffer, 0);
        byte type = buffer[2];
        switch (type) {
            case Frame.TYPE_COMMAND_SEND_CANCEL:
                return new CancelReceiveFrame(buffer);
            case Frame.TYPE_PACKET_HEADER:
                return new ReceiveHeaderFrame(buffer);
            case Frame.TYPE_PACKET_ENTITY:
                return new ReceiveEntityFrame(buffer);
            default:
                throw new UnsupportedOperationException("unsupported frame type" + type);
        }
    }

... 


/**
 * this is frame header handling logic
 * build a new frame with help the frame header bytes (6 bytes)
 *
 * @param args
 * @return
 */
private Frame buildNewFrame(IoArgs args) {
    AbsReceiveFrame frame = ReceiveFrameFactory.createInstance(args);
    if (frame instanceof CancelReceiveFrame) {
        cancelReceivePacket(frame.getBodyIdentifier());
        return null;
    } else if (frame instanceof ReceiveEntityFrame) {
        WritableByteChannel channel = getPacketChannel(frame.getBodyIdentifier());
        ((ReceiveEntityFrame) frame).bindPacketChannel(channel);
    }
    return frame;
}
```

The first time we read a HeaderFrame, we could build a outputStream and conserve it into a packetMap. Then when it comes to a EntityFrame, we could refind the outputStream for given readable socket channel and write to its corresponding outputStream.

```
 if (frameTemp == null) {
            Frame temp;
            do {
                temp = buildNewFrame(args);
            } while (temp == null && args.remained());

            if (temp == null) {
                return;
            }

            frameTemp = temp;
            if (!args.remained()) {
                return;
            }
        }

Frame currentFrame = frameTemp;
        do {
            try {
                // read frame body
                // if header frame then write header content to header frame property
                // if entity frame then write to channel
                if (currentFrame.handle(args)) {
                    if (currentFrame instanceof ReceiveHeaderFrame) {
                        ReceiveHeaderFrame headerFrame = (ReceiveHeaderFrame) currentFrame;
                        // build packet with header frame body information
                        ReceivePacket packet = provider.takePacket(headerFrame.getPacketType(),
                                headerFrame.getPacketLength(),
                                headerFrame.getPacketHeaderInfo()
                        );
                        // construct packetModel from packet and add it to packetMap
                        appendNewPacket(headerFrame.getBodyIdentifier(), packet);
                    } else if (currentFrame instanceof ReceiveEntityFrame) {
                        completeEntityFrame((ReceiveEntityFrame) currentFrame);
                    }
                    frameTemp = null;
                    break;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        } while (args.remained());

...


private void appendNewPacket(short identifier, ReceivePacket packet) {
        PacketModel model = new PacketModel(packet);
        packetMap.put(identifier, model);
}

...


/**
 * a data model to manage packet and channel couple for later recovery
 */
static class PacketModel {
    final ReceivePacket packet;
    final WritableByteChannel channel;
    volatile long unreceivedlength;

    PacketModel(ReceivePacket<?, ?> packet) {
        this.packet = packet;
	// packet.open() will create a outputStream via ReceivePacket
        this.channel = Channels.newChannel(packet.open());
        this.unreceivedlength = packet.length();
    }
}
```

## Conclusion
To sum up, with the help of framing. The server will be able to distinguish different packets arriving in parallel. According to different types of received frames, we could define different ways of handling. Treating big volume of file transfer with small pieces of data chunk is an effective way to address the problems we could face in a long connection transfer.



