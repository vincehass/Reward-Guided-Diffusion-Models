# Reward-Guided-Diffusion-Models

This repo omplements the "Reward-Guided Diffusion Models for Bayesian Inference of Cyclic Gene Regulatory Networks" paper.


The core idea is:
1.  **Base Model:** Define a diffusion model $\ppre(G)$ for graphs, representing the prior $P(G)$.
2.  **Reward:** Define a reward function $r(G) = \log P(D|G)$ based on data evidence.
3.  **Target:** Aim to sample from the posterior $P(G|D) \propto P(G) P(D|G) \approx \ppre(G) \exp(r(G))$.
4.  **Guidance:** Use techniques *from the alignment paper* to modify the reverse sampling process of the base model $\ppre(G)$ at *inference time*, incorporating the reward $r(G)$ to steer sampling towards the target posterior.

Here's where the alignment concepts appear in the structure:

1.  **Section 1 (Introduction) & Abstract:**
    *   **Explicit Framing:** They state the goal is to adapt "inference-time alignment techniques" using the "log evidence $\log P(D|G)$ as the reward function guiding the reverse diffusion process." This establishes the core application of the alignment framework.

2.  **Section 3 (Foundations: Reward-Guided Graph Diffusion):**
    *   **Direct Analogy:** This section directly mirrors the theoretical foundation of the alignment paper, applying it to graphs.
    *   **Soft Value Function $V_t(\Gt)$ (Def 1):** This is the direct analogue of the soft value function in the alignment paper, defined over noisy graph states, representing expected future log evidence under the base model.
    *   **Soft Optimal Score $\score^*$ (Def 2):** Defines the score function of the *ideal* reverse process that samples the target posterior. It shows this optimal score is the base score plus the gradient of the value function ($\score^* = \score_\psi + (1/\alpha)\nabla V_t$), exactly mirroring the optimal policy/score derivation in the alignment framework.
    *   **Theorem 1:** States that sampling with the optimal score $\score^*$ yields the target posterior distribution, providing the theoretical justification for guidance, analogous to Theorem 1/2 in the alignment paper.

3.  **Section 4 (Derivative-Free Guidance):**
    *   **Direct Application:** This section applies methods *from the alignment paper* that don't require reward gradients.
    *   **Importance Sampling (Eq.~\ref{eq:is_weights_graph}):** Directly uses the final reward $r(G_0) = \log P(D|G_0)$ to re-weight samples generated by the base model $\ppre$. This is a standard technique mentioned in the alignment context.
    *   **Value-Based Resampling:** Suggests adapting methods like SVDD, which were discussed as derivative-free guidance options in the alignment context, using an approximate value function $\hat{V}$.

4.  **Section 5 (Derivative-Based Guidance):**
    *   **Core Mechanism:** This section implements the primary guidance mechanism analogous to "classifier guidance."
    *   **Classifier Guidance Score (Eq.~\ref{eq:classifier_guidance_graph_score}):** This equation, $\score^* \approx \score_{\vpsi} + (1/\alpha) \nabla_{\Gt} r(\hat{G}_0(\Gt))$, *is* the application of classifier guidance. It modifies the base score ($\score_\psi$) at inference time using the gradient of the reward ($r(G)$ evaluated on the denoised estimate $\hat{G}_0$).
    *   **Algorithm 1:** Details the step-by-step *inference-time* procedure for calculating the reward gradient ($\mathbf{g}_r$) and modifying the score ($\score^*$) during the reverse sampling process.
    *   **Doob Transform Justification (Sec 5.3):** Provides the continuous-time theoretical underpinning (Eq.~\ref{eq:reverse_sde_graph_optimal}) for why adding the value/reward gradient to the drift correctly steers the process towards the target posterior, directly referencing the theory used in alignment.

5.  **Section 6 (Discrete Guidance):**
    *   **Adaptation:** Explores how to apply the *ideas* of derivative-based guidance (modifying generation based on reward gradients) to discrete diffusion processes, adapting the core concept from the alignment framework.

6.  **Section 7 (Tree Search):**
    *   **Guided Search:** Proposes using MCTS where the node evaluation ($Q(\Gt)$) and potentially the expansion policy are influenced by the reward $r(G_0)$ or the value $V_t(\Gt)$ derived from the alignment framework. This uses the reward/value information *at inference time* to guide the search for good graph samples.

7.  **Section 8 (Editing/Refinement):**
    *   **Guided Modification:** Describes starting diffusion from a partially noised existing graph and running the *guided* reverse process (using $\score^*$) to refine it towards the posterior. This is an inference-time application of the guided dynamics for modification rather than generation from scratch.

8.  **Section 10 (Distillation):**
    *   **Baking in Guidance:** Proposes fine-tuning the base score model $\score_\psi$ to directly output the *guided* score $\score^*$. This leverages the (slow) inference-time guidance procedure to create a faster model that approximates the guided sampling distribution.

In essence, the entire methodology proposed *after* setting up the base graph diffusion model (Sec 2) and the Bayesian reward relies on adapting and applying the concepts, theory, and algorithms developed for inference-time alignment/guidance of diffusion models. Sections 3, 5, and Algorithm 1 are the most direct implementations of this framework.
