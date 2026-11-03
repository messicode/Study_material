# Endianness: A Complete 30-Minute Crash Course

Computer architecture · C/C++ · Memory layout · Assembly · Binary protocols

## Learning roadmap

This lesson progresses from the basic mental model of memory to hardware behavior, C/C++ implementation details, binary formats, performance, and advanced edge cases.

| Time      | Topic                                              |
| --------- | -------------------------------------------------- |
| 0–4 min   | Fundamentals: bytes, addresses, integers           |
| 4–10 min  | Big-endian vs. little-endian, with memory diagrams |
| 10–16 min | C/C++, pointers, assembly, and hardware            |
| 16–22 min | Advanced concepts, optimizations, and edge cases   |
| 22–26 min | Practical applications and debugging               |
| 26–30 min | Knowledge check and implementation exercises       |

## 1. Fundamentals: Build the right mental model

### 1.1 What is endianness?

Endianness is the order in which the bytes of a multi-byte value are arranged in memory.

It matters when a value occupies more than one byte, such as a 16-bit integer, 32-bit integer, or 64-bit integer.

There are two primary conventions:

- Big-endian (BE): The most significant byte is stored at the lowest memory address.

- Little-endian (LE): The least significant byte is stored at the lowest memory address.

The key distinction is the order of bytes, not the order of individual bits.

### 1.2 Prerequisites: Bits, bytes, and addresses

A bit is a binary digit, either `0` or `1`.

A byte is a group of 8 bits on the systems discussed in this course. In standard C and C++, a byte is formally the size of a `char`, and it is at least 8 bits; modern general-purpose platforms almost universally use 8-bit bytes.

A memory address identifies a location in memory. On ordinary byte-addressable computers, consecutive addresses identify consecutive bytes.

For example:

text

```
Address       Stored byte
0x1000        0x12
0x1001        0x34
0x1002        0x56
0x1003        0x78
```

Each byte has its own address. A four-byte integer occupies four consecutive byte locations.

### 1.3 How an integer is divided into bytes

Consider the 32-bit hexadecimal value:

text

```
0x12345678
```

Each pair of hexadecimal digits represents one byte:

text

```
0x12  0x34  0x56  0x78
  ↑     ↑     ↑     ↑
Byte 3 Byte 2 Byte 1 Byte 0
```

The most significant byte (MSB) is `0x12`: it contributes the greatest place value.

The least significant byte (LSB) is `0x78`: it contributes the smallest place value.

The numerical value is:

This evaluates to the same integer regardless of how its bytes are stored in memory.

Crucial mental model: The number is an abstract value. Endianness describes one way of laying out its bytes in memory.

## 2. Core mechanics: Big-endian vs. little-endian

### 2.1 The same number, two memory layouts

Suppose a four-byte integer has the value `0x12345678` and starts at address `0x1000`.

## Big-endian (BE)

Most significant byte first in memory

### 0x12

0x1000

### 0x34

0x1001

### 0x56

0x1002

### 0x78

0x1003

## Little-endian (LE)

Least significant byte first in memory

### 0x78

0x1000

### 0x56

0x1001

### 0x34

0x1002

### 0x12

0x1003

Both layouts represent the same integer:

text

```
0x12345678
```

Only the mapping from byte significance to memory address changes.

### 2.2 Why the names make sense

Imagine writing the hexadecimal value from left to right:

text

```
12 34 56 78
```

- Big-endian puts the big end (the most significant byte, `12`) first at the lowest address.

- Little-endian puts the little end (the least significant byte, `78`) first at the lowest address.

The names describe byte significance order in memory, not how a number is written on paper.

### 2.3 A systematic way to calculate memory layouts

For an -byte value, let its bytes in significance order be:

Here, is the least significant byte, and is the most significant byte.

At base address :

- Little-endian: byte goes to address .

- Big-endian: byte goes to address .

For example, a 16-bit value `0xABCD`:

| Address | Big-endian | Little-endian |
| ------- | ---------- | ------------- |
| `A`     | `AB`       | `CD`          |
| `A+1`   | `CD`       | `AB`          |

This rule generalizes to 16-, 32-, 64-, and larger-width integers.

### 2.4 Endianness does not change individual bits

The byte `0x12` is binary `00010010` in both conventions. Its bits are not reversed simply because the machine is little-endian.

For example:

text

