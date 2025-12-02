# Wireless Medical Telemetry System - Code Analysis

## System Overview
This MATLAB program simulates a complete FM-based wireless medical telemetry system for transmitting ECG (electrocardiogram) signals from a patient to a doctor's monitoring station.

---

## Part 1: Initialization & User Interaction

### Purpose
- Clears workspace and prompts user to select patient condition
- Sets up simulation parameters based on heart condition

### Key Components
```matlab
clear; close all; clc;
choice = input('Enter your choice (A/B): ', 's');
```

### Built-in Functions
- **`clear`**: Removes all variables from workspace (No inputs, no return)
- **`close all`**: Closes all figure windows (No inputs, no return)
- **`clc`**: Clears command window (No inputs, no return)
- **`fprintf()`**: Prints formatted text to command window
  - Input: Format string and values
  - Return: None (displays to console)
- **`input()`**: Gets user input from command window
  - Input: Prompt string, format specifier 's' for string
  - Return: User's input value
- **`strcmpi()`**: Case-insensitive string comparison
  - Input: Two strings to compare
  - Return: 1 (true) if equal, 0 (false) otherwise

### Output Variables
- `heart_rate`: 70 BPM (normal) or 150 BPM (critical)
- `status`: Patient condition string
- `alarm_flag`: Boolean for emergency alert

---

## Part 2: Signal Parameters Setup

### Purpose
Defines all critical parameters for signal generation, modulation, and transmission

### Key Parameters
```matlab
fs = 2000;              % Sampling frequency
duration = 3;           % Signal duration
t = 0:1/fs:duration-1/fs;
```

### Built-in Functions
- **`length()`**: Returns number of elements in array
  - Input: Array/vector
  - Return: Integer count

### Output Variables
- `fs`: 2000 Hz sampling rate
- `t`: Time vector from 0 to 3 seconds
- `fc`: 400 Hz carrier frequency
- `kf`: 50 Hz/V frequency sensitivity
- `SNR_dB`: 18 dB signal-to-noise ratio

---

## Part 3: ECG Signal Generation (Synthetic)

### Purpose
Creates a realistic synthetic ECG signal with P, QRS, and T waves using Gaussian pulses

### Algorithm
```matlab
for beat = 0:num_beats-1
    % P-wave, Q-wave, R-wave, S-wave, T-wave
    ecg_signal = ecg_signal + amp * exp(-((t - center).^2) / (2*width^2));
end
```

### Built-in Functions
- **`floor()`**: Rounds down to nearest integer
  - Input: Numeric value
  - Return: Integer (rounded down)
- **`zeros()`**: Creates array of zeros
  - Input: Size dimensions
  - Return: Zero-filled array
- **`size()`**: Returns dimensions of array
  - Input: Array
  - Return: Size vector
- **`exp()`**: Exponential function (e^x)
  - Input: Numeric array
  - Return: Exponential of each element
- **`sin()`**: Sine function
  - Input: Angle in radians
  - Return: Sine value
- **`max()`**: Maximum value in array
  - Input: Array
  - Return: Maximum value
- **`abs()`**: Absolute value
  - Input: Numeric array
  - Return: Absolute values

### Output Variables
- `ecg_signal`: Raw synthetic ECG waveform
- `m_t`: Normalized ECG signal (message signal)

---

## Part 4: FM Modulation (Transmitter)

### Purpose
Applies Frequency Modulation to the ECG signal for wireless transmission

### FM Equation
```
s(t) = Ac * cos(2π*fc*t + 2π*kf*∫m(τ)dτ)
```

### Built-in Functions
- **`cumsum()`**: Cumulative sum (numerical integration)
  - Input: Array
  - Return: Array of cumulative sums
- **`cos()`**: Cosine function
  - Input: Angle in radians
  - Return: Cosine value
- **`pi`**: Mathematical constant π (3.14159...)
  - No input (constant)
  - Return: π value

### Output Variables
- `integral_m`: Integrated message signal
- `phase`: Phase deviation
- `fm_signal`: FM modulated signal

---

## Part 5: Channel Model (AWGN)

### Purpose
Simulates realistic wireless channel by adding white Gaussian noise

### Built-in Functions
- **`mean()`**: Average/mean value
  - Input: Array
  - Return: Scalar mean value
- **`sqrt()`**: Square root
  - Input: Numeric value/array
  - Return: Square root
- **`randn()`**: Random numbers from normal distribution
  - Input: Size dimensions
  - Return: Array of random values (mean=0, std=1)

### Output Variables
- `noise`: Gaussian noise vector
- `received_signal`: FM signal + noise

---

## Part 6: FM Demodulation (Phase Discriminator Method)

### Purpose
**MOST COMPLEX PART** - Recovers original ECG signal from noisy FM signal using industry-standard phase discriminator technique

### Multi-Stage Process

#### Stage 1: Bandpass Filtering
```matlab
[b_bp, a_bp] = butter(6, [f_low f_high]/(fs/2), 'bandpass');
received_filtered = filtfilt(b_bp, a_bp, received_signal);
```
**Functions:**
- **`butter()`**: Designs Butterworth filter
  - Input: Order, cutoff frequency(ies), filter type
  - Return: Filter coefficients (b, a)
- **`filtfilt()`**: Zero-phase digital filtering
  - Input: Filter coefficients, signal
  - Return: Filtered signal (no phase distortion)

