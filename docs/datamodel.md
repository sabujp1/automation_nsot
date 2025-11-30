এবার আসি Nautobot-এর ডেটা মডেলে। এটা বোঝাটা খুব জরুরি, কারণ এটাই ঠিক করে দেয় আপনি কীভাবে আপনার নেটওয়ার্ক ডেটা সাজাবেন।

Nautobot-এর ডেটা মডেল কয়েকটা মূল ক্যাটাগরিতে ভাগ করা:

### Organization - কে কী করবে

প্রথম ক্যাটাগরি হলো Organization। এখানে আছে:

**Tenants (টেন্যান্ট):** ধরুন আপনার আইএসপি একাধিক ব্র্যান্ড অপারেট করে। অথবা আপনার কিছু এন্টারপ্রাইজ ক্লায়েন্ট আছে যাদের ডেডিকেটেড নেটওয়ার্ক আছে। এদের আলাদা করে রাখার জন্য Tenant ইউজ করা হয়। উদাহরণ: "Residential Customers", "Corporate Clients", "Government Sector"।

**Teams (টিম):** আপনার অর্গানাইজেশনের বিভিন্ন টিম। যেমন "Network Operations", "Field Technicians", "NOC Team"। প্রতিটা টিমের আলাদা পার্মিশন থাকতে পারে।

**Roles (রোল):** ডিভাইস বা সার্কিটের ভূমিকা। যেমন একটা রাউটার "Core Router" রোলে থাকতে পারে, আরেকটা "Distribution Router" রোলে। এতে সহজে ফিল্টার করা যায়।

একটা আইএসপির উদাহরণ দিই। ধরুন "SkyNets Bangladesh" নামে একটা আইএসপি। তাদের দুটো ব্র্যান্ড - "SkyNets Home" (রেসিডেনশিয়াল) আর "SkyNets Business" (কর্পোরেট)। Nautobot-এ তারা দুটো টেন্যান্ট বানাবে। তাদের তিনটা টিম - NOC, Field Ops, আর Management। আর তাদের ডিভাইসগুলোর কয়েকটা রোল - Core, Distribution, Access, Customer Premises Equipment (CPE)।

### DCIM - ফিজিক্যাল নেটওয়ার্ক

DCIM মানে Data Center Infrastructure Management। এখানে আপনার ফিজিক্যাল নেটওয়ার্কের সব কিছু থাকে।

**Sites (সাইট):** আপনার নেটওয়ার্কের ফিজিক্যাল লোকেশন। আইএসপির ক্ষেত্রে এগুলো হলো পপ (Point of Presence)। উদাহরণ: "মিরপুর পপ", "উত্তরা পপ", "ধানমন্ডি পপ"।

প্রতিটা সাইটের কিছু তথ্য থাকবে:
- নাম: "Mirpur POP"
- অ্যাড্রেস: "রোড ১০, মিরপুর-১২, ঢাকা"
- GPS Coordinates: (23.8103, 90.3654)
- Contact Person: "জাহিদ হাসান"
- Phone: +880 1712-345xxx

**Racks (র‍্যাক):** প্রতিটা পপে কয়েকটা র‍্যাক থাকে যেখানে ডিভাইস বসানো। র‍্যাকের তথ্য:
- নাম: "Rack-A1"
- Height: 42U
- Site: "Mirpur POP"

**Devices (ডিভাইস):** আপনার নেটওয়ার্ক ডিভাইস - রাউটার, সুইচ, OLT, ONU ইত্যাদি।

একটা কংক্রিট উদাহরণ দিই। ধরুন মিরপুর পপে একটা কোর রাউটার আছে:

```bash
Device Name: R-MIR-CORE-01
Device Type: Cisco ISR 4431
Role: Core Router
Site: Mirpur POP
Rack: Rack-A1
Position: 20U (মানে র‍্যাকের ২০ নাম্বার ইউনিট থেকে শুরু)
Status: Active
Serial Number: FTX1234ABCD
Asset Tag: SN-RTR-001
```

**Interfaces (ইন্টারফেস):** প্রতিটা ডিভাইসের পোর্ট বা ইন্টারফেস। উদাহরণ:

```bash
Device: R-MIR-CORE-01
Interface Name: GigabitEthernet0/0/0
Type: 1000BASE-T
Enabled: Yes
MTU: 1500
Description: "Uplink to ColoAsia"
```

**Cables (ক্যাবল):** কোন ডিভাইসের কোন ইন্টারফেস কোন ডিভাইসের কোন ইন্টারফেসে কানেক্টেড সেটা ট্র্যাক করার জন্য ক্যাবল অবজেক্ট। উদাহরণ:

```bash
Cable ID: CBL-001
Type: Fiber (Single-Mode)
Length: 50 meters
Termination A: R-MIR-CORE-01, Interface GigabitEthernet0/0/1
Termination B: SW-MIR-DIST-01, Interface GigabitEthernet0/1
Status: Connected
```