```
Integer value: 0x1234

Big-endian bytes:    12 34
Little-endian bytes: 34 12
```

Neither representation turns `0x12` into `0x48`, nor does it reverse the bits within each byte.

Common misconception: Little-endian does not mean that all bits or all characters are stored backward. It means that the least significant byte occupies the lowest address.

### 2.5 How this relates to numeric significance

For an unsigned 32-bit integer, the mathematical value is:

where is the byte with significance .

In little-endian memory, the bytes already appear in increasing significance order as addresses increase. In big-endian memory, they appear in decreasing significance order.

The processor's load operation interprets the bytes according to its applicable memory-access rules and produces the intended register value.

### 2.6 Endianness applies to the interpretation of byte sequences

Consider the following four bytes:

text

```
01 00 00 00
```

If interpreted as a 32-bit unsigned integer:

- Little-endian: `0x00000001` = 1.

- Big-endian: `0x01000000` = 16,777,216.

The bytes themselves are not inherently the number 1 or 16,777,216. Their meaning depends on the format and interpretation.

This is the essential principle behind binary file parsing, network communication, and debugging raw memory.

## 3. Core mechanics in C and C++

### 3.1 Inspecting endianness with C

The following program checks the byte layout of a 32-bit value on the current machine.

c

```
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint32_t x = UINT32_C(0x12345678);
    const unsigned char *p = (const unsigned char *)&x;

    printf("Value: 0x%08X\n", (unsigned int)x);
    printf("Bytes in increasing address order:\n");

    for (size_t i = 0; i < sizeof x; ++i) {
        printf("%02X ", (unsigned int)p[i]);
    }

    printf("\n");
    return 0;
}
```

Typical output on a little-endian system:

text

```
Value: 0x12345678
Bytes in increasing address order:
78 56 34 12
```

Typical output on a big-endian system:

text

```
Value: 0x12345678
Bytes in increasing address order:
12 34 56 78
```

Why does this work?

- `x` contains a 32-bit integer value.

- `&x` obtains its address.

- Converting the address to `const unsigned char *` allows the program to inspect the object's representation one byte at a time.

- `p[i]` reads the byte at offset `i`.

In C and C++, inspecting an object's representation through a character-type pointer is permitted. `unsigned char` is particularly useful because its values represent byte patterns without signed-character interpretation issues.

The exact output depends on the platform and its execution environment.

### 3.2 A portable C++ version

C++ provides `std::endian` in C++20 through `<bit>`.

cpp

```
#include <bit>
#include <cstdint>
#include <iostream>

int main() {
    if constexpr (std::endian::native == std::endian::little) {
        std::cout << "Little-endian\n";
    } else if constexpr (std::endian::native == std::endian::big) {
        std::cout << "Big-endian\n";
    } else {
        std::cout << "Mixed or other native endianness\n";
    }
}
```

Important qualifications:

- `std::endian` describes the native endianness characteristics of the implementation.

- `std::endian::native` can equal neither `little` nor `big` on unusual implementations.

- Some systems can have different endianness characteristics for different types or data paths.

- The C++20 feature reports an implementation property; it does not automatically convert values for a file format or network protocol.

Use a byte-inspection program when you want to see the actual representation of a particular object.

### 3.3 Endianness is not the same as integer signedness

Consider:

c

```
uint32_t u = 0xFFFFFFFFu;
int32_t  s = -1;
```

On a conventional system using 32-bit two's-complement integers, both commonly have the byte representation:

text

```
Little-endian: FF FF FF FF
Big-endian:    FF FF FF FF
```

Because all four bytes are identical, their order is invisible.

Yet their types and values differ:

- `u` is the unsigned value 4,294,967,295.

- `s` is the signed value -1.

Endianness describes byte order. Signedness describes how a bit pattern is interpreted numerically.

For an ordinary 32-bit two's-complement signed integer, the top bit is the sign bit. However, C and C++ type semantics and historical language versions have their own rules; do not assume that all arbitrary signed-integer representations are interchangeable.

### 3.4 Reconstructing an integer manually

A portable way to decode four bytes in little-endian order is to combine them explicitly.

c

```
#include <stdint.h>

uint32_t read_u32_le(const uint8_t b[4]) {
    return ((uint32_t)b[0])
         | ((uint32_t)b[1] << 8)
         | ((uint32_t)b[2] << 16)
         | ((uint32_t)b[3] << 24);
}
```

For big-endian:

c

