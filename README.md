# AtmosProp
### Atmospheric Sound Propagation Analyzer

AtmosProp is a professional-grade, single-file HTML acoustic analysis tool built for live production engineers. It requires no installation, no server, and no dependencies beyond a modern browser. Open the file and work.

---

## Origin & Disclaimer

AtmosProp was created by a working live audio engineer and system technician who found that practical, accessible tools for understanding atmospheric effects on sound propagation simply did not exist. Questions about how temperature and humidity affect high-frequency absorption at distance, or how much delay alignment drifts as conditions change through the day, are common on the show floor — yet the answers have historically required referring to academic literature or a variety of websites of questionable accuracy. This project exists to provide a more practical and concise solution to any engineer working in the field today.

The broader goal is **dBench**: a consolidated, freely accessible platform covering the full range of calculations a live production engineer encounters before or during an event — acoustics, signal processing, power, and beyond — available to anyone, anywhere, on any device, without accounts or internet dependencies.

> **This tool is currently in active development. While every effort has been made to ensure accuracy — and all formula implementations have been cross-validated against recognized reference sources — AtmosProp is provided for educational and informational purposes only. It should not be used as the sole basis for critical engineering decisions. The developer assumes no liability for decisions made in reliance on its outputs. Users are encouraged to take measurements, verify data, and provide feedback whenever possible.**

---

## Overview

AtmosProp currently consists of two tools — the **Absorption Tool** and the **Delay Alignment Tool** — accessible from the AtmosProp logo dropdown in the upper-left corner. Additional tools are in development (see Roadmap).

---

## Absorption Tool

Calculates atmospheric sound absorption across the full audio frequency range, showing total attenuation in dB at a given distance and set of atmospheric conditions.

### Controls

**Atmospheric Conditions** (shared between both tools)

- **Temperature** — Range: −4°F to 122°F / −20°C to +50°C. The °F slider snaps to whole integers; the °C slider snaps to one decimal place (0.1°C resolution). The internal value is always stored in °C with full floating-point precision, so switching between Imperial and Metric units does not cause rounding drift.
- **Relative Humidity** — 0–100% RH.
- **Altitude / Baro Press** — Toggle between altitude mode (ft or m) and barometric pressure mode (inHg or kPa). Internally stored in kPa. Valid range: below sea level down to −328 ft (−100 m) to 18,000 ft (5,500 m) / 50–110 kPa. Pressure has a meaningful effect on absorption but a negligible effect on speed of sound.
- **IMP / MET** — Unit Standards toggle located inside the ⚙ Settings dropdown.

**Live Positions**

Up to six live positions can be active simultaneously, each with an independently configurable label and distance from sound source. Each position renders as a live trace on the absorption canvas. Toggle positions on/off with the pill switch. Use the × button to remove a position (minimum one position required). Use the **+ ADD POSITION** button to add positions up to the six-position maximum.

**Traces**

Static traces lock a set of conditions (distance, temperature, humidity, pressure) as a reference line. Use the **ADD TRACE** button to capture current conditions. Traces can be toggled visible/hidden or deleted individually. The circle button cycles visibility; the × button removes the saved trace.

**Canvas Interaction**

- **Pan** — Click and drag horizontally to pan the frequency view.
- **Zoom** — Mouse wheel zooms toward the cursor position. Use the +, −, and ⊡ buttons in the upper-right of the display header for keyboard-accessible zoom control.
- **Hover readout** — Moving the mouse over the canvas displays **FREQ** and **ATT** (attenuation) at the cursor position.
- **Probe** — Double-click anywhere on the canvas to lock a crosshair probe at a specific frequency. Drag horizontally to scrub the probe across the frequency axis (scrub speed varies with the vertical location of the pointer for fine/coarse control). The probe displays frequency, attenuation, and the live position label at that point. Horizontal pan is disabled while the probe is active; zoom and vertical movement are maintained. Double-click again to release.

**EQ Mode — Atmospheric Delta Viewer**

The **EQ** button activates a specialized view designed for Run-Of-Show use, showing the real-time impact of changing atmospheric conditions relative to a previously tuned system.

When a PA system is tuned, the engineer applies EQ corrections to compensate for changes in atmospheric conditions that differ from the time of tuning. As temperature and humidity change throughout the day or event, the absorption characteristics of the air shift — meaning the original EQ corrections are no longer perfectly matched to current conditions. EQ Mode makes this drift visible in real time.

Each Captured Trace represents a reference — a distance and the atmospheric conditions at the time the system was tuned. EQ Mode computes the difference between absorption of the current live conditions and absorption of each trace's stored conditions, over that trace's distance:

```
Δ(f) = Absorption(f, live conditions, distance) − Absorption(f, stored conditions, distance)
```

The result is a delta curve for each Captured Trace, drawn in that trace's original color. When live conditions exactly match a trace's stored conditions, its curve sits flat at 0 dB — no correction drift. As conditions change, the curves deviate, showing precisely how much additional boost or cut is needed at each frequency to maintain the original tuning intent. The view is symmetric around 0 dB (default ±15 dB range) and covers 1 kHz–20 kHz by default, where atmospheric effects are most significant. Pan, zoom, and probe interactions work identically to the standard Absorption View.

Live Position traces are hidden in EQ Mode — the delta curves already account for all distance-condition relationships — and are restored automatically when EQ Mode is deactivated.

**UPDATE TRACE** replaces the + CAPTURE TRACE button while in EQ Mode. Pressing it snapshots the current state of all visible delta curves as static reference lines — useful after re-tuning the system to the current conditions to create a persistent visual record. Each snapshot appears in the Updated Traces log below the Captured Traces list, labeled with the source distance and timestamp. Individual updated traces can be toggled or removed; multiple snapshots can accumulate over an event, building a visual timeline of how conditions and correction requirements have evolved. Updated traces are displayed in colors distinct from Captured Traces for clear visual separation.

Updated Traces are session-only and are not saved to the browser. If the user desires a record of changes made throughout a show then screenshots are suggested. Toggling EQ Mode off hides the Updated Traces and restores the standard Absorption View; toggling it back on restores them exactly as they were.

### Formula Engine

AtmosProp implements the full ANSI/ASA S1.26-2014 atmospheric absorption formula chain with Bass et al. 1995 pressure corrections applied. This is the internationally recognized standard for audio-frequency absorption calculation.

**Formula chain summary:**

1. Convert temperature to Kelvin; compute pressure ratio `pr = P / 101.325`
2. Saturation vapor pressure via Bass et al. 1995, Form A
3. Molar water vapor concentration: `h_mol = h × Psat / pr` — the division by `pr` is the critical Bass et al. pressure correction omitted by many online implementations
4. Oxygen and nitrogen relaxation frequencies, both scaled by `pr`
5. Absorption coefficient: `α = 8.686 × f² × (1/pr) × [classical term + vibrational terms]` — classical term uses `(T/T0)^(+0.5)` per ANSI S1.26 §5.1
6. Total attenuation: `α × distance`

**Valid operating ranges:**

| Parameter   | Range                        |
|-------------|------------------------------|
| Temperature | −20°C to +50°C (−4°F to 122°F) |
| Humidity    | 0–100% RH                    |
| Pressure    | 50–110 kPa                   |
| Frequency   | 50 Hz – 20 kHz               |
| Distance    | Any (α scales linearly)      |

Outputs have been cross-validated against the National Physical Laboratory (NPL) online calculator — the UK's gold-standard reference implementation — and agree within 0.05 dB/100m across the 1 kHz–16 kHz range.

> **Scope note:** AtmosProp calculates atmospheric absorption only. It does not model ground reflection, refraction, turbulence, wind shear, barriers, or source directivity. Results represent free-field estimates and should be validated against measurement when used for critical applications.

Click the **ANSI/ASA S1.26-2014 LOADED** status pill (green pulsing dot, upper right of the header) to open the formula summary dropdown. From there, click **↗ View Citations** to open the full citations panel with complete bibliographic references and links.

---

## Delay Alignment Tool

Calculates the acoustically correct delay setting for any delay measurement, as well as the optional distance from the two sources aligned to each other, compensating for changes in atmospheric conditions between the original measurement session and the current show conditions.

### Concept

When a delay alignment is measured with SMAART or a similar system, that measurement captures the delay required at a specific temperature and humidity. When conditions change — a warm afternoon becomes a cool evening, or a dry cool venue becomes a hot humid one — the speed of sound changes, and the previously aligned delay is no longer correct. AtmosProp calculates the optimal delay for current conditions using a speed-of-sound ratio applied to the reference measurement.

Speed of sound is calculated using the full **Cramer (1993)** equation, which accounts for temperature, humidity, CO₂ concentration, and pressure. At practical audio engineering scales, pressure has a negligible effect on speed of sound (approximately 0.01 m/s across the full altitude range) and is therefore excluded from delay calculations. Only temperature and humidity are operationally significant.

### Measurements

AtmosProp supports up to **8 simultaneous measurement slots** (Measurement 1 through Measurement 8), each representing an independent delay speaker position. On desktop, all active slots display in a grid. On mobile, slots are accessed via the tab bar at the top of the delay panel.

