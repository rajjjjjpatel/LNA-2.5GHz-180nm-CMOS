# 2.5 GHz Low-Noise Amplifier in 180 nm CMOS

A single-stage cascode LNA with inductive source degeneration, designed for a 2.5 GHz antenna-integrated wearable receiver. The design was sized by hand, refined using Cadence operating-point parameters, and verified in Spectre.

**Course:** RFIC Design (ES679), IIT Gandhinagar
**Tools:** Cadence Virtuoso, Spectre
**Technology:** 180 nm CMOS, 1.8 V supply

The full report, with the schematic and all simulation plots, is in [`report/RFIC_Design.pdf`](report/RFIC_Design.pdf).

---

## Results at 2.5 GHz

| Parameter | Specification | Achieved |
|---|---|---|
| Supply voltage | 1.8 V | 1.8 V |
| Gain, S21 | > 18 dB | **25.94 dB** |
| Noise figure, NF | < 1.2 dB | **0.691 dB** |
| Input return loss, S11 | < −15 dB | **−20.66 dB** |
| Output return loss, S22 | < −15 dB | **−21.50 dB** |
| Reverse isolation, S12 | — | −38.56 dB |
| Input / output impedance | 50 Ω | 50 Ω |
| DC power | — | 13.1 mW |
| Input P1dB | — | −30.26 dBm |
| Stability | — | K > 1 up to 18.8 GHz |

NF stays below 1.2 dB from roughly 1.6 to 3.3 GHz. The achieved NF is only 0.14 dB above the device NFmin of 0.554 dB, so the input is close to noise-matched.

---

## Topology

A cascode common-source stage with inductive source degeneration.

- **Input match without a resistor.** The real part of the input impedance is set by the source inductor (Re{Zin} = ωT·Ls), so no noisy resistor is needed.
- **Passive voltage gain at the input.** The series-resonant input network gives voltage gain (Qin) before the transistor, which lowers its input-referred noise.
- **Cascode benefits.** The cascode device removes the Miller effect of Cgd on M1, improves reverse isolation and stability, and raises output resistance, so a single stage gives more than 18 dB of gain.

**RF path:** PORT0 → C0 → Lg → gate of M1 → M0 (cascode) → Ld load → C1 → PORT1.
**Bias:** M2 and R0 form a 1:25 current mirror that sets the bias current of M1. R1 and C2 isolate the RF gate from the bias circuit.

### Final component values

| Element | Value |
|---|---|
| M1, M0 | W = 250 µm (total), L = 0.18 µm |
| M2 | W = 10 µm, L = 0.18 µm |
| R0 (Rref) | 4.62 kΩ |
| R1 (Rbias) | 20 kΩ |
| C2 | 5 pF |
| C0 (Cin) | 10 pF |
| Lg (Q = 40) | 10.26 nH + 4.03 Ω |
| Ls (Q = 20) | 0.343 nH + 0.27 Ω |
| Ld (Q = 10) | 7.15 nH + 11.2 Ω |
| C1 (Cout) | 306 fF |

Inductors are modelled as ideal inductors with a series resistor that sets the quality factor (R = ω₀L/Q).

---

## Design approach

1. **Transistor width.** Optimum width from the noise-power trade-off, Wopt = 1 / (3·ω₀·L·Cox·Rs) ≈ 280 µm. W = 250 µm was chosen.
2. **Bias.** Id = 7 mA as a balance between gain and power.
3. **Input match.** Ls = Rs / ωT sets the 50 Ω real part. Lg resonates the input network at 2.5 GHz, also cancelling the reactance of the 10 pF coupling capacitor.
4. **Output match.** Ld resonates the drain capacitance, and the series capacitor Cout forms an L-match down to 50 Ω.
5. **Refinement.** The hand-calculation values were replaced with the transistor parameters from the Cadence operating point, and the final values were fine-tuned by solving the full small-signal circuit for S11 = S22 = 0 at 2.5 GHz.

### Hand calculation vs. simulation

| Quantity | Hand calculation | Refined calculation | Simulation |
|---|---|---|---|
| S21 | 23.2 dB | 26.7 dB | 25.94 dB |
| NF | 0.70 dB | 0.98 dB | 0.691 dB |
| NFmin | 0.80 dB | 0.61 dB | 0.554 dB |

The first-pass element values were within about 20% of the final ones, and the refined calculation matched the simulated gain within 0.8 dB.

---

## Simulations

