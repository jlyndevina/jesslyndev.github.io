---
layout: page
title: SeedBot 🌱
permalink: /projects/seedbot/
order: 5
---

## The Problem

The BC greenhouse vegetable farming sector represents eight percent of agricultural cash receipts in BC (Government of British Columbia and Government of Canada). Unfortunately, roughly fifty percent of the seeds planted in greenhouses do not reach fruit bearing age.

<div class="project-hero-image">
  <img src="/assets/img/projects/traffibot/seed_1.png" alt="Journey from seed to plate" class="project-image">
</div>

Seed sorting technology is common in the farming sector, but it is limited in a variety of ways. Many methods of predicting plant traits and disease status require time consuming or destructive methods. Technologies that use non-destructive, high throughput methods are limited in what they can assess. Insporos' goal is to bridge this gap, thereby reducing the losses in the BC greenhouse sector.

For our capstone project, we partnered with **Insporos Inc.**, a developer of a seed-sorting technology designed to improve modern greenhouse operations. Their goal is to solve this problem.

---

## Our Prototype

For our test bench prototype, we prioritised iteration, scalability, and user-friendliness. The prototype will streamline our sponsor's workflow by automating repetitive tasks such as seed movement, scanning, and providing the sensors with a convenient placement.

<div class="project-hero-image">
  <img src="/assets/img/projects/traffibot/seed_2.png" alt="Seedbot Prototype" class="project-image">
</div>

Our final design meets all requirements. It comprises a 3-axis gantry from Zaber which moves a high-resolution camera, RGBW LED ring light, and the sensors required (let's call them Bob and Alice).

Take a look at this short 12 second video on how it works:

<div class="project-video">
  <iframe 
    width="100%" 
    height="400" 
    src="https://www.youtube.com/embed/UorzVzRqjPw" 
    title="Seedbot Operational Plan" 
    frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
    allowfullscreen>
  </iframe>
</div>

<div class="project-hero-image">
  <img src="/assets/img/projects/traffibot/seed_3.png" alt="System level diagram of the seed scanning process" class="project-image">
  <p><em>Figure: System level diagram of the seed scanning process.</em></p>
</div>

This design's simplicity makes it highly robust. By moving the sensors over the stationary seeds instead of moving the seeds to stationary sensors we decreased the risk of dropping or losing the seeds, since they only need to be moved once into the microwell plate. The types and arrangement of the sensors can be quickly changed by changing the mount that attaches them to the gantry, which makes the prototype versatile. It is also easy for the operator to load the seeds for interrogation since they are simply spread out by hand on the platform.

A video live from our robot demo here:

<div class="project-video">
  <iframe 
    width="100%" 
    height="400" 
    src="https://www.youtube.com/embed/oVhCLcLitak" 
    title="Seedbot Working Robot Demo" 
    frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
    allowfullscreen>
  </iframe>
</div>

---

<a href="/" class="back-link">← Back to Home</a> 