# ISL Tool — Development Notes
### dBench-Audio | Tier 1 Planned

This document outlines the recommended development path for the ISL (Inverse Square Law) Tool, the next major tool in the AtmosProp / dBench-Audio family. It is intended as a working reference for the developer and is not user-facing.

---

## What the ISL Tool Is

The ISL Tool extends AtmosProp's atmospheric absorption calculations by adding the geometric spreading component — the loss of level with distance due to the physics of wave propagation — and modeling how that spreading behavior differs across source types encountered in live production: point sources, line sources, and horns/waveguides.

Together with atmospheric absorption (already handled by AtmosProp), geometric spreading gives the engineer a complete picture of total SPL loss with distance for a given source and set of conditions.

---

## Core Physics to Implement

### 1. Inverse Square Law — Point Source
The foundation. Every doubling of distance produces a 6 dB drop in SPL.

```
SPL(r) = SPL(r₀) − 20 × log₁₀(r / r₀)
```

Where `r₀` is the reference distance (typically 1 m) and `r` is the target distance. This is free-field, on-axis, omnidirectional behavior. It is the correct baseline for a single loudspeaker or cluster behaving as a point source at the distances relevant to live production (beyond the near field).

### 2. Line Source Behavior
A true line source (or a vertically continuous line array operating in its near field) spreads cylindrically rather than spherically — every doubling of distance produces only a 3 dB drop.

```
SPL(r) = SPL(r₀) − 10 × log₁₀(r / r₀)
```

In practice, no real line array behaves as a pure line source at all distances. The transition from line-source behavior (−3 dB/doubling) to point-source behavior (−6 dB/doubling) occurs at a crossover distance dependent on the array length. This transition is one of the most practically useful things to model.

**Crossover distance approximation:**
```
r_crossover ≈ L² / λ
```
Where `L` is the array length and `λ` is the wavelength at the frequency of interest. Below this distance the array behaves as a line source; above it, as a point source.

### 3. Horn / Waveguide Behavior
Horns and waveguides control directivity and therefore modify the effective spreading behavior within their coverage pattern. Within the horn's coverage angle, the level drop can be significantly less than the ISL would predict because energy is concentrated. Outside the coverage angle, level drops off steeply.

This is best handled via Directivity Index (DI) or Q factor:

```
DI = 10 × log₁₀(Q)
Q = 10^(DI / 10)
SPL_on_axis(r) = SPL(r₀) − 20 × log₁₀(r / r₀) + DI
```

For the initial implementation, a simplified model using a single DI/Q value is appropriate. Full polar data import can be a Tier 2 enhancement.

### 4. Combined Loss — Spreading + Absorption
The full picture for any given source, frequency, and distance:

```
Total Loss(f, r) = Spreading Loss(r) + Atmospheric Absorption(f, conditions, r)
```

AtmosProp already calculates the right half of this equation correctly. The ISL Tool adds the left half and combines them.

---

## Practical Scope for Live Production

Based on real-world production practice, the tool should be optimized for:

- **Distance range:** 1 m to approximately 100 m. Delay stacks are typically deployed every 30–60 m; 100 m from a single source is an outer limit for practical coverage without reinforcement.
- **Frequency range:** Inherit AtmosProp's 50 Hz–20 kHz range. Atmospheric absorption effects are negligible below ~500 Hz at production distances, so the tool's combined output will naturally emphasize the mid-to-high frequency region where both spreading and absorption are meaningful.
- **Source types:** Point source (single cabinet or cluster), line source (vertical array), horn/waveguide (with DI/Q input). These cover the vast majority of live production scenarios.

---

## Validation Checklist

These are the checks to run before the tool is considered production-ready. Adapted from ASA S1.26 guidance and general propagation modeling best practice.

**Spreading math**
- Verify point source uses `20 × log₁₀(r / r₀)` — the pressure form for SPL, not the intensity form
- Verify line source uses `10 × log₁₀(r / r₀)`
- Verify sign convention: loss increases with distance (negative dB values)
- Verify reference distance handling is consistent — `r₀ = 1 m` as default, user-configurable

**Combined loss**
- Confirm atmospheric absorption is applied per frequency band, not as a single broadband term (this is already correct in AtmosProp; carry the behavior forward)
- Confirm spreading loss is added to absorption loss, not multiplied
- Verify total loss at reference distance equals zero (or the stated reference level)

**Horn/waveguide DI**
- Verify `DI = 10 × log₁₀(Q)` and `Q = 10^(DI/10)` conversions are correct
- Confirm on-axis gain is applied correctly and does not affect off-axis calculations unless polar data is provided
- Validate that DI = 0 (Q = 1, omnidirectional) produces identical results to the point source model

**Line array crossover**
- Verify crossover distance calculation is frequency-dependent
- Confirm the model transitions smoothly from line-source to point-source behavior at the crossover distance
- Test at multiple frequencies (125 Hz, 1 kHz, 4 kHz, 16 kHz) to confirm frequency-dependent behavior

**Numerical stability**
- Test at very short distances (0.5 m, 1 m) and at maximum practical distance (100 m)
- Confirm no overflow, underflow, or divide-by-zero conditions

**Unit consistency**
- Confirm meters/feet, dB, Hz, °C/°F, % RH, and kPa/inHg all handled consistently
- Confirm IMP/MET toggle applies to all distance and temperature inputs and outputs

---

## Suggested UI Structure

The ISL Tool should feel like a natural extension of AtmosProp — same design language, same atmospheric conditions panel (shared), same canvas interaction model.

**Left panel inputs:**
- Source type selector (Point Source / Line Source / Horn-Waveguide)
- Array length field (visible only when Line Source is selected)
- DI / Q input field (visible only when Horn-Waveguide is selected)
- Reference distance and reference SPL
- Distance positions (inherit the Live Positions pattern from AtmosProp)

**Canvas:**
- X axis: frequency (same log scale as AtmosProp)
- Y axis: total SPL loss in dB
- Separate toggleable layers for spreading loss alone, absorption alone, and combined total
- Live traces per position, same color coding as AtmosProp

**Readout:**
- Hover: frequency + total loss at cursor
- Probe: same scrub behavior as AtmosProp absorption canvas

---

## Tier 2 Enhancements (not for initial release)

These are noted here for reference but should not block the Tier 1 launch:

- Polar data import (CSV or standard format) for full off-axis horn/waveguide modeling
- Path segmentation for varying conditions along distance (relevant for very long outdoor throws; less critical at production distances)
- SPL overlay with Equal Loudness contours (this is the separate SPL/Distance tool in the Tier 2 roadmap but shares infrastructure with the ISL Tool)
- Line array impulse response analysis

---

## Relationship to Existing AtmosProp Code

- The `iso9613alpha()` function in AtmosProp calculates `α` (dB/m) correctly and can be called directly from the ISL Tool's combined loss calculation
- The `speedOfSound()` function is not needed in the ISL Tool (it belongs to the Delay tool) but the atmospheric conditions panel it reads from is shared infrastructure
- The canvas rendering, zoom/pan/probe system, and live position architecture are all reusable with minimal modification

The ISL Tool should be built as a new tool within the existing AtmosProp single-file structure, following the same `switchTool()` pattern used for the Absorption and Delay tools.

---

*This document is a living reference. Update it as implementation decisions are made.*
