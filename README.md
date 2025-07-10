# Wiscreation ASN.1 API Service

## 1. ASN.1 History - The Universal Protocol Language

### 1.1 The Birth of a Universal Data Language

> "Imagine every country speaking a different language - that was computer networks in the 1980s."
>

**The Problem:** In the early 1980s, networks exploded with incompatible systems (email servers, routers, databases). Each used unique data formats, causing chaos.

**The Solution:** ASN.1 (Abstract Syntax Notation One) was created in 1984 by ITU-T and ISO as a universal data language. It became foundational for:

| Domain     | Protocols           | ASN.1's Role                          | Real-World Example           |
| ---------- | ------------------- | ------------------------------------- | ---------------------------- |
| Networking | SNMP, LDAP, 5G NR   | Defines messages like "get CPU usage" | Monitoring 10,000 routers    |
| Security   | X.509 (SSL/TLS)     | Encodes digital certificates          | HTTPS website encryption     |



- Telecom (SS7 call routing)
- Internet (SNMP/LDAP)
- Security (X.509 certificates)

### 1.2 Real-World Applications

| Domain     | Protocols           | ASN.1's Role                          | Real-World Example           |
| ---------- | ------------------- | ------------------------------------- | ---------------------------- |
| Networking | SNMP, LDAP, 5G NR   | Defines messages like "get CPU usage" | Monitoring 10,000 routers    |
| Security   | X.509 (SSL/TLS)     | Encodes digital certificates          | HTTPS website encryption     |
| Telecom    | SS7, SMS, 4G/5G NAS | Routes calls/texts                    | Sending SMS between carriers |

### 1.3 ASN.1 in Telecom: The Invisible Backbone

**Protocol Stack Diagram:**

| Layer        | Telecom Protocol   | ASN.1 Role                |
| ------------ | ------------------ | ------------------------- |
| Application  | SMS, Call Control  | Defines message structure |
| Presentation | ASN.1              | Translates data to binary |
| Network      | SS7, Diameter      | Routes messages           |
| Physical     | Fiber, Radio Waves | Bit transmission          |

**Telecom Protocol Evolution:**

| Generation | Key Protocols  | ASN.1 Application                  |
| ---------- | -------------- | ---------------------------------- |
| 2G/3G      | GSM MAP, RANAP | SMS delivery, call handovers       |
| 4G LTE     | S1AP, NAS      | Base station handovers             |
| 5G NR      | NGAP, F1AP     | Network slicing, ultra-low latency |

*Example:* When your phone switches towers during a call, ASN.1-encoded `HandoverCommand` messages (4G S1AP/5G NGAP) manage the transition.

---

## 2. The Three Pillars of ASN.1

### 2.1 Types

Types define the architecture of protocol messages. They enforce data validity rules while remaining implementation-agnostic.

**Analogy: Computer Manufacturing**

***Primitive Types*** = Component Properties:

- `REAL` — floating-point numbers
- `INTEGER` — whole numbers
- `ENUMERATED` — named options
- `UTF8String` — Unicode text
- `OCTET STRING` — byte sequences

***Structured Types*** = Components:

```asn1
CPU-Component ::= SEQUENCE {
  brand      UTF8String,
  frequency  REAL (1.0..6.0),
  cores      INTEGER (2..128)
}

RAM-Component ::= SEQUENCE {
  brand  UTF8String,
  size   INTEGER (1..256),
  type   ENUMERATED {DDR4, DDR5},
  speed  INTEGER (1600..6400)
}

Motherboard-Component ::= SEQUENCE {
  model     UTF8String,
  cpuSocket CPU-Component,
  ramSlots  SEQUENCE SIZE (4) OF RAM-Component
}
```

**Telecom Example(5G Handover Component):**

```asn1
HandoverCommand ::= SEQUENCE {
  targetCellID  OCTET STRING (SIZE(6)),
  encryptionAlg ENUMERATED {aes256, snow3g},
  urgencyLevel  INTEGER (1..3)
}
```

### 2.2 Values

Values are concrete protocol messages — instances built from type blueprints.

