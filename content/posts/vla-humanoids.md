# Training Humanoids with 14 Demonstrations

> **TLDR**: I tried to deploy a vision-language-action model on a humanoid robot. It didn't work. I spent a semester debugging, bought a robot arm to diagnose the problem, implemented multiple teleoperation methods, and eventually discovered that LoRA fine-tuning with just 14 VR-teleoperated trajectories beats full fine-tuning with 100k steps. Parameter-efficient training isn't just convenient—it's essential for small datasets.

---

## The Original Vision (and Why It Failed)

I started this project with a simple goal: get a Unitree G1 humanoid to do chemistry. Specifically, an autonomous pick-and-pour sequence for elephant toothpaste. The motivation was that embodied AI could push machine intelligence into physical domains where real scientific discovery happens—not just text generation.

What I thought would take a week turned into a semester-long education in why current VLAs are harder than they look.

Here's what I assumed:
- VLAs have out-of-box generalization
- I could use pre-collected internet data
- The hard part would be the robotics, not the ML

Here's what actually happened:
- Internet-sourced training data failed completely on my setup
- Teleoperation infrastructure for humanoids didn't exist
- Every layer of the pipeline required building from scratch

---

## Why Physical Intelligence Matters

Through the struggle, I came to a realization that changed how I think about AI.

Text-based LLMs are incredible, but they can't create anything truly novel. An LLM isn't going to formulate a chemical that cures a disease—at least not yet. The biggest wins from current AI systems come from automation-level intelligence.

But physical intelligence is different. Think about how scientists actually work: they start in the books, but once they've read everything, they go to the lab. That's where real knowledge emerges—through trial and error, creating new methods, reasoning in the physical world.

Enabling machines to learn through embodied experience could push all machine intelligence forward in ways that language models alone cannot.

---

## The Journey

### Phase 1: Infrastructure (September)

Set up Isaac Sim and Isaac Lab with the G1 humanoid. Trained a basic locomotion policy so the robot could walk. Research revealed something important: **no single model currently handles both locomotion and dexterous manipulation for humanoids**.

This led to the architecture I'm building toward—a System-1/System-2 hierarchy:
- System 2: LLM planner for reasoning and task decomposition
- System 1: Parallel execution of specialized policies (GR00T for manipulation, LeVERB for locomotion)

### Phase 2: GR00T Integration (October)

Established streaming connection between NVIDIA's GR00T N1.5 and Isaac Sim. Built a task environment with the G1 at a table with objects to manipulate.

First major problem: the G1 kept falling over. GR00T only controls the upper body—arms and hands. When it moves the arms, the center of mass shifts outside the support polygon. Fixed this by cranking up joint stiffness and damping for the lower body, essentially locking everything below the waist.

Second major problem: the model was producing identical actions regardless of the language prompt. Complete failure to generalize. I couldn't figure out why.

### Phase 3: Diagnostic Experiments (October-November)

When you can't figure out what's broken, simplify.

I bought an SO-ARM101 robot arm to test whether the problem was GR00T itself or my integration. The strategy: collect real-world teleoperation data on simpler hardware, fine-tune GR00T, deploy to physical robot.

If it works on the SO-101, the problem is my G1 pipeline or data quality.
If it fails the same way, the problem is GR00T.

**Result: It worked.** GR00T learned the task with just 20 teleoperation trajectories and deployed successfully to hardware.

Conclusion: GR00T is fine. My data was the problem.

(Side note: discovered that Feetech STS3215 servos require `Goal_Time=0` for responsive position mode. Completely undocumented. Cost me days.)

### Phase 4: Data Collection (November-December)

Here's the key insight: **current VLAs cannot generalize across substantially different scenes or task spaces.** They only handle small perturbations from the training distribution.

My original approach—training on pre-collected internet data with different normalizations, action spaces, and camera configurations—was fundamentally flawed. The data has to match the exact inference setup.

I needed to collect teleoperation data myself. Two approaches:

**VideoMimic (implemented but incomplete)**

Built a pipeline to convert webcam video of myself into robot trajectories:
- SAM2 for human segmentation
- ViTPose for 2D pose estimation (133 keypoints)
- VIMO for 3D mesh reconstruction

