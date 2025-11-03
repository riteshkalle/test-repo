# DNP3 Protocol Data Units and Messages

## Detailed Analysis of Master-Outstation Communication

This document provides an in-depth look at the Protocol Data Units (PDUs) and message exchanges between DNP3 master stations and outstations, complementing the general IEEE 1815-2012 summary.

---

## Table of Contents
1. [Message Architecture Overview](#message-architecture-overview)
2. [Application Layer Message Structure](#application-layer-message-structure)
3. [Function Codes](#function-codes)
4. [Request-Response Patterns](#request-response-patterns)
5. [Message Fragmentation](#message-fragmentation)
6. [Object Headers and Data Objects](#object-headers-and-data-objects)
7. [Common Message Exchanges](#common-message-exchanges)

---

## Message Architecture Overview

### Three-Layer Message Structure

DNP3 messages traverse three protocol layers before transmission:

```
┌─────────────────────────────────────────────────────┐
│           APPLICATION LAYER                          │
│  • Request/Response/Confirmation                     │
│  • Function Codes                                    │
│  • Data Objects                                      │
│  • Creates: FRAGMENT (ASDU)                          │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│           TRANSPORT FUNCTION                         │
│  • Segmentation/Reassembly                          │
│  • Sequence Numbers                                  │
│  • Creates: SEGMENT (max 250 octets)                │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│           DATA LINK LAYER                            │
│  • Frame Transmission                                │
│  • Error Detection (CRC-16)                         │
│  • Creates: FRAME (max 292 octets with header/CRC)  │
└─────────────────────────────────────────────────────┘
```

### Master-Outstation Communication Model

```
    MASTER                           OUTSTATION
      │                                   │
      │   REQUEST (Function Code 1-128)   │
      │──────────────────────────────────>│
      │                                   │
      │  RESPONSE (Function Code 129-255) │
      │<──────────────────────────────────│
      │                                   │
      │    CONFIRMATION (if requested)    │
      │──────────────────────────────────>│
      │                                   │
```

---

## Application Layer Message Structure

### Message Types

DNP3 defines three application message types:

1. **REQUEST** - Master to Outstation (Function Codes 0-128)
2. **RESPONSE** - Outstation to Master (Function Codes 129-255)
3. **CONFIRMATION** - Acknowledges receipt (Function Code 0)

### Application Request Header (Master → Outstation)

```
┌──────────────────────────────────────────────────┐
│  Octet 0: Application Control                    │
│  ┌─────┬─────┬─────┬────────────────────────┐   │
│  │ FIR │ FIN │ CON │    SEQ (4 bits)        │   │
│  └─────┴─────┴─────┴────────────────────────┘   │
├──────────────────────────────────────────────────┤
│  Octet 1: Function Code                          │
│  (Values 0-128 for requests)                     │
└──────────────────────────────────────────────────┘

Field Descriptions:
• FIR (1 bit): First fragment indicator
  - 0 = Not first fragment
  - 1 = First fragment of message
  
• FIN (1 bit): Final fragment indicator
  - 0 = More fragments follow
  - 1 = Final fragment
  
• CON (1 bit): Confirmation required
  - 0 = No confirmation needed
  - 1 = Receiver must send confirmation
  
• SEQ (4 bits): Sequence number (0-15)
  - Increments with each new message
  - Used for duplicate detection
  
• Function Code (8 bits): Message purpose
  - Defines the operation requested
```

### Application Response Header (Outstation → Master)

```
┌──────────────────────────────────────────────────┐
│  Octet 0: Application Control                    │
│  (Same structure as request header)              │
├──────────────────────────────────────────────────┤
│  Octet 1: Function Code                          │
│  (Values 129-255 for responses)                  │
├──────────────────────────────────────────────────┤
│  Octets 2-3: Internal Indications (IIN)          │
│  ┌──────────────────────────────────────────┐   │
│  │  IIN1 (Octet 2) │  IIN2 (Octet 3)        │   │
│  └──────────────────────────────────────────┘   │
└──────────────────────────────────────────────────┘

Internal Indications (IIN) Bits:
IIN 1.0: Device restart occurred
IIN 1.1: DNP functionality not supported
IIN 1.2: Broadcast message received
IIN 1.3: Time synchronization required
IIN 1.4: Object quality not current
IIN 1.5: Event buffer overflow
IIN 1.6: Device in local mode
IIN 1.7: Device trouble

IIN 2.0: Configuration corrupt
IIN 2.1: Already executing
IIN 2.2: Event buffer overflow (by class)
IIN 2.3: Event buffer overflow (by class)
IIN 2.4: Event buffer overflow (by class)
IIN 2.5: Unknown object in request
IIN 2.6: Parameters out of range or invalid
IIN 2.7: Reserved for future use
```

---

## Function Codes

### Master Request Function Codes (0-128)

| Code | Hex  | Name              | Purpose                                                          |
|------|------|-------------------|------------------------------------------------------------------|
| 0    | 0x00 | CONFIRM           | Confirm receipt of fragment                                      |
| 1    | 0x01 | READ              | Request data from outstation                                     |
| 2    | 0x02 | WRITE             | Write data to outstation                                         |
| 3    | 0x03 | SELECT            | Arm output points for operation (2-step control)                 |
| 4    | 0x04 | OPERATE           | Execute previously selected operation                            |
| 5    | 0x05 | DIRECT_OPERATE    | Single-step control (select + operate combined)                  |
| 6    | 0x06 | DIRECT_OPERATE_NR | Direct operate, no response expected                             |
| 7    | 0x07 | IMMED_FREEZE      | Freeze counter values immediately                                |
| 8    | 0x08 | IMMED_FREEZE_NR   | Freeze counters, no response                                     |
| 9    | 0x09 | FREEZE_CLEAR      | Freeze and clear counters                                        |
| 10   | 0x0A | FREEZE_CLEAR_NR   | Freeze and clear, no response                                    |
| 11   | 0x0B | FREEZE_AT_TIME    | Freeze counters at specified time                                |
| 12   | 0x0C | FREEZE_AT_TIME_NR | Freeze at time, no response                                      |
| 13   | 0x0D | COLD_RESTART      | Restart device (cold reboot)                                     |
| 14   | 0x0E | WARM_RESTART      | Restart device (warm reboot)                                     |
| 15   | 0x0F | INITIALIZE_DATA   | Reset data to default values                                     |
| 16   | 0x10 | INITIALIZE_APPL   | Reset application                                                |
| 17   | 0x11 | START_APPL        | Start application                                                |
| 18   | 0x12 | STOP_APPL         | Stop application                                                 |
| 19   | 0x13 | SAVE_CONFIG       | Save configuration to non-volatile memory                        |
| 20   | 0x14 | ENABLE_UNSOLICITED| Enable unsolicited responses                                     |
| 21   | 0x15 | DISABLE_UNSOLICITED| Disable unsolicited responses                                   |
| 22   | 0x16 | ASSIGN_CLASS      | Assign point to event class                                      |
| 23   | 0x17 | DELAY_MEASURE     | Measure communication delay                                      |
| 24   | 0x18 | RECORD_CURRENT_TIME| Record current time                                            |
| 25   | 0x19 | OPEN_FILE         | Open file for transfer                                           |
| 26   | 0x1A | CLOSE_FILE        | Close file                                                       |
| 27   | 0x1B | DELETE_FILE       | Delete file                                                      |
| 28   | 0x1C | GET_FILE_INFO     | Get file information                                             |
| 29   | 0x1D | AUTHENTICATE_FILE | Authenticate file                                                |
| 30   | 0x1E | ABORT_FILE        | Abort file transfer                                              |
| 31   | 0x1F | ACTIVATE_CONFIG   | Activate configuration                                           |
| 32   | 0x20 | AUTHENTICATE_REQ  | Authentication request (Secure Authentication)                   |
| 33   | 0x21 | AUTHENTICATE_ERR  | Authentication error                                             |

### Outstation Response Function Codes (129-255)

| Code | Hex  | Name              | Purpose                                                          |
|------|------|-------------------|------------------------------------------------------------------|
| 129  | 0x81 | RESPONSE          | Solicited response to master request                             |
| 130  | 0x82 | UNSOLICITED_RESPONSE | Unsolicited response (event report)                           |
| 131  | 0x83 | AUTHENTICATE_RESP | Authentication response (Secure Authentication)                  |

---

## Request-Response Patterns

### Pattern 1: Polled Data Request (READ)

**Scenario**: Master requests current values from outstation

```
Master                                    Outstation
  │                                            │
  │  READ (FC=1)                               │
  │  Request: Binary Inputs 0-15               │
  │  FIR=1, FIN=1, CON=1, SEQ=5               │
  │───────────────────────────────────────────>│
  │                                            │
  │                                            │ (Process request)
  │                                            │ (Collect data)
  │                                            │
  │  RESPONSE (FC=129)                         │
  │  Contains: 16 Binary Input values          │
  │  FIR=1, FIN=1, CON=0, SEQ=5               │
  │<───────────────────────────────────────────│
  │                                            │
  │  CONFIRM (FC=0)                            │
  │  SEQ=5                                     │
  │───────────────────────────────────────────>│
  │                                            │
```

**Message Details**:
- **Request**: 
  - Function Code: 1 (READ)
  - Object: Group 1 Variation 2 (Binary Input with Status)
  - Qualifier: 0x00 (Start-Stop indexes)
  - Range: Start=0, Stop=15
  
- **Response**:
  - Function Code: 129 (RESPONSE)
  - IIN: Status flags
  - Object: Group 1 Variation 2
  - Data: 16 binary input values with quality flags

### Pattern 2: Select-Before-Operate Control

**Scenario**: Master controls an output using 2-step safety procedure

```
Master                                    Outstation
  │                                            │
  │  SELECT (FC=3)                             │
  │  Control Point 5: Close                    │
  │  FIR=1, FIN=1, CON=1, SEQ=7               │
  │───────────────────────────────────────────>│
  │                                            │
  │                                            │ (Validate control)
  │                                            │ (ARM output)
  │                                            │
  │  RESPONSE (FC=129)                         │
  │  Status: Success (0)                       │
  │  FIR=1, FIN=1, CON=0, SEQ=7               │
  │<───────────────────────────────────────────│
  │                                            │
  │  CONFIRM (FC=0), SEQ=7                     │
  │───────────────────────────────────────────>│
  │                                            │
  │  OPERATE (FC=4)                            │
  │  Control Point 5: Close (same parameters)  │
  │  FIR=1, FIN=1, CON=1, SEQ=8               │
  │───────────────────────────────────────────>│
  │                                            │
  │                                            │ (Execute control)
  │                                            │ (CLOSE BREAKER)
  │                                            │
  │  RESPONSE (FC=129)                         │
  │  Status: Success (0)                       │
  │  FIR=1, FIN=1, CON=0, SEQ=8               │
  │<───────────────────────────────────────────│
  │                                            │
  │  CONFIRM (FC=0), SEQ=8                     │
  │───────────────────────────────────────────>│
  │                                            │
```

### Pattern 3: Unsolicited Response (Event Report)

**Scenario**: Outstation spontaneously reports important event

```
Master                                    Outstation
  │                                            │
  │                                            │ (Event occurs)
  │                                            │ (Alarm triggered)
  │                                            │
  │  UNSOLICITED_RESPONSE (FC=130)             │
  │  Event: Binary Input 12 changed to ON      │
  │  FIR=1, FIN=1, CON=1, SEQ=3 (unsol seq)   │
  │<───────────────────────────────────────────│
  │                                            │
  │  CONFIRM (FC=0)                            │
  │  SEQ=3 (matches unsolicited sequence)      │
  │───────────────────────────────────────────>│
  │                                            │
  │                                            │ (Clear event from buffer)
  │                                            │
```

**Key Points**:
- Unsolicited responses use a **separate sequence number** from solicited communications
- Events remain in outstation buffer until confirmed by master
- Master must enable unsolicited responses before outstation will send them

### Pattern 4: Class-Based Event Poll

**Scenario**: Master requests events by priority class

```
Master                                    Outstation
  │                                            │
  │  READ (FC=1)                               │
  │  Request: Class 1 Events (High Priority)   │
  │  Object: Group 60 Var 2                    │
  │  FIR=1, FIN=1, CON=1, SEQ=10              │
  │───────────────────────────────────────────>│
  │                                            │
  │                                            │ (Retrieve Class 1 events)
  │                                            │
  │  RESPONSE (FC=129)                         │
  │  Contains: Multiple event objects          │
  │  - Binary Input Event (G2 V2)             │
  │  - Analog Input Event (G32 V3)            │
  │  IIN 2.2 = 1 (more events available)      │
  │  FIR=1, FIN=1, CON=0, SEQ=10              │
  │<───────────────────────────────────────────│
  │                                            │
  │  CONFIRM (FC=0), SEQ=10                    │
  │───────────────────────────────────────────>│
  │                                            │
  │                                            │ (Clear confirmed events)
  │                                            │
```

---

## Message Fragmentation

### Why Fragmentation is Needed

- Application messages can be large (data from many points)
- Transport layer limits segments to **250 octets**
- Data Link layer limits frames to **292 octets** (including overhead)
- Large messages must be split into multiple fragments

### Fragmentation Process

```
Large Application Message (e.g., 800 octets)
            ↓
┌───────────────────────────────────────────────┐
│        FRAGMENT 1 (250 octets)                │
│  FIR=1, FIN=0, SEQ=5                          │
└───────────────────────────────────────────────┘
            ↓
┌───────────────────────────────────────────────┐
│        FRAGMENT 2 (250 octets)                │
│  FIR=0, FIN=0, SEQ=5                          │
└───────────────────────────────────────────────┘
            ↓
┌───────────────────────────────────────────────┐
│        FRAGMENT 3 (250 octets)                │
│  FIR=0, FIN=0, SEQ=5                          │
└───────────────────────────────────────────────┘
            ↓
┌───────────────────────────────────────────────┐
│        FRAGMENT 4 (50 octets)                 │
│  FIR=0, FIN=1, SEQ=5                          │
└───────────────────────────────────────────────┘
```

**Fragment Control Bits**:
- **First Fragment**: FIR=1, FIN=0
- **Middle Fragments**: FIR=0, FIN=0
- **Last Fragment**: FIR=0, FIN=1
- **Single Fragment Message**: FIR=1, FIN=1

**Sequence Number**:
- All fragments of the same message use the **same sequence number**
- Receiver reassembles fragments using sequence number matching

---

## Object Headers and Data Objects

### Object Header Structure

Every data object group in a message has a header:

```
┌──────────────────────────────────────────────────┐
│  Octet 0: Group Number                           │
│  (Identifies object type, e.g., Binary Input)    │
├──────────────────────────────────────────────────┤
│  Octet 1: Variation Number                       │
│  (Identifies specific format/size)               │
├──────────────────────────────────────────────────┤
│  Octet 2: Qualifier Code                         │
│  (Specifies range/count format)                  │
├──────────────────────────────────────────────────┤
│  Octets 3+: Range Field                          │
│  (Start/Stop indexes, count, etc.)               │
├──────────────────────────────────────────────────┤
│  Data Objects (variable length)                  │
│  • Object 1                                      │
│  • Object 2                                      │
│  • ...                                           │
└──────────────────────────────────────────────────┘
```

### Common Qualifier Codes

| Code | Hex  | Range Specifier        | Use Case                                      |
|------|------|------------------------|-----------------------------------------------|
| 0    | 0x00 | 1-octet start-stop     | Request/response for sequential points 0-255  |
| 1    | 0x01 | 2-octet start-stop     | Request/response for sequential points        |
| 6    | 0x06 | No range (all points)  | Request all points of this type               |
| 7    | 0x07 | 1-octet count          | Limited quantity or single non-indexed value  |
| 8    | 0x08 | 2-octet count          | Limited quantity of objects                   |
| 23   | 0x17 | 1-octet count + index  | Non-sequential points (events)                |
| 40   | 0x28 | 2-octet count + index  | Non-sequential points (events)                |
| 91   | 0x5B | Variable length        | File transfer, variable-size objects          |

### Example: Reading Binary Inputs 0-7

**Master Request**:
```
Application Control:  0xC4 (FIR=1, FIN=1, CON=1, SEQ=4)
Function Code:        0x01 (READ)
Object Header:
  Group:              0x01 (Binary Input)
  Variation:          0x02 (Binary Input with Status)
  Qualifier:          0x00 (1-octet start-stop)
  Start Index:        0x00 (Point 0)
  Stop Index:         0x07 (Point 7)
```

**Outstation Response**:
```
Application Control:  0xC4 (FIR=1, FIN=1, CON=0, SEQ=4)
Function Code:        0x81 (RESPONSE)
IIN:                  0x0000 (No issues)
Object Header:
  Group:              0x01 (Binary Input)
  Variation:          0x02 (Binary Input with Status)
  Qualifier:          0x00 (1-octet start-stop)
  Start Index:        0x00
  Stop Index:         0x07
Data Objects (8 objects):
  Object 0:           0x81 (ON, quality=ONLINE)
  Object 1:           0x01 (OFF, quality=ONLINE)
  Object 2:           0x81 (ON, quality=ONLINE)
  Object 3:           0x01 (OFF, quality=ONLINE)
  Object 4:           0x81 (ON, quality=ONLINE)
  Object 5:           0x01 (OFF, quality=ONLINE)
  Object 6:           0x81 (ON, quality=ONLINE)
  Object 7:           0x81 (ON, quality=ONLINE)
```

---

## Common Message Exchanges

### 1. Integrity Poll (Complete Data Snapshot)

**Purpose**: Obtain all current values from outstation

```
Master Request:
  Function: READ (1)
  Objects:
    - Group 1 Var 2 Qualifier 0x06 (All Binary Inputs)
    - Group 30 Var 5 Qualifier 0x06 (All Analog Inputs)
    - Group 20 Var 5 Qualifier 0x06 (All Counters)

Outstation Response:
  Function: RESPONSE (129)
  Objects:
    - All Binary Input values with status
    - All Analog Input values
    - All Counter values
  IIN: Current status flags
```

### 2. Event Poll (Change-of-State Data)

**Purpose**: Retrieve only changed values since last poll

```
Master Request:
  Function: READ (1)
  Objects:
    - Group 60 Var 2 (Class 1 Events)
    - Group 60 Var 3 (Class 2 Events)
    - Group 60 Var 4 (Class 3 Events)

Outstation Response:
  Function: RESPONSE (129)
  Objects:
    - Binary Input Event objects (G2)
    - Analog Input Event objects (G32)
    - Counter Event objects (G22)
    - Each with timestamp
  IIN: May indicate more events buffered
```

### 3. Time Synchronization

**Purpose**: Set outstation clock to master time

```
Sequence:
1. Master sends DELAY_MEASURE request
2. Outstation responds immediately
3. Master calculates round-trip delay
4. Master sends WRITE with current time + (delay/2)
5. Outstation sets its clock
```

### 4. Enable Unsolicited Reporting

**Purpose**: Allow outstation to send events without polling

```
Master Request:
  Function: ENABLE_UNSOLICITED (20)
  Objects:
    - Group 60 Var 2 (Class 1 Events enabled)
    - Group 60 Var 3 (Class 2 Events enabled)
    - Group 60 Var 4 (Class 3 Events enabled)

Outstation Response:
  Function: RESPONSE (129)
  Status: Success

Future Event Occurrence:
  Outstation sends UNSOLICITED_RESPONSE (130)
  with event data
  Master sends CONFIRM (0)
```

---

## Summary

### Key Takeaways

1. **Three Message Types**: Request, Response, Confirmation
2. **Function Codes**: Define message purpose (0-128 for requests, 129-255 for responses)
3. **Sequence Numbers**: Track messages and detect duplicates
4. **Fragmentation**: Large messages split into 250-octet segments
5. **Object-Oriented**: Data organized by Group/Variation with flexible addressing
6. **Confirmation**: Critical operations require explicit acknowledgment
7. **Event-Driven**: Supports both polling and unsolicited reporting
8. **Status Reporting**: IIN bits convey outstation health and issues

### Message Flow Best Practices

- **Poll Integrity Periodically**: Get complete snapshot (e.g., every 5 minutes)
- **Poll Events Frequently**: Check for changes (e.g., every 1-5 seconds)
- **Use Unsolicited for Critical Events**: Immediate notification without polling delay
- **Always Confirm Unsolicited**: Prevents event buffer overflow
- **Use Select-Before-Operate**: For safety-critical controls
- **Monitor IIN Bits**: Detect device problems proactively
- **Implement Timeouts**: Retry on communication failures

---

**Related Documents**:
- [IEEE_1815_2012_SUMMARY.md](IEEE_1815_2012_SUMMARY.md) - Complete standard overview
- [ANALYSIS_SUMMARY.md](ANALYSIS_SUMMARY.md) - Malware analysis document

**Standard Reference**: IEEE Std 1815-2012, Clauses 4-5 (Application Layer specifications)