**Computer Manufacturing Analogy:**
Values are physical components manufactured from blueprints:

```asn1
corsairRAM RAM-Component ::= {
  brand = "Corsair Dominator",
  size  = 64,
  type  = DDR5,
  speed = 5600
}

ryzenCpu CPU-Component ::= {
  brand     = "AMD Ryzen 9 7950X",
  frequency = 5.7,
  cores     = 16
}

asusBoard Motherboard-Component ::= {
  model     = "ASUS ProArt X670E",
  cpuSocket = ryzenCpu,
  ramSlots  = { corsairRAM, corsairRAM }
}
```

**Telecom Value:**

```asn1
liveHandover HandoverCommand ::= {
  targetCellID = '5C3A91'H,
  encryptionAlg = snow3g,
  urgencyLevel  = 2
}
```

### 2.3 Modules

Modules are container units that organize related protocol definitions - like specialized
engineering manuals for different telecom subsystems. Just as a 5G network separates
radio control (RRC) from core network signaling (NGAP), modules compartmentalize
specifications to avoid conflicts and enable team collaboration.

**Computer Manufacturing Analogy:**

> "Think of modules as computer specification documents (e.g., 'Dell Precision Workstation
Technical Manual'). Each manual defines:
1. Compatible components (structured types)
2. Their properties (primitive types)
3. Assembly rules (imports/exports as needed)

```asn1
High-Performance-Workstation DEFINITIONS ::= BEGIN
  CPU-Component ::= SEQUENCE { -- STRUCTURED TYPE
    brand UTF8String, -- Manufacturer
    frequency REAL (1.0..6.0), -- Constrained REAL property
    cores INTEGER (2..128) -- Core count
  }

  RAM-Component ::= SEQUENCE { -- STRUCTURED TYPE
    brand UTF8String, -- Manufacturer
    size INTEGER(1..256), -- RAM size 1-256 GB
    type ENUMERATED {DDR4, DDR5}, -- Constrained ENUMERATED p
    roperty
    speed INTEGER (1600..6400) -- MHz constraint
  }

  Motherboard-Component ::= SEQUENCE { -- STRUCTURED TYPE
    model UTF8String,
    cpuSocket CPU-Component, -- Nested component
    ramSlots SEQUENCE SIZE (4) OF RAM-Component -- Array
  }

  -- RAM COMPONENT INSTANCES
  corsairRAM RAM-Component ::= { -- VALUE
    brand = "Corsair Dominator",
    size = 64, -- Valid 1-256 GB
    type = DDR5,
    speed = 5600 -- Valid 1600-6400 MHz
  }

  -- CPU COMPONENT INSTANCE
  ryzenCpu CPU-Component ::= { -- VALUE
    brand = "AMD Ryzen 9 7950X",
    frequency = 5.7, -- Valid 1.0-6.0 GHz
    cores = 16 -- Valid 2-128 cores
  }
    -- MOTHERBOARD WITH INSTALLED COMPONENTS
  asusBoard Motherboard-Component ::= { -- VALUE
    model = "ASUS ProArt X670E",
    cpuSocket = ryzenCpu, -- Component instance
    ramSlots = { corsair32gb, corsair32gb } -- RAM instances
  }
END
```

An example of how 3GPP defines separate modules for RRC (radio components), NAS
(security components), and NGAP (core network assembly):

```asn1
5G-Radio-Components DEFINITIONS ::= BEGIN
    EXPORTS Antenna-Array, FrequencyBand; -- Public components
    IMPORTS EncryptionKey FROM Security-Components; -- Cross-module part
    -- Component definitions
    Antenna-Array ::= SEQUENCE { ... } -- Structured type
    FrequencyBand ::= INTEGER (1..100) -- Primitive property
END
```

### 2.4 How Do Things Work Together

**Workflow:**

1. Design schema:

```asn1
SMS-Deliver ::= SEQUENCE {
  sender    OCTET STRING (SIZE(12)),
  timestamp GeneralizedTime,
  message   UTF8String (SIZE (0..160))
}
```

2. Encode (PER, BER)
3. Transmit over network
4. Decode with matching schema


