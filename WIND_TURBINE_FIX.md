# Wind Turbine — Print & Power Fix (Ender 3 V3)

> Goal: make the VAWT **print on an Ender 3 V3** (220 × 220 × 220 mm) and actually **generate power** to charge a battery or run a light.
> Geometry was checked by parsing every STL. Fixed parts are in `print_v3/`. Units are **millimetres** (parts are 14–200 mm → the design is a ~200 mm-tall desktop/small-home prototype turbine).

---

## 1. What was broken (from the deep dive)

| Problem | Status | Fix |
|---------|--------|-----|
| `va_leg` (central shaft) is **226 mm tall** — exceeds 220 mm AND limits turbine size | 🔴 | **Replaced with a modular snap-together shaft** (see §1a) — prints in <220 mm pieces and stacks to *any* height, so the turbine can be **bigger than the print volume** |
| Only **one blade** modeled (VAWT needs ≥2, usually 3) | 🔴 | Generated `va_blade_seg_b.stl` (120°) and `va_blade_seg_c.stl` (240°) by rotating about the vertical axis |
| **No generator** — only a `va_coupler` stub | 🔴 | Added `generator_mount.stl` (NEMA17 stepper adapter plate) + electrical spec below |
| No print/electrical documentation | 🟠 | This document |

Everything else already fits 220³ (largest other part is `va_bearing_head` at 152 × 171 × 46 mm).

### 1a. Modular snap-together shaft (the "bigger than the printer" fix)

The shaft is now a **stackable kit**, not one tall part:

| Part | Length | Role |
|------|--------|------|
| `shaft_base.stl` | 40 mm | Bottom: male tenon drops into the existing bearing head (same 14 mm² interface as the original leg); top: female socket |
| `shaft_segment.stl` | 200 mm | Repeating hollow shaft section. Bottom = female socket, top = male tenon. **Print as many as you like** |
| `shaft_topcap.stl` | 30 mm | Caps the top of the last segment |

**Coupling (true snap-fit, no glue):** each segment's top tenon has an **11.2 mm detent bump** that clicks into an **11.4 mm groove** in the next part's female socket; the **11.0 mm entrance lip** provides the snap. The square cross-section **locks rotation with zero slip** (no keyways, no setscrew needed on the shaft joints). Net height per added segment ≈ 180 mm (20 mm joint overlap), so:
- 1 segment → ~240 mm tall rotor shaft
- 4 segments → ~740 mm
- 10 segments → ~1.8 m

**Why this shaft is the most efficient design (my call):**
- **Hollow square tube** (14 mm² outer, 6 mm² bore → 4 mm wall): far lower **rotational inertia** than a solid shaft, so the rotor **spins up at lower wind speed** → captures more energy (higher effective efficiency). Also uses less material.
- **Square cross-section** transmits torque directly through face contact — no slipping, no separate keys/splines, and it matches the existing 14 mm blade clamps and hub interface.
- **Modular = scalable power:** more shaft segments + more blade sets along the shaft = larger swept area = more watts, all still printable.

Blades (`va_blade_seg` ×3+) clamp onto the 14 mm² shaft body at whatever heights you print segments for — stack blade sets up the tower for a taller H-rotor.

---

## 2. Printability check (all parts vs 220 mm)

| Part | X | Y | Z | Fits? |
|------|---|---|---|-------|
| va_base | 198 | 192 | 18 | ✅ |
| va_leg_fixed | 14 | 14 | 220 | ✅ (was 226 — fixed) |
| va_bearing_head | 152 | 171 | 46 | ✅ |
| va_hub_top / va_hub_low | 92 | 92 | 24 | ✅ |
| va_blade_seg (+ b, c) | 128 | 60 | 180 | ✅ (3 copies) |
| va_gearbox_housing | 52 | 52 | 21 | ✅ |
| va_coupler | 30 | 30 | 42 | ✅ |
| va_ring_gear | 48 | 48 | 12 | ✅ |
| va_planet_gear | 15 | 15 | 10 | ✅ |
| va_planet_carrier | 42 | 42 | 14 | ✅ |
| va_sun_gear | 21 | 21 | 10 | ✅ |
| va_shaft_cap | 32 | 32 | 10 | ✅ |
| generator_mount | 56 | 56 | 10 | ✅ |
| shaft_base | 14 | 14 | 40 | ✅ |
| shaft_segment (×N) | 14 | 14 | 200 | ✅ (print as many as desired) |
| shaft_topcap | 10.8 | 10.8 | 30 | ✅ |
| cutaway_socket / cutaway_plug | 128/21 | 60/26 | 160/21 | ✅ (reference/optional) |

