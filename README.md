# OFDM Channel Estimation using SDRs

This repository contains tools and modules for OFDM channel estimation using Software Defined Radios (SDRs), including a GNU Radio Out-of-Tree (OOT) module for delta pulse generation.

## Contents

- **gr-delta_pulse**: GNU Radio OOT module for generating delta-like pulses for channel sounding
- **MATLAB**: MATLAB scripts and examples for OFDM channel estimation
- **python**: Python utilities for delta pulse generation

---

## Installing the GNU Radio OOT Module (gr-delta_pulse)

The `gr-delta_pulse` module provides a Delta Pulse Source block for GNU Radio Companion, useful for channel sounding and estimation with SDRs like the Pluto SDR.

### Prerequisites

- **GNU Radio 3.8 or later**
- **CMake 3.10 or later**
- **Python 3.x** with NumPy
- **Conda environment** (recommended) with GNU Radio installed

### Installation

1. **Create build directory:**
   ```bash
   cd gr-delta_pulse
   mkdir -p build
   cd build
   ```

2. **Configure with CMake:**
   ```bash
   cmake ..
   ```

3. **Build the module:**
   ```bash
   make
   ```

4. **Install the module:**
   ```bash
   sudo make install
   ```

The module should now be available in GNU Radio Companion under the category `[delta_pulse]`.

---

## Usage

### In GNU Radio Companion (GRC)

1. **Open GNU Radio Companion** (make sure you're using the conda environment if applicable)

2. **Find the block**: The block should appear in the block tree under:
   - Category: `[delta_pulse]`
   - Block: `Delta Pulse Source`

3. **Add to flowgraph**: Drag the block into your flowgraph

4. **Configure parameters**:
   - **Pulse Length (samples)**: Number of samples in each pulse (default: 1024)
   - **Amplitude**: Output amplitude, keep ≤ 1.0 for SDRs (default: 0.8)
   - **Apply Hann Window**: Apply windowing to reduce ringing (default: True)
   - **Center Pulse**: Center the pulse in the array (default: True)
   - **Repeat Pulses**: Continuously repeat pulses (default: True)
   - **Number of Pulses**: Number of pulses to generate, -1 for infinite (default: -1)

5. **Connect and run**: Connect the output to your SDR sink or visualization blocks

### In Python

```python
from gnuradio import gr
from delta_pulse import delta_pulse_source

# Create a delta pulse source block
pulse_source = delta_pulse_source(
    pulse_length=1024,
    amplitude=0.8,
    window=True,
    center=True,
    repeat=True,
    num_pulses=-1
)
```

### Example Flowgraph

A typical channel sounding flowgraph might include:

1. **Delta Pulse Source** → Generates the test pulse
2. **Throttle** → Controls sample rate (if not using hardware)
3. **UHD/USRP Sink** or **Pluto SDR Sink** → Transmits the pulse
4. **UHD/USRP Source** or **Pluto SDR Source** → Receives the signal
5. **QT GUI Time/Freq Sink** → Visualizes the received signal

---

## Block Description

### Delta Pulse Source

The Delta Pulse Source block generates a delta-like pulse suitable for channel sounding. The pulse is created using an OFDM-style IFFT approach:

1. Creates a frequency-domain signal with all ones
2. Performs an IFFT to get a time-domain impulse
3. Optionally centers the pulse in the time array
4. Optionally applies a Hann window to reduce spectral leakage
5. Normalizes to the specified amplitude

**Output**: Complex-valued samples (complex64)

**Parameters**:
- `pulse_length`: Number of samples in each pulse
- `amplitude`: Peak amplitude of the pulse (recommended ≤ 1.0 for SDRs)
- `window`: Apply Hann windowing to reduce ringing
- `center`: Center the pulse in the time array
- `repeat`: Continuously repeat pulses
- `num_pulses`: Number of pulses to generate (-1 for infinite)

---

## Troubleshooting

### Module Not Found in GRC

If the block doesn't appear in GNU Radio Companion:

1. **Verify installation**:
   ```bash
   python3 -c "from delta_pulse import delta_pulse_source; print('OK')"
   ```

2. **Check block location**:
   ```bash
   python3 -c "from gnuradio import gr; import os; print(os.path.join(gr.prefix(), 'share', 'gnuradio', 'grc', 'blocks'))"
   ls -la <path_from_above>/delta_pulse_source.block.yml
   ```

3. **Restart GRC**: Close and reopen GNU Radio Companion

4. **Check GRC preferences**: Verify block paths in GRC settings

5. **For conda environments**: Ensure GRC is launched from within the conda environment:
   ```bash
   conda activate gnuradio
   gnuradio-companion
   ```

### Import Errors in Python

If you get `ModuleNotFoundError: No module named 'delta_pulse'`:

1. **Verify Python environment**: Make sure you're using the same Python where GNU Radio is installed
   ```bash
   conda activate gnuradio
   which python3
   ```

2. **Check installation location**:
   ```bash
   python3 -c "import site; print(site.getsitepackages()[0])"
   ls -la <path_from_above>/delta_pulse/
   ```

3. **Reinstall if needed**: Follow the installation steps again

### CMake Configuration Errors

If CMake can't find GNU Radio:

1. **Check GNU Radio installation**:
   ```bash
   pkg-config --modversion gnuradio-runtime
   ```

2. **Verify CMake can find GNU Radio modules**:
   ```bash
   find /opt/homebrew -name "GrPlatform.cmake" 2>/dev/null
   # or
   find /usr -name "GrPlatform.cmake" 2>/dev/null
   ```

3. **For conda environments**: Ensure the conda environment is activated before running CMake

### Build Errors

If you encounter build errors:

1. **Check dependencies**: Ensure all prerequisites are installed
2. **Clean build**: Remove the build directory and start fresh
   ```bash
   rm -rf build
   mkdir build && cd build
   cmake ..
   make
   ```

---

## Uninstallation

To remove the module:

```bash
cd gr-delta_pulse/build
sudo make uninstall
```

Or manually remove:
- Python module: `$CONDA_PREFIX/lib/python3.x/site-packages/delta_pulse/` (or system site-packages)
- GRC block: `$CONDA_PREFIX/share/gnuradio/grc/blocks/delta_pulse_source.block.yml` (or system GRC blocks directory)

---

## License

GPL-3.0-or-later

---

## Additional Resources

- **MATLAB Examples**: See the `MATLAB/` directory for OFDM channel estimation examples
- **Python Utilities**: See `python/deltaPulse.py` for standalone delta pulse generation

---

## Notes

- The module is designed to work with conda environments, which is the recommended way to use GNU Radio
- The installation automatically detects whether you're using a conda environment or system-wide GNU Radio
- For best results with SDRs, keep the amplitude ≤ 1.0 to avoid clipping
- The Hann window is recommended to reduce spectral leakage and ringing effects