**Adding measurements** — Click the **+** button in the header of the last slot (desktop) or the **+** tab at the end of the mobile tab bar to add a new measurement. Slots can be toggled active/inactive with the pill switch (Measurement 1 is always active).

**Slot header controls:**

- **Name field** — Editable. Changes are reflected immediately in the mobile tab bar.
- **Toggle switch** — Activates or deactivates the slot (Measurement 1 cannot be deactivated).
- **+ button** — Appears on the last slot header; adds a new measurement.
- **💾 / 🔒 button** — Save this slot's reference data to browser local storage. When saved, the icon changes to 🔒 and glows cyan. Click the 🔒 to clear the saved data for that slot. Saved slots survive page reloads and are included in Export/Import.

### Reference Card (per slot)

Enter the conditions at the time of the original SMAART measurement:

- **Mains → Mic** — Distance in feet or meters from the main speaker array or zero point sound source to the measurement microphone.
- **Delay Src → Mic** — Distance from the delayed speaker to the measurement microphone.
- **Temp at Measurement** — Temperature at the time of measurement.
- **RH at Measurement** — Relative humidity at the time of measurement.
- **Applied Delay** — The delay value (in milliseconds) that was set and verified as aligned during the measurement session.

All distance and temperature fields respect the current IMP/MET unit selection. Distances are stored internally in meters; temperatures are stored internally in °C.

### Current Card (per slot)

Displays the calculated results based on current atmospheric conditions (set via the shared Atmospheric Conditions panel):

- **Speed of Sound (Ref)** — Speed of sound at reference measurement conditions, in m/s. Located at the bottom of the "Validation" container.
- **Speed of Sound (Now)** — Speed of sound at current conditions, in m/s. Located in the top right above the Measurement Containers and under the settings button. 
- **Optimal Delay** — The delay value you should set right now to maintain alignment, in milliseconds.
- **Δ from Reference** — Difference between Optimal Delay and Applied Delay. Color-coded: neutral grey below ±0.026 ms (half-wavelength of 19 kHz), magenta for positive delta, orange for negative delta.
- **Consistency check** — Compares the Optimal Delay against the geometry-implied expected delay (if distances were entered). Green ✓ = within ±0.5 ms; yellow ⚠ = within ±2 ms; orange ⚠ = greater than ±2 ms. If no distances are entered, shows the Implied Distance instead.

### Comb Filter Visualization

Each slot includes a mini comb filter canvas showing the interference pattern between the mains and the delay speaker at the current level ratio. The pattern reflects whether the delay is early, late, or aligned.

**Expanding the comb filter** — Double-click the mini canvas (or double-tap on mobile) to open the full-screen comb filter overlay.

**Expanded overlay controls:**

- **FREQ / GAIN readout** — Displays frequency and gain at the mouse cursor position as you move across the canvas.
- **Level Ratio selector** — Sets the amplitude relationship between mains and delay speaker: Equal (0 dB), Mains +3 dB, +6 dB, +12 dB, or +18 dB. This affects comb filter null depth.
- **+, −, ⊡** — Zoom in, zoom out, reset zoom.
- **Probe** — Double-click the expanded canvas to lock a crosshair probe at a specific frequency. Drag horizontally to scrub the probe across the frequency axis (scrub speed varies with the vertical location of the pointer for fine/coarse control). Double-click again to release.
- **✕** — Close the overlay.

---

## Settings & Defaults

### ⚙ Settings Dropdown

Located in the header, upper right.

- **UNITS** — Toggle between Imperial (IMP) and Metric (MET). Affects all distance, temperature, and pressure displays across both tools.
- **Defaults** — Opens the Customize Defaults modal.
- **Export** — Downloads the current configuration as a JSON file, including saved defaults, active traces, and any saved delay slot data.
- **Import** — Loads a previously exported JSON file, restoring all saved settings.

### Customize Defaults Modal

Defines the traces and live positions that load automatically on startup.

**Live Positions** — Configure the default label, distance, and enabled state for each of the live positions. Changes take effect the next time the page loads or defaults are applied.

**Default Traces** — Add, remove, and configure static traces that appear by default. Each trace stores distance, temperature, humidity, and pressure. The pressure field includes a per-trace toggle between Altitude and Barometric Pressure display mode.

**Buttons:**
- **SAVE TO BROWSER** — Persists the current defaults to local storage.
- **APPLY NOW** — Applies the current modal settings to the active session without saving.
- **CLEAR SAVED** — Removes all saved defaults and saved delay slot data from local storage.
- **CANCEL** — Closes without applying changes.

---

## Formula Citations

[1] **ANSI/ASA S1.26-2014 (R2024)** — *American National Standard Method for the Calculation of the Absorption of Sound by the Atmosphere.* Acoustical Society of America. Reaffirmed 2024.
https://webstore.ansi.org/standards/asa/asaansis1262014r2024