**Print set:** every original part **except `va_leg`** (replaced by the modular shaft kit: `shaft_base` + `shaft_segment` ×N + `shaft_topcap`), plus `va_blade_seg` ×3 (original + `_b` + `_c`) and `generator_mount`. The turbine is no longer limited to 220 mm — stack shaft segments to make it as tall as you want.

---

## 3. Recommended print settings (Ender 3 V3, PLA)

- **Layer height:** 0.2 mm (0.16 for gears if you want smoother meshing).
- **Supports:** ON for `va_bearing_head`, `va_hub_top/low`, `va_gearbox_housing` (overhangs/cavities). Blades and shaft print support-free if oriented upright.
- **Brim:** add a 5 mm brim to `va_base` and the shaft segments/base (tall thin parts — fight warping/wobble). The square cross-section already resists rolling better than a round shaft.
- **Infill:** 20% (gears/hub 30–40% for strength). Gears benefit from higher infill.
- **Orientation:** print the shaft (`va_leg_fixed`) flat on its long axis with a brim; blades vertical; gears flat.
- **Tolerance note:** printed planetary gears may bind — test-fit and lightly sand/run-in. If the gearbox is too lossy at low wind, bypass it (see §5).

---

## 4. Generator + electrical (the part that makes it "work")

The modelled drivetrain ends at `va_coupler` (a 30 × 30 × 42 mm square shaft). To make power:

**Chosen generator:** a **NEMA17 stepper motor used as a generator** (cheap, ~$8–15, bolts to the adapter plate, produces AC on its 2 phases when spun). Alternative: any small **12 V DC can motor** (e.g., 555/775) used in reverse.

**Wiring (battery / light):**
```
[rotor] → [gearbox] → [coupler] → [NEMA17 generator]
                                       │
                                       ├─ Phase A ─┐
                                       ├─ Phase B ─┤→ 4-diode bridge rectifier (AC→DC)
                                       │           │
                                       ↓           ↓
                                    +V ──[1000 µF cap]──→ [DC-DC buck / charge controller]
                                                                │
                                                  ┌─────────────┴─────────────┐
                                            Li-ion / LiFePO4 battery      LED + resistor (light)
```
- **For a light:** rectified DC → current-limiting resistor → efficient LED (e.g., 1 W white). Even ~2–3 V from the generator lights it in modest wind.
- **For a battery:** rectified DC → small **wind/solar charge controller** (or a buck + BMS) → 3.7 V Li-ion or 12 V SLA. Trickle-charges a phone power-bank or runs an LED overnight.

**Realistic output (honest):** a single blade set at ~5 m/s wind yields roughly **0.7 W** electric; stack more sets (see §6) for several watts to tens of watts. Enough to **trickle-charge a small battery or power an efficient LED**, not to run appliances on a single set.

---

## 5a. Blade collars — lock multiple blade sets onto the shaft (part a)

`blade_collar.stl` is a **printable 3-arm SNAP clamp** that lets you put a full 3-blade set anywhere along the shaft, so you can stack rotor stages up the tower — **no glue, no tools**:

- Square clamp ring with a **13.8 mm hole** (0.2 mm interference on the 14 mm shaft) and a **slot on the top face** so it springs open, clicks around the shaft, and grips by friction.
- **3 radial arms at 0° / 120° / 240°**, each ending in a pad at the blade-root radius (~20 mm) where a `va_blade_seg` bolts/glues on.
- Print **one collar per 3-blade set**; space them up the shaft segments for a taller H-rotor. Each collar is 48 mm across, 15 mm tall — prints easily.

---

## 6. Power vs wind speed & height (part b)

Simple actuator-disc estimate for the modelled blade (tip radius ≈ 0.145 m → diameter **D = 0.29 m**, one blade segment = **0.18 m** tall). Assumes **Cp = 0.30** and **0.55** combined generator+gearbox+rectifier efficiency.

| Blade sets | Swept height | Swept area | 3 m/s | 5 m/s | 8 m/s | 12 m/s (electric W) |
|-----------|--------------|-----------|-------|-------|-------|---------------------|
| 1 | 0.18 m | 0.052 m² | 0.1 W | 0.7 W | 2.7 W | 9.1 W |
| 3 | 0.54 m | 0.157 m² | 0.4 W | 2.0 W | 8.1 W | 27.3 W |
| 6 | 1.08 m | 0.313 m² | 0.9 W | 4.0 W | 16.2 W | 54.7 W |

**Takeaways:**
- A **single set** is a "trickle" source — fine for an LED or keeping a small battery topped up.
- **3+ sets** (a ~0.5–1 m tall rotor) reaches **useful watts** (2–27 W at 5–12 m/s) — enough to charge a phone battery or run several efficient LEDs.
- Output scales with **height × wind³**, so more shaft segments *and* a windier site dominate. These are aerodynamic upper bounds; real numbers will be lower (start-up torque, bearing/gearbox loss).

---