```python
decoded_sms = PER.decode(encoded_binary, SMS-Deliver)
print(decoded_sms.message)  # Output: "Hello"
```

---

## 3. Mastering ASN.1 Types

### 3.1 Primitive Types (Atoms)

#### 3.1.1 BOOLEAN Type

BOOLEAN is a primitive ASN.1 type with universal tag `[UNIVERSAL 1]`. It represents logical values with exactly two states: TRUE or FALSE. Used for flags, presence indicators, and binary conditions.

**Basic Examples of Type Definition:**

```asn1
BooleanExample DEFINITIONS ::= BEGIN
  Enabled ::= BOOLEAN  -- Basic declaration
END
```

**Constraint Considerations:**

```asn1
ConstraintedBooleanExample DEFINITIONS ::= BEGIN
  FixedTrue ::= BOOLEAN (TRUE)
  FixedFalse ::= BOOLEAN (FALSE)
END
```

| Constraint      | Valid Value | Invalid Value |
| --------------- | ----------- | ------------- |
| BOOLEAN (TRUE)  | TRUE        | FALSE         |
| BOOLEAN (FALSE) | FALSE       | TRUE          |

**Step-by-Step Encoding of `TRUE`:**

**Basic Encoding Rules**:
| Identifier octets | Length octets | Contents octets   | End-of-content Octets |
| ----------------- | ------------- | ------------------| ---------------- |

The encoding of a data value shall consist of three or four components, which appear in the order
of Identifier octets, Length octets, Contents octets and End-of-content Octets (optional).

**Identifier**: `0x01` (UNIVERSAL 1 = BOOLEAN)
1. Class: Boolean is Universal type = 00b
2. P/C: Boolean is Primitive type = 0b
3. Tag: Boolean type’s universal tag is 1 = 00001b
4. Identifier Octet: 00 0 00001b (`0x01`)
<table>
  <tr> <td>8</td> <td>7</td> <td>6</td> <td>5</td> <td>4</td> <td>3</td> <td>2</td> <td>1</td> </tr>
  <tr> <td colspan="2">Class<br>0 0 : Universal<br>0 1 : Application <br>1 0 : Context Specific<br> 1 1 : Private</td> <td>P/C<br>0: Primitive<br>1: Constructed</td>  <td colspan="5">Tag Number</td> </tr>
  <tr> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>1</td> </tr>
</table>

**Length Octet**:
Value always occupies 1 byte → `0x01`

**Contents Octets**:
`TRUE` = `0xFF` (any non-zero byte accepted, but 0xFF is standard)

**→ Full BER Encoding**: `01 01 FF`

**Master encoding hands-on**: Follow the steps below to [try it yourself](https://wiscreationsoft.com/apps/asn1codec) (click 'Login' to
start)

![3.1.1](/docs/images/3.1.1.png)
---

#### 3.1.2 INTEGER Type

INTEGER is a primitive ASN.1 type with universal tag `[UNIVERSAL 2]`. It represents whole numbers and supports constraints.

**Basic Examples of Type Definition:**

```asn1
IntegerExample DEFINITIONS ::= BEGIN
  Temperature ::= INTEGER
END
```

**Constraint Examples:**

| Constraint Type | Syntax           | Valid Values | Invalid Values |
| --------------- | ---------------- | ------------ | -------------- |
| Value Range     | INTEGER (0..100) | 0, 50, 100   | -1, 101        |
| Single Value    | INTEGER (42)     | 42           | 41, 43         |
| Telecom Example | RSRP-Value ::= INTEGER (-140..-44)     | -86           | 41, 43         |

```asn1
ConstraintedIntegerExample DEFINITIONS ::= BEGIN
  Temperature ::= INTEGER (0..100)
  Age ::= INTEGER (42)
END
```