[2] **Bass et al. 1995** — Bass, H.E., Sutherland, L.C., Zuckerwar, A.J., Blackstock, D.T., and Hester, D.M. "Atmospheric absorption of sound: Further developments." *JASA* 97(1), 680–683. Erratum: JASA 99, 1259 (1996).
https://doi.org/10.1121/1.412989

[3] **ISO 9613-1:1993** — *Acoustics — Attenuation of sound during propagation outdoors — Part 1: Calculation of the absorption of sound by the atmosphere.* ISO. Confirmed current 2021.
https://www.iso.org/standard/17426.html

[4] **Bohn 1988** — Bohn, D.A. "Environmental effects on the speed of sound." *JAES* 36(4), 223–231. AES Convention 83, New York, 1987.
https://aes.org/publications/elibrary-page/?id=5156

[5] **NPL Calculator** (Cross-validation reference) — National Physical Laboratory. *Calculation of Absorption of Sound by the Atmosphere.*
https://www.npl.co.uk/acoustics

---

## Roadmap

AtmosProp is the flagship tool of **dBench** — a forthcoming platform of professional production engineering tools organized across three focused web applications:

- **dBench-Audio** — Live audio calculations: acoustics, signal processing, psychoacoustics
- **dBench-Power** — Live power calculations: load balancing, amplifier/driver relationships, safety
- **dBench-Logic** — Educational content and community discussion

Each lives as a self-contained single-page tool set under the dBench umbrella.

---

**Tier 1 — Launch (current)**
- ✅ AtmosProp (Absorption + Delay) *(dBench-Audio)*

**Tier 1 — Planned**
- Wavelength / Frequency Calculator *(dBench-Audio)*
- Frequency-to-Note with Harmonic Series *(dBench-Audio)*
- dBench platform development — umbrella site, navigation, dBench-Audio / dBench-Power / dBench-Logic structure
- Expanded browser compatibility — verified support across Firefox (desktop), mobile Firefox, mobile Safari, and Chromium-based browsers
- ISL Tool — frequency-dependent behavior across point source, line source, and horn; includes line array impulse response analysis *(dBench-Audio)*

**Tier 2**
- ISL Tool — User Discussion and Database Platform *(dBench-Logic)*
- SPL / Distance with Equal Loudness overlay *(dBench-Audio)*
- Fletcher-Munson Equal Loudness reference + SPL delta visualization *(dBench-Audio)*
- System Tuning Target Curve Generator — SMAART companion *(dBench-Audio)*
- Compressor Attack/Release Frequency Interaction Visualizer *(dBench-Audio)*
- Basic Power Load Calculator *(dBench-Power)*
- Amplifier / Driver Voltage Calculator *(dBench-Power)*
- Amplifier / Driver Cable Voltage Drop Calculator *(dBench-Power)*
- Three-phase Load Balancing *(dBench-Power)*

**Tier 3**
- Dynamic EQ recommendations *(dBench-Logic)*
- Coverage gradient tool *(dBench-Logic)*
- Hearing safety dose calculator *(dBench-Audio)*
- Compressor type comparison — VCA vs. optical vs. vari-mu *(dBench-Logic)*
- Recommended compressor settings guide *(dBench-Logic)*
- Educational articles *(dBench-Logic)*
- Community forum layer *(dBench-Logic)*
- Atmospheric Conditions API and Tool integration *(dBench-Audio)*

---

## Technical Notes

**Single-file architecture** — AtmosProp is a self-contained HTML file. All CSS, JavaScript, and the embedded logo are bundled inline. No CDN dependencies, no build step, no server required.

**Browser compatibility** — Developed and verified in Firefox (desktop) and Mobile Safari (iOS). Chromium-based browsers are supported. Mobile Firefox compatibility testing is planned as part of Tier 1 development. Touch handling is implemented for pan, pinch-to-zoom, scrub, and double-tap probe interactions on mobile.

**Local storage** — Saved defaults and saved delay slots are stored in browser local storage under the keys `atmosprop_defaults_v1` and `atmosprop_delay_slots_v1`. Clearing browser site data will remove these. Use Export before clearing if you want to preserve your settings.

**No data transmission** — AtmosProp performs all calculations locally in the browser. No data is sent to any server.

---

*AtmosProp is the flagship tool of dBench — a production engineering platform in active development. Built for engineers, by engineers.*

*Development of this tool has been through the sole efforts of Erick Crumb, High Fidelity Systems Expert, Live Sound Professional, and Event Technical Specialist. Donations are warmly welcomed to encourage and hasten the development of this project.*
