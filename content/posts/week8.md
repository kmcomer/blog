+++
title = 'Week 8: July 29 - August 2'
date = 2024-08-05T12:53:50-04:00
+++

Last week, I made progress on the testbench. I created a Python file that simulates a communication chain. The user configures the codec scheme, signal-to-noise ratio range, and message size parameters through command-line arguments. For each Eb/N0 SNR value, a loop counts the number of bit and frame errors and re-runs the flowgraph until the frame errors reaches 100 (or a different, user-specified value). Each iteration, a new random-data message is created and encoded, noisy data is added, and the data is decoded. The original message is compared to the decoded data to count the number of frame and bit errors. A command-line log updates after each iteration with frame and bit error numbers and rates, throughput, and elapsed time. When the number of frame errors reached the specified value, the data for the iteration is saved to a csv file.

I have used the testbench to simulate a communication chain for turbo codes, and I plan to plot and compare the data to an AFF3CT simulation with the same parameters. I am still working on adding user options for BCH and RA, but I still need to resolve errors with the BCH and RS codecs without noise.

Next, I plan to continue developing the testbench and adding AFF3CT-based implementations of codecs already in gr-fec, including polar, LDPC, TPC, and repetition. Then I will use the testbench to compare the two versions of each codec. This may involve some debugging or reconfiguration of the testbench to make it more adaptable. These updates will benefit the existing FEC blocks and make it easier to use the testbench for future additions.