### IPAM - আইপি অ্যাড্রেস ম্যানেজমেন্ট

IPAM হলো IP Address Management। এখানে আপনার সব আইপি রিলেটেড ডেটা থাকবে।

**Namespaces:** আইপি নেমস্পেস হলো আইপি রেঞ্জের একটা কনটেইনার। সাধারণত একটাই নেমস্পেস থাকে "Global"। কিন্তু যদি আপনার ওভারল্যাপিং আইপি রেঞ্জ থাকে (যেমন একাধিক VPN), তাহলে আলাদা নেমস্পেস বানাতে পারেন।

**Prefixes (প্রিফিক্স):** এগুলো হলো আইপি সাবনেট। উদাহরণ:

```bash
Prefix: 10.10.0.0/16
Namespace: Global
Site: Mirpur POP
VLAN: VLAN 100
Status: Active
Description: "Mirpur Customer Network"
```

আমাদের আইএসপিদের জন্য একটা রিয়েল উদাহরণ দিই। ধরুন আপনার কাছে কয়েকটা লিংকের জন্য /22 পাবলিক আইপি ব্লক আছে: 103.92.40.0/22। এটাকে আপনি ভাগ করবেন:

```bash
Parent: 103.92.40.0/22 (Own Assigned)
  ├─ 103.92.40.0/24 (Residential Customers - Mirpur)
  ├─ 103.92.41.0/24 (Residential Customers - Uttara)
  ├─ 103.92.42.0/25 (Corporate Clients)
  └─ 103.92.42.128/25 (Infrastructure & Management)
```

**IP Addresses (আইপি অ্যাড্রেস):** ইন্ডিভিজুয়াল আইপি অ্যাড্রেস। উদাহরণ:

```
IP Address: 10.10.1.1/24
Namespace: Global
Status: Active
Role: Loopback
Assigned To: R-MIR-CORE-01, Interface Loopback0
DNS Name: r-mir-core-01.skynet.local
Description: "Core router management IP"
```

**VLANs (ভিল্যান):** Virtual LAN। উদাহরণ:

```
VLAN ID: 100
Name: MANAGEMENT
Site: Mirpur POP
Status: Active
Description: "Management VLAN for network devices"
```

একটা কমপ্লিট ভিল্যান স্ট্রাকচার দেখি আমাদের আইএসপির:

```
VLAN 10: INFRASTRUCTURE (Router-to-router links)
VLAN 20: MANAGEMENT (Device management)
VLAN 100-199: RESIDENTIAL_CUSTOMERS
VLAN 200-299: CORPORATE_CLIENTS
VLAN 300: VOIP
VLAN 999: QUARANTINE (Problematic customers)
```

**VLAN Groups:** যদি আপনার একাধিক সাইটে একই ভিল্যান আইডি ইউজ করেন, তাহলে ভিল্যান গ্রুপ ইউজ করতে পারেন কনফ্লিক্ট এড়াতে।

### Circuits - আপলিংক ম্যানেজমেন্ট

Circuits হলো আপনার আপস্ট্রিম কানেক্টিভিটি - মানে যেসব লিংকের মাধ্যমে আপনি ইন্টারনেট পান।

**Providers (প্রোভাইডার):** আপনার আপস্ট্রিম প্রোভাইডার। বাংলাদেশি আইএসপির জন্য উদাহরণ:

```
Provider Name: BTCL
ASN: AS17494
Portal URL: https://isp.btcl.gov.bd
NOC Contact: +880 2-9555555
```

অন্যান্য প্রোভাইডার হতে পারে: Summit, Cybergate, F@H Global, বা আন্তর্জাতিক প্রোভাইডার অথবা আপনার IIG লাইসেন্স থাকে।

**Circuit Types:** সার্কিটের ধরন। যেমন:
- Fiber Optic
- Microwave Link
- IPLC (International Private Leased Circuit)
- IX Peering

**Circuits:** আসল সার্কিট। উদাহরণ:

```
Circuit ID: BTCL-MIR-001
Provider: BTCL
Type: Fiber Optic
Status: Active
Install Date: 2024-01-15
Commit Rate: 10 Gbps
Description: "Primary uplink from Mirpur POP"
Termination A: Provider Network
Termination Z: R-MIR-CORE-01, Interface GigabitEthernet0/0/0
Monthly Cost: BDT 500,000
Contract End Date: 2026-12-31
```

এই সার্কিট ডেটা খুব গুরুত্বপূর্ণ। কারণ এখান থেকে আপনি জানতে পারবেন:
- মোট কত ব্যান্ডউইথ আছে
- কোন সার্কিটের কন্ট্র্যাক্ট কবে শেষ হবে
- মাসিক খরচ কত
- কোন ডিভাইসে কানেক্টেড

### Extras - কাস্টমাইজেশন

Extras হলো সেই জিনিসগুলো যা দিয়ে আপনি Nautobot-কে কাস্টমাইজ করতে পারবেন।

