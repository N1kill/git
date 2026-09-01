---
tags: [theory, ai, generative-models, gan, diffusion]
---

# Generative Models — GANs & Diffusion

Related: [[Autoencoders & GNNs]] · [[Neural Networks]] · [[DL]]

## The core idea
Everything covered elsewhere in this folder is mostly about *understanding* data — classifying it, predicting from it, retrieving relevant pieces of it. Generative models flip the goal: instead of mapping input to an answer, learn the underlying distribution of the data well enough to produce *new*, plausible examples that weren't in the training set — new images, new audio, new molecules. GANs and Diffusion models are the two dominant approaches, and they get there through genuinely different mechanisms.

## GANs (Generative Adversarial Networks)
Two networks trained against each other in direct competition:
- The **Generator** takes random noise and tries to produce something that looks like real data.
- The **Discriminator** looks at a mix of real and generator-produced samples and tries to tell which is which.

They're trained together in a min-max game: the generator gets better by learning to fool the discriminator, and the discriminator gets better by learning to catch the generator's fakes — each one's improvement forces the other to improve in response. At a successful equilibrium, the generator produces samples the discriminator can't reliably distinguish from real data.

**Why this is hard in practice**: this adversarial setup is notoriously unstable to train — if the discriminator gets too good too fast, the generator stops receiving a useful learning signal (nothing it does fools the discriminator, so there's no gradient telling it *how* to improve); if the generator finds one type of output that reliably fools the discriminator, it can collapse to only producing that (**mode collapse**), losing diversity.

**Use when**: you need fast generation once trained (a single forward pass through the generator), and can tolerate — or have mitigations for — the training instability. Historically strong for image generation and style transfer, though diffusion models have overtaken GANs as the default for most high-quality image generation today.

## Diffusion Models
A different strategy: learn to gradually reverse a noising process, rather than compete against a discriminator.
- **Training**: take real data, gradually add random noise to it over many steps until it's pure noise. Train a model to predict/reverse each individual noising step — i.e. given a slightly-noisier version, recover the slightly-less-noisy version.
- **Generation**: start from pure random noise, and run the trained model step by step in reverse, gradually denoising until a coherent sample emerges.

**Why this tends to work better than GANs in practice**: there's no adversarial game to destabilize — the training objective at each step is a well-defined, direct prediction task (predict the noise that was added), which makes training far more stable than a GAN's competing-network setup. The tradeoff is generation speed: producing a sample requires running the reverse process across many steps, which is much slower than a GAN's single forward pass (though a large amount of recent work focuses specifically on reducing the number of steps needed).

**Use when**: quality and training stability matter more than raw generation speed — this is why diffusion is the current default approach behind most state-of-the-art image, video, and audio generation systems.

## Choosing between them, practically
For most new projects today, **diffusion is the more common default** for generative image/audio/video work, specifically because it's more stable to train and tends to produce higher-quality, more diverse outputs. GANs remain relevant where **generation speed at inference time** is the binding constraint (real-time applications), or in specific domains (certain style-transfer and image-to-image tasks) where GAN-based approaches are still well-established and effective.

## How this connects to Autoencoders (see [[Autoencoders & GNNs]])
A VAE (Variational Autoencoder) is also a generative model — it's worth noticing where it sits relative to these two: VAEs are generally faster to sample from than diffusion models but tend to produce blurrier, lower-fidelity results than either GANs or diffusion at their best. The three approaches represent a genuine three-way tradeoff between training stability, generation speed, and output quality/diversity — no single one dominates on all three axes.
