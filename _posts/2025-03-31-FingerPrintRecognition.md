# Finger print recognition system

In the first task of Assignment 2 we were asked to:

Design a GUI and template file/database which allows you to:

- [a. Enroll a fingerprint and associate a name](#a-enroll-a-fingerprint-and-associate-a-name)
- [b. Compare a new fingerprint to the fingerprints in the gallery](#b-compare-a-new-fingerprint-to-the-fingerprints-in-the-gallery)
- [c. Evaluate your system on several fingerprints and adjust the threshold for good performance with low error rates](#c-evaluate-your-system-on-several-fingerprints-and-adjust-the-threshold-for-good-performance-with-low-error-rates)
- [d. Produce a ROC curve showing error rates versus threshold](#d-produce-a-roc-curve-showing-error-rates-versus-threshold)
- [e. Estimate the false positive rate (false alarm rate) for a false negative rate of 1%](#e-estimate-the-false-positive-rate-false-alarm-rate-for-a-false-negative-rate-of-1)

In this blogpost I will share some thoughts and results from the task that I found interesting!

---

## a. Enroll a fingerprint and associate a name

The enrollment process includes several steps from classical biometric image processing:

- **Segmentation:** Sobel filters and thresholding isolate the fingerprint area.
- **Orientation Estimation:** Local ridge directions are calculated using gradients.
- **Enhancement:** Gabor filters enhance ridge clarity.
- **Minutiae Extraction:** The image is skeletonized and ridge endings/bifurcations are detected.
- **Template Storage:** Extracted features are saved using Python’s Pickle format for later matching.

Simple GUI setup for fingerprint enrollment:

```python
image_files = [f for f in os.listdir("DB1_B") if f.endswith(".tif")]
image_dropdown = Dropdown(options=image_files, description="Image:")
name_input = Text(description="Name:")
button = Button(description="Enroll Fingerprint", button_style="success")
display(VBox([name_input, image_dropdown, button]))
```

<figure style="text-align: center;">
  <img src="/assets/GUI.png" alt="Simple GUI setup" width="70%">
  <figcaption>Figure: Simple GUI setup for fingerprint enrollment" width.</figcaption>
</figure>

---

## b. Compare a new fingerprint to the fingerprints in the gallery

To match a new fingerprint, the system extracts minutiae points and compares them to those in the stored templates. A simple distance-based score is used — the more matched points, the higher the score.

I noticed that even prints from the same finger sometimes scored worse than expected. This is likely due to small shifts, rotation, or partial overlap between scans. The matcher doesn’t currently handle alignment or distortion, which makes it sensitive to small variations. Improving this would definitely help the system be more reliable.


---

## c. Evaluate your system on several fingerprints and adjust the threshold for good performance with low error rates

I ran tests on multiple fingerprint pairs and adjusted the matching threshold to see how it affected accuracy. A higher threshold reduced false matches but also missed some genuine ones. After a bit of tuning, I found that a threshold around 14 gave the best balance between false positives and false negatives for my dataset.

---

## d. Produce a ROC curve showing error rates versus threshold

I plotted a ROC curve by sweeping through different thresholds and recording the error rates. It showed the usual trade-off: as the threshold increases, false positives go down, but false negatives go up. The Equal Error Rate (EER) — where FPR and FNR are roughly equal — was around threshold 14–15, which helped guide my tuning.

<figure style="text-align: center;">
  <img src="/assets/output.png" alt="Threshold vs Error Rate" width="70%">
  <figcaption>Figure: ROC curve showing error rates at different thresholds.</figcaption>
</figure>



---

## e. Estimate the false positive rate (false alarm rate) for a false negative rate of 1%

I wrote a simple loop to estimate the FPR when the FNR was fixed at 1%. The result was pretty extreme — the FPR shot up to over 99%, meaning the system was matching almost everything. This clearly shows that our current matching method is too basic, and more robust techniques (like alignment or smarter descriptors) are needed to improve real-world reliability.