#### Stage 2: Analytic Signal Conversion
```matlab
analytic = hilbert(received_filtered);
```
**Functions:**
- **`hilbert()`**: Hilbert transform (creates complex analytic signal)
  - Input: Real signal
  - Return: Complex signal with imaginary part as Hilbert transform

#### Stage 3: Delay & Multiply Discriminator
```matlab
analytic_delayed = [analytic(1), analytic(1:end-1)];
discriminator_output = analytic .* conj(analytic_delayed);
```
**Functions:**
- **`conj()`**: Complex conjugate
  - Input: Complex array
  - Return: Conjugated values

#### Stage 4: Phase Extraction
```matlab
instantaneous_phase_diff = angle(discriminator_output);
instantaneous_freq = instantaneous_phase_diff * fs / (2*pi);
```
**Functions:**
- **`angle()`**: Phase angle of complex number
  - Input: Complex array
  - Return: Phase in radians (-π to π)

#### Stage 5-7: Multi-Stage Low-Pass Filtering
Removes high-frequency noise and shapes bandwidth for ECG recovery

#### Stage 8-9: Amplitude Normalization
Scales recovered signal to match original amplitude

### Output Variables
- `demodulated_signal`: Recovered ECG signal

---

## Part 7: Frequency Analysis (FFT)

### Purpose
Analyzes frequency content of FM signal for visualization

### Built-in Functions
- **`nextpow2()`**: Next power of 2
  - Input: Number
  - Return: Exponent for next power of 2
- **`fft()`**: Fast Fourier Transform
  - Input: Signal, FFT length
  - Return: Complex frequency spectrum
- **`linspace()`**: Linearly spaced vector
  - Input: Start value, end value, number of points
  - Return: Evenly spaced vector

### Output Variables
- `fm_fft_magnitude`: Magnitude spectrum
- `f_axis`: Frequency axis for plotting

---

## Part 8: Visualization (Medical Dashboard)

### Purpose
Creates comprehensive 4-panel medical monitoring dashboard

### Subplots
1. **Original ECG** (Patient Side)
2. **Frequency Spectrum** (FFT Analysis)
3. **FM Modulated Signal** (Transmission)
4. **Recovered ECG** (Doctor Side)

### Built-in Functions
- **`figure()`**: Creates new figure window
  - Input: Property-value pairs
  - Return: Figure handle
- **`subplot()`**: Creates subplot in figure
  - Input: Rows, columns, index
  - Return: Axes handle
- **`plot()`**: 2D line plot
  - Input: x-data, y-data, formatting
  - Return: Line object handle
- **`xlabel()`, `ylabel()`, `title()`**: Add labels
  - Input: Text string, properties
  - Return: Text object handle
- **`xlim()`, `ylim()`**: Set axis limits
  - Input: [min, max]
  - Return: Current limits
- **`set()`, `get()`**: Set/get object properties
  - Input: Handle, property-value pairs
  - Return: Values (for get)
- **`text()`**: Add text to plot
  - Input: x, y coordinates, string
  - Return: Text object
- **`annotation()`**: Add annotation to figure
  - Input: Type, position, properties
  - Return: Annotation object
- **`legend()`**: Add legend to plot
  - Input: Labels, properties
  - Return: Legend object
- **`grid on/off/minor`**: Toggle grid display
  - No return value

---

## Part 9: Audio Alarm System

### Purpose
Generates audible alarm for critical patient conditions

### Built-in Functions
- **`sound()`**: Play audio through speakers
  - Input: Audio vector, sampling rate
  - Return: None (plays sound)

### Output
Pulsating 800 Hz alarm tone (3 beeps)

---

## Part 10: Performance Metrics

### Purpose
Quantifies system performance and signal recovery quality

### Built-in Functions
- **`corrcoef()`**: Correlation coefficient
  - Input: Two vectors
  - Return: 2×2 correlation matrix
- **`log10()`**: Base-10 logarithm
  - Input: Numeric value/array
  - Return: Logarithm

### Calculated Metrics
- **Signal Similarity**: Correlation percentage (how similar original and recovered signals are)
- **Recovered SNR**: Signal-to-noise ratio of demodulated signal
- **Frequency Deviation**: Maximum frequency shift from carrier

---

## Signal Flow Summary

```
User Input → ECG Generation → FM Modulation → Channel (Noise) 
→ Demodulation → Signal Recovery → Visualization → Performance Analysis
```

## Key Algorithms

1. **FM Modulation**: Numerical integration + phase modulation
2. **Phase Discriminator Demodulation**: Hilbert transform + delay-multiply + frequency extraction
3. **Multi-stage filtering**: Bandpass → Low-pass (2 stages)
4. **Synthetic ECG**: Superposition of Gaussian pulses (P-QRS-T waves)

---

## Critical Technical Notes

- **Sampling Rate**: 2000 Hz (adequate for ECG: 0.5-100 Hz bandwidth)
- **Carrier Frequency**: 400 Hz (chosen for visibility in spectrum)
- **Frequency Sensitivity**: 50 Hz/V (controls frequency deviation)
- **Demodulation Method**: Phase discriminator (industry standard, superior to envelope detection)
- **Filtering**: Zero-phase `filtfilt()` prevents signal distortion