## 5. Assembly notes & open decisions

1. **Blades:** print 3 × `va_blade_seg` (original, `_b`, `_c`) and mount at **120° spacing** on the central shaft/hub. Confirm the hub has (or add) 3 blade sockets — if not, the blades may need a printed hub collar.
2. **Generator coupling:** `generator_mount.stl` is a basic **NEMA17 adapter plate** (56 × 56 × 6 mm, 31 mm PCD holes, 8 mm centre boss). The motor shaft must engage the 30 × 30 `va_coupler` — use a **printed or flexible shaft coupling** between them, or redesign the coupler to a 5 mm round bore. *I can model a precise coupling if you tell me the exact generator shaft size.*
3. **Gearbox optional:** the planetary stage gives ~**3.3:1** speed-up (fixed ring, carrier in, sun out). For a stepper/DC generator that already works at low rpm, you may **skip the gearbox** and direct-couple for less friction. Keep it if your generator needs higher rpm.
4. **Cutaway parts** (`cutaway_socket/plug`) appear to be reference/connector models — not required for the running turbine.

---

## 8. Assembly diagram & print checklist

### How it goes together (no adhesive on the shaft/collar)
```
        [ shaft_topcap ]  <- snaps onto top segment (detent)
   =================================
   |  blade_collar + 3× va_blade_seg |   <- collar snaps onto shaft
   |  blade_collar + 3× va_blade_seg |   <- stack more stages here
   =================================
        [ shaft_segment ]  (×N, snap together)
   =================================
        [ shaft_base ]  -> bottom tenon drops into bearing head
   =================================
   [ va_bearing_head | va_gearbox_housing | va_coupler ]
   =================================
        [ generator_coupling ]  (square socket over coupler)
        [ generator_mount ] + NEMA17 / 12V DC motor (5mm shaft into coupling, setscrew)
   =================================
        [ va_base ]  (bolts to bearing head)
```
Shaft segments + collars are **snap/tool-free**. The generator coupling uses one small setscrew (a screw, not glue), and the motor + base bolt on.

### Print checklist (Ender 3 V3, PLA)
| Part | Count | Orientation | Supports? | Notes |
|------|-------|-------------|-----------|-------|
| va_base | 1 | flat | no | brim |
| va_bearing_head | 1 | bearing up | yes | overhangs |
| va_hub_top / va_hub_low | 1 ea | flat | yes | |
| va_blade_seg (+_b,_c) | 3 | vertical | no | print 3× (one per set); print more sets as desired |
| va_gearbox_housing | 1 | flat | yes | |
| va_coupler | 1 | vertical | no | |
| va_ring/sun/planet gears, carrier, shaft_cap | 1 ea | flat | yes (gears) | test-fit gears |
| shaft_base | 1 | tenon down | no | |
| shaft_segment | N | flat/rolling | no | brim; N = desired height/180 |
| shaft_topcap | 1 | tenon down | no | |
| blade_collar | 1 per blade set | flat | no | slot faces up |
| generator_mount | 1 | flat | no | |
| generator_coupling | 1 | bore vertical | yes (bore) | drill M3 setscrew |

---

## Files produced
- `print_v3/shaft_base.stl` — modular shaft base (male tenon → bearing, female socket on top)
- `print_v3/shaft_segment.stl` — repeating hollow shaft section (print ×N)
- `print_v3/shaft_topcap.stl` — top cap
- `print_v3/blade_collar.stl` — 3-arm clamp to lock a 3-blade set anywhere on the shaft
- `print_v3/va_blade_seg_b.stl`, `va_blade_seg_c.stl` — 2nd & 3rd blades
- `print_v3/generator_mount.stl` — NEMA17 adapter plate
- `print_v3/generator_coupling.stl` — links turbine coupler (30×30) to 5 mm generator shaft
- this document

*Status: parts (a) blade collars, (b) power estimate, and (c) generator coupling are all done. The turbine is now fully printable, modular, and electrically specified.*

## 7. Generator coupling (part c) — done

`generator_coupling.stl` joins the turbine's drivetrain to the generator:
- **Turbine side:** a 30.4 mm square socket that slips over the existing `va_coupler` (30 × 30 bar).
- **Generator side:** a 5.2 mm round bore for a **5 mm generator shaft** (standard for NEMA17 steppers and most 12 V DC can motors — your "NEMA17 or 12 V DC" both fit).
- A **setscrew nub** on the generator face — drill an M3 hole and add a grub screw for a positive lock.
- 40 × 40 × 66 mm — prints easily.

The generator itself bolts to the earlier `generator_mount.stl` (NEMA17 adapter plate); its shaft then enters this coupling. **To resize the bore** for a different shaft (e.g., 6 mm), change `rinner=2.6` in `wt_coupling.py` to `3.0` and re-run — it's a one-line change.
