# Rain + Solar Hybrid Water Heating System — Interactive Prototype

An educational, browser-based simulation of a hybrid water heater that combines
solar thermal heating with electricity recovered from falling rainwater
(roof → turbine → generator → battery → heating element), both feeding one
shared insulated hot-water tank. This is a **simulation/prototype for a college
project** — it does not claim the underlying concept is novel or "world's first."

## 1. How to run it locally

No installation, build step, server, or internet connection is required.

1. Unzip the project folder.
2. Double-click `index.html` (or right-click → Open with → your browser).
3. Use the sliders/dropdown on the left to set conditions, then press
   **▶ Start Simulation**.

Folder structure:
```
rain-solar-heater/
├── index.html      (page structure)
├── css/style.css   (styling + animations)
├── js/script.js    (simulation logic)
└── README.md       (this file)
```

## 2. How the simulation calculations work

All numbers are clearly labeled **SIMULATED VALUES** — they are reasonable
educational estimates for a small demo rig, not measured hardware data.

**Solar thermal heat delivered to the tank**
```
solarHeatW = (sunlight% / 100) × 600 W        // collector's peak heat output
usableSolarW = solarHeatW × 0.55              // ~55% reaches the tank water
```

**Rainwater → turbine → generator**
```
rainFlowLpm = (rainfall% / 100) × 6 L/min     // collected flow rate
turbineW    = rainFlowLpm × 1.1               // turbine mechanical→electrical output
generatorW  = turbineW × 0.85                 // generator conversion efficiency
```

**Electrical heating element**
The controller leans more heavily on generated/stored electricity as sunlight
drops (a "supplement" factor from 1× at full sun to 3× at no sun), capped at a
150 W heating element and by whatever the small buffer battery can supply:
```
supplementFactor = 1 + (1 − sunlight/100) × 2
elecHeatingW = min(150 W, generatorW × supplementFactor)
```

**Battery (30 Wh simulated buffer)**
```
batteryWh += (generatorW − elecHeatingW) × (Δt / 3600)   // clamped 0–30 Wh
```

**Water temperature**
Standard heat-energy balance, updated every simulation tick:
```
heatLossW = max(0, waterTemp − ambientTemp) × 1.2         // loss to surroundings
netHeatW  = usableSolarW + elecHeatingW − heatLossW
ΔT        = (netHeatW × Δt_seconds) / (volume_kg × 4186)  // 4186 J/kg°C = specific heat of water
waterTemp += ΔT
```
When both sunlight and rainfall are 0, `netHeatW` becomes negative (loss only),
so the water cools gradually toward ambient — matching the required behavior.

**Simulation clock:** each real 200 ms tick advances the model by 6 simulated
seconds, so temperature changes are visible within a short live demo instead
of requiring you to wait in real time.

## 3. Assumptions used

- Solar thermal collector: 600 W peak output at 100% sunlight, ~55% of that
  heat is assumed to reach the tank (piping/exchanger losses).
- Rainwater collection: up to 6 L/min at 100% rainfall intensity.
- Micro turbine: ~1.1 W generated per L/min of flow (small pico-hydro range).
- Generator efficiency: 85%.
- Electrical heating element: 150 W maximum.
- Buffer battery: 30 Wh capacity, purely to smooth turbine output for the heater.
- Ambient/room temperature: 20 °C; tank loses heat proportionally to how far
  above ambient the water is (simple lumped-loss model, not a full insulation model).
- Water: 1 liter ≈ 1 kg, specific heat 4186 J/kg°C (standard value for water).
- All constants are declared at the top of `js/script.js` so they can be
  tuned to match a real prototype's measured performance.

## 4. Suggested future hardware implementation

- **Roof + gutter routing:** simple mesh pre-filter at the gutter to stop
  leaves/debris before the storage tank.
- **Storage/filter stage:** small sediment filter + a header tank to give the
  turbine a steadier flow than raw rainfall intensity.
- **Micro turbine:** a low-cost pico-hydro turbine (e.g., Pelton-wheel style)
  sized for low-head, low-flow residential rainwater rather than a river.
- **Generator + regulator:** small DC generator with a buck/boost regulator
  and a battery management IC to protect the battery from over/under-charge.
- **Battery:** a small sealed lead-acid or LiFePO₄ pack sized to the expected
  rainfall duty cycle.
- **Heating element:** low-voltage resistive heating cartridge rated for the
  battery's output voltage, with a thermal cutoff fuse.
- **Solar thermal side:** flat-plate or evacuated-tube collector plumbed
  directly into the shared insulated tank via a heat exchanger coil.
- **Controller:** a microcontroller (e.g., Arduino/ESP32) reading a light
  sensor and a flow/rain sensor to automatically prioritize solar vs.
  rain-recovered power, mirroring the logic in `js/script.js`.
- **Safety:** all rain-recovered electrical components should stay in the
  low-voltage DC domain, with fusing, a thermal cutoff on the heater, and
  waterproof enclosures for anything near the collection path.

## 5. 30-second pitch for judges

> "Most solar water heaters go idle on cloudy or rainy days — right when
> there's the least sunlight. Our prototype recovers energy from that same
> rain: water falling off the roof spins a micro turbine, which charges a
> battery that powers a supplementary electric heating element. A simple
> controller automatically shifts between solar and rain-recovered power, so
> both sources feed one shared insulated tank. It's a low-cost way to keep a
> water heater working even when the sun isn't out."

---
*Educational simulation only. Actual turbine output, heating power, and water
temperature depend on real-world rainfall, flow rate, turbine efficiency,
solar intensity, tank insulation, and hardware specifications. Real hardware
should use appropriate low-voltage components and proper electrical protection.*
