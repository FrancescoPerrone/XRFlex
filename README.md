jk# Semi-quatitative spectra quantification (L1S2H33N)

## Status

## To-Do List

### High Priority:
- [ ] **Label plots with action taken**
- [X] **Implement Background Substraction**
- [ ] **Implement Matrix Correction**
- [ ] **Implement Normalisation**


### Medium Priority:
- [ ] *Design UI in C++ (proof of concept)*
- [ ] *Code documentation*

### Low Priority:
- [ ] Write presentation for Tue meeting?


## Introduction

X-ray fluorescence (XRF) spectroscopy is a non-destructive analytical technique used to determine the elemental composition of materials. XRF spectrometers measure the fluorescent (or secondary) X-ray emitted from a sample when it is excited by a primary X-ray source.

To quantify an XRF spectrum, you generally need calibration standards that match your samples as closely as possible. However, we test an approach to address situation in which there is no access to such standards. Instead, you are interested in a "relative" set of quantities, which I take to mean a semi-quantitative analysis.

In such cases, you can follow these general steps:

1. **Peak Identification**: Identify the characteristic peaks in each spectrum, which correspond to specific elements. This can be done by comparing the energies of the observed peaks with the known characteristic energies of different elements.

2. **Background Subtraction**: Once you've identified the peaks, you need to subtract the background to get accurate peak areas. There are several methods for estimating the background, such as fitting a polynomial to the non-peak regions of the spectrum.

3. **Peak Area Calculation**: After background subtraction, calculate the area under each peak. The area of a peak in an XRF spectrum is proportional to the concentration of the corresponding element in the sample. Therefore, the relative quantities of different elements can be estimated by comparing their peak areas.

4. **Matrix Correction**: Since the matrix in which an element is found can affect its fluorescence, some sort of correction is often necessary. For semi-quantitative analysis, you could use a simple correction based on the overall shape of the spectrum.

5. **Normalization**: To provide relative quantities, you can normalize the peak areas to the total peak area, which would give the fraction of each element.

That said, we are aware that this kind of semi-quantitative analysis can only provide approximate compositions, and the accuracy can be quite variable, depending on the specific elements and the matrix in which they're found. For more accurate quantification, we would need to use calibration standards that are similar to your samples.

### References:

- Bertin, E. P. (1978). Principles and practice of X-ray spectrometric analysis. Plenum Press.

- Jenkins, R., & de Vries, J. L. (2012). Practical X-ray spectrometry. Springer Science & Business Media.

- Van Grieken, R., & Markowicz, A. (2002). Handbook of X-ray spectrometry: methods and techniques. CRC Press.

- Adams, F., & Van Espen, P. (1986). Quantitative analysis by X-ray fluorescence spectrometry. In X-ray spectrometry: Recent technological advances (pp. 265-305).

## Implementation Status

1. **Peak Identification**:<br>
   **Status**: Implemented.<br>
   **Details**: The code identifies peaks using either a simple threshold method or the scipy.signal.find_peaks function. Once detected, these peaks are then matched with known elemental energies to identify the corresponding elements.

2. **Background Subtraction**:<br>
   **Status**: Partially implemented.<br>
   **Details**: While we implemented data smoothing using the Savitzky-Golay filter, which can help in reducing noise, a dedicated background subtraction method has not been incorporated. One common method for background subtraction in XRF is to fit a polynomial or another function to the parts of the spectrum where no peaks are present and then subtract this fit from the spectrum.

3. **Peak Area Calculation**:<br>
   **Status**: Implemented.<br>
   **Details**: The area under each detected peak is calculated using a simple summation around the peak. This approach assumes that the peaks are relatively narrow and the background is relatively flat in the vicinity of each peak.

4. **Matrix Correction**:<br>
   **Status**: Not implemented.<br>
   **Details**: Matrix effects can be quite complex in XRF analysis, and correcting for them usually requires knowledge of the sample composition or a standard sample for calibration. Simple corrections based on the overall spectrum shape are possible but can be limited in accuracy.

5. **Normalization**:<br>
   **Status**: Not implemented.<br>
   **Details**: While the code does quantify each peak, it doesn't normalize the peak areas to provide relative quantities. This can be a straightforward addition where after quantifying all peaks, each peak's area is divided by the total area of all detected peaks.

6. **Background Substracion**:<br>
   **Status**: Implemented.<br>
   **Details**: Matrix correction accounts for the effects that the sample matrix can have on the XRF spectrum. After applying matrix correction, you should indeed re-plot and re-quantify to reflect these corrections.

7. **Matrix Correction**:<br>
   **Status**: On development
   **Details**: 


## Implemnetation details:

## 1. Peak Detection

Peak detection is the first step in quantifying the elemental composition. 
### Mathematical Approach

The simplest approach to detect peaks is to set a threshold, $T$, and consider any data point above this threshold as a peak if it's greater than its neighboring points.

$$
f(x_i) > T \ \text{and} \ f(x_i) > f(x_{i-1}) \ \text{and} \ f(x_i) > f(x_{i+1}) 
$$

Where $f(x_i)$ is the intensity at data point $i$.

### Code Implementation

```python
def threshold_detect_peaks(data, threshold):
    peaks = []
    for i in range(1, len(data)-1):
        if data[i] > threshold and data[i-1] < data[i] and data[i+1] < data[i]:
            peaks.append(i)
   ``return peaks
```

## 2. Data Smoothing

Real-world data can often be noisy, which can hinder peak detection. A common strategy to counteract this is to use smoothing techniques.
### Savitzky-Golay Filter

The Savitzky-Golay filter is a type of convolution filter that fits a polynomial of a defined order to a window of data points. The central idea is to smooth the data while preserving the features (like peaks) in the dataset.

The smoothed value, $y_i$, for the data point $y_i$ is given by:

$$ 
y_i' = \sum_{j=-k}^{k} c_j y_{i+j}
$$

where $k$ is the window size, and $c_j$ are the convolution coefficients.

### Code Implementation

```python
from scipy.signal import savgol_filter
smoothed_counts = savgol_filter(counts, window_length=window_size, polyorder=2)
```

## 3 Quantification

Once peaks are detected, the next step is to quantify them, i.e., estimate the amount of each element present.
### Mathematical Approach

The area under each peak is proportional to the amount of the corresponding element present. For a peak detected at position $i$ the area, $A$, is approximated as:#
$$
A = \sum_{j=i-5}^{i+5} f(x_j)
$$
### Code Implementation

```python
def quantify_peaks(energy, counts, peaks):
    quantified = {}
    for peak in peaks:
        area = np.sum(counts[peak-5:peak+5])
        quantified[energy[peak]] = area
    return quantified 
```

## References

1. Savitzky, A., & Golay, M. J. E. (1964). Smoothing and differentiation of data by simplified least squares procedures. Analytical chemistry, 36(8), 1627-1639.
2. Puchinger, J., & O'Leary, D. P. (1997). Comparison of three peak detection methods for MALDI mass spectrometry data. The Journal of Chemical Physics, 107(3), 873-879.