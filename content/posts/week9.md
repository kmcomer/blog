+++
title = 'Week 9: August 5-9'
date = 2024-08-12T08:42:25-04:00
+++

Last week, I continued developing the testbench. I restructured how I counted frame errors for clarity and fixed the calculation of sigma, the standard deviation of noise, for Es/N0 (energy per signal over noise). Now the simulation matches the AFF3CT results more closely by visual inspection. However, I still need to plot the BER and FER curves to compare my use of AFF3CT through GNU Radio blocks with the AFF3CT simulation itself. Since the simulations take a long time to run, I need to run the program over night to generate data I can plot.

I started adding other codecs from AFF3CT into GNU Radio. I added AFF3CT's Repetition codec, which is simple. I added "skeleton" blocks for LDPC, Polar, TPC, and RSC codes. The code compiles and builds, but the blocks do not do anything.

This week, I will continue working on the testbench, compare the testbench to AFF3CT using plots, and continue adding the new codec blocks.