```
uint32_t read_u32_be(const uint8_t b[4]) {
    return ((uint32_t)b[0] << 24)
         | ((uint32_t)b[1] << 16)
         | ((uint32_t)b[2] << 8)
         | ((uint32_t)b[3]);
}
```

Example:

text

```
Bytes: 12 34 56 78

read_u32_be -> 0x12345678
read_u32_le -> 0x78563412
```

Why use explicit casts?

In C, operands smaller than `int` undergo integer promotions. Casting to `uint32_t` before shifting makes the intended unsigned arithmetic clear and avoids problematic signed shifts.

These routines assume the caller supplies four valid bytes. Production parsers must also check buffer lengths before reading them.

### 3.5 Why naïve pointer casts are dangerous

You may see code like this:

c

```
uint8_t bytes[4] = {0x12, 0x34, 0x56, 0x78};

uint32_t x = *(uint32_t *)bytes;  /* Avoid this pattern */
```

It appears to load four bytes into an integer, but it has multiple problems:

1. Endianness: The numerical result depends on the machine's byte order.

2. Alignment: `bytes` may not satisfy the alignment required for `uint32_t`.

3. C/C++ object rules: Accessing a declared array of byte elements through a `uint32_t` lvalue can violate the language's aliasing and object-access rules.

4. Bounds: The operation assumes four accessible bytes.

5. Format mismatch: The byte sequence may have a specified order that differs from the machine's native order.

For a native representation on a known platform, `memcpy` can be appropriate:

c

```
#include <stdint.h>
#include <string.h>

uint32_t x;
memcpy(&x, bytes, sizeof x);
```

This avoids alignment and aliasing problems, but it does not perform endianness conversion. The result still uses the machine's native representation.

For external data, explicit byte decoding or a suitable byte-order conversion routine is generally clearer.

### 3.6 Byte swapping

A byte swap reverses the order of bytes in a fixed-width value.

For example:

text

```
Before: 0x12345678
After:  0x78563412
```

In C++20:

cpp

```
#include <bit>
#include <cstdint>

uint32_t x = 0x12345678;
uint32_t y = std::byteswap(x);  // C++23, not C++20
```

Note that `std::byteswap` was introduced in C++23, not C++20. It reverses the bytes of an integral value; it does not automatically know the source and destination protocol formats.

In C, compiler-specific intrinsics such as `__builtin_bswap32` are common, but they are not ISO C interfaces.

A portable manual implementation is:

c

```
#include <stdint.h>

uint32_t bswap32(uint32_t x) {
    return ((x & UINT32_C(0x000000FF)) << 24)
         | ((x & UINT32_C(0x0000FF00)) << 8)
         | ((x & UINT32_C(0x00FF0000)) >> 8)
         | ((x & UINT32_C(0xFF000000)) >> 24);
}
```

The masks isolate bytes, and the shifts move them to their new positions.

Important: Byte swapping is not the same as bit reversal. For example, swapping `0x12` and `0x34` in `0x1234` produces `0x3412`; it does not reverse the bits inside `0x12` or `0x34`.

## 4. Hardware and assembly: What actually happens?

### 4.1 Registers and memory are different things

A CPU register holds a value used by instructions. Memory holds bytes at addresses.

For example, suppose a little-endian CPU executes a load of a 32-bit integer from address `0x1000`.

Memory:

text

```
Address     Byte
0x1000      78
0x1001      56
0x1002      34
0x1003      12
```

The load can produce this register value:

text

```
Register: 0x12345678
```

The CPU's load machinery interprets the bytes according to the architecture's rules. Software normally does not need to reverse bytes manually after every load.

### 4.2 Conceptual assembly example

On x86-64, a load such as:

asm

```
mov eax, DWORD PTR [rdi]
```

loads four bytes from the address in `RDI` into `EAX`.

If memory contains:

text

```
Address RDI+0: 78
Address RDI+1: 56
Address RDI+2: 34
Address RDI+3: 12
```

then the resulting register value is:

text

```
EAX = 0x12345678
```

x86-64 uses little-endian byte order for ordinary integer memory representations.

The instruction does not need to execute four separate software instructions to reverse the bytes. The architecture defines the semantics of the load.

A store performs the corresponding operation in the opposite direction: storing `0x12345678` as a 32-bit value in little-endian memory writes bytes `78 56 34 12` at increasing addresses.

