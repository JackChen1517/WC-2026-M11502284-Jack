# Lab 2: gNB TDD Traffic Analysis

## 1. Goal

In this lab, you will compare three 5-second UDP iperf3 tests. You will learn how TDD slot allocation can affect downlink and uplink performance.

You will also read the broadcast TDD configuration from SIB1 in Wireshark, draw the downlink/uplink time allocation, and relate the configuration to the iperf3 results.

You do not need to understand every packet. Follow the steps, complete the tables, and answer the short questions.

## 2. Test Scenarios and Files

| Scenario | TDD setting | Traffic direction | Main idea |
| --- | --- | --- | --- |
| A | DL-heavy | Downlink | The TDD setting favors the traffic direction |
| B | DL-heavy | Uplink | There are fewer opportunities for uplink traffic |
| C | Balanced | Uplink | More uplink opportunities are available than in B |

[Download the lab files from Google Drive](https://drive.google.com/drive/u/0/folders/1gTgosNkWFuak_sXbHFTxqaRqXb15Wz9T).

### Two Types of Packet Captures

Two types of packet captures are provided in this lab:

- **Configuration captures:** Recorded separately during initial cell access to show the broadcast SIB1 messages for the two TDD settings. Use these files to read and draw the common TDD configuration. Only two configuration captures are required because Scenarios A and B share the DL-heavy setting.
- **Scenario captures:** Recorded during the iperf3 tests. Use these files to analyze user-plane packets and the TCP control connection. Separate captures are provided for A, B, and C because each scenario uses a different combination of TDD setting and traffic direction.

The configuration captures are reference recordings for the corresponding settings, not simultaneous recordings of the original iperf3 runs. The scenario-to-configuration mapping below is supplied by the instructor; it cannot be established from the iperf3 JSON alone. Configuration captures are not used to calculate test throughput, loss, jitter, RTT, or packet intervals.

### Configuration Captures — Task 4 Only

| TDD setting | Applies to | Configuration capture |
| --- | --- | --- |
| DL-heavy | Scenarios A and B | `tdd-dl-heavy-sib1.pcapng` |
| Balanced | Scenario C | `tdd-balanced-sib1.pcapng` |

### Performance Files — Tasks 1–3 and 5–8

| Scenario | iperf3 JSON | Scenario capture |
| --- | --- | --- |
| A | `scenario-a-downlink.json` | `scenario-a-dl-heavy-downlink.pcapng` |
| B | `scenario-b-uplink.json` | `scenario-b-dl-heavy-uplink.pcapng` |
| C | `scenario-c-uplink.json` | `scenario-c-balanced-uplink.pcapng` |

> The JSON files report the traffic direction and performance results. They do not contain the gNB TDD configuration. Use the configuration captures for SIB1 and the scenario captures for traffic analysis.

## 3. Task 1 — Confirm the Traffic Direction

Open each JSON file and find:

```text
start → test_start → reverse
```

- `reverse: 1` means downlink in this lab.
- `reverse: 0` means uplink in this lab.

Complete the table:

| Scenario | Reverse value | Direction |
| --- | ---: | --- |
| A | 1 | Downlink |
| B | 0 | Uplink |
| C | 0 | Uplink |

**Question:** Does the traffic direction shown in the JSON match the scenario description? Show the proof.

**Answer:** Yes. All three JSON files match the scenario description. Scenario A has `reverse: 1`, which means downlink. Scenarios B and C have `reverse: 0`, which means uplink.

Proof from `start → test_start` in each JSON file (line 32):

```text
scenario-a-downlink.json   "protocol": "UDP",   "reverse": 1   → Downlink
scenario-b-uplink.json     "protocol": "UDP",   "reverse": 0   → Uplink
scenario-c-uplink.json     "protocol": "UDP",   "reverse": 0   → Uplink
```

Screenshots of `start → test_start` in each JSON file:

Scenario A (`scenario-a-downlink.json`):

![reverse value in the Scenario A JSON](images/task1-reverse-a.png)

Scenario B (`scenario-b-uplink.json`):

![reverse value in the Scenario B JSON](images/task1-reverse-b.png)

Scenario C (`scenario-c-uplink.json`):

![reverse value in the Scenario C JSON](images/task1-reverse-c.png)

## 4. Task 2 — Compare Throughput

In each JSON file, find:

```text
end → sum → bits_per_second
```

Convert the value to Mbps:

```text
Throughput (Mbps) = bits_per_second ÷ 1,000,000
```

Complete the table:

| Scenario | Direction | Throughput (Mbps) |
| --- | --- | ---: |
| A | Downlink | 90.65 |
| B | Uplink | 109.12 |
| C | Uplink | 124.42 |

Source values from `end → sum → bits_per_second`:

```text
scenario-a-downlink.json   bits_per_second = 90,654,195.84    → 90.65 Mbps
scenario-b-uplink.json     bits_per_second = 109,117,249.69   → 109.12 Mbps
scenario-c-uplink.json     bits_per_second = 124,416,749.13   → 124.42 Mbps
```

Screenshots of `end → sum → bits_per_second` in each JSON file:

Scenario A (`scenario-a-downlink.json`):

![bits_per_second in the Scenario A JSON](images/task2-throughput-a.png)

Scenario B (`scenario-b-uplink.json`):

![bits_per_second in the Scenario B JSON](images/task2-throughput-b.png)

Scenario C (`scenario-c-uplink.json`):

![bits_per_second in the Scenario C JSON](images/task2-throughput-c.png)

Calculate how much C improves over B:

```text
Improvement (%) = (C - B) ÷ B × 100
```

```text
Improvement (%) = (124.42 - 109.12) ÷ 109.12 × 100 ≈ 14.02%
```

Answer:

1. What is the downlink throughput in Scenario A?
   - **About 90.65 Mbps.**
2. What is the uplink throughput in Scenario B?
   - **About 109.12 Mbps.**
3. Is the uplink throughput higher in B or C?
   - **C.** Scenario C reaches 124.42 Mbps, compared with 109.12 Mbps in Scenario B.
4. By approximately what percentage does C improve over B?
   - **About 14.0%.**

> Use the JSON `bits_per_second` value. Do not recalculate throughput using the other time fields.

## 5. Task 3 — Check the Trade-off

In each JSON file, find:

```text
end → sum → lost_percent
end → sum → jitter_ms
```

Complete the table:

| Scenario | Throughput (Mbps) | Packet loss (%) | Jitter (ms) |
| --- | ---: | ---: | ---: |
| B | 109.12 | 0.24 | 0.116 |
| C | 124.42 | 2.13 | 0.162 |

Source values from `end → sum`:

```text
scenario-b-uplink.json   lost_percent = 0.2357   jitter_ms = 0.1162   (lost 111 of 47,102 packets)
scenario-c-uplink.json   lost_percent = 2.1339   jitter_ms = 0.1620   (lost 1,146 of 53,704 packets)
```

Screenshots of `end → sum` in each JSON file:

Scenario B (`scenario-b-uplink.json`):

![lost_percent and jitter_ms in the Scenario B JSON](images/task3-loss-jitter-b.png)

Scenario C (`scenario-c-uplink.json`):

![lost_percent and jitter_ms in the Scenario C JSON](images/task3-loss-jitter-c.png)

Answer:

1. Does C have higher uplink throughput than B?
   - **Yes.** C reaches 124.42 Mbps, compared with 109.12 Mbps in B, an increase of about 14.02%.
2. Does C have lower packet loss than B?
   - **No.** C has higher packet loss: 2.13% compared with 0.24% in B, an increase of about 1.9 percentage points.
3. What trade-off, if any, is visible when changing from B to C?
   - **C gains throughput but loses some stability.** Throughput rises by about 14%, while packet loss rises from 0.24% to 2.13% and jitter rises from 0.116 ms to 0.162 ms. The gain still outweighs the extra loss: multiplying each `bits_per_second` by (1 − loss) gives an estimated delivered rate of about 108.86 Mbps for B and 121.76 Mbps for C.

A simple answer format is:

> Compared with Scenario B, Scenario C has ______ throughput, ______ packet loss, and ______ jitter. These results suggest ______.

**Answer:** Compared with Scenario B, Scenario C has higher throughput (124.42 vs. 109.12 Mbps), higher packet loss (2.13% vs. 0.24%), and higher jitter (0.162 vs. 0.116 ms). These results suggest that C carries more uplink traffic, but with less reliable and slightly less stable delivery, so higher throughput did not come without a cost in this test.

Use the measured values. If one result improves while another worsens, explain that trade-off. Do not assume that increasing UL resources always increases packet loss.

## 6. Task 4 — Read SIB1 and Draw the TDD Configuration

Use these **two configuration captures** for this task:

- `tdd-dl-heavy-sib1.pcapng`: Reference configuration for Scenarios A and B.
- `tdd-balanced-sib1.pcapng`: Reference configuration for Scenario C.

These files were recorded separately from the original iperf3 tests. Use them to examine the supplied reference settings.

### 4.1 What Is SIB1?

A gNB broadcasts system information so that a UE can learn how to access the cell.

- **MIB (Master Information Block):** Provides basic information, including parameters needed to find SIB1.
- **SIB1 (System Information Block Type 1):** Provides cell-access and common radio configuration information. In this TDD lab, it includes `tdd-UL-DL-ConfigurationCommon`.

You will read the **common TDD configuration in SIB1**, not decode every MIB/SIB field. This configuration describes DL, UL, and flexible time resources; it is not a record of the actual data scheduled in every slot.

### 4.2 Enable MAC-NR Decoding and Find SIB1

These captures contain NR MAC messages carried over UDP. **Enable MAC-NR-over-UDP decoding before applying the SIB1 filter.** Without this decoder, the packets may appear only as UDP and the SIB1 filter may return no results.

1. Open `tdd-dl-heavy-sib1.pcapng` in Wireshark.
2. Open **Analyze → Enabled Protocols**. Locate **MAC-NR over UDP** (`mac_nr_udp`) in the heuristic dissectors list and enable it. Depending on the Wireshark version, the heuristic list may be shown separately. Apply the change and reload the capture if needed.
3. Apply this display filter:

   ```text
   nr-rrc.systemInformationBlockType1_element
   ```

4. Select a SIB1 packet. In the packet details, expand:

   ```text
   NR Radio Resource Control (RRC) protocol
     → BCCH-DL-SCH-Message
       → message
         → c1
           → systemInformationBlockType1
             → servingCellConfigCommon
               → tdd-UL-DL-ConfigurationCommon
   ```

5. Read `referenceSubcarrierSpacing`, then expand `pattern1` and record:

   ```text
   dl-UL-TransmissionPeriodicity
   nrofDownlinkSlots
   nrofDownlinkSymbols
   nrofUplinkSlots
   nrofUplinkSymbols
   ```

6. Check whether `pattern2` is also present.
7. Repeat for `tdd-balanced-sib1.pcapng`.

Record the **decoded labels and units** shown in Wireshark. For example, record `30 kHz` or `5 ms`, not the enumeration number displayed in parentheses.

You can also locate packets containing the TDD configuration with:

```text
nr-rrc.tdd_UL_DL_ConfigurationCommon_element
```

Confirm that the selected TDD fields are inside **SIB1**. If a filter is not recognized in your version, try `nr-rrc` and locate `systemInformationBlockType1` manually.

> **Troubleshooting:** If packets still appear only as UDP, check that the **MAC-NR over UDP heuristic dissector** is enabled, rather than only the MAC-NR protocol. If the file still does not decode, ask the instructor to check the settings. Do not infer TDD parameters from packet timing. Graphical menu labels may vary by version.

### 4.3 Record the TDD Parameters

Complete the table for `pattern1`:

| Item | DL-heavy: A/B | Balanced: C |
| --- | --- | --- |
| Configuration capture filename | `tdd-dl-heavy-sib1.pcapng` | `tdd-balanced-sib1.pcapng` |
| SIB1 packet number | 2 | 2 |
| `referenceSubcarrierSpacing` (kHz) | 30 kHz | 30 kHz |
| `dl-UL-TransmissionPeriodicity` (ms) | 5 ms | 2.5 ms |
| `nrofDownlinkSlots` | 7 | 2 |
| `nrofDownlinkSymbols` | 6 | 6 |
| `nrofUplinkSlots` | 2 | 2 |
| `nrofUplinkSymbols` | 4 | 4 |
| Is `pattern2` present? | No | No |

If `pattern2` is present, add another table containing its period and four slot/symbol counts. It uses the same reference SCS.

`pattern2` is not present in either configuration, so no additional table is needed.

Take **one screenshot per configuration** showing the selected SIB1 packet and the expanded TDD fields. Include `pattern2` in an additional screenshot if it is present and does not fit.

DL-heavy (`tdd-dl-heavy-sib1.pcapng`, packet 2):

![SIB1 TDD fields for the DL-heavy configuration](images/task4-sib1-dl-heavy.png)

Balanced (`tdd-balanced-sib1.pcapng`, packet 2):

![SIB1 TDD fields for the balanced configuration](images/task4-sib1-balanced.png)

### 4.4 Draw the TDD Time Allocation

Draw one diagram for DL-heavy and one for balanced. **Use the same 0–5 ms time axis for both diagrams.** Derive each repeating period from SIB1 and repeat the shorter pattern as needed to fill the 5 ms interval. Mark every repeating-period boundary.

A spreadsheet, a drawing tool, or a hand-drawn diagram is acceptable. Include the diagrams as images in your report.

For the normal cyclic prefix used in this exercise, each reference slot contains **14 OFDM symbols**, numbered **0–13**. Use the SIB1 **reference SCS** to construct the common pattern:

| Reference SCS | Reference slot duration |
| --- | ---: |
| 15 kHz | 1 ms |
| 30 kHz | 0.5 ms |
| 60 kHz | 0.25 ms |
| 120 kHz | 0.125 ms |

```text
Number of reference slots in a pattern = Pattern period ÷ Reference slot duration
```

For each pattern:

1. Place `nrofDownlinkSlots` full DL slots at the beginning.
2. If `nrofDownlinkSymbols` is nonzero, mark that many symbols at the beginning of the next slot as DL.
3. Place `nrofUplinkSlots` full UL slots at the end.
4. If `nrofUplinkSymbols` is nonzero, mark that many symbols at the end of the slot immediately before the full UL slots as UL. If there are no full UL slots, use the last slot of the pattern.
5. Mark all remaining symbols as **F (flexible)**. The partial DL and partial UL portions may be in the same slot or in different slots.

Use **D = downlink**, **U = uplink**, and **F = flexible**. Include a legend, the common 0–5 ms time scale, slot indices, and repeating-period boundaries. You may restart the local slot index at zero at each period boundary, but label the boundaries clearly. Expand any mixed slot to show all 14 symbols. Do not automatically label flexible symbols as UL or guard time.

If `pattern2` is present, draw `pattern1` followed by `pattern2`; the combined sequence repeats. Label each pattern boundary and show the full combined duration.

DL-heavy diagram (one 5 ms period):

![DL-heavy TDD time allocation over 0 to 5 ms](images/task4-tdd-dl-heavy.png)

Balanced diagram (two 2.5 ms periods):

![Balanced TDD time allocation over 0 to 5 ms](images/task4-tdd-balanced.png)

First, count the symbols over **one complete repeating sequence** for each configuration. The two sequences may have different durations, so their raw UL symbol counts alone cannot be used to compare resources per unit time:

```text
Configured DL symbols = 14 × Full DL slots + Partial DL symbols
Configured UL symbols = 14 × Full UL slots + Partial UL symbols
Flexible symbols = Total symbols - Configured DL symbols - Configured UL symbols
Configured UL symbol share (%) = Configured UL symbols ÷ Total symbols × 100
```

For two patterns, sum the counts and durations of both patterns before calculating the share. These are symbol-count shares of the common configuration, not exact airtime measurements or measured throughput shares.

Calculation with a 0.5 ms reference slot (30 kHz):

```text
DL-heavy: slots = 5 ÷ 0.5 = 10      total = 10 × 14 = 140 symbols
          DL = 14 × 7 + 6 = 104     UL = 14 × 2 + 4 = 32     F = 140 - 104 - 32 = 4
          UL share = 32 ÷ 140 × 100 = 22.86%

Balanced: slots = 2.5 ÷ 0.5 = 5     total = 5 × 14 = 70 symbols
          DL = 14 × 2 + 6 = 34      UL = 14 × 2 + 4 = 32     F = 70 - 34 - 32 = 4
          UL share = 32 ÷ 70 × 100 = 45.71%
```

| TDD setting | Repeating duration (ms) | Total reference slots | DL symbols | UL symbols | Flexible symbols | Configured UL symbol share (%) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| DL-heavy | 5 | 10 | 104 | 32 | 4 | 22.86 |
| Balanced | 2.5 | 5 | 34 | 32 | 4 | 45.71 |

Next, count the symbols over the **same 5 ms interval** shown in your diagrams. Multiply each per-sequence count by the number of complete repetitions in 5 ms. For these captures, 5 ms contains a whole number of repetitions for both configurations.

| TDD setting | Repeating duration (ms) | Repetitions in 5 ms | DL symbols in 5 ms | UL symbols in 5 ms | Flexible symbols in 5 ms |
| --- | ---: | ---: | ---: | ---: | ---: |
| DL-heavy | 5 | 1 | 104 | 32 | 4 |
| Balanced | 2.5 | 2 | 68 | 64 | 8 |

**Question:** Can two patterns have the same UL symbol count per period but provide different numbers of UL symbols per unit time? Explain using your two tables.

**Answer:** Yes. Both configurations have 32 UL symbols per period, but DL-heavy repeats every 5 ms and balanced repeats every 2.5 ms. Over the same 5 ms interval, DL-heavy therefore provides 32 UL symbols (1 repetition), while balanced provides 64 UL symbols (2 repetitions). This is why the shares differ (22.86% vs. 45.71%), and why raw per-period counts cannot be compared directly.

### 4.5 Connect SIB1 to the Performance Results

Use the supplied mapping between the reference configuration captures and scenarios. Complete this table using Tasks 1–3 and your SIB1 analysis:

| Scenario | Reference TDD setting | JSON traffic direction | Configured UL symbol share (%) | Throughput (Mbps) | Packet loss (%) |
| --- | --- | --- | ---: | ---: | ---: |
| A | DL-heavy | Downlink | 22.86 | 90.65 | 0.44 |
| B | DL-heavy | Uplink | 22.86 | 109.12 | 0.24 |
| C | Balanced | Uplink | 45.71 | 124.42 | 2.13 |

Answer:

1. Which decoded TDD parameters differ between DL-heavy and balanced?
   - **`dl-UL-TransmissionPeriodicity` (5 ms vs. 2.5 ms) and `nrofDownlinkSlots` (7 vs. 2).** The other fields are the same: `referenceSubcarrierSpacing` is 30 kHz, `nrofDownlinkSymbols` is 6, `nrofUplinkSlots` is 2, `nrofUplinkSymbols` is 4, and `pattern2` is absent in both.
2. Which configuration provides more configured UL symbols per unit time? Support your answer with the diagram and calculated share.
   - **Balanced.** Its diagram shows the UL portion twice in 5 ms, giving 64 UL symbols compared with 32 for DL-heavy. Its configured UL symbol share is 45.71%, twice the 22.86% of DL-heavy.
3. How does that difference help explain the B/C throughput comparison?
   - **Balanced offers more uplink transmission opportunities per unit time, which is consistent with C reaching higher uplink throughput than B** (124.42 vs. 109.12 Mbps). The gain is not proportional: UL symbols double, but throughput rises by only about 14%. So the TDD pattern helps explain the result, but it is not shown to be the only cause.
4. Why can A and B use the same SIB1 TDD configuration even though their traffic directions differ?
   - **SIB1 broadcasts a cell-wide common configuration that does not depend on the direction of any iperf3 test.** The same pattern contains both DL and UL resources. A uses the DL portion for its traffic and B uses the UL portion.

> More configured UL resources can provide more opportunities for uplink traffic, but they do not guarantee a proportional throughput increase. Scheduling, radio conditions, overhead, offered traffic, and other radio settings also affect the result. This comparison does not establish that the TDD pattern is the only cause of a performance difference. SIB1 does not show actual slot-by-slot data scheduling. Because the configuration captures were recorded separately, use them as references for the supplied settings, not as independent proof of what was broadcast during each original performance run.

### Optional Observation — PRACH Configuration Index

In the same SIB1, find:

```text
servingCellConfigCommon
  → uplinkConfigCommon
    → initialUplinkBWP
      → rach-ConfigCommon
        → setup
          → rach-ConfigGeneric
            → prach-ConfigurationIndex
```

Wireshark may omit or display the `setup` layer differently. You can also use this display filter, then confirm the field is inside SIB1:

```text
nr-rrc.prach_ConfigurationIndex
```

| TDD setting | SIB1 packet number | PRACH configuration index |
| --- | ---: | ---: |
| DL-heavy | 2 | 98 |
| Balanced | 2 | 98 |

PRACH is used to send a random-access preamble. Its configuration index is not the DL/UL slot ratio and does not directly explain iperf3 throughput. Equal indices in the two configurations are a valid observation. Drawing PRACH occasions requires additional parameters and the applicable TS 38.211 table; it is outside the required TDD drawing task.

## 7. Task 5 — Is the TDD Setting Suitable?

Complete the table using the supplied scenario mapping, the SIB1 reference configurations from Task 4, and your throughput results:

| Scenario | TDD setting | Traffic direction | Is the setting suitable for this direction? Why? |
| --- | --- | --- | --- |
| A | DL-heavy | Downlink | **Yes.** 104 of 140 symbols (74.29%) are configured as DL, so the setting favors this direction. A reached 90.65 Mbps with 0.44% loss. |
| B | DL-heavy | Uplink | **Not ideal for heavy uplink.** Only 32 of 140 symbols (22.86%) are configured as UL, which limits uplink opportunities. B still reached 109.12 Mbps with low loss (0.24%), but C achieved more uplink throughput. |
| C | Balanced | Uplink | **Yes, better than DL-heavy for uplink.** 45.71% of symbols are configured as UL (64 vs. 32 UL symbols in 5 ms), and throughput reached 124.42 Mbps. Packet loss was higher, at 2.13%. |

Suitability here is judged from how the configured resources match the traffic direction. It cannot be judged by comparing throughput across directions: A's downlink result (90.65 Mbps) is lower than B's uplink result (109.12 Mbps) even though A uses the setting that favors its direction, and offered traffic, scheduling, and radio settings also affect these values.

Use these ideas:

- More DL resources provide more downlink transmission opportunities.
- More UL resources provide more uplink transmission opportunities.
- A DL-heavy setting is suitable for downlink traffic.
- A DL-heavy setting may limit heavy uplink traffic.
- A balanced setting gives uplink traffic more opportunities than the DL-heavy setting in this lab.

> The configuration captures show the reference broadcast TDD settings. The iperf3 JSON reports performance; it does not contain TDD parameters. The scenario captures are used for traffic analysis and need not include SIB1.

**Question:** Why might Scenario C achieve better uplink throughput than Scenario B? Refer to your SIB1 diagrams and measured results.

**Answer:** The diagrams show that balanced repeats its UL portion every 2.5 ms, giving 64 UL symbols in 5 ms, while DL-heavy gives only 32. With twice the configured UL resources (45.71% vs. 22.86%), the UE has more opportunities to transmit, which is consistent with throughput rising from 109.12 Mbps in B to 124.42 Mbps in C (about 14.02%). The increase is much smaller than the doubling of UL symbols, so other factors such as scheduling, radio conditions, overhead, and offered traffic also matter.

## 8. Task 6 — Find GTP-U Information

Open a **scenario capture** in Wireshark and enter this display filter:

```text
gtp && ip
```

Choose a packet that contains two IPv4 headers. In the packet details, expand:

1. the first `Internet Protocol Version 4` header;
2. `User Datagram Protocol`;
3. `GPRS Tunneling Protocol`; and
4. the second `Internet Protocol Version 4` header.

Record the following information from one packet:

| Item | Value |
| --- | --- |
| Scenario capture filename | `scenario-a-dl-heavy-downlink.pcapng` |
| Packet number | 1892 |
| Outer source IP | 192.168.70.134 |
| Outer destination IP | 192.168.70.129 |
| Inner source IP | 192.168.70.135 |
| Inner destination IP | 10.0.0.6 |
| TEID | 0xcddc4529 (3453764905) |

Meaning:

- **Outer IP addresses:** The GTP-U tunnel endpoints.
- **Inner IP addresses:** The original UE/application packet.
- **TEID:** Identifies the GTP-U tunnel at the receiving endpoint.

Take one screenshot showing the selected packet and these fields.

![GTP-U fields of packet 1892 in Scenario A](images/task6-gtp-fields.png)

Packet 1892 is a G-PDU (`Message Type: T-PDU (0xff)`) carrying an iperf3 UDP data packet from port 5201 to port 51624, so it contains an inner IPv4 packet and is not an echo or control message.

> Some GTP packets are echo or control-related packets. Select one that contains an inner IPv4 packet.

## 9. Task 7 — TCP Retransmissions

Use the **scenario captures** for this task.

The performance traffic is UDP, but iperf3 also uses a small TCP control connection.

Apply this Wireshark filter:

```text
tcp.analysis.retransmission
```

Record the number of displayed packets:

| Scenario | Displayed TCP retransmissions |
| --- | ---: |
| A | 14 |
| B | 12 |
| C | 12 |

> Report the number displayed by Wireshark using the specified filter. Counts may vary with Wireshark analysis settings. Include screenshots showing the filter and displayed count for each scenario. The filter includes any TCP flows that Wireshark can decode in the capture; a displayed retransmission is not, by itself, proof of loss in the UDP test.

Scenario A:

![TCP retransmission filter and count for Scenario A](images/task7-retrans-a.png)

Scenario B:

![TCP retransmission filter and count for Scenario B](images/task7-retrans-b.png)

Scenario C:

![TCP retransmission filter and count for Scenario C](images/task7-retrans-c.png)

**Question:** Do these TCP retransmissions directly represent lost UDP performance packets?

**Answer:** No. They belong to the iperf3 TCP control connection on port 5201, not to the UDP data stream. UDP does not retransmit, and UDP loss is reported separately by the JSON `lost_packets` field (185 in A, 111 in B, and 1,146 in C), which does not track these counts (14, 12, and 12).

Most displayed packets are also not real retransmissions. The capture contains two observations of the same packet, a few microseconds apart, and Wireshark flags the second one because it has the same sequence number. Comparing the inner IP ID shows that 13 of the 14 packets in A and all 12 packets in B and C have the same inner IP ID as an earlier packet. The one exception is packet 6712 in A, which has a new inner IP ID and resends the 2-byte segment from packet 2538 about 234 ms later. Packet 6713 is a second observation of packet 6712.

## 10. Task 8 — RTT and Packet Interval

Use a **scenario capture** for both parts of this task.

### TCP RTT

Apply:

```text
tcp.analysis.ack_rtt
```

Select one packet from the iperf3 TCP control connection and record its ACK RTT:

| Scenario | Packet number | Example TCP ACK RTT |
| --- | ---: | --- |
| A | 1842 | 18.162 ms (0.018162 s) |

Packet 1842 is the final ACK of the TCP handshake on the iperf3 control connection (10.0.0.6:55141 to 192.168.70.135:5201). It acknowledges the SYN-ACK in packet 1833.

![TCP ACK RTT of packet 1842 in Scenario A](images/task8-ack-rtt.png)

This value is the RTT of the TCP control connection. It is not the RTT of the UDP data packets.

### Packet Interval

Apply a display filter to select UDP data packets from the same flow.

Select two consecutive, distinct UDP data packets in the filtered packet list. Avoid duplicated observations caused by merged captures; use the same capture point/interface when possible.

In the packet details of the second packet, find:

```text
Time delta from previous displayed frame
```

Record the scenario, display filter, packet numbers, and time interval.

| Scenario | Display filter | Packet numbers | Time interval |
| --- | --- | --- | --- |
| A | `gtp && ip.src==192.168.70.135 && ip.dst==10.0.0.6 && udp.srcport==5201 && udp.dstport==51624 && frame.len==1540` | 1898 → 1900 | 81 µs (0.000081 s) |

Packets 1898 and 1900 are consecutive iperf3 datagrams (`[3]` and `[4]`) with different IP IDs (inner 0x8ea3 and 0x8ea4), so they are distinct data packets. The filter returns 41,765 packets with no repeated IP ID, which matches the 41,765 packets in `scenario-a-downlink.json`. This confirms that it selects one observation of each data packet. The filter excludes the separate IP-fragment observations of the same data (for example, packet 1897), which do not decode as GTP.

![Time delta from previous displayed frame for packet 1900 in Scenario A](images/task8-packet-interval.png)

**Note:** The time interval depends on the applied display filter. Make sure the selected packets are distinct data packets, not duplicated observations from different capture points. Do not use their interval to infer the TDD slot pattern.

## 11. Final Questions

Answer each question in two or three sentences:

1. Does Scenario A use a TDD setting that matches its traffic direction?

   **Answer:** Yes. Scenario A carries downlink traffic and uses the DL-heavy setting, where 104 of 140 symbols per 5 ms period are configured as DL. The result was 90.65 Mbps with 0.44% packet loss.
2. Why is Scenario B expected to have limited uplink resources? Cite a parameter or count from the DL-heavy SIB1 reference configuration.

   **Answer:** The DL-heavy SIB1 sets `nrofUplinkSlots` to 2 and `nrofUplinkSymbols` to 4 in a 5 ms period, compared with 7 DL slots and 6 DL symbols. That gives only 32 UL symbols out of 140, or 22.86%, so uplink traffic has fewer transmission opportunities.
3. How much does uplink throughput change from B to C?

   **Answer:** Uplink throughput increases from 109.12 Mbps in B to 124.42 Mbps in C. That is a gain of 15.30 Mbps, or about 14.02%.
4. What negative result, if any, is visible in C? Support your answer with packet loss or jitter values.

   **Answer:** Packet loss and jitter are both worse in C. Packet loss rises from 0.24% to 2.13%, and jitter rises from 0.116 ms to 0.162 ms.
5. Why can increasing UL resources improve uplink throughput?

   **Answer:** More configured UL symbols per unit time give the UE more opportunities to be scheduled and to transmit. In this lab, UL symbols in 5 ms increase from 32 to 64, and throughput increases from 109.12 to 124.42 Mbps, although not proportionally.
6. Is adding more UL resources always the best solution? Explain using throughput and packet loss.

   **Answer:** No. Moving to balanced raised throughput by about 14%, but packet loss rose from 0.24% to 2.13%. It also reduced DL symbols in 5 ms from 104 to 68, so the best setting depends on the traffic mix and on how much loss the application can tolerate.
7. Suggest one simple follow-up test that could improve throughput without producing too much packet loss.

   **Answer:** Keep the balanced setting and repeat the UDP uplink test at several offered rates using `iperf3 -u -b`, for example 100, 110, 120, and 130 Mbps. Then choose the highest rate whose packet loss stays below a target such as 1%.
8. What can the configuration captures tell you that the iperf3 JSON cannot? Why are separately recorded configuration captures not direct proof of the configuration during an earlier test?

   **Answer:** The configuration captures show the broadcast TDD parameters in SIB1, such as the period and the DL and UL slot and symbol counts, while the JSON reports only traffic direction and performance. Because they were recorded separately, they show what was broadcast during that recording, not during the iperf3 runs. The link between each scenario and its TDD setting therefore relies on the instructor's supplied mapping.

## 12. Submission Checklist

Submit one Markdown file containing:

- the completed direction table;
- the throughput table and B-to-C calculation;
- the B/C packet-loss and jitter table;
- the SIB1 TDD parameter table, including filenames and packet numbers;
- one SIB1 TDD screenshot per configuration, plus `pattern2` screenshots if needed;
- two TDD diagrams: DL-heavy and balanced, with D/U/F labels and mixed-slot symbol details;
- the per-sequence symbol-count / UL-share table, the common 5 ms comparison table, and the configuration-to-performance table;
- the Task 4.5 answers and the TDD suitability table;
- one GTP-U screenshot and its completed table;
- the TCP retransmission counts and screenshots;
- one TCP ACK RTT example;
- one packet-interval example; and
- answers to the final questions.

The PRACH observation is optional and carries no required points. Include all referenced image files alongside the Markdown file, or submit a ZIP containing the report and images.

## 13. Grading Rubric

| Item | Points |
| --- | ---: |
| Traffic direction | 5 |
| Throughput values and calculation | 20 |
| B/C packet-loss and jitter comparison | 15 |
| SIB1 TDD parameter extraction and screenshot evidence | 10 |
| TDD diagrams, symbol counts, and UL shares | 10 |
| TDD suitability and connection to measured performance | 10 |
| GTP-U fields and screenshot | 15 |
| TCP retransmission, RTT, and packet interval | 10 |
| Clear final answers | 5 |
| **Total** | **100** |

## References

These references support the field names and interpretation. Reading the full specifications is not required.

- [Wireshark NR RRC display filter reference](https://www.wireshark.org/docs/dfref/n/nr-rrc.html): SIB1 and TDD display-filter fields.
- [3GPP TS 38.331, Release 18](https://www.etsi.org/deliver/etsi_ts/138300_138399/138331/18.08.00_60/ts_138331v180800p.pdf): SIB1, TDD-UL-DL-ConfigCommon, and TDD-UL-DL-Pattern definitions.
- [3GPP TS 38.213, Release 17, Section 11.1](https://www.etsi.org/deliver/etsi_ts/138200_138299/138213/17.06.00_60/ts_138213v170600p.pdf): Interpretation of common TDD slot and symbol allocation.