**Step-by-Step Encoding of 30:**
**Identifier**: `0x02` (UNIVERSAL 2 = INTEGER))
1. Class: Integer is Universal type = 00b
2. P/C: Integer is Primitive type = 0b
3. Tag: Integer type’s universal tag is 2 = 00010b
4. Identifier Octet: 00 0 00010b (`0x02`)
<table>
  <tr> <td>8</td> <td>7</td> <td>6</td> <td>5</td> <td>4</td> <td>3</td> <td>2</td> <td>1</td> </tr>
  <tr> <td colspan="2">Class<br>0 0 : Universal<br>0 1 : Application <br>1 0 : Context Specific<br> 1 1 : Private</td> <td>P/C<br>0: Primitive<br>1: Constructed</td>  <td colspan="5">Tag Number</td> </tr>
  <tr> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>1</td> <td>0</td> </tr>
</table>

**Length Octet**:
Value always occupies 1 byte → `0x01`

**Contents Octets**:
`TRUE` = `0x1E` (hex for decimal 30)

**→ Full BER Encoding**: `02 01 1E`

**Master encoding hands-on**: Follow the steps below to [try it yourself](https://wiscreationsoft.com/apps/asn1codec) (click 'Login' to
start)

![3.1.2](/docs/images/3.1.2.png)
---

#### 3.1.3 BIT STRING Type

BIT STRING is a primitive ASN.1 type with universal tag `[UNIVERSAL 3]`. It represents
binary data as a sequence of bits (0s and 1s). Useful for flags, configurations, and compact signaling.

**Basic Examples of Type Definition:**

```asn1
BitStringExample DEFINITIONS ::= BEGIN
  Flags ::= BIT STRING  -- Basic declaration
  AccessFlags ::= BIT STRING {
    admin(0),
    user-read(1),
    user-write(2)
  }
END
```

**Constraint Examples:**

| Constraint Type | Syntax                         | Valid Values | Invalid Values     |
| --------------- | ------------------------------ | ------------ | ------------------ |
| Size            | BIT STRING (SIZE(8))           | '11001100'B  | '101'B (too short) |
| Named Bits      | BIT STRING {read(0), write(1)} | {read}       | {execute}          |
| Telecom Example | RRC-Config ::= BIT STRING (SIZE(5)) | '11001'B | '11001100'B       |


```asn1
ConstraintedBitStringExample DEFINITIONS ::= BEGIN
    PCI-Config ::= BIT STRING (SIZE(5)) -- Fixed 5-bit ID
END
```

**Step-by-Step Encoding of **`'11001'B`**:**
```JSON
{
"length": 5,
"value": "C8"
}
```
**Note:** To encode ASN.1 bit string '11001'B: 'C8' is the hex representation of '`11001`000'B (5-bit
value padded to 8 bits), where 'length' denotes actual bit count.


**Identifier**: `0x03` (UNIVERSAL 3 = BIT STRING)
1. Class: BIT STRING is Universal type = 00b
2. P/C: BIT STRING is Primitive type = 0b
3. Tag: BIT STRING type’s universal tag is 3 = 00011b
4. Identifier Octet: 00 0 00011b (`0x03`)
<table>
  <tr> <td>8</td> <td>7</td> <td>6</td> <td>5</td> <td>4</td> <td>3</td> <td>2</td> <td>1</td> </tr>
  <tr> <td colspan="2">Class<br>0 0 : Universal<br>0 1 : Application <br>1 0 : Context Specific<br> 1 1 : Private</td> <td>P/C<br>0: Primitive<br>1: Constructed</td>  <td colspan="5">Tag Number</td> </tr>
  <tr> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>1</td> <td>1</td> </tr>
</table>

**Length Octet**: Total contents length = 2 octets (1 unused bits count + 1 value octet) `0x02`

**Content Octets**:
1. Unused Bits Count: '11001'B has 5 bits → 3 unused bits in last octet → 0x03
2. Value: Original `11001` is padded to octet: `11001000` (append 3 zeros) → `0xC8`(hex)

**→ Full BER Encoding**: `03 02 03 C8`

**Master encoding hands-on**: Follow the steps below to [try it yourself](https://wiscreationsoft.com/apps/asn1codec) (click 'Login' to
start)

![3.1.3](/docs/images/3.1.3.png)
---


#### 3.1.4 OCTET STRING Type

