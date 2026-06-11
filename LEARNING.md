# What I'm Currently Studying

## Active Track: Linear Algebra
Working through the fundamentals with ML and graphics applications in mind.

Current focus areas:
- **Matrix decompositions** — LU, QR, eigendecomposition, SVD
  - The goal: understand *why* you factor a matrix (numerical stability, geometric clarity, dimensionality insight)
- **Positive (semi-)definite matrices** — what they are, how to guarantee them, how to parameterize them for gradient descent
- **Eigenvalues and eigenvectors** — the geometric reading: directions that only stretch, never rotate

### Where this lands in the research
The covariance matrix in 3D Gaussian splatting is stored as Σ = RSS^T R^T, where R is orthogonal (rotation) and S is diagonal (scale). This IS eigendecomposition applied to scene representation: you factor a symmetric matrix into its natural form so gradient descent never has to "stay" inside the PSD cone. Studying eigendecomposition is studying how Gaussian splatting decides where to place and how to shape every primitive in a scene.

## Up Next: Optimization
- Gradient descent, SGD, Adam — mechanics and intuitions
- Convexity and why it makes problems tractable
- Lagrange multipliers and constrained optimization

## Later: Computer Graphics Fundamentals
- Homogeneous coordinates and projection matrices
- The rendering equation (Kajiya 1986)
- Rasterization vs. ray marching vs. splatting

## Running Note
Trying to feel each prerequisite as *aimed at a target*, not as a detour from it. When a concept from the study track shows up in a paper, that's the note worth taking.