Run in Cadence Spectre:

- DC operating point
- S-parameter and noise analysis, 1–4 GHz
- Stability factors Kf and B1f, 0.1–20 GHz
- Transient analysis with a 2.5 GHz sine input
- PSS power sweep at 2.5 GHz for the 1-dB compression point

### DC operating point

| Quantity | Hand calculation | Simulation |
|---|---|---|
| Rref (R0) | 3.93 kΩ | 4.62 kΩ |
| Iref | 0.28 mA | 0.251 mA |
| Id of M1 and M0 | 7 mA | 7.006 mA |
| VGS of M1 | 0.701 V | 0.638 V |
| VDS of M1 | ≈ 0.92 V | 1.028 V |
| VDS of M0 | ≈ 0.88 V | 0.692 V |
| Region (M0, M1, M2) | saturation | saturation |
| DC power | 13.1 mW | 13.1 mW |

Rref was raised from the hand value to 4.62 kΩ to reach Id = 7.0 mA, because channel-length modulation changes the mirror ratio.

### S-parameters

S21 = 25.94 dB at 2.5 GHz (peak of 25.98 dB at 2.48 GHz). The S11 dip lies slightly low, at 2.46 GHz (−24.2 dB), and the S22 dip is at 2.49 GHz.

### Noise figure

NF = 0.691 dB at 2.5 GHz, only 0.14 dB above the device minimum NFmin = 0.554 dB. NF stays below 1.2 dB over roughly 1.6 to 3.3 GHz.

### Stability

Kf > 1 and B1f > 0 from 0.1 GHz up to 18.81 GHz, more than seven times the operating frequency. Above 18.8 GHz Kf falls below 1, because the inductors are modelled with a frequency-independent series resistance, which leaves the input network almost lossless near fT. Real inductors have loss that rises with frequency. A 100 Ω resistor in parallel with Ls would remove this with little effect at 2.5 GHz.

### Transient response

With a 2.0 mV peak-to-peak input, the output is a clean 2.5 GHz sine of 39.14 mV peak-to-peak. The voltage gain is 19.6 V/V (25.8 dB), within 0.15 dB of S21.

### Gain compression

Small-signal gain in the sweep is 26.0 dB. The input-referred 1-dB compression point is −30.26 dBm, with about −5.3 dBm at the output.

---

## Key takeaways

- **Gate-inductor Q is the most important noise parameter.** The design assumes a high-Q (Q = 40) off-chip gate inductor. A lower-Q on-chip inductor would noticeably degrade the noise figure.
- **The output match is the most sensitive part.** A 10% change in Ld noticeably degrades S22.
- **Biasing needed simulation tuning.** The PDK width field is the total device width, and channel-length modulation shifts the mirror ratio, so Rref had to be tuned in simulation.
- **Power headroom.** The 7.9 dB gain margin leaves room to reduce bias current and power, which matters for a wearable receiver.

## Limitations and future work

- Inductors are ideal models with a series resistance, not extracted layout or foundry inductor models.
- Process and temperature corners have not been simulated.
- Layout and a two-tone IIP3 simulation are not done yet.


## References

1. A. Çağlar and M. B. Yelten, "A high linearity LNA using 180 nm CMOS technology for S-Band," in *Proc. ECCTD*, Catania, Italy, 2017, doi: 10.1109/ECCTD.2017.8093230.
2. P. K. S. K and Venkateshappa, "LNA design for low noise and high performance in 180 nm Cadence technology," in *Proc. ICITEICS*, Bangalore, India, 2024, doi: 10.1109/ICITEICS61368.2024.10624903.
3. M. Kumar et al., "Design and analysis of a single-stage cascode LNA for S-band application in 180-nm CMOS technology," *Journal of Electrical Engineering*, vol. 76, 2025, doi: 10.2478/jee2025-0055.
4. D. K. Shaeffer and T. H. Lee, "A 1.5-V, 1.5-GHz CMOS low noise amplifier," *IEEE J. Solid-State Circuits*, vol. 32, no. 5, pp. 745–759, May 1997.
5. T. H. Lee, *The Design of CMOS Radio-Frequency Integrated Circuits*, 2nd ed. Cambridge University Press, 2004.
6. B. Razavi, *RF Microelectronics*, 2nd ed. Prentice Hall, 2012.

## Author

Raj Patel
B.Tech–M.Tech Dual Degree, Electrical Engineering, IIT Gandhinagar
