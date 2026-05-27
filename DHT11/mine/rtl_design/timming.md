\## Time analysis table



Timing Parameter            Value in Datasheet  Used in Code        Meaning

──────────────────────────  ──────────────────  ──────────────────  ─────────────────────────

MCU pulls line low (Start)  At least 18 ms      20000 \* freq        MCU wake sensor up (DHT's dectection MCU's signal)

MCU release time            20–40 µs            30 \* freq ± margin  Handover (MCU is ready to listen)

DHT11 responds low          80 µs               80 \* freq           DHT11 acknowledges

DHT11 responds high         80 µs               80 \* freq           DHT11 ready to send data

Each bit starts with low    50 µs               50 \* freq           Start of every bit

Bit = 0 (high pulse)        26–28 µs            \~20–35 µs range     Short high pulse

Bit = 1 (high pulse)        70 µs               \~65–75 µs range     Long high pulse





\## The whole conversation that it must be:



MCU: "Hi. Wake up, DHT11"                     -- \[18 ms Low] (The Master Request)

MCU: "I will listen to you now, DHT11"        -- \[20–40 µs High] (The Handover)



DHT11: "Hi, MCU. I'm here."                   -- \[80 µs Low] (The ACK Arrival)

DHT11: "I'm ready to tell you."               -- \[80 µs High] (The ACK Handshake)



\-- REPEAT 40 TIMES FOR THE DATA BITS --

DHT11: "Look out, here comes a bit!"          -- \[50 µs Low] (The Clock Sync)

DHT11: "It is a 0!"                           -- \[26–28 µs High] 

&#x20;  -OR-

DHT11: "It is a 1!"                           -- \[70 µs High]









\## Protocol Timing → FSM States Mapping



Timing Parameter    	  Duration     Used in Code   		FSM States Involved         	Purpose in FSM

──────────────────  ────────  ────────────  ──────────────────────────  ───────────────────────────

MCU pulls line low  	    ≥ 18 ms   20000 \* freq  		CONFIG\_COUNT\_18M,           Wake up DHT11

(Start)                                     			EN\_COUNT\_18M

MCU release time    	   20–40 µs  	30 \* freq ±   		CONFIG\_COUNT\_20,            Release line \& wait for

&#x20;                             		margin        		EN\_COUNT\_20                 DHT11 response

DHT11 responds low  	   80 µs     	80 \* freq     		CONFIG\_COUNT\_80,            Wait for DHT11

&#x20;                                           			EN\_COUNT\_80                 acknowledgment

DHT11 responds      	   80 µs     	80 \* freq     		CONFIG\_COUNT\_80\_2,          Wait for DHT11 ready signal

high                                        			EN\_COUNT\_80\_2

Each bit starts     	   50 µs     	50 \* freq     		CONFIG\_COUNT\_50,            Detect start of each bit

with low                                    			EN\_COUNT\_50 + CONFIG\_COUNT\_26

Bit = 0 (high       	   26–28 µs  	20–35 µs      		EN\_COUNT\_26 +               Detect short high pulse →

pulse)                        		range         		SECOND\_COMPARATOR           Bit 0

Bit = 1 (high       	   70 µs     	65–75 µs      		EN\_COUNT\_26 +               Detect long high pulse →

pulse)                        		range         		SECOND\_COMPARATOR           Bit 1

Store received bit  —         —             			REC\_0, REC\_1                Shift bit into SIPO

&#x20;                                                                       			register





\## Analysising each state: 

* MCU pulls line low: Wake the sensor up

  * CONFIG\_COUNT\_18M: 
  * EN\_COUNT\_18M

