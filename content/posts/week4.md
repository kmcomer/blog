+++
title = 'Week 4: June 17-21'
date = 2024-06-24T10:13:12-04:00
+++

## Continued progress on AFF3CT Turbo codec implementation in OOT module

Last week, I continued to work on implementing the Turbo encoder and decoder blocks in GNU Radio. The blocks call the AFF3CT library to encode/decode the input vector, but the encoder block's output is not right.

I started using the debugger in VS Code to step through the code. This was helpful as I was dealing with segmentation faults and needed to track down where the code was accessing invalid memory (with the help of my mentors).

I am using simple flowgraphs to test. One exists in Python only, while the other is visualized in GRC. While the Python-only flowgraph prints output from the decoder, the QT GUI Time Sink block, which I only call in the GRC flowgraph, does not display decoded output. The input and output overlap exactly in the time sink.

Now, I am trying to determine where the error in encoding lies. I am using Daniel's LTE code, which has been validated, to compare the expected encoded output with the actual output. Then, I can work on the encoder and decoder separately with known output.
