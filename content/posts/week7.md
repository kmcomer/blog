+++
title = 'Week 7: July 15-19'
date = 2024-07-19T16:27:05-04:00
+++

This week, I added the Repeat and Accumulate (RA) codec and worked on the Reed-Solomon (RS) codec. RA codes are fairly simple, and I did not have much trouble. However, I still need to add QA codes for them and check some additional input.

RS codes are a subtype of BCH codes. I still had trouble with the BCH codes this week, and I could not get the RS codes to work at all. I am encountering a segmentation fault error, which may be due to incorrect data types and memory management. The problem is similar to what I was facing with the turbo codec a few weeks ago.

I started adapting the Python flowgraph I use to test new codecs into a testbench. So far, I have added random noise to the encoded input before it is passed to the decoder. Then, I compare the output from the decoder to the original input and count the errors. This represents one simulation in the testbench, with one Eb/N0 (SNR). The next step is to cycle through different SNRs and compare. I will then make the testbench parameters configurable, so the user chooses the data size, SNR range, and minimum number of errors.

I will not work on the project next week, as I had something come up. I am planning to add another week onto the end of the coding period to make up for this, so that I can get some more work on the project done.