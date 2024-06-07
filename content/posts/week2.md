+++
title = 'Week 2: June 3-7'
date = 2024-06-07T10:01:02-04:00
+++

At the end of the first two weeks, I am unfortunately behind schedule. Setting up the development environment so that I can work in-tree has required troubleshooting every step of the way. Based on my issues, I plan to update the MacOS installation guide on the wiki in the next few weeks, or at least point out some outdated instructions.

I've decided to start working on new blocks in an OOT module called fec_dev before moving my code over to the main tree. This allows me to use gr_modtool to create new blocks but introduced some challenges with linking to the FEC source code. Generally, gr-fec implements different coding schemes by using a variable block to modify generic encoder and decoder blocks. This variable includes the main work functions.

My first week goal was to create the Turbo Parallel codec blocks, and my second week goal was to code 3 more. I have created Turbo encoder and decoder blocks based on gr-fec's Dummy codec. It does not do any processing and directly passes the input bits to the output.

My next step is to build the Turbo encoder and decoder blocks from the AFF3CT source code. I am still working on linking AFF3CT into my project and will then need to incorporate AFF3CT's Turbo code into the encoder and decoder blocks. Turbo Parallel also relies on a Puncturer, which I will need to add as a new block. gr-fec has a Puncture block, but I'm not yet sure how I'll need to modify it for Turbo.

Though I am already behind, I know that implementing the Turbo Parallel codec will be the biggest hurdle. From there, it will be easier to add in additional blocks.
