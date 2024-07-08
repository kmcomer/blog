+++
title = 'Week 5: June 24-28'
date = 2024-07-08T09:02:33-04:00
+++

This update is for my progress two weeks ago, June 24-28. Last week, I took a vacation and did not work on the project.

### Turbo codec

The Turbo encoder and decoder blocks now work! In the previous code, I had a typo in the default argument for the polynomials parameter of the encoder, so the encoded and decoded implementation did not match. The quantizer for the decoder also had a misconfigured parameter.

I still need to expand the Turbo codec to include CCSDS in addition to LTE, along with more QA tests. Andrej created a QA test helper and basic test, so I can use that to create further tests. I also need to document my code and how I have used the AFF3CT library. One problem with the current FEC code is that it lacks clear documentation, so I want to make sure to explain everything in detail.

### BCH, RA, and RS

I started working on a few more codecs: Bose-Chaudhuri-Hocquenghem (BCH), Repeat-Accumulate (RA), and Reed-Solomon (RS) codes. Like the Turbo codec, I started by creating new encoder and decoder blocks that pass the input directly to the output, mimicking the Dummy codec in gr-fec. I used gr-modtool and kept track of the changes I made in a guide for myself. If it would be useful to others, I could develop the guide to explain how to add an external library's codec into an OOT module.

I have only started integrating the AFF3CT library code for one of the new codecs: BCH. RA and RS are simply Dummy codecs. The BCH encoder and decoder are partially functional. With a certain length input vector, the output of the decoder matches. For other input, the blocks decode part of the message but cut off the end. I have a little more work to do to fix the blocks.