**Custom Fields:** ধরুন Nautobot-এর ডিফল্ট ফিল্ডে আপনার প্রয়োজনীয় কিছু নেই। যেমন আপনি চান প্রতিটা ডিভাইসে "Warranty Expiry Date" ফিল্ড থাকুক। এটা ডিফল্টে নেই। তাহলে কাস্টম ফিল্ড বানাতে পারবেন।

উদাহরণ:

```
Field Name: warranty_expiry
Label: "Warranty Expiry Date"
Object Type: Device
Type: Date
Required: No
Default Value: null
```

এখন প্রতিটা ডিভাইসে এই ফিল্ড দেখাবে।

**Tags (ট্যাগ):** অবজেক্টগুলোকে ক্যাটাগরাইজ করার জন্য ট্যাগ ইউজ করতে পারেন। উদাহরণ:

```
Tag: Production
Color: Green
Description: "Production devices"

Tag: Maintenance
Color: Orange
Description: "Devices under maintenance"

Tag: End-of-Life
Color: Red
Description: "Devices nearing end of life"
```

তারপর যেকোনো ডিভাইস, সাইট, সার্কিট - যেকোনো কিছুতে ট্যাগ লাগাতে পারবেন। এতে সহজে ফিল্টার করা যায়।

**Custom Links:** Nautobot-এর UI-তে কাস্টম বাটন বা লিংক যুক্ত করতে পারেন। যেমন ধরুন আপনার একটা সেপারেট মনিটরিং সিস্টেম আছে। আপনি চান Nautobot-এ কোনো ডিভাইস দেখার সময় একটা বাটন থাকবে "View in Monitoring" যেটায় ক্লিক করলে সেই মনিটরিং সিস্টেমে চলে যাবে।

## আরেকটা কমপ্লিট আইএসপি উদাহরণ

চলুন আরেকটা কমপ্লিট উদাহরণ দেখি। ধরুন "RapidNets Bangladesh" নামে একটা ছোট আইএসপি। তাদের দুটো পপ - মিরপুর আর উত্তরা। প্রায় তিন হাজার রেসিডেনশিয়াল কাস্টমার আর পঞ্চাশটা কর্পোরেট ক্লায়েন্ট। তারা Nautobot সেটআপ করছে।

**Sites তৈরি করা:**
```
Site 1: Mirpur POP
- Address: House 25, Road 10, Mirpur-12, Dhaka
- Facility: Rented
- Rack Count: 2

Site 2: Uttara POP
- Address: Sector 7, Uttara, Dhaka
- Facility: Owned
- Rack Count: 1
```

**Providers যুক্ত করা:**
```
Provider: BTCL (primary uplink)
Provider: CyberGate Communications (secondary uplink)
```

**Circuits যুক্ত করা:**
```
Circuit 1: BTCL-MIR-001
- Provider: BTCL
- Type: Fiber
- Bandwidth: 5 Gbps
- Site: Mirpur POP

Circuit 2: CYBERGATE-UTT-001
- Provider: Cybergate
- Type: Fiber
- Bandwidth: 3 Gbps
- Site: Uttara POP
```

**Devices যুক্ত করা:**
```
Device 1: R-MIR-CORE-01
- Type: MikroTik CCR2004
- Role: Core Router
- Site: Mirpur POP
- Rack: Rack-A
- Interfaces: 
  - ether1: Uplink to BTCL
  - ether2: Link to Uttara POP
  - ether3-24: Distribution to access switches

Device 2: SW-MIR-ACCESS-01
- Type: Cisco Catalyst 2960
- Role: Access Switch
- Site: Mirpur POP
- Rack: Rack-A

...
```

**IP Prefixes সেট করা:**
```
Prefix: 103.92.40.0/23 (IIG assigned public IPs)
  ├─ 103.92.40.0/24 (Mirpur customers)
  └─ 103.92.41.0/24 (Uttara customers)

Prefix: 10.10.0.0/16 (Internal management)
  ├─ 10.10.1.0/24 (Mirpur infrastructure)
  ├─ 10.10.2.0/24 (Uttara infrastructure)
  └─ 10.10.100.0/22 (Corporate clients private IPs)
```

**VLANs ডিফাইন করা:**
```
VLAN 10: MGMT (Management)
VLAN 20: UPLINK
VLAN 100: RES_MIRPUR (Residential Mirpur)
VLAN 101: RES_UTTARA (Residential Uttara)
VLAN 200: CORP_CLIENTS (Corporate)
```

এভাবে একটু একটু করে তারা পুরো নেটওয়ার্ক Nautobot-এ ডকুমেন্ট করবে। শুরুতে সময় লাগবে, কিন্তু একবার হয়ে গেলে পরে সব সহজ হয়ে যায়।

পরের চ্যাপ্টারে আমরা হাতে-কলমে UI ইউজ করে ডেটা এন্ট্রি করব। তাহলে থিওরি থেকে প্র্যাকটিসে পুরোপুরি নামা যাবে।