### 4.3 Does the CPU always read memory one byte at a time?

No.

A source-level operation that reads a 32-bit integer does not imply four independent machine-level byte loads. A processor may use a single load instruction, multiple internal transfers, cache-line data, or other implementation-specific mechanisms.

The architecture specifies the observable behavior; the microarchitecture determines how the processor implements it.

The same distinction applies to stores. A machine instruction may describe one architectural store even when internal execution involves buffers, cache operations, or split transfers.

### 4.4 Why endianness is usually transparent to arithmetic

Suppose a program executes:

c

```
uint32_t a = 100;
uint32_t b = 200;
uint32_t c = a + b;
```

The result is 300 on a conforming implementation where these values are representable.

The CPU performs arithmetic on values in registers or through its execution units. It does not normally need to think about the memory byte order while adding two already-loaded integers.

Endianness matters at the boundary where multi-byte values are represented in memory or transferred between components that interpret those bytes differently.

### 4.5 Alignment and unaligned loads

An address is aligned for a type if it meets the type's alignment requirements. For example, a platform might naturally align 32-bit integers to addresses divisible by four.

An unaligned load reads a value from an address that does not meet the architecture's preferred or required alignment.

Possible consequences vary by architecture:

- Some CPUs support unaligned loads efficiently.

- Some support them but incur performance penalties.

- Some instructions or memory regions require alignment and may fault.

- A load crossing a cache-line or page boundary may need extra work.

- The compiler must still obey the language's alignment and object-access rules.

Endianness and alignment are distinct properties. A little-endian machine can require aligned accesses for particular instructions, and a big-endian machine can support unaligned accesses.

### 4.6 Does endianness change cache behavior?

Not inherently.

Caches organize data in lines or blocks, and processors fetch and store data according to the architecture and microarchitecture. Byte order determines how a multi-byte value maps to those bytes, but does not by itself imply that one endianness is faster.

Performance depends on the workload, memory access pattern, instruction set, data representation, compiler, and processor implementation.

## 5. Advanced concepts

### 5.1 Host byte order versus wire byte order

Host byte order is the native order used by a machine's ordinary multi-byte representations.

Wire byte order is the order specified by a communication protocol. Despite the name, it also applies to serialized files and other external byte streams.

Network protocols frequently specify big-endian order, historically called network byte order in common Internet programming interfaces.

Example: a protocol specifies a 16-bit port number as `0x1234`.

text

```
Protocol bytes: 12 34
```

A little-endian host normally stores the equivalent native integer in memory as:

text

```
34 12
```

The protocol bytes must not be confused with the host's native object representation.

In POSIX-style networking code, conversion functions such as `htons` (host to network short) and `ntohs` (network to host short) are commonly used for 16-bit values. The `htonl` and `ntohl` functions cover 32-bit values.

These functions are provided by platform networking libraries, not by ISO C itself. On systems where host and network byte order already agree, a conversion may compile to a no-op.

### 5.2 Serialization and deserialization

Serialization converts a value into a defined sequence of bytes.

Deserialization reconstructs a value from that sequence.

A robust binary format specifies:

- Field widths

- Byte order for multi-byte fields

- Signed or unsigned interpretation

- Encoding for text

- Whether padding or reserved bytes exist

- Versioning and validation rules

For example, a simple four-byte field defined as an unsigned big-endian integer:

text

```
Input bytes: 00 00 01 2C
Decoded value: 300
```

A little-endian decoder would instead interpret the same bytes as `0x2C010000`, which is 738,263,040.

This is a format-interpretation bug, not an arithmetic bug.

### 5.3 Endianness and floating-point values

Floating-point numbers also have object representations. IEEE 754 defines widely used binary floating-point formats, such as binary32 and binary64, but endianness and byte layout are separate concerns.

On a common system using IEEE 754 binary32, the value `1.0f` has the bit pattern:

text

```
0x3F800000
```

Its memory bytes on common systems are:

text

```
Little-endian: 00 00 80 3F
Big-endian:    3F 80 00 00
```

However, do not assume that every platform's floating-point representation, byte ordering, or mixed-width layout is identical. A portable file format must specify the floating-point encoding and byte serialization explicitly.

Reversing the bytes of a floating-point object is not the same as converting a number mathematically. It rearranges the representation and can produce a completely different value, infinity, NaN, or another bit pattern.

### 5.4 Mixed-endian and unusual systems

