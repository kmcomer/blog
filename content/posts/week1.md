+++
title = 'Week 1: May 28-31'
date = 2024-05-28T15:10:24-04:00
draft = false
+++

This summer, I am contributing to GNU Radio through Google Summer of Code. I recently completed my Bachelor's degree in Electrical Engineering, and I'm starting a PhD program in Networks/Communications in the fall. My project involves overhauling GNU Radio's forward error correction (FEC) / error coding package (gr-fec).

Other open source libraries like AFF3CT (https://aff3ct.github.io/) include additional FEC implementations compared to GNU Radio. I will add FEC techniques from AFF3CT's library into GNU Radio and develop a testbench to compare throughput and error rate for different techniques and implementations. My test bench will be adaptable to any existing or newly developed FEC blocks.

The GSOC coding period starts this week and lasts until the end of August. I will share my progress on the project through weekly blog posts, which will be hosted here (https://kaylacomer.github.io/blog/).

During this week, I will add encoder and decoder blocks for Turbo Parallel, using the AFF3CT library.

Feel free to check out the rest of my project proposal for more details and my intended coding schedule (https://www.overleaf.com/read/kxgcwzctxhxg#ac7704). 