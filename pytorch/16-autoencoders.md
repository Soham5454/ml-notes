# 16 — Autoencoders

Every architecture so far (MLP, CNN, RNN, Transformer) has been trained with labeled data — a known correct answer to predict. Autoencoders are genuinely different: they're trained to reconstruct their OWN input, with no external labels needed at all. This is the entry point into unsupervised representation learning and generative modeling.

## The core idea

An autoencoder has two parts:
- **Encoder** — compresses the input down into a smaller representation (the "bottleneck" or "latent space")
- **Decoder** — reconstructs the original input from that compressed representation

Training objective: make the reconstructed output as close to the original input as possible. The genuinely interesting part isn't the reconstruction itself (that's not useful on its own) — it's that the bottleneck forces the network to learn a **compressed, meaningful representation** of the data, since it can't just copy the input through unchanged.

## A basic autoencoder for image data

```python
import torch
import torch.nn as nn

class Autoencoder(nn.Module):
    def __init__(self, input_dim=784, latent_dim=32):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 128),
            nn.ReLU(),
            nn.Linear(128, latent_dim)
        )
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 128),
            nn.ReLU(),
            nn.Linear(128, input_dim),
            nn.Sigmoid()   # output pixels scaled to [0,1], matches normalized image input
        )

    def forward(self, x):
        latent = self.encoder(x)
        reconstructed = self.decoder(latent)
        return reconstructed, latent

model = Autoencoder(input_dim=784, latent_dim=32)   # 784 = flattened 28x28 MNIST image
```

784 input dimensions compressed down to just 32 — a genuinely aggressive compression (about 24x), and yet a well-trained autoencoder can still reconstruct recognizable digits from just those 32 numbers. That compression is exactly the "meaningful representation" being learned.

## Training loop — reconstruction loss instead of classification loss

```python
criterion = nn.MSELoss()   # or nn.BCELoss() since pixels are in [0,1]
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

for epoch in range(20):
    for images, _ in train_loader:            # note: labels are IGNORED, genuinely unsupervised
        images_flat = images.view(images.size(0), -1)   # flatten to [batch, 784]

        optimizer.zero_grad()
        reconstructed, latent = model(images_flat)
        loss = criterion(reconstructed, images_flat)      # compare output to the INPUT, not a label
        loss.backward()
        optimizer.step()
    print(f"Epoch {epoch+1}, reconstruction loss: {loss.item():.4f}")
```

Same five-step training rhythm from file 07 — the only genuine difference is what gets compared in the loss: the output against the original input itself, rather than against a separate label.

## What the latent space is actually useful for

1. **Dimensionality reduction** — same practical goal as PCA (recap from scikit-learn notes), but learned non-linearly instead of via linear projection, so it can capture more complex structure.
2. **Anomaly detection** — train an autoencoder only on "normal" data; if a new sample reconstructs poorly (high reconstruction error), it's likely anomalous, since the model never learned to compress patterns like it.
3. **Denoising** — feed in a corrupted/noisy input, train the decoder to reconstruct the CLEAN version. Forces the network to learn robust features rather than just memorizing exact pixel values.

## Denoising autoencoder — a small but genuinely useful variant

```python
def add_noise(images, noise_factor=0.3):
    noisy = images + noise_factor * torch.randn_like(images)
    return torch.clamp(noisy, 0., 1.)

# training loop change: feed noisy input, compare reconstruction against the ORIGINAL clean image
for images, _ in train_loader:
    images_flat = images.view(images.size(0), -1)
    noisy_flat = add_noise(images_flat)

    reconstructed, _ = model(noisy_flat)
    loss = criterion(reconstructed, images_flat)   # target is the CLEAN image, input was noisy
    # ... backward/step as usual
```

Genuinely elegant idea — the model never sees clean data as input during training, only noisy data, but it's always graded against the clean version. This forces it to learn what's actually signal versus what's noise.

## Variational Autoencoders (VAE) — the bridge toward true generative modeling

A regular autoencoder's latent space isn't guaranteed to be smooth or well-structured — nearby points in latent space don't necessarily decode into similar, realistic outputs. A VAE fixes this by forcing the latent space to follow a known distribution (typically Gaussian), which makes it possible to **sample** new latent vectors and decode genuinely new, realistic outputs — not just reconstructions of things already seen.

```python
class VAE(nn.Module):
    def __init__(self, input_dim=784, latent_dim=32):
        super().__init__()
        self.encoder = nn.Sequential(nn.Linear(input_dim, 128), nn.ReLU())
        self.fc_mu = nn.Linear(128, latent_dim)       # mean of the latent distribution
        self.fc_logvar = nn.Linear(128, latent_dim)   # log-variance of the latent distribution
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 128), nn.ReLU(),
            nn.Linear(128, input_dim), nn.Sigmoid()
        )

    def reparameterize(self, mu, logvar):
        std = torch.exp(0.5 * logvar)
        eps = torch.randn_like(std)
        return mu + eps * std    # the "reparameterization trick" — keeps sampling differentiable

    def forward(self, x):
        h = self.encoder(x)
        mu, logvar = self.fc_mu(h), self.fc_logvar(h)
        z = self.reparameterize(mu, logvar)
        return self.decoder(z), mu, logvar
```

The **reparameterization trick** is genuinely the clever piece here — directly sampling from a distribution isn't differentiable (breaks autograd, recap file 03), so instead the randomness is isolated into `eps` (sampled independently) while `mu` and `std` remain part of the differentiable computation graph. This is what lets gradients flow back through a "random" sampling step at all.

VAE loss combines reconstruction loss with a KL-divergence term that pushes the learned distribution toward standard Gaussian — not implementing that term's derivation in depth here since it's a genuinely deep statistics topic on its own, just flagging it exists and why (`loss = reconstruction_loss + kl_divergence_term`).

## Try this

```python
# Train the basic Autoencoder on MNIST (from file 08/10's DataLoader pattern) for 10 epochs
# Then take one test image, encode it, decode it, and visually compare original vs reconstructed
# Also try feeding in RANDOM noise as the latent vector directly into just the decoder half
# and see what garbage/interesting output comes out
```

That last part — decoding pure random noise through a plain (non-variational) autoencoder — genuinely demonstrates WHY the latent space needs the VAE's distributional constraint: a regular autoencoder's decoder usually produces garbage from random latent input, while a trained VAE's decoder produces at least roughly plausible-looking output, since its latent space was explicitly shaped to be smooth and samplable.

## What's next
File 17 — GANs, a different and genuinely more famous approach to generating realistic new data, using two networks competing against each other.
