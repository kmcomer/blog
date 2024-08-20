+++
title = 'Week 10: August 12-16'
date = 2024-08-20T08:45:36-04:00
+++

Last week, I worked on comparing the testbench results to AFF3CT's simulation and adding new codecs into the OOT module.

I implemented the polar codec with hard-coded values to get it up and running. Now, I need to go back and adjust the code so a user can configure the simulation to their preferences. I also started adding the turbo product code (TPC) codec but encountered errors along the way and have more work to do with it.

I created a plot_testbench tool to plot the data recorded by the testbench. Currently, it only plots the testbench's data and not AFF3CT's simulation data. I plan to restructure the testbench's created data file to match AFF3CT's format. That way, users can upload their data to AFF3CT's BER/FER comparator tool on its website and use its PyBER Python GUI tool. Users can use my plotting tool for convenience and AFF3CT's plotting tools if they want to view the data in more detail.

I revisited the BCH code implementation, which had been unexpectedly and inconsistently returning segmentation faults. I made some progress on the codec, but ultimately, I am still experiencing issues with the codec.

This week, my main focuses are to generate plots from the codecs I have working, and update incomplete but functional implementations to allow more user configurations.