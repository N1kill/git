---
tags: [theory, ai, cnn, computer-vision]
---

# CNNs & Computer Vision

Related: [[Neural Networks]] · [[DL]]

## The core idea
Images have structure a plain fully-connected network ignores: nearby pixels are related, and a pattern (an edge, a texture, an eye) can appear anywhere in the frame and should be recognized the same way regardless of where it shows up. A CNN builds both assumptions directly into its architecture.

Instead of learning a separate weight for every pixel, a CNN learns small **filters** (also called kernels) — tiny grids of weights, maybe 3×3 or 5×5 — and slides each one across the entire image, computing a dot product at every position. The same filter is reused everywhere in the image, which is *why* it can detect "an edge" no matter where the edge appears — this weight-sharing is the entire trick, and it's also why CNNs need far fewer parameters than a fully-connected network would to process the same image.

Stacking convolutional layers builds a hierarchy the same way depth does in any neural network: early layers' filters learn to detect simple things (edges, color gradients), middle layers combine those into textures and parts, later layers combine those into whole objects. **Pooling** layers (typically max-pooling) periodically shrink the spatial size, keeping the strongest signal from each region — this reduces computation and adds a bit of tolerance to small shifts in exactly where a feature appears.

## Where to use CNNs
Primarily image data — but the same "local pattern, position-independent" assumption extends to anything with local structure: 1D CNNs are used on audio waveforms or even text/time-series where local patterns matter more than distant relationships. For images specifically, CNNs (and increasingly Vision Transformers, when enough data/compute is available) are the default over a plain MLP, which would need an unreasonable number of parameters and wouldn't generalize a learned pattern across positions at all.

## The three vision tasks — and why they're genuinely different problems
It's tempting to think of these as "harder versions of the same thing," but each answers a structurally different question, which is why each needs its own architecture family rather than just a bigger classifier.

**Classification — "what is this image, overall?"**
Assigns one label to the entire image. The simplest task: a CNN's convolutional layers extract features, then a final fully-connected layer maps those features to a class probability.
- **Use when**: you only need to know *whether* something is present, not *where* — e.g. "does this X-ray show pneumonia."

**Object Detection — "what's in this image, and where?"**
Finds *multiple* objects and draws a bounding box around each, with a label per box. This is a fundamentally harder problem than classification because the number of objects isn't known in advance.
- **YOLO** ("You Only Look Once"): processes the whole image in a single pass, predicting boxes and classes simultaneously — fast enough for real-time use (video, robotics), at some accuracy cost relative to two-stage approaches.
- **Faster R-CNN**: a two-stage approach — first proposes candidate regions that might contain an object, then classifies each region — historically more accurate but slower.
- **Use when**: you need to know both *what* and *where*, especially with multiple objects per image (e.g. counting cars in a parking lot, detecting defects on a production line) — and choose YOLO when speed/real-time matters more than squeezing out the last bit of accuracy, Faster R-CNN when accuracy matters more than speed.

**Segmentation — "which exact pixels belong to what?"**
Classifies every single pixel, not just a box around a region. Splits further into two genuinely different goals:
- **Semantic segmentation**: labels each pixel's class, but doesn't distinguish between multiple instances of the same class (all cars are just "car," not "car #1" vs "car #2"). **U-Net** is the common architecture here, especially in medical imaging (e.g. outlining a tumor's exact boundary).
- **Instance segmentation**: does semantic segmentation *and* separates individual objects of the same class. **Mask R-CNN** extends Faster R-CNN's detection with a per-object pixel mask.
- **Use when**: bounding boxes aren't precise enough for your task — e.g. measuring the exact area of a tumor, precisely outlining a product for background removal, autonomous driving's need to know the exact drivable road surface, not just "there's a road somewhere in this box."

## Choosing between the three, practically
The question to ask is what you actually need downstream: a single "yes/no/which category" answer → classification. Needing to count or locate distinct objects, boxes are precise enough → detection. Needing exact boundaries, areas, or pixel-level precision → segmentation. Each step up costs more in labeling effort (segmentation masks are far more expensive to annotate than a single image label) and compute, so it's worth confirming you actually need the precision before reaching for it.