OCTET STRING is a primitive ASN.1 type with universal tag `[UNIVERSAL 4]`. It represents a sequence of octets (bytes) of arbitrary length, used for binary data like encryption keys or file payloads.

**Basic Example:**

```asn1
OctetStringExample DEFINITIONS ::= BEGIN
  HashValue ::= OCTET STRING
END
```

**Constraint Examples:**

| Constraint Type | Syntax                      | Valid Values        | Invalid Values  |
| --------------- | --------------------------- | ------------------- | --------------- |
| Fixed Size      | OCTET STRING (SIZE(8))      | 'A1B2C3D4E5F60718'H | 'A1B2'H         |
| Range Size      | OCTET STRING (SIZE(1..128)) | '112233'H           | 129-byte string |
| RTelecom Example| 5G-Key ::= OCTET STRING (SIZE(32)) | 32-byte key  | 16-byte key     |


```asn1
ConstraintedOctetStringExample DEFINITIONS ::= BEGIN
  IV ::= OCTET STRING (SIZE(2)) -- 2-byte Initialization Vector
  Salt ::= OCTET STRING (SIZE(8..16)) -- Salt of variable length
END
```

**Step-by-Step Encoding of**`'A1B2'H`

**Identifier**: `0x04` (UNIVERSAL 4 = OCTET STRING)
1. Class: OCTET STRING is Universal type = 00b
2. P/C: OCTET STRING is Primitive type = 0b
3. Tag: OCTET STRING type’s universal tag is 4 = 00100b
4. Identifier Octet: 00 0 00100b (`0x04`)
<table>
  <tr> <td>8</td> <td>7</td> <td>6</td> <td>5</td> <td>4</td> <td>3</td> <td>2</td> <td>1</td> </tr>
  <tr> <td colspan="2">Class<br>0 0 : Universal<br>0 1 : Application <br>1 0 : Context Specific<br> 1 1 : Private</td> <td>P/C<br>0: Primitive<br>1: Constructed</td>  <td colspan="5">Tag Number</td> </tr>
  <tr> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>1</td> <td>0</td> <td>0</td> </tr>
</table>

**Length Octet**: Total contents length = 2 octets → `0x02`

**Content Octets**:
`0xA1 0xA2` → `0xA1A2`


**→ Full BER Encoding**: `04 02 A1 B2`

**Master encoding hands-on**: Follow the steps below to [try it yourself](https://wiscreationsoft.com/apps/asn1codec) (click 'Login' to start)

![3.1.4](/docs/images/3.1.4.png)
---
#### 3.1.5 OBJECT IDENTIFIER Type
OBJECT IDENTIFIER (OID) is a primitive ASN.1 type with universal tag `[UNIVERSAL 6]`. It represents a globally unique ID in a hierarchical namespace tree.

**Basic Declaration:**

```asn1
ObjectIdentifierExample DEFINITIONS ::= BEGIN
  RSA-Algorithm ::= OBJECT IDENTIFIER
END
```

**OID Tree Example:**
OIDs form a hierarchical namespace
```text
1 - ISO
 └── 3 - Identified Organization
      └── 6 - US DoD
           └── 1 - Internet
                └── 4 - Private
                     └── 1 - Enterprise
```

**Constraint Examples:**

| Constraint Type | Syntax        | Valid Values | Invalid Values |
| --------------- | ------------- | ------------ | -------------- |
| Fixed OID       | {1 3 6 1}     | 1.3.6.1      | 1.3.6.2        |
| OID Prefix      | {1 3 6 1 5}.. | 1.3.6.1.5.9  | 1.3.14         |

```asn1
ConstrainedOIDExample DEFINITIONS ::= BEGIN
  SNMP-Root ::= OBJECT IDENTIFIER ( {1 3 6 1} ) -- Fixed OID
  ETSI-Branch ::= OBJECT IDENTIFIER ( {0 4 0}..) -- Must start with 0.4.0
END
```

**Encoding **``**:**
**Step-by-Step Encoding of:** `1.3.6.1.4.1`


**Identifier**: `0x06` (UNIVERSAL 6 = OBJECT IDENTIFIER))
1. Class: OBJECT IDENTIFIER) is Universal type = 00b
2. P/C: OBJECT IDENTIFIER) is Primitive type = 0b
3. Tag: OBJECT IDENTIFIER) type’s universal tag is 6 = 00110b
4. Identifier Octet: 00 0 00110b (`0x06`)
<table>
  <tr> <td>8</td> <td>7</td> <td>6</td> <td>5</td> <td>4</td> <td>3</td> <td>2</td> <td>1</td> </tr>
  <tr> <td colspan="2">Class<br>0 0 : Universal<br>0 1 : Application <br>1 0 : Context Specific<br> 1 1 : Private</td> <td>P/C<br>0: Primitive<br>1: Constructed</td>  <td colspan="5">Tag Number</td> </tr>
  <tr> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> <td>1</td> <td>1</td> <td>0</td> </tr>