The simple BE/LE distinction covers most modern general-purpose cases, but historical and specialized architectures have used more complicated arrangements.

For example, an architecture might have a defined byte order for integers but a different word or byte arrangement for certain floating-point formats.

Terminology such as mixed-endian can describe these situations, though the exact behavior depends on the architecture and data type.

The safe rule is to follow the target architecture's documentation and the external format specification rather than assuming a universal layout for every type.

### 5.5 Bit fields, packed structures, and padding

C and C++ structures may contain padding bytes between members or at the end of the structure. Bit-field allocation order and layout are also implementation-dependent in important respects.

Example:

c

```
struct Example {
    uint8_t  a;
    uint32_t b;
};
```

A common ABI may place `b` at an aligned offset after `a`, leaving padding between the members. The exact structure layout depends on the implementation's ABI and alignment rules.

Therefore, sending raw structure bytes over a network or saving them as a portable file format is generally unsafe unless the format and ABI explicitly define the layout.

Even if every member has a known byte order, padding, member offsets, and bit-field layout can still differ.

### 5.6 Endianness versus bit numbering

A protocol may define a field as occupying particular bits within a byte. This is a separate issue from the byte order of a multi-byte integer.

Engineers often use terms such as most-significant-bit first or least-significant-bit first when describing bit streams. Those rules govern how fields are packed or transmitted at the bit level.

Do not infer bit-field placement from the endianness of the host machine. Follow the bit numbering and packing rules of the protocol or file format.

### 5.7 Endianness and atomicity

Atomicity concerns whether a memory operation is observed as an indivisible operation under the applicable memory model.

Endianness concerns byte ordering.

They are different properties. A multi-byte value does not automatically become atomic because its byte order is known, and byte swapping does not make a data race safe.

Correct concurrent access requires suitable atomic operations, synchronization, and memory-ordering rules. Low-level code must follow the language memory model and the processor's architectural guarantees.

### 5.8 Vector instructions and SIMD

SIMD (single instruction, multiple data) instructions operate on several values in parallel within vector registers.

A vector load usually transfers a sequence of bytes into a vector register. Instructions that interpret those bytes as lanes may have architecture-specific lane ordering and semantics.

Endianness becomes relevant when code maps memory bytes to vector lanes or serializes vector contents.

Do not assume that reversing a scalar integer's bytes is equivalent to reversing all lanes of a vector, or that a vector shuffle performs the same operation as a scalar byte swap.

The required transformation depends on the data format and the vector instruction set.

### 5.9 Optimizations and compiler intrinsics

A compiler may optimize an explicit byte-swap operation into a dedicated instruction where the target supports one, or into a short sequence of shifts and masks.

Similarly, a native-to-little-endian conversion may disappear entirely on a little-endian target if the compiler can establish that no conversion is needed.

For performance-sensitive code:

- Prefer clear, correct conversions first.

- Use standard library functions or documented intrinsics where available.

- Inspect generated assembly when the cost matters.

- Benchmark on the actual target.

- Avoid assuming that a hand-written sequence is faster than a compiler intrinsic.

A byte swap may be cheap, but bounds checking, unaligned access handling, cache misses, parsing, and copying can dominate the overall cost.

## 6. Practical understanding: Real-world scenarios

### 6.1 Network packet debugging

Suppose a packet contains these bytes for a 16-bit field:

text

```
01 BB
```

If the protocol specifies big-endian order:

The field represents decimal 443.

If a little-endian decoder mistakenly interprets the bytes in native little-endian order, it calculates:

The packet bytes are correct; the decoder is wrong.

Debugging procedure:

1. Capture the raw bytes.

2. Identify the protocol's field width.

3. Read the protocol's byte-order specification.

4. Decode the bytes explicitly.

5. Compare the result with the expected value.

6. Check whether a conversion function has already been applied.

Avoid blindly swapping every field. A protocol can contain fields of different widths, individual bytes, text, and explicitly defined encodings.

### 6.2 Binary file formats

Suppose a file begins with this header:

text

```
4D 53 47 31 00 00 00 2A
```

Assume the format defines:

- Bytes 0–3: four-byte ASCII magic value `MSG1`

- Bytes 4–7: unsigned 32-bit big-endian record count

The magic value identifies the format. The count is:

text

```
00 00 00 2A = 42
```

A correct parser should decode the count using the file specification, not the host's native byte order.

