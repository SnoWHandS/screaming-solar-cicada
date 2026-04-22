# Screaming Solar Cicada

A sun-fed, MCU-free noise bug that trickle-charges from a tiny **Anysolar KXOB** “SolarBIT” module, then screams in bursts when the storage node crosses a “battery good” threshold—then goes quiet until the cap refills. No code, no sleep states—just **PV → boost → cap → hysteresis → load switch → discrete astable → magnetic buzzer.**

---

## How it works (signal chain)

### 1. Harvest (KXOB)

The main energy source is a compact monocrystalline **KXOB** module—for example **KXOB25-04X3F** / **-TR** class (~22 × 7 mm, ~2 V MPP neighborhood, ~10–15 mA-scale current under decent light).

- The panel feeds `VIN_DC` through a **BAT54-class Schottky** (polarity, forward drop, and saner behavior when the charger isn’t drawing).
- **Stringing:** This design targets **1S** (one module) so `VIN_DC` stays inside the **BQ25505** window. **2S** only if you prove worst-case open-circuit never violates `VIN_DC` limits (**~5.1 V** recommended, **5.5 V** abs max). **Parallel** modules are fine for more charging current in weak light.

### 2. PMIC

**TI BQ25505** — **22 µH** boost from `VIN_DC` ↔ `LBOOST`; cold-start from **~600 mV** typical at the input (after the diode, plan margin), then normal operation down to very low `VIN` once running.

**MPPT** uses a high-impedance divider on `VOC_SAMP` from `VIN_DC` so the chip tracks a fraction of the panel’s open-circuit voltage (KXOB **V<sub>OC</sub>** is usually **~2–3 V** class depending on SKU and illumination).

### 3. Storage and thresholds

Energy stacks on the storage / `VSTOR`–`VBAT_*` network. The main reservoir is a large electrolytic (**1000 µF / 6.3 V** on the tank in `crktboi`).

| Function | Programming |
|----------|-------------|
| **`VBAT_OV`** | Two-resistor divider from `VRDIV` — build uses **~6.8 MΩ + 6.2 MΩ** (~**13 MΩ** total class). |
| **`VBAT_OK`** | Three-resistor string (**~5.1 MΩ + 5.11 MΩ + 5.11 MΩ** class) — rising vs falling “turn load on/off” vs **`VBIAS` ≈ 1.21 V**. |

Sanity-check **`VBAT_OV` ≥ `VBAT_OK` rising threshold** per TI ordering rules; divider pin mapping matters.

### 4. Load switch

`VBAT_OK` drives **2N7002** → **AO3401A** high-side: **`Vstore` → `+3V0`** to the audio section. When the tank sags past the low trip, `+3V0` collapses and the chirp ends.

### 5. Noise

**BC847** astable — **~174 kΩ** + **1 nF** timing (**~4 kHz** ballpark), **~1 kΩ** collectors. **GSC1102YB-3V4000** buzzer does the actual scream.

---

## Early direction (not the hero)

The first concept used **arrays of VBPW34S photodiodes** (**3S×3P** strings, per-string Schottkys, picky light economics). That stays interesting as a stunt, but this description treats it as **initial exploration** — the **KXOB** is the practical, repeatable harvester you’d actually leave on a windowsill.

---

## Repo / CAD

**KiCad** project **`crktboi`**:

- **`power`** sheet — KXOB / `VIN_DC`, BQ25505, load switch  
- **`oscillator`** sheet — astable + buzzer  

**ERC** and **Highlight net** are the real documentation.

---

## Expectations

- **Chirp cadence** — charge rate from the KXOB in your real light vs load current when the PFET is on — scale like **Δ*t* ~ *C*·Δ*V* / *I*<sub>load</sub>**.
- **Louder / longer** — more µF or lower mean buzzer current.
- **More frequent chirps** — more mW from the cell or a tighter OK window (trade noise against TI threshold constraints).