</table>

**Length Octet**: Calculate encoded value length = 5 octets → `0x05`

**Content Octets**:
1. First two arcs: `1.3` → 1*40 + 3 = 43 → `0x2B`
2. Subarcs: `6` → `0x06`  `1` → `0x01` `4` → `0x04` `1` → `0x01`
3. Full Value: 2B 06 01 04 01

**→ Full BER Encoding**: `06 05 2B 06 01 04 01`

**Master encoding hands-on**: Follow the steps below to [try it yourself](https://wiscreationsoft.com/apps/asn1codec) (click 'Login' to
start)

![3.1.5](/docs/images/3.1.5.png)
---

### 3.2 Structured Types

#### 3.2.1 SEQUENCE

SEQUENCE is a structured ASN.1 type with universal tag `[UNIVERSAL 16]`. It represents an ordered collection of heterogeneous elements. Think of it like a C `struct` — field order matters.

**Basic Examples of Type Definition:**

```asn1
SequenceExample DEFINITIONS ::= BEGIN
  HandoverCommand ::= SEQUENCE {
    transactionId  INTEGER (0..255),
    targetCell     OCTET STRING (SIZE(6)),
    algorithm      OBJECT IDENTIFIER,
    parameters     SEQUENCE {
      encryptionKey  OCTET STRING (SIZE(32)),
      integrityKey   OCTET STRING (SIZE(32))
    },
    urgent         BOOLEAN DEFAULT FALSE
  }
END
```

**Step-by-Step Encoding Example Using **`handoverCmd`** Value:**

```JSON
{
  "transactionId": 42,
  "targetCell": "A1B2C3D4E5F6",
  "algorithm": "0.4.0.127.0.7.1.1.4",
  "parameters": {
    "encryptionKey": "4F3C2B7E151628AED2A6ABF7158809CF4F3C2B7E151628AED2A6BF7158809CF4",
    "integrityKey": "8E9F0A1B2C3D4E5F60718293A4B5C6D7E8F901A2B3C4D5E6F785C6D7E8F901A0"
  }
}

```
**Identifier**: `0x30` (UNIVERSAL 16 = SEQUENCE))
1. Class: SEQUENCE) is Universal type = 00b
2. P/C: SEQUENCE) is Constructed type = 1b
3. Tag: SEQUENCE) type’s universal tag is 16 = 10000b
4. Identifier Octet: 00 1 10000b (`0x30`)
<table>
  <tr> <td>8</td> <td>7</td> <td>6</td> <td>5</td> <td>4</td> <td>3</td> <td>2</td> <td>1</td> </tr>
  <tr> <td colspan="2">Class<br>0 0 : Universal<br>0 1 : Application <br>1 0 : Context Specific<br> 1 1 : Private</td> <td>P/C<br>0: Primitive<br>1: Constructed</td>  <td colspan="5">Tag Number</td> </tr>
  <tr> <td>0</td> <td>0</td> <td>1</td> <td>1</td> <td>0</td> <td>0</td> <td>0</td> <td>0</td> </tr>
</table>

**Length Octet**: Sum of encoded fields: 3 (INT) + 8 (OCTSTR) + 10 (OID) + 70 (nested SEQ) + 3 (bool)
= 94 bytes → `0x5e`