For untrusted input, it should also check that the header is complete before accessing its fields.

### 6.3 Memory dumps and debuggers

A debugger may show:

text

```
Address       Bytes
0x2000        78 56 34 12
```

You expect the integer `0x12345678`, but the displayed bytes appear reversed.

On a little-endian system, this is normal. The debugger is displaying bytes in increasing address order, while the conventional hexadecimal notation for the integer displays its most significant digits first.

When debugging, distinguish between:

- The numerical value shown by an integer expression.

- The raw bytes shown in a memory window.

- The architecture's native byte order.

- The byte order required by an external format.

### 6.4 Cross-platform data exchange

Imagine writing a binary file on a little-endian computer and reading it on a big-endian computer.

If the file stores the raw native representation of a 32-bit integer, the reader can obtain the wrong value.

A better design specifies a fixed external representation:

text

```
Field: uint32
Encoding: big-endian
```

Both systems can then serialize and deserialize the same value, regardless of their native byte order.

This is one reason portable file formats and protocols define their binary layouts explicitly.

### 6.5 Embedded systems and memory-mapped devices

Embedded software often communicates with hardware through memory-mapped registers or peripheral interfaces.

For example, a device might define a 32-bit register with a specified layout. The processor's native load/store semantics, the peripheral bus, the register definition, and any required access width all matter.

Do not assume that changing the pointer type or swapping bytes is sufficient. Consult the processor and device documentation for:

- Register width and alignment

- Required access sizes

- Endianness of the register interface

- Reserved bits

- Read/write side effects

- Ordering requirements

- Required barriers or synchronization

A register read may have side effects, so replacing a single access with multiple byte reads can be incorrect even if the reconstructed integer looks right.

### 6.6 Database storage and distributed systems

A database or distributed application may move binary data between processes running on different architectures.

Endianness becomes relevant when data is represented as raw bytes rather than as a logical value encoded by a defined serialization format.

Many higher-level formats avoid exposing native object representations. Others explicitly define field encodings, which makes them more portable.

The important question is not simply, "Which CPU is this?" It is, "What byte representation does this interface promise?"

## 7. Common pitfalls and misconceptions

| Mistake                                                      | Why it is wrong                                                                 | Correct approach                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Thinking little-endian reverses every bit                    | Endianness orders bytes, not bits within each byte                              | Draw the bytes at increasing addresses                     |
| Assuming every computer is little-endian                     | Many modern systems are, but not all                                            | Check the architecture or use defined conversions          |
| Assuming a cast converts endianness                          | A pointer cast does not rearrange bytes                                         | Decode explicitly or use an appropriate conversion routine |
| Swapping every integer unconditionally                       | Some interfaces already use host order or a particular external order           | Convert only at representation boundaries                  |
| Writing raw C/C++ structures to files                        | Padding, alignment, field layout, and byte order can vary                       | Define and serialize the format explicitly                 |
| Assuming floating-point layout always matches integer layout | Floating-point encodings and platform layouts require separate consideration    | Follow the format and ABI specification                    |
| Assuming endianness determines performance                   | Memory access costs depend on the architecture and workload                     | Benchmark the actual implementation                        |
| Assuming a byte swap makes an access safe                    | It does not solve alignment, bounds, or concurrency problems                    | Validate the access independently                          |
| Confusing ASCII order with integer order                     | Text bytes have character encodings; they are not automatically a numeric field | Parse text according to its encoding and grammar           |

## 8. A compact implementation toolkit

The following examples cover the most useful operations for everyday systems programming.

### 8.1 Read a big-endian 16-bit unsigned integer

c

```
#include <stdint.h>

uint16_t read_u16_be(const uint8_t b[2]) {
    return ((uint16_t)b[0] << 8)
         |  (uint16_t)b[1];
}
```

Example:

text

```
Bytes: 12 34
Value: 0x1234 = 4660
```

### 8.2 Read a little-endian 16-bit unsigned integer

c

```
#include <stdint.h>

uint16_t read_u16_le(const uint8_t b[2]) {
    return ((uint16_t)b[1] << 8)
         |  (uint16_t)b[0];
}
```

Example:

text

```
Bytes: 12 34
Value: 0x3412 = 13330
```

### 8.3 Write a big-endian 32-bit unsigned integer

c

