# GCE

GCE is a hardware design that corrects nonlinear distortion, such as the distortion a power amplifier adds to a signal. This repository has the results for GCE-REF, one block of GCE. It takes a 26-bit input word and returns the corrected output code, with much lower delay than a polynomial or a lookup table at the same accuracy.
![GCE layout](https://github.com/robinbishtt/gcebenchmark/blob/main/results/physical/gce_ref/asap7/gds_full.png)
## Benchmarks

All three designs approximate the same reference function and meet the same accuracy limit (maximum error 16 LSB, RMS error 5 LSB), checked on all 2^26 inputs.

| | GCE-REF | Polynomial, degree 6 | Piecewise-linear LUT, 32 segments |
|---|---|---|---|
| Delay on ASAP7 (7 nm), ns | 1.478 | 4.305 | 2.564 |
| Delay on sky130 HD, ns | 10.19 | 33.45 | 20.98 |
| Stored coefficient bits | 560 | 494 | 3168 |
| Area on ASAP7, um^2 | 446 | 968 | 403 |
| Area on sky130 HD, um^2 | 33616 | 73304 | 31562 |
| Maximum error, LSB | 9.63 | 5.13 | 2.88 |
| RMS error, LSB | 3.22 | 3.31 | 1.63 |

On ASAP7, GCE-REF is 2.9 times faster than the polynomial and 1.7 times faster than the LUT, and it stores 5.7 times fewer coefficient bits than the LUT. Delay and area are Yosys synthesis estimates (ASAP7 and sky130 HD libraries) and leave out wires and registers.

Files: `results/ppa/ppa_comparison.csv`, `results/accuracy/err_hist_*.csv`

## Place and route

Flow: OpenROAD-flow-scripts, from synthesis through routing and fill, with input and output registers added around the block.

| | ASAP7 | sky130 HD |
|---|---|---|
| Clock target | 2000 ps | 12 ns |
| Worst slack | +54.2 ps | -6.01 ns |
| Implied critical path | 1.946 ns | 18.01 ns |
| Die | 43.6 x 43.6 um | 410.1 x 410.1 um |
| Cell area after floorplan | 537 um^2 | 40799 um^2 |
| Total power (vectorless estimate) | 7.77 mW | 23.0 mW |
| Routing DRC / antenna violations | 0 / 0 | 0 / 0 |

Layout images for both platforms are in `results/physical/gce_ref/`: `gds_full.png` (whole die), `gds_zoom_center.png` (close view) and `floorplan.png` (cell and pin placement).

Files: `results/physical/physical_summary.csv`

## Verification

| Check | Result |
|---|---|
| RTL against golden model, 105,472 directed and random vectors | 0 errors |
| RTL against golden model, all 2^26 inputs | identical |
| Gate netlist against golden model, all 2^26 inputs | identical |
| Inferred latches | 0 |
| Injected RTL faults caught by the testbench | 13 of 13 |

Files: `results/verification/verification_summary.csv`

## Power-amplifier predistortion (OpenDPD)

The same approach was tried as a power-amplifier predistorter on the APA_200MHz dataset from the open OpenDPD project (github.com/lab-emi/OpenDPD). The predistorters were simulated on an amplifier model fitted to the measurements.

| Predistorter | ACPR, dBc | NMSE against linear, dB |
|---|---|---|
| None | -30.99 | -19.85 |
| GCE-style, memoryless | -39.64 | -22.57 |
| Memoryless polynomial, order 7 | -39.87 | -22.70 |
| Generalized memory polynomial | -38.50 | -33.26 |

The GCE-style predistorter improves ACPR by 8.65 dB and ends 0.23 dB behind the order-7 polynomial.

Files: `results/opendpd/feasibility_APA200.json`, `results/opendpd/psd_APA200.csv`

## Files

```
README.md
LICENSE
results/
  ppa/ppa_comparison.csv
  accuracy/err_hist_*.csv
  physical/physical_summary.csv
  physical/gce_ref/asap7/*.png
  physical/gce_ref/sky130hd/*.png
  verification/verification_summary.csv
  opendpd/feasibility_APA200.json
  opendpd/psd_APA200.csv
```

## License

MIT. See `LICENSE`.