**Content Octets**:
1. transactionId (42): 02 01 2A
2. targetCell: 04 06 A1 B2 C3 D4 E5 F6
3. algorithm (OID): 06 08 04 00 7F 00 07 01 01 04
4. parameters (nested SEQUENCE):
```text
30 44 -- Nested SEQUENCE tag/length (64 bytes)
04 20 -- encryptionKey
4F3C...09CF
04 20 -- integrityKey
8E9F...F78
```
5. Urgent: 01 01 00


**→ Full BER Encoding**:
```text
30 5e -- SEQUENCE tag/length
   02 01 2a -- "transactionId"
   04 06 a1 b2 c3 d4 e5 f6 -- "targetCell"
   06 08 04 00 7f 00 07 01 01 04 -- "algorithm"
   30 44 -- "parameters"(Nested SEQUENCE)
      04 20 4f 3c 2b 7e 15 16 28 ae d2 a6 ab f7 15 88 09 cf
            4f 3c 2b 7e 15 16 28 ae d2 a6 bf 71 58 80 9c f4 -- "encryptionKey"
      04 20 8e 9f 0a 1b 2c 3d 4e 5f 60 71 82 93 a4 b5 c6 d7
            e8 f9 01 a2 b3 c4 d5 e6 f7 85 c6 d7 e8 f9 01 a0 -- "integrityKey"
   01 01 00 -- "urgent": default False
```

**Master encoding hands-on**: Follow the steps below to [try it yourself](https://wiscreationsoft.com/apps/asn1codec) (click 'Login' to start)

![3.2.1](/docs/images/3.2.1.png)


---

## 4. The Power of Values

### 4.1 Value Assignment Syntax

```asn1
authVector5G ::= SEQUENCE {
  rand 'A3F209D4B5C6E7F8'H,
  xres '89F2C4A6'H,
  autn '51BF6084D9810A3B'H
}
```

### 4.2 Constraints = Validation Rules

```asn1
PCI-Value ::= INTEGER (0..503)
validPCI PCI-Value ::= 301
```

### 4.3 Telecom Value Patterns

```asn1
CellConfig ::= SEQUENCE {
  cellId    [0] INTEGER (1..65535),
  freqBand [1] INTEGER (1..256) DEFAULT 78
}

siteA-Cell3 CellConfig ::= {
  cellId 143
  -- freqBand uses default 78
}
```

---

## 5. Encoding Rules - The Wire Format

### 5.1 BER vs. PER in Telecom

| Encoding | Use Case         | Size Comparison         |
| -------- | ---------------- | ----------------------- |
| BER      | X.509 certs      | 42 bytes for INTEGER 30 |
| PER      | 5G RRC signaling | 1 byte for INTEGER 30   |

### 5.2 PER Optimization

```asn1
RRC-Short ::= SEQUENCE {
  msgType INTEGER (0..3),
  urgent  BOOLEAN,
  padding BIT STRING (SIZE(5)) OPTIONAL
}
```

Encodes in 1 byte (vs. 8+ in BER)

---

## Appendix: ASN.1 Specification Library

### X.680 Series (Core Specs)

| Standard | Focus               | Key Feature                  |
| -------- | ------------------- | ---------------------------- |
| X.680    | Basic notation      | Type/value definition syntax |
| X.681    | Information objects | Protocol extensibility       |
| X.682    | Constraints         | Value validation rules       |
| X.683    | Parameterization    | Template-based reuse         |

### X.690 Series (Encoding)

| Standard | Encoding      | Primary Use                   |
| -------- | ------------- | ----------------------------- |
| X.690    | BER, CER, DER | BER, CER, DER Encoding Rules  |
| X.691    | PER           | Packed Encoding Rules         |
| X.692    | ECN           | Encoding Control Notation     |
| X.693    | XER           | XML Encoding Rules            |
| X.694    | XML           | XML schema definitions        |
| X.695    | PER           | Registration and application of PER|
| X.696    | OER           | Octet Encoding Rules          |
| X.697    | JER           | JSON Encoding Rules           |
