# DHT11 Protocol Timing Analysis

## Time Analysis Table

| Timing Parameter | Value in Datasheet | Used in Code | Meaning |
|---|---|---|---|
| MCU pulls line low (Start) | At least 18 ms | 20000 × freq | MCU wake sensor up (DHT's detection of MCU's signal) |
| MCU release time | 20–40 µs | 30 × freq ± margin | Handover (MCU is ready to listen) |
| DHT11 responds low | 80 µs | 80 × freq | DHT11 acknowledges |
| DHT11 responds high | 80 µs | 80 × freq | DHT11 ready to send data |
| Each bit starts with low | 50 µs | 50 × freq | Start of every bit |
| Bit = 0 (high pulse) | 26–28 µs | ~20–35 µs range | Short high pulse |
| Bit = 1 (high pulse) | 70 µs | ~65–75 µs range | Long high pulse |

## Protocol Communication Flow

The MCU and DHT11 follow a handshake protocol:

**MCU Phase:**
- `MCU: "Hi. Wake up, DHT11"` → [18 ms Low] (The Master Request)
- `MCU: "I will listen to you now, DHT11"` → [20–40 µs High] (The Handover)

**DHT11 Acknowledgment:**
- `DHT11: "Hi, MCU. I'm here."` → [80 µs Low] (The ACK Arrival)
- `DHT11: "I'm ready to tell you."` → [80 µs High] (The ACK Handshake)

**Data Transmission (40 bits):**
Repeat 40 times:
- `DHT11: "Look out, here comes a bit!"` → [50 µs Low] (The Clock Sync)
- `DHT11: "It is a 0!"` → [26–28 µs High] **OR**
- `DHT11: "It is a 1!"` → [70 µs High]

## Protocol Timing → FSM States Mapping

| Timing Parameter | Duration | Used in Code | FSM States Involved | Purpose in FSM |
|---|---|---|---|---|
| MCU pulls line low (Start) | ≥ 18 ms | 20000 × freq | CONFIG_COUNT_18M, EN_COUNT_18M | Wake up DHT11 |
| MCU release time | 20–40 µs | 30 × freq ± margin | CONFIG_COUNT_20, EN_COUNT_20 | Release line & wait for DHT11 response |
| DHT11 responds low | 80 µs | 80 × freq | CONFIG_COUNT_80, EN_COUNT_80 | Wait for DHT11 acknowledgment |
| DHT11 responds high | 80 µs | 80 × freq | CONFIG_COUNT_80_2, EN_COUNT_80_2 | Wait for DHT11 ready signal |
| Each bit starts with low | 50 µs | 50 × freq | CONFIG_COUNT_50, EN_COUNT_50 + CONFIG_COUNT_26 | Detect start of each bit |
| Bit = 0 (high pulse) | 26–28 µs | 20–35 µs range | EN_COUNT_26 + SECOND_COMPARATOR | Detect short high pulse → Bit 0 |
| Bit = 1 (high pulse) | 70 µs | 65–75 µs range | EN_COUNT_26 + SECOND_COMPARATOR | Detect long high pulse → Bit 1 |
| Store received bit | — | — | REC_0, REC_1 | Shift bit into SIPO register |

## FSM State Analysis

### MCU Pulls Line Low (Sensor Wake-up)

Initializes the communication by pulling the data line low for 18 ms to wake the DHT11 sensor.

- **CONFIG_COUNT_18M**: Configure the counter for 18 ms timing
- **EN_COUNT_18M**: Enable the counter to measure the wake-up period