```
#include <stdint.h>

void write_u32_be(uint8_t b[4], uint32_t x) {
    b[0] = (uint8_t)(x >> 24);
    b[1] = (uint8_t)(x >> 16);
    b[2] = (uint8_t)(x >> 8);
    b[3] = (uint8_t)x;
}
```

For `x = 0x12345678`, the output is:

text

```
12 34 56 78
```

### 8.4 Write a little-endian 32-bit unsigned integer

c

```
#include <stdint.h>

void write_u32_le(uint8_t b[4], uint32_t x) {
    b[0] = (uint8_t)x;
    b[1] = (uint8_t)(x >> 8);
    b[2] = (uint8_t)(x >> 16);
    b[3] = (uint8_t)(x >> 24);
}
```

For `x = 0x12345678`, the output is:

text

```
78 56 34 12
```

These functions illustrate the core technique: extract or combine bytes using shifts and masks instead of relying on the host's native memory layout.

In production, validate buffer lengths before calling these functions, and ensure that the chosen integer types exist with the required widths. `uint32_t`, for example, is provided only on implementations that support an exact 32-bit unsigned integer type.

## 9. Knowledge check: Challenging questions with answers

Try to answer each question before revealing the explanation.

## Test your understanding

0 / 10 answers revealed

1. A 32-bit integer is 0xA1B2C3D4. What bytes appear at increasing addresses on a little-endian machine?

Reveal answer

2. Memory contains 11 22 33 44 at increasing addresses. What 32-bit value does a native little-endian load produce?

Reveal answer

3. Does reversing a 32-bit integer's bytes reverse the bits within each byte?

Reveal answer

4. A network protocol specifies a 32-bit big-endian value. Should a little-endian CPU always byte-swap the integer after parsing?

Reveal answer

5. Why can reading a uint32_t by casting a uint8_t array pointer be unsafe?

Reveal answer

6. Does big-endian arithmetic inherently run slower than little-endian arithmetic?

Reveal answer

7. A binary32 value has the common IEEE 754 bit pattern 0x3F800000. What value does it represent?

Reveal answer

8. Two CPUs have different native endianness. How should they exchange a 32-bit counter reliably?

Reveal answer

9. Does using memcpy to copy four external bytes into a uint32_t perform endian conversion?

Reveal answer

10. Can endianness alone tell you the memory layout of a C++ structure containing a uint8_t, a uint32_t, and bit fields?

Reveal answer

Hide all answers Reveal all answers

## 10. Final summary

Endianness determines how a multi-byte value's bytes are ordered in memory. Big-endian stores the most significant byte at the lowest address; little-endian stores the least significant byte there.

The underlying numerical value does not change merely because the byte layout changes. Problems arise when software interprets bytes using the wrong representation rules.

The most important practical principles are:

1. Distinguish values from representations. The integer `0x12345678` and the byte sequence `78 56 34 12` are not interchangeable without a defined interpretation.

2. Follow external format specifications. Protocols and files must define their field widths and byte order.

3. Convert at boundaries. Decode external bytes into native values, and serialize native values into the specified external representation.

4. Respect C/C++ rules. Endianness is only one concern; alignment, aliasing, padding, object representation, and bounds also matter.

5. Understand hardware behavior. Load and store instructions follow architecture-defined semantics; the CPU normally handles native byte interpretation automatically.

6. Measure performance rather than guessing. Endianness alone does not determine speed.

## 11. Final checklist: What you should be able to explain or implement

Use this as your self-assessment after the 30-minute lesson.

- Define endianness and distinguish big-endian from little-endian.

- Identify the most and least significant bytes of a hexadecimal integer.

- Draw the memory layout of a 16-, 32-, or 64-bit value in either byte order.

- Explain why the same bytes can decode to different integer values.

- Inspect native byte order in C or C++.

- Explain how CPU load and store instructions interact with memory representation.

- Distinguish byte swapping from bit reversal.

- Implement functions to read and write 16- and 32-bit values in a specified byte order.

- Explain why a pointer cast does not perform endian conversion.

- Identify alignment, aliasing, bounds, and structure-padding hazards.

- Explain host byte order versus network or file-format byte order.

- Recognize when floating-point representations require additional care.

- Debug an incorrect numeric value from a packet capture or memory dump.

- Explain why endianness is not the same as signedness, atomicity, or performance.

- Design a portable binary format with explicit field encodings.

The single most important takeaway: Never assume that a sequence of bytes has a particular numerical meaning until you know how that sequence is supposed to be interpreted.
