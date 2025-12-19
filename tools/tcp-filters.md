## TCP Filter Breakdown

The Syntax for this capture filter is defined by libpcap/WinPcap library.

In the TCPDump example, we have the following targeted examples to retrieve only GET and POST packets
```d
// GET
$ sudo tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420'
//POST
$ sudo tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354'
```

First, some notation:

- `var[n]` means the nth byte of var.
- `var[n,c]` means c bytes of var starting at offset n, e.g. `var[3:4]` would return bytes 3,4,5,6 of var.
- `&` is the bitwise AND operation.
- `>>` means bitwise shift right.

So, what do we have?
```d
tcp[((tcp[12:1] & 0xf0) >> 2):4]
```

Let's deconstruct this into its individual parts:

**`tcp[12:1]`** 
takes 1 byte of the TCP segment (i.e. the packet including header) at offset 12. We can see from the structure  that offset 12 (0xC) is the Data Offset field. Its definition is as follows:

> Data offset (4 bits) specifies the size of the TCP header in 32-bit words. The minimum size header is 5 words and the maximum is 15 words thus giving the minimum size of 20 bytes and maximum of 60 bytes, allowing for up to 40 bytes of options in the header. This field gets its name from the fact that it is also the offset from the start of the TCP segment to the actual data.

<img width="712" height="457" alt="Screen Shot 2025-04-08 at 1 17 22 PM" src="https://github.com/user-attachments/assets/660dbdbb-8fd1-4ae8-9f2b-c2f37b9612ff" />



Ok, cool. This field is only 4 bits, though, and `tcp[12:1]` takes a whole byte (8 bits). We only want the first four.

**`tcp[12:1] & 0xf0`**
takes that Data Offset byte and applies a bitwise AND operation, using the constant 0xF0. This is often known as a _masking_ operation, since any bits in the constant set to 0 will also be set to zero in the output. It's easier to think about this in binary. 0xF0 is just 11110000, and since `x & 0` for any bit x is always 0, and `x & 1` for any bit x is always x, it follows that the zero bits in 0xF0 will "switch off" or _mask_ the given bits, but leave the rest alone. In this case, if we imagine that `tcp[12:1]` is 10110101, the result would be 10110000 because the last four bits are masked to zero. The idea here is that, since the Data Offset field is only 4 bits long, but we have 8 bits, we mask out the irrelevant bits so we only have the first 4.

The problem here is that, numerically speaking, our 4 bits are in the "top" side of the byte. This means that instead of just having 1101 (our Data Offset bits), we have 11010000. If we just wanted to get the "raw" value of the Data Offset field, we'd right shift by four places. `10110000 >> 4 = 1101`, i.e. we throw away the "bottom" 4 bits and shift the top four bits right. However, in this case you'll notice the filter only shifts across by 2 places, not 4.

This is where we want to refer back to the definition of the Data Offset field: it specifies the size of the header in _32-bit words_, not bytes. So, if we want to know the length of the header in bytes, we need to multiply it by 4. As it turns out, a bitwise left-shift of 1 is the same as multiplying a number by 2, and a bitwise left-shift of 2 is the same as multiplying a number by 4.

Now, combine this with what we've already seen: **`>> 4`** would make sense in the filter if you wanted to get that raw value of Data Offset, but then we want to multiply it by 4, which is equivilent to left shifting (`<< 2`), which cancels out part of that right shift, resulting in `>> 2`.

So, `(tcp[12:1] & 0xf0) >> 2` extracts the Data Offset field and multiplies it by 4 to get us the size of the TCP header in bytes.

But wait, there's more! In the filter you provided, we still need to do one more operation:
```d
tcp[((tcp[12:1] & 0xf0) >> 2):4]
```

This is easier looked at if we used an intermediate variable:
```d
$offset = ((tcp[12:1] & 0xf0) >> 2)
tcp[$offset:4]
```

What this does is get the first 4 bytes _after_ the TCP header, i.e. the first 4 data bytes of the payload. In the filter you pulled this from, they were looking for HTTP GET requests using this filter:
```d
port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420
```

The `0x47455420` constant is actually a numeric encoding of the ASCII bytes for `GET` (that last character is a space), where the ASCII values of those characters are 0x47, 0x45, 0x54, 0x20.

So, how does this work in full? It extracts the 4-bit Data Offset field from the TCP header, multiplies it by 4 to compute the size of the header in bytes (which is also the offset to the data), then extracts 4 bytes at this offset to get the first 4 bytes of the data, which it then compares to "GET " to check it's a HTTP GET.
