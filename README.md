# 🖥️ 10-Inch Mini Rack Homelab Build

A fully documented 10-inch homelab rack housing four Dell OptiPlex Micro servers, a Synology NAS, Raspberry Pi status display, PoE switch, and PDU. Clean cable management with a keystone patch panel and colour-coded cabling throughout.

---

## 📦 Hardware Overview

### In the Rack

| Device | Role | Switch |
|---|---|---|
| Dell OptiPlex 7070 Micro (i5-9500T) | JANET — main OpenClaw host | KeepLiNK |
| Dell OptiPlex 3060 Micro | Homelab server | KeepLiNK |
| Dell OptiPlex 3050 Micro ×2 | Homelab servers | KeepLiNK |
| Synology DS1019+ (5-bay, ~30TB) | NAS | KeepLiNK |
| Raspberry Pi 4B 2GB + Wisecoco 7.84" bar display | Status panel (top of rack) | KeepLiNK |
| KeepLiNK 10-Port PoE Switch (8×PoE + 2×uplink) | In-rack switching | — |
| PDU 10-inch rack mount (4× universal outlets) | Rack power | — |

### On the Desk (beside rack)

| Device | Role |
|---|---|
| GoodTop 16-Port 2.5G Switch (2× 10G SFP+) | Core switch — too wide for 10-inch rack |
| Starlink Router | Internet uplink |
| HADES (Dell Precision 7910, dual Xeon, 384GB RAM) | Hyper-V host — on KeepLiNK for NAS proximity |
| Laptop, TV, PS5, Chromecast | On GoodTop via existing cables |

---

## 🏗️ Rack Design