The pipeline works, but retargeting human poses to G1 joint positions is complex—different joint limits, link lengths, degrees of freedom. Put this on hold.

**Meta Quest VR (what actually worked)**

Bought a Meta Quest 3. Integrated Unitree's XR_teleoperate package. Direct hand and arm tracking mapped to G1 joint positions in Isaac Sim.

Way more straightforward. Immediately started collecting high-quality data.

---

## The Experiment That Mattered

I collected 14 teleoperation episodes (15,791 frames) for a simple task: move a red block to a yellow target zone.

Then I ran the central experiment of the project: **full fine-tuning vs. LoRA fine-tuning on identical data.**

### Full Fine-Tuning (100k steps): Complete Failure

- Training time: 10 hours
- Loss dropped 250x in early training, then flatlined near zero
- At inference: identical trajectory every run regardless of prompt or object position
- The model memorized the 14 trajectories instead of learning manipulation

This is catastrophic overfitting. The model has too many parameters relative to the dataset size, so it just memorizes everything.

### LoRA Fine-Tuning (8k steps): Success

- Training time: <1 hour
- LoRA rank: 16, alpha: 32
- Loss showed proper decay without premature convergence

Results:
- Distinct trajectories for each inference run
- Appropriate responses to different initial object positions
- Correct behavior changes based on language prompts
- Successful task completion

**14 trajectories. That's it.**

---

## Why LoRA Works

LoRA (Low-Rank Adaptation) freezes the pretrained weights and adds small trainable matrices. Instead of updating all parameters, you're updating a tiny subset.

For full fine-tuning with a large model and small dataset, the model can trivially memorize every example—destroying the pretrained representations that enable generalization.

LoRA constrains the parameter count, forcing the model to adapt without forgetting what it learned during pretraining.

This isn't just computationally convenient. **For small datasets, it's essential.**

| Metric | Full Fine-Tune | LoRA |
|--------|----------------|------|
| Training Steps | 100,000 | 8,000 |
| Training Time | 10 hours | <1 hour |
| Unique Trajectories | No | Yes |
| Task Success | 0% | Yes |

---

## What I Learned

1. **Data quality > model choice.** GR00T worked fine—my internet-sourced data just didn't match my setup. Don't assume pre-collected data will transfer.

2. **Diagnose systematically.** The SO-101 experiment isolated the problem in days instead of weeks of guessing. When something doesn't work, simplify until you can identify the cause.

3. **LoRA should be default for small datasets.** Full fine-tuning will overfit. This is the most important technical finding of the project.

4. **VLAs don't generalize as much as you think.** They handle small perturbations from training distribution, not substantially different scenes or tasks.

5. **Hardware configs will block you.** Undocumented settings like `Goal_Time=0` can waste days. Check everything.

---

## What's Next

**Immediate:**
- Integrate LeVERB for whole-body locomotion (so the robot can walk while manipulating)
- Sim-to-real transfer on physical G1
- More diverse tasks (pouring, tool use, bimanual manipulation)

**Data scaling with DreamGen:**
A promising direction is synthetic data generation via video world models—train a world model on real trajectories, have it "dream" imagined trajectories, filter for physical plausibility, then train policies on the combined data.

**Long-term: System-1/System-2 architecture**
- System 2: LLM planner for task decomposition and goal monitoring
- System 1: GR00T for manipulation, LeVERB for locomotion, running in parallel

This would enable complex multi-step tasks like "prepare a chemical solution" requiring reasoning, navigation, and precise manipulation.

---

## Bottom Line

What started as "deploy a foundation model on a humanoid" became an education in the current limits of embodied AI.

Physical intelligence is harder than language-based AI. Robots have to contend with real-world physics, hardware failures, and the challenge of collecting compatible training data.

But that's exactly why it matters. Enabling machines to learn through physical trial and error could push all intelligence forward in ways that text alone cannot.

Foundation model + right training strategy + good data = results.

---

*This research was conducted in KIMLAB at UIUC under Prof. Joohyung Kim, Fall 2025.*
