# 17 — GANs (Generative Adversarial Networks)

Recap from file 16's closing note: VAEs generate new data through a smooth, statistically-constrained latent space. GANs take a genuinely different approach — two networks trained in direct competition against each other, no explicit probability distribution math required.

## The core idea — two networks, opposing goals

- **Generator (G)** — takes random noise as input, tries to produce fake data realistic enough to fool the discriminator.
- **Discriminator (D)** — a binary classifier, tries to correctly tell real data apart from the generator's fakes.

They're trained together, adversarially: the generator gets better by learning what fools the discriminator, and the discriminator gets better by learning what gives the generator away. Genuinely elegant setup — neither network is ever given a direct "make it look like X" instruction; the realism comes purely from this competitive pressure.

## The generator

```python
import torch
import torch.nn as nn

class Generator(nn.Module):
    def __init__(self, noise_dim=100, output_dim=784):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(noise_dim, 256),
            nn.ReLU(),
            nn.Linear(256, 512),
            nn.ReLU(),
            nn.Linear(512, output_dim),
            nn.Tanh()    # output in [-1, 1] — matches normalized image range, common GAN convention
        )

    def forward(self, z):
        return self.net(z)
```

`noise_dim=100` — the generator's input is genuinely just random noise (`torch.randn`), no real data involved at all. The network's whole job is learning a mapping from "random noise space" to "realistic image space."

## The discriminator

```python
class Discriminator(nn.Module):
    def __init__(self, input_dim=784):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, 512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 1),
            nn.Sigmoid()    # output: probability the input is real
        )

    def forward(self, x):
        return self.net(x)
```

`LeakyReLU` gets used here specifically (recap file 05) rather than plain ReLU — GAN training is notoriously unstable, and avoiding dead neurons (which plain ReLU risks) genuinely helps keep gradients flowing through the discriminator more reliably.

## The adversarial training loop — the genuinely tricky part

```python
G = Generator()
D = Discriminator()

criterion = nn.BCELoss()
opt_G = torch.optim.Adam(G.parameters(), lr=0.0002, betas=(0.5, 0.999))
opt_D = torch.optim.Adam(D.parameters(), lr=0.0002, betas=(0.5, 0.999))

for epoch in range(num_epochs):
    for real_images, _ in train_loader:
        real_images = real_images.view(real_images.size(0), -1)
        batch_size = real_images.size(0)
        real_labels = torch.ones(batch_size, 1)
        fake_labels = torch.zeros(batch_size, 1)

        # ---- Train Discriminator ----
        opt_D.zero_grad()
        real_preds = D(real_images)
        d_loss_real = criterion(real_preds, real_labels)

        noise = torch.randn(batch_size, 100)
        fake_images = G(noise)
        fake_preds = D(fake_images.detach())    # detach! don't backprop into G during D's step
        d_loss_fake = criterion(fake_preds, fake_labels)

        d_loss = d_loss_real + d_loss_fake
        d_loss.backward()
        opt_D.step()

        # ---- Train Generator ----
        opt_G.zero_grad()
        fake_preds = D(fake_images)              # NOT detached this time — need gradients into G
        g_loss = criterion(fake_preds, real_labels)   # generator WANTS discriminator to say "real"
        g_loss.backward()
        opt_G.step()

    print(f"Epoch {epoch+1}: D_loss={d_loss.item():.4f}, G_loss={g_loss.item():.4f}")
```

Two genuinely critical details worth flagging explicitly:

1. **`.detach()` during the discriminator's step** — when training D, the fake images are only used as input data, not something D should backpropagate through into G. `.detach()` (recap file 03) cuts the tensor off from its computation graph so gradients don't flow back into the generator during D's update.
2. **The generator's loss uses `real_labels`, not `fake_labels`** — this is the genuinely clever adversarial part. The generator's goal is literally "make the discriminator output 'real' for my fakes," so its loss is computed as if the fakes SHOULD have been classified real — driving gradients that push the generator to produce more convincing fakes.

## Why GAN training is genuinely harder than a normal training loop

Two networks improving against each other is a fundamentally less stable optimization problem than the usual single-network-toward-fixed-target setup from files 06/07. Common real failure modes worth knowing about even without deep-diving fixes here:
- **Mode collapse** — generator finds one or a few outputs that reliably fool the discriminator and stops producing diverse outputs.
- **Discriminator overpowering the generator** — if D gets too good too fast, its gradients to G become uninformative (it's just confidently right everywhere), and G stops learning.

Genuinely why GAN training often needs more careful learning rate tuning, architecture choices (DCGAN conventions, batch norm placement), and sometimes techniques like label smoothing — flagging these exist without implementing every fix here, since GAN stability alone is a deep rabbit hole.

## Generating new samples after training

```python
G.eval()
with torch.no_grad():
    noise = torch.randn(16, 100)
    generated = G(noise)
    generated = generated.view(16, 1, 28, 28)   # reshape back to image format for viewing
```

Genuinely the payoff moment — feeding pure random noise into the trained generator and getting recognizable, novel (never-seen-in-training) digit images out.

## GANs vs VAEs — the honest comparison

| | VAE | GAN |
|---|---|---|
| Training stability | Genuinely more stable | Notoriously unstable, needs tuning |
| Output sharpness | Tends toward blurrier outputs | Tends toward sharper, more realistic outputs |
| Latent space structure | Explicitly smooth/structured (recap file 16) | No explicit structure guarantee |
| Has an explicit likelihood/probability | Yes | No |

## Try this

```python
# Train the Generator/Discriminator pair on MNIST for a modest number of epochs
# Every few epochs, generate and save a small grid of fake digit images
# Watch how the outputs evolve from pure noise-looking blobs toward recognizable digit shapes
```

Watching the generated images visibly improve across training checkpoints is genuinely one of the most rewarding things to see in this whole roadmap — actual evidence of the adversarial competition working.

## What's next
File 18 — Model Deployment: getting a trained model (any of the ones built so far) out of a Jupyter notebook and into something that can actually serve predictions.