**Rack:** [KWS Rack V2 — Heavy Duty 10-Inch Homelab Rack](https://makerworld.com/en/models/2139130-kws-rack-v-2-heavy-duty-10-inch-homelab-rack) (3D printed)

**OptiPlex mounts:** [Lenovo Tiny Vertical Holder for DeskPi 10-Inch Rack](https://makerworld.com/en/models/1216152-lenovo-tiny-vertical-holder-for-deskpi-10-rack) (3D printed — compatible with Dell OptiPlex Micro)

**Keystone patch panel:** [Patch Keystones Panel 2–3U for 10-Inch KWS Rack](https://makerworld.com/en/models/2154991-patch-keystones-panel-2-3u-for-10-inch-kws-rack) (3D printed)

### 10-Inch Rack Specs
- Usable internal width: ~220mm (~210mm safe with tolerance)
- Rail hole spacing: 236.5mm
- Standard 1U height: 44.45mm

### OptiPlex Holder Compatibility
| Dimension | Lenovo ThinkCentre Tiny | Dell OptiPlex Micro |
|---|---|---|
| Height | 183mm | 183mm ✅ |
| Depth | 179mm | 178mm ✅ |
| Thickness | 34.5mm | 36mm (~1.5mm diff) ✅ |

> ⚠️ Print one first and test fit before printing all four.

---

## 🌐 Network Architecture

```
Starlink Router
      │
      ▼
GoodTop 16-Port 2.5G (desk)
  ├── Laptop          (existing cable)
  ├── TV              (existing cable)
  ├── PS5             (existing cable)
  ├── Chromecast      (existing cable)
  └── uplink ──────────────────────────┐
                                       │
                              KeepLiNK PoE Switch (rack)
                                  ├── OptiPlex 7070
                                  ├── OptiPlex 3060
                                  ├── OptiPlex 3050 #1
                                  ├── OptiPlex 3050 #2
                                  ├── Synology DS1019+
                                  ├── Raspberry Pi 4
                                  ├── HADES
                                  └── [PoE 8 — spare]
```

**Laptop → Synology:** Even though the laptop is on GoodTop and the NAS is on KeepLiNK, they route through the uplink transparently. Full 1G speed, no configuration needed.

### KeepLiNK Port Map

| Port | Device | Notes |
|---|---|---|
| PoE 1 | OptiPlex 7070 | |
| PoE 2 | OptiPlex 3060 | |
| PoE 3 | OptiPlex 3050 #1 | |
| PoE 4 | OptiPlex 3050 #2 | |
| PoE 5 | Synology DS1019+ | Start with 1 port; add 2nd for link agg later |
| PoE 6 | Raspberry Pi 4 | |
| PoE 7 | HADES | Existing long cable via patch panel keystone |
| PoE 8 | **SPARE** | Reserved for Synology 2nd port |
| Uplink 1 | GoodTop | Core uplink |
| Uplink 2 | **SPARE** | |

---

## 🔌 Cabling Plan

Cabling uses a **keystone patch panel** between the switch and devices for a clean, professional look.

```
[KeepLiNK port] ──0.25m purple──▶ [Keystone front | Keystone back] ──0.5m──▶ [Device]
```

### Front Side (switch → keystone) — ESSCable 0.25m purple
| Connection | Cable |
|---|---|
| KeepLiNK → keystone (OptiPlex 7070) | 0.25m purple |
| KeepLiNK → keystone (OptiPlex 3060) | 0.25m purple |
| KeepLiNK → keystone (OptiPlex 3050 #1) | 0.25m purple |
| KeepLiNK → keystone (OptiPlex 3050 #2) | 0.25m purple |
| KeepLiNK → keystone (Synology) | 0.25m purple |
| KeepLiNK → keystone (Pi) | 0.25m purple |
| KeepLiNK → keystone (HADES) | 0.25m purple |
| KeepLiNK uplink → GoodTop | 0.25m purple (direct, no keystone) |

**Ordered:** 10× ESSCable ERT-600-HV (8 used, 2 spare) ✅

### Back Side (keystone → device) — Vention 0.5m coloured
| Connection | Cable |
|---|---|
| Keystone → OptiPlex 7070 | 0.5m |
| Keystone → OptiPlex 3060 | 0.5m |
| Keystone → OptiPlex 3050 #1 | 0.5m |
| Keystone → OptiPlex 3050 #2 | 0.5m |
| Keystone → Synology | 0.5m |
| Keystone → Pi | 0.5m |
| Keystone → HADES | Existing long cable ✅ |

**Keystones:** 12× ZoeRax CAT6A STP (7 used, 5 spare) ✅

---

## ⚡ Power Plan

| Device | Power Method |
|---|---|
| OptiPlex 7070 | UGREEN Nexode 100W 4-Port GaN → KYMISON 4.5×3.0mm adapter → 0.25m Toocki cable |
| OptiPlex 3060 | UGREEN Nexode 65W 3-Port GaN → KYMISON 4.5×3.0mm adapter → 0.25m Toocki cable |
| OptiPlex 3050 ×2 | UGREEN Nexode 65W 3-Port GaN (×2) → KYMISON adapters → 0.25m Toocki cables |
| Synology DS1019+ | Own power brick |
| KeepLiNK switch | Own UK plug |
| Raspberry Pi 4 | Meanwell 5V PSU (from bundle) via USB-C |
| Wisecoco display board | Spare UGREEN 65W port → USB-C to Micro-USB |

> ✅ OptiPlex 7070 covered by the 100W charger — no boot warning.

### Idle Power Estimate

| Device | Idle Draw |
|---|---|
| OptiPlex 7070 | ~13W |
| OptiPlex 3060 | ~6W |
| OptiPlex 3050 ×2 | ~20W |
| Synology DS1019+ (5 drives) | ~30W |
| Raspberry Pi 4 | ~5W |
| KeepLiNK switch | ~10W |
| **Total** | **~84W** |

~$88/year at US average electricity rates.

---

## 🖥️ Rack Control Display Module (Top of Rack — 2U)

**Model:** [2U Rack Control Display Module for KWS 10-inch Rack](https://makerworld.com/en/models/2233953-2u-rack-control-display-module) (3D printed)

Houses the Raspberry Pi 4B, Wisecoco bar display, and 2× keystone panel ports in a clean 2U front panel.

**Keystone slots populated with:**
- USB-A 3.0 pass-through (×2 ordered)
- USB-C 3.1 pass-through (×2 ordered)
- 4K HDMI pass-through (×1 ordered)

**Internal cable:** USB 2.0 Male-to-Male 0.25m for internal connections.

---

## 📺 Status Display (Raspberry Pi + Wisecoco 7.84" Bar LCD)

**Display:** Wisecoco 7.84" 1280×400 MIPI bar LCD with external driver board

**Connection:**
```
Pi 4 Micro-HDMI ──0.3m──▶ Mini-HDMI (driver board)
Driver board Micro-USB ──▶ USB-C to Micro-USB ──▶ 65W charger
Pi USB-C power ──▶ Meanwell 5V PSU
```

**Display config** (`/boot/config.txt`):
```ini
hdmi_group=2
hdmi_mode=87
hdmi_cvt=1280 400 60 6 0 0 0
```

**Software:** Chromium in kiosk mode → Homarr dashboard (already running on hades-vm)
```bash
chromium-browser --kiosk --noerrdialogs --disable-infobars http://hades-vm:7575
```

---

## 🛒 Purchase List

> All prices in USD. GBP converted at $1.3368 (15 Mar 2026 rate).

### Networking & Rack

| Item | Link | Price |
|---|---|---|
| KeepLiNK 10-Port PoE Switch (UK plug) | [AliExpress](https://www.aliexpress.com/item/1005007045849781.html) | $34.11 |
| PDU 10-inch Rack Mount #1 — OptiPlex power (4× universal, UK) | [AliExpress](https://www.aliexpress.com/item/1005011824742917.html) | $41.93 |
| PDU 10-inch Rack Mount #2 — Switch/NAS/Pi power (4× universal, UK) | [AliExpress](https://www.aliexpress.com/item/1005011824742917.html) | $41.93 |
| ~~ZoeRax CAT6A STP Keystone Couplers ×12~~ | ~~[AliExpress](https://www.aliexpress.com/item/1005006532881465.html)~~ | ~~$23.64~~ CANCELED |
| Linkwylan CAT6A STP Coloured Keystone Jacks ×12 (Violet, tool-free) | [AliExpress](https://www.aliexpress.com/item/1005005844843325.html) | $40.08 |
| ESSCable ERT-600-HV 0.25m Purple CAT6 ×10 | [Broadband Buyer](https://www.broadbandbuyer.com/products/38577-esscable-ert-600-hv-10x/) | $14.30 |
| VEnTIOn CAT6 Coloured Ethernet Cables | [AliExpress](https://www.aliexpress.com/item/1005002524876303.html) | $7.34 |

### OptiPlex Power

| Item | Link | Price |
|---|---|---|
| UGREEN Nexode 100W 4-Port GaN Charger ×1 | [Amazon UK](https://www.amazon.co.uk/dp/B091N7FVDL) | (see order) |
| UGREEN Nexode 65W 3-Port GaN Charger ×3 | [Amazon UK](https://www.amazon.co.uk/dp/B0B7N4DX1Z) | (see order) |
| UGREEN order total (Amazon, Order #204-8358283-3717925) | — | $128.89 |
| KYMISON USB-C to Dell 4.5×3.0mm Adapter ×4 | [AliExpress](https://www.aliexpress.com/item/1005001603542560.html) | $9.20 |
| Toocki USB-C to USB-C 0.25m Black ×4 | [AliExpress](https://www.aliexpress.com/item/1005006844576361.html) | $6.20 |

### Display & Raspberry Pi

| Item | Link | Price |
|---|---|---|
| Wisecoco 7.84" 1280×400 Bar LCD (MIPI) | [AliExpress](https://www.aliexpress.com/item/1005004986951553.html) | $54.00 |
| Mini HDMI to Micro HDMI Cable 0.3m | [AliExpress](https://www.aliexpress.com/item/1005009277513110.html) | $7.46 |
| USB-C to Micro-USB Cable 1m | [AliExpress](https://www.aliexpress.com/item/1005006716758195.html) | $4.41 |
| Toocki USB-C to USB-C 1m 60W | [AliExpress](https://www.aliexpress.com/item/1005008152768741.html) | $1.98 |
| 0.91" OLED SSD1306 I2C Display | [AliExpress](https://www.aliexpress.com/item/32879702750.html) | $3.07 |
| Raspberry Pi 4B 2GB + SD + DIN Rail + PSU | [eBay](https://www.ebay.co.uk/itm/358067141087) | $69.20 |
| Vention USB 2.0 Male-to-Male Cable 0.25m ×2 | [AliExpress](https://www.aliexpress.com/item/1005001560424564.html) | $3.47 |
| USB-C 3.1 Keystone Panel Insert ×2 | [AliExpress](https://www.aliexpress.com/item/1005009430447241.html) | (see below) |
| USB-A 3.0 Keystone Panel Insert ×2 | [AliExpress](https://www.aliexpress.com/item/1005009430447241.html) | (see below) |
| 4K HDMI Keystone Panel Insert ×1 | [AliExpress](https://www.aliexpress.com/item/1005009430447241.html) | (see below) |
| Keystone panel inserts total (Order 8209266578349865) | — | $8.26 |

### Tools & Hardware

| Item | Link | Price |
|---|---|---|
| 43pc Magnetic Screwdriver Set | [AliExpress](https://www.aliexpress.com/item/1005007494840464.html) | $33.10 |
| Heat Set Thread Insert Kit (soldering iron kit) | [AliExpress](https://www.aliexpress.com/item/1005007550647561.html) | $11.16 |
| Brass Hot Melt Insert Nuts M5 ×30pcs ×6 bags | [AliExpress](https://www.aliexpress.com/item/1005003582355741.html) | $19.86 |
| Phillips Truss Head Screws M5×10 (20pcs) | [AliExpress](https://www.aliexpress.com/item/1005004681098987.html) | TBD |
| Hex Socket Cap Head Allen Bolts M6×10mm (20pcs) | [AliExpress](https://www.aliexpress.com/item/1005009393815442.html) | TBD |
| Hex Socket Cap Head Allen Bolts M6×8mm (20pcs) | [AliExpress](https://www.aliexpress.com/item/1005009393815442.html) | TBD |
| M6 Nylon Lock Nuts ×100 | [AliExpress](https://www.aliexpress.com/item/1005006565106519.html) | $8.62 |
| Velcro Cable Organiser 5M ×2 | [AliExpress](https://www.aliexpress.com/item/1005008165192843.html) | $3.32 |
| 10×5mm Round Magnets ×200 | [AliExpress](https://www.aliexpress.com/item/1005009558516455.html) | $32.01 |
| 6×2mm Neodymium Magnets ×50 | [AliExpress](https://www.aliexpress.com/item/1005010158237241.html) | $2.83 |

### Total Spent

| Category | Amount |
|---|---|
| AliExpress orders (excl. swapped items) | $260.74 |
| Linkwylan keystones ×12 (replaces ZoeRax) | $40.08 |
| 2nd PDU (AliExpress) | $41.93 |
| Brass insert nuts ×6 bags | $19.86 |
| ESSCable (Broadband Buyer) | $14.30 |
| Raspberry Pi bundle (eBay) | $69.20 |
| UGREEN chargers (Amazon UK) | $128.89 |
| Keystone panel inserts (USB-C ×2, USB-A ×2, HDMI ×1) | $8.26 |
| Vention USB 2.0 M-M cable ×2 | $3.47 |
| Screws & bolts (×3 orders) | TBD |
| **Grand Total** | **~$586.73 USD + TBD** |

> ❌ Canceled: Toocki 1m Purple USB-C ×4 (-$9.16)

> GBP converted at $1.3368 (15 Mar 2026)

---

## ✅ Still To Do

- [ ] Order 3 more Vention 0.5m patch cables (back-side device connections)
- [ ] Print keystone patch panel: [2–3U for 10-inch KWS Rack](https://makerworld.com/en/models/2154991-patch-keystones-panel-2-3u-for-10-inch-kws-rack)
- [ ] Measure Synology DS1019+ against rack internal width (230mm vs ~220mm usable)
- [ ] Print one OptiPlex vertical holder, test fit, then print ×3 more
- [x] ~~Order USB-C GaN chargers for OptiPlexes~~ ✅ UGREEN order placed

---

## 📐 Rack Layout (Planned)

```
          FRONT                          BACK
┌───────────────────────────┐   ┌───────────────────────────┐
│  2U Control Module        │   │                           │
│  Pi + Display + USB/HDMI  │   │                           │
├───────────────────────────┤   │       [4U empty]          │
│   KeepLiNK PoE Switch     │   │                           │
├───────────────────────────┤   ├───────────────────────────┤
│   Keystone Patch Panel    │   │      PDU #2 (Switch/NAS/  │
├───────────────────────────┤   │           Pi power)       │
│   OptiPlex ×2 (vertical)  │   ├───────────────────────────┤
├───────────────────────────┤   │                           │
│   OptiPlex ×2 (vertical)  │   │       [2U empty]          │
├───────────────────────────┤   │                           │
│   Synology DS1019+ ⚠️     │   ├───────────────────────────┤
│   (measure fit first)     │   │    PDU #1 (OptiPlex       │
└───────────────────────────┘   │         power)            │
                                └───────────────────────────┘
```

**Front (bottom → top):** NAS → OptiPlexes ×4 (vertical, 2 per U) → Keystone panel → KeepLiNK switch → 2U Control Module (Pi + display + USB/HDMI keystones)

**Back (bottom → top):** PDU #1 (OptiPlex power) → 2U empty → PDU #2 (switch/NAS/Pi power) → 4U empty

---

*Documentation maintained by Janet 🐟 — last updated 15 March 2026 16:43 GMT*
