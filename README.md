# LoRa-trans - L2 transport improvements for LoRa

This project adds a Level-2 layer over LoRa transport, as well as an
easy-to-use API. The idea is to make LoRa easier to use in IoT projects.

Check the Arduino sketch which is a complete example of API usage.

Our target hardware is TTGO-ESP32, so the low-level code and testing is
tested only on this platform.

LoRa-trans adds FEC (Forward Error Correction) and optional encryption
to LoRa packets.

## Error correction

LoRa has layer-1 error correction (named CR) as well as layer-1 error
detection using CRC. Why add FEC in software?

In our experiments, we found that LoRa packets arrive with errors even under
the best of circunstances. CR is not enough to prevent this, even if configured
to maximum redundancy. Due to this, layer-1 CRC discards many almost-perfect
packets.

CR and CRC use algorithms that are fast in hardware, but software error correction
can be much more powerful.

Every packet is trailed by a FEC (Forward Error Code). The code can be
Reed-Solomon RS(50,10), RS(100,14) or RS(200,20) depending on packet
size. This means the maximum payload is 200 octets.

Since Reed-Solomon codes demand a fixed-size message, it is calculated as if
the network packet was padded with nulls (binary zeros) up to 50, 100 or 200
octets.

The idea of using several codes is to keep the redundancy and error correction
power roughly similar to all packet sizes, with a discreet advantage given to
shorter packets.

The reference FEC RS implementation is https://github.com/simonyipeter/Arduino-FEC .

Packets are transmitted using LoRa explicit mode, so the packet size is known
when it reaches the network layer to be parsed. CRC is disabled.

## Encryption

Optional encryption is based on AES256, and happens before the calculation
of FEC code.

If encryption is on, maximum payload is 166 octets, due to additional preambles
(IV, length) and encryption block round-up.

## Testing

Unit testing is carried out on PC, given the superior tools (code coverage,
Valgrind) available. Embedded APIs are mocked up.

This project uses slightly modified versions of RS-FEC, AES256, and SHA-256 
libraries. Their copyrights belong to their respective authors (mentioned 
directly or indirectly in the header of each file).

# LoRa parameters

In LoRaL2 constructor, some LoRa parameters (frequency, spread and bandwidth)
are pased as parameters. The suggested parameters in Lora-trans.ino deliver
5kbps of speed and have a good balance between reliability, range and speed,
especially for testing.

If you need really good range, try to reduce bandwidth to 62500 (if your LoRa
chip supports it) and increase spread to SF9. We have had very good range in
urban areas with this setup, and reach several km of range in almost-clear sight.
This is 9db better, at the cost of low speed (0.8kbps). Another option is 125kHz
bandwidth and SF10, but the narrower bandwidth did better in our tests (this may
vary, depending on each specific hardware).

Be sure to generate very few packets withvery small payloads if you go very low speed.
An encrypted packet has a minimum of 44 octets, which takes half a second
in 0.8kbps speed.

The project disables CRC mode, enables explicit mode and uses the lowest
L1 redundancy possible (5/4). This is hardcoded within Radio.cpp, and should
be kept as they are. In particular the explicit mode should not be turned off,
since the code depends on it to function.

# License

The code authored by us is distributed under MIT license.
