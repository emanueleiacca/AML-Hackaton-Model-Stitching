# 🔍 **Diagnostics Introduced (after Exp.1)**

After seeing the poor baseline results, I introduced a diagnostic suite to understand the geometry of the task before modifying the model.

### 🔧 What I added

* **Ridge Linear Transferability** → measures best possible linear map.
* **Procrustes (orthogonal vs linear)** → checks if mapping is isometric or anisotropic.
* **Mutual kNN alignment** → measures structural/topological match.

### 💡 What the diagnostics revealed

* The mapping is mostly linear.
* The geometry is strongly anisotropic.
* KL/VAE-like smoothing destroys useful structure.

### 🔍 What I found out

* Ridge achieved strong alignment → any nonlinear translator must beat this.
* Orthogonal Procrustes underperformed linear Procrustes → the mapping is not a simple rotation.
* Mutual-kNN was extremely low → the cross-modal topology is fragile.
* These confirmed that aggressive geometric constraints or priors would likely harm performance.

### 📚 Papers referenced

* *Harnessing the Universal Geometry of Embeddings*
* *Revisiting Model Stitching*
* *Platonic Representation Hypothesis*
* *Relative Representations Enable Zero-Shot Communication*

---

# 🧪 **Experiment 2 — Deterministic Contrastive Translator**

This was built as a direct response to the diagnostics pointing to: “avoid smoothing, keep the mapping simple”.

### 🔧 What I applied

* **Removed KL + sampling** → avoids Gaussian smoothing.
* **Symmetric InfoNCE** → CLIP-style contrastive alignment.
* **Learned temperature** → stabilizes gradient scaling.
* **Memory-bank hard negatives** → stronger contrastive signals.
* **Ridge distillation** → pushes model toward optimal linear map.
* **Label smoothing** → more stable CE gradients.

### 🔍 What I found out

* This experiment gave a **slight improvement**.
* Removing KL was critical; contrastive alignment helped.
* However, conflict between InfoNCE and ridge teacher prevented major gains.
* Hard negatives increased instability.

### 📚 Papers referenced

* **CLIP** (InfoNCE, temp scaling)
* **MoCo / SupCon** (memory bank)
* **Universal Geometry** (ridge distillation)
* **Relative Representations** (angle preservation)

---

# 🧪 **Experiment 3 — Multi-Positive SupCon + Geometry Polishing**

This attempt tried to use multi-caption positives + geometric constraints.

### 🔧 What I applied

* **Multi-positive SupCon** → uses all captions for an image.
* **Grouped sampling by image-ID** → ensures multi-view training.
* **Whitening + orthoreg** → enforces cleaner geometry.
* **Uniformity loss** → spreads embeddings uniformly.
* **Ridge-blend + dynamic clamp** → stabilizes norms.
* **Optional k-reciprocal re-ranking** → improves final retrieval.

### 🔍 What I found out

* Performance **worsened**.
* Too many geometric constraints conflicted with the natural anisotropy.
* Whitening + uniformity flattened the structure → bad for retrieval.
* SupCon created tight local clusters but harmed global ranking.

### 📚 Papers referenced

* **SupCon (Khosla et al.)**
* **Revisiting Model Stitching / Procrustes**
* **Platonic Representation Hypothesis**
* **SimCLR** (uniformity)
* **Retrieval re-ranking literature**

---

# 🧪 **Experiment 4**

This phase focused on stability and avoiding harmful geometry modifications.

### 🔧 What I applied

* **Mixup warmup** → smoother early gradients.
* **Small Gaussian noise conditioning** → robustness (borrowed from diffusion models).
* **Fixed-temperature CE** → prevents temperature collapse.
* **Hard triplet (margin=0.06)** → stronger local ranking.
* **DynamicThresholdingClamp** → inspired by Imagen; prevents norm blow-ups.
* **Ridge-blend inference** → improves ranking quality.
* **Removed EMA, KL** → avoid instability.

### 🔍 What I found out

* Performance **decreased**.
* Triplet + CE + geometric control created conflicting optimization signals.
* Noise + clamp altered latent scales in ways that hurt retrieval.
* The model became stable but not expressive enough.

### 📚 Papers referenced

* **CLIP stability heuristics**
* **Imagen diffusion model** (dynamic thresholding)
* **Diffusion noise conditioning**
* **Triplet margin loss literature**