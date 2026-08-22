---
layout: home
title: "Ben Robison | RTL, FPGA & Embedded Systems"
meta_description: "UW Electrical and Computer Engineering student building RTL, FPGA, and embedded systems, including a hardware-verified 8x8 GEMM accelerator."
permalink: /index.html
homepage: true
header: false
---

<section class="home-hero">
  <div class="row">
    <div class="large-8 columns">
      <p class="home-eyebrow">UW Electrical &amp; Computer Engineering</p>
      <h1>Ben Robison</h1>
      <p class="home-lead">I build digital hardware and embedded systems, from synthesizable RTL through board-level integration and verification.</p>
      <div class="home-actions">
        <a class="button radius" href="/portfolio/">View projects</a>
        <a class="button radius secondary" href="/assets/docs/Ben_Robison_Resume.pdf">Resume (PDF)</a>
        <a class="home-text-link" href="https://github.com/Ae0lis" target="_blank" rel="noopener noreferrer">GitHub</a>
        <a class="home-text-link" href="mailto:benprobison@gmail.com">Email</a>
      </div>
    </div>
  </div>
</section>

<section class="home-section">
  <div class="row">
    <div class="small-12 columns">
      <p class="home-eyebrow">Featured project</p>
      <div class="featured-project">
        <div class="large-7 columns featured-project__copy">
          <h2>8x8 GEMM Accelerator on FPGA</h2>
          <p>A 64-MAC signed INT8 matrix-multiply accelerator built in SystemVerilog, integrated with the ARM processing system on a DE1-SoC, and verified against a Python golden model.</p>
          <div class="home-metrics">
            <div><strong>64</strong><span>parallel MACs</span></div>
            <div><strong>77.48 MHz</strong><span>post-fit Fmax</span></div>
            <div><strong>9 cycles</strong><span>core operation</span></div>
            <div><strong>1,000</strong><span>randomized interface tests</span></div>
          </div>
          <p class="home-project-links">
            <a class="button radius small" href="/projects/gemm_accelerator/">Read the project</a>
            <a class="button radius small secondary" href="https://github.com/Ae0lis/Systolic-Array" target="_blank" rel="noopener noreferrer">View source</a>
          </p>
        </div>
        <div class="large-5 columns featured-project__image">
          <img src="/images/projects/gemm-de1-soc.jpg" alt="DE1-SoC development board used for the GEMM accelerator">
        </div>
      </div>
    </div>
  </div>
</section>

<section class="home-section home-section--muted">
  <div class="row">
    <div class="small-12 columns">
      <p class="home-eyebrow">More hardware work</p>
      <div class="home-card-grid">
        <article class="home-card">
          <span class="home-card__type">PCB + controls</span>
          <h3><a href="/projects/line_robot/">Line-Following Robot</a></h3>
          <p>Designed a seven-channel sensor PCB, assembled the sensing hardware, and tuned the controller through a full-track finish.</p>
          <a class="home-card__link" href="/projects/line_robot/">Read the project <span aria-hidden="true">&rarr;</span></a>
        </article>
      </div>
    </div>
  </div>
</section>
