---
layout: default
title: AI Infrastructure
description: A hands-on Machine Learning Systems course on GPU/NPU architecture, accelerator programming, distributed training, LLM inference, and agentic AI systems development.
---

<section class="course-hero">
  <div class="wrap">
    <p class="eyebrow">Machine Learning Systems · Fall 2026</p>
    <h1>AI Infrastructure</h1>
    <p class="lead">Learn the full stack behind modern AI—and build, profile, and optimize the systems that make it work.</p>
    <div class="course-meta" aria-label="Course information">
      <span><strong>Instructor</strong> Li Shang</span>
      <span><strong>Term</strong> Term 1, AY 2026–2027</span>
      <span><strong>Format</strong> Lectures + in-class hackathons</span>
      <span><strong>Contact</strong> <a href="mailto:lishang@slai.edu.cn">lishang@slai.edu.cn</a></span>
    </div>
  </div>
</section>

<div class="course-content wrap" markdown="1">

## Course overview {#overview}

AI Infrastructure—also known as Machine Learning Systems—examines how algorithms, systems software, and hardware are co-designed to make AI workloads efficient, scalable, and reliable. The course spans GPU and NPU architecture, memory hierarchies, communication systems, accelerator programming, compilers and runtimes, distributed training, and large language model (LLM) inference.

This is a hands-on, engineering-driven course. Lectures establish architectural and systems foundations; in-class hackathons ask students to build, profile, debug, and optimize components used in modern training and inference systems. The work emphasizes quantitative reasoning, reproducible measurement, and iterative improvement.

<div class="callout">
  <strong>A central theme: agentic AI for systems development.</strong>
  Students will explore multi-agent workflows for accelerator profiling, kernel generation and optimization, experiment management, and training and inference infrastructure—while learning how to verify correctness and performance.
</div>

The course will also feature industry-defined project opportunities. Candidate topics may be contributed by AI-infrastructure organizations, including Huawei, Tencent, and ByteDance. Final partners, topics, awards, research opportunities, and internships are subject to confirmation.

### Prerequisites

Recommended preparation includes computer architecture, operating systems, machine learning, and compiler fundamentals. Basic programming proficiency is expected; prior GPU or NPU programming experience is helpful but not required.

## Learning outcomes {#outcomes}

By the end of the course, students will be able to:

1. **Explain the architectural foundations of modern AI infrastructure** and analyze how workloads interact with accelerators, memory, interconnects, compilers, runtimes, training frameworks, and inference systems.
2. **Develop practical optimization skills** by implementing, profiling, debugging, and optimizing accelerator kernels and system components.
3. **Apply systems thinking and hardware–software co-design** to identify bottlenecks and translate algorithmic requirements into efficient implementations.
4. **Build and evaluate agentic systems-development workflows** for profiling, analysis, kernel generation, debugging, experiment management, and optimization.
5. **Work across emerging AI-computing ecosystems**, including GPU, NPU, and 国产算力生态 platforms, with attention to portability and deployment.
6. **Design next-generation intelligent computing systems** using system expertise, empirical evidence, and agentic methods.

## Tentative schedule {#schedule}

| Week | Theme | Format | Topic |
|:---:|---|---|---|
| 1 | Introduction | Lecture | The Intellectual Map of AI Infrastructure |
| 2 | GPU/NPU Architecture | Lecture | GPU Architecture |
| 3 | GPU/NPU Architecture | Lecture | NPU Architecture and 国产算力生态 |
| 4 | CUDA Programming | Lecture | CUDA Programming Through the Lens of GPU Architecture |
| 5 | CUDA Programming | Lecture | CUDA Programming as Hardware–Software Co-design |
| 6 | GPU Kernel Development | In-class Hackathon | Agentic CUDA Kernel Development and Optimization |
| 7 | NPU Kernel Development | In-class Hackathon | Agentic NPU Kernel Development and Optimization |
| 8 | NPU Kernel Development | In-class Hackathon | Agentic FlashAttention Development and Optimization on NPUs |
| 9 | LLM Training | Lecture | LLM Training Systems A–Z: Data, Tensor, Pipeline, and Expert Parallelism |
| 10 | LLM Training | Lecture | Advanced Topics in Large-scale LLM Training Systems |
| 11 | LLM Training | In-class Hackathon | Agentic Communication-kernel Development and Optimization |
| 12 | LLM Training | In-class Hackathon | Agentic Parallel-training Infrastructure Development and Optimization |
| 13 | LLM Inference | Lecture | LLM Inference Systems A–Z: Performance Optimization |
| 14 | LLM Inference | In-class Hackathon | Agentic LLM-inference Infrastructure Development and Optimization |

## Assessment {#assessment}

Assessment emphasizes engineering ability, quantitative evidence, reproducibility, and clear technical communication.

| Component | Primary evidence |
|---|---|
| In-class hackathons | Correct implementations, profiling evidence, engineering logs, and concise reflections |
| Industry-defined semester project | System design, measured improvement over a baseline, reproducibility package, final report, and demonstration |
| Technical communication and participation | Design reviews, presentations, constructive collaboration, and evidence-based discussion |

> **To be finalized:** component weights, submission rules, late-work policy, and individual/team contribution requirements will be confirmed before the course begins.

## Readings and tools {#resources}

The course does not rely on a single required textbook. Readings will be drawn from research papers, architecture and programming manuals, compiler and runtime documentation, technical reports, and instructor notes.

- **Core tools:** Git, Python, PyTorch, accelerator programming toolchains, profilers, and experiment-tracking utilities.
- **Compute access:** environment setup, submission channels, and platform-specific instructions will be announced before the relevant labs.
- **Course materials:** announcements and required readings will be published here at **mlsys.github.io**.

## Course expectations

**AI and automation tools.** Their use is expected when appropriate, but students remain responsible for correctness, safety, and performance claims. Submissions should disclose material tool use and preserve the prompts, configurations, generated artifacts, and validation evidence needed to reproduce the work.

**Teamwork and attribution.** Collaboration may be permitted for hackathons and the semester project. Each submission must identify contributors, external sources, reused code, and individual responsibilities.

**Reproducible and responsible computing.** Students should retain code, configurations, environment details, experiment logs, and baselines. Credentials, restricted data, and shared computing resources must be handled according to institutional and platform policies.

<p class="closing-note">The schedule is tentative and may change in response to platform availability, project partnerships, and developments in the field.</p>

</div>
