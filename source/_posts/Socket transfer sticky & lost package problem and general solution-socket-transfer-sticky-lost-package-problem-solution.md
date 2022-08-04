---
title: Socket transfer sticky & lost package problem and general solution
date:  2019-07-07 12:00:00
tags:
- data transfer
---

> Recently, I have been learning some socket programing skills. Thanks to [@_qiujuer_](https://github.com/qiujuer), I finally came to get the hang of the mysterious Sticky & Lost Package Problem and had a way to address it. Here is my little summary about what I have discovered.

# What is Socket Sticky Package and Lost Package

You would have wondered: TCP as a stable protocole should never have sticky package problem as the order and integrity is guaranteed by the protocole itself. 

And that is very true. As a matter of fact, instead of being a transport level problem, Sticky Package and Lost Package are actually logical level problems which could occur if we don't handle it right. 

I will illustrate it by some images.

### Sticky Package

Ideally, datatransfer could be like this so that the receiver could always distinguish each independant package. 

![socket-sticky-packet-1](/img/2019/7/socket-sticky-packet-1-26db2c799016481a9a7b16d112d9de8f.jpg)

However, in the real life, it is very often that the client send two or more packages continuously, which make it possible for server to treat two packages as a single package and hence leads to the famous sticky package problem just like the following image.

 ![socket-sticky-packet-2](/img/2019/7/socket-sticky-packet-2-9e4dfd837f7b47fc98bbeb03cb24304d.jpg)

### Lost Package

Basically, as server receives data, it will read from the channel and transfer it to user space buffer. If the server receives a single biiiiig package that exceeds the buffer's capacity, only part of the data will be extracted and hence results in problems like the following cases. 

![socket-sticky-packet-3](/img/2019/7/socket-sticky-packet-3-8bfa294dfd5c4406aa60c293360a4ee2.jpg)

![socket-sticky-packet-4](/img/2019/7/socket-sticky-packet-4-c8899d75bb1d48b9b97a9dba09476cca.jpg)

You see that together with sticky packages, you server could ended with a mess and all the packages just get mixed.


# General Solution

The simplest solution is to add a length header at the beginning of each message so that it will be easy for the server to know how many bytes to read in order to get the whole message and only its own message. 

### Design a box to wrap the complete message

Here we have designed a packet box with a supplementary length property and position property.
```
public class StringReceivePacket extends ReceivePacket {
    private byte[] buffer;
    private int position;

    public StringReceivePacket(int len) {
        this.buffer = new byte[len];
        length = len;
    }

    @Override
    public void save(byte[] bytes, int count) {
        System.arraycopy(bytes, 0, buffer, position, count);
        position += count;
    }

    public String string() {
        return new String(buffer);
    }

    @Override
    public void close() throws IOException {
    }
}
```

### Read data in an asynchronous manner

Everytime when the socket channel became readable, the selector that was registered for this channel will be notified and the selectionkey which corresponds to the socket channel will be selected. Hence, we could possibly define a callback runnable and put it together with the selectionkey in a map-like structure. 

```
abstract class HandleInputCallback implements Runnable {
    @Override
    public final void run() {
        canProviderInput();
    }

    protected abstract void canProviderInput();
}
```

In this callback handler, we need to define the strategy about how we receive the bytes from the socket channel. Obviously, here we need an intermediate object that works as a transit medium to transport data from socket channel to StringReceivePacket continuously. 

Here is our transit medium called IoArgs, which is just like a truck with fixed sized container (the buffer) that can be used as a intermediate relay.

```
public class IoArgs {
    private int limit = 256;
    private byte[] byteBuffer = new byte[256];
    private ByteBuffer buffer = ByteBuffer.wrap(byteBuffer);

    ...

    /**
     * writeTo data from buffer into bytes
     *
     * @param bytes
     * @param offset
     * @return
     */
    public int writeTo(byte[] bytes, int offset) {
        // to prevent bytes from being overflowed
        int size = Math.min(bytes.length - offset, buffer.remaining());
        // This method transfers bytes from this buffer into the given destination array.
        buffer.get(bytes, offset, size);
        return size;
    }


    /**
     * read data from SocketChannel
     *
     * @param channel
     * @return
     * @throws IOException
     */
    public int readFrom(SocketChannel channel) throws IOException {
        startWriting(); // init buffer
        int bytesProduced = 0;
        while (buffer.hasRemaining()) {
            int len = channel.read(buffer);
            if (len < 0) {
                throw new EOFException();
            }
            bytesProduced += len;
        }
        finishWriting(); // set buffer ready to be read
        return bytesProduced;
    }

    /**
     * set single writing limit
     *
     * @param limit
     */
    public void limit(int limit) {
        this.limit = limit;
    }

    public void writeLength(int total) {
        buffer.putInt(total);
    }


    public int readLength() {
        return buffer.getInt();
    }

    public int capacity() {
        return buffer.capacity();
    }

    public interface IoArgsEventListener {
        void onStarted(IoArgs args);
        void onCompleted(IoArgs args);
    }

}

```

Firstly, we need to know the size of the arriving message by reading 4 bytes in the first place(int type represents 4 bytes). After knowing the size, we could initiate the StringReceivePacket, the final cutomizable container.

Then we will get all remaining data until it fills all the IoArg's(the transit medium) fixed buffer.

> It is definitely possible that the remaining data is far more larger than the fixed buffer of the transit medium, which makes it impossible to carry all the data from the socket channel. But never fear, as the rest of the data is still in socket channel, it will be carried the next time by the transit medium through the callback runnable. What's more important is that the transit medium should be capable of reading exactly what StringReceivePacket is expecting, not a byte more, not a byte less! 

```
int receiveSize;
if (packetTemp == null) {
    // first to receive the message length
    receiveSize = 4;
} else {
    // receive the rest of the message
    // to prevent Ioargs buffer from being overflowed
    receiveSize = Math.min(total - position, args.capacity());
}
// set the size of data to receive to prevent sticky message
args.limit(receiveSize);

...

// readFrom operation
if (args.readFrom(socketChannel) > 0) {
    // complete readFrom callback
    listener.onCompleted(args);
} else {
    throw new IOException("Cannnot readFrom any data from current socketChannel");
}
...

if (packetTemp == null) {
    // no existent current packet -> a new packet to receive
    // read the message length
    int length = args.readLength();
    // init a receive packet with the exact length
    packetTemp = new StringReceivePacket(length);
    // init a new buffer with the exact length
    buffer = new byte[length];
    total = length;
    position = 0;
}

```

### Fill the box as a Japanese
> Japanese are famous for their precision and that is exactly what we need in socket transport, to be accurate to bytes!

You should have noticed that in the code, we had some indicators like **total, position, count...** And these are exactly what we depends on to achieve our precision objective.

When we first get the length of the expecting message, we will manage a new variable called total. And in the mean time, we create a new variable called postion with a initial value of 0.

Each time the transit medium tries to read from the socket channel, we will set a limit **[Math.min(total - position, args.capacity()]** to make sure that read not more not less. 

Afterwards, transit medium will give us a count variable for us to know how many bytes has been actually read from socket channel.

Then, when transit medium transports the data to StringReceivePacket, we need to accumulate the position variable by the count variable. Finally, by comparing the position and total variable, we could know if the StringReceivePacket has get its expected complete message.

```
// transfer data received from socket channel via transit medium IoArgs to our dispatcher buffer
int count = args.writeTo(buffer, 0);
// sometimes you will have zero as buffer args has already been read previously <= args.readLength();
if (count > 0) {
    packetTemp.save(buffer, count);
    // register current packet position to judge whether message has been completely received
    position += count;

    if (position == total) {
        //transfer complete packet to outside invoker
        completePacket();
        //removed current packet as it has been fully received by outside invoker
        packetTemp = null;
    }
}
```

# Conclusion
To sum up, we have a StringReceivePacket as a final container and a IoArgs as a transit medium which continuously relay data from socket channel to our final container in an asynchronous manner. With some extra indicators (total, position, count ...), all participants knows exactly answers to the below questions.

- what's the size of expected message
- how many bytes have I read
- how many bytes yet to read
- Am I finished reading

By this way, we are able to solve the cumbersome sticky & lost package problem.

 





