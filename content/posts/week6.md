+++
title = 'Week 6: July 8-12'
date = 2024-07-15T07:09:07-04:00
+++

Last week, I continued developing the Turbo codec. I added the CCSDS configuration, which runs correctly, though it still needs some work. The CCSDS implementation requires one of a few specific frame sizes (1784, 3568, 7136, or 8920 bits). Currently, my workaround pads with zeros and transmits the full corresponding codeword, which is not efficient. If CCSDS is a systematic code, I could transmit without the extra zeros and add them back before decoding.

I did a similar thing with the BCH codec to allow for more flexibility in choosing number of input frames. The encoder calculates the lowest allowable N (codeword size) for the user's number of frame bits and t (number of errors that can be corrected). It then calculates K, the number of input bits needed to produce N, and pads frame bits with (K - frame bits) zeros. Before transmitting, it deletes these extra zeros. The decoder inserts the same number of zeros to the received frame, then removes the extra zeros from the decoded output.

The BCH codec works for some settings but not all. I get a segmentation fault when t is above 7 for the frame size I have been testing with. I need to try some more inputs to track down the specific error condition.

My next main step is developing the testbench. Similar to AFF3CT's simulation tool, the testbench should generate bit error data for the user's configured SNR range. I will add random noise to the encoded message before passing to the decoder. Then, I'll plot the data to compare different configurations of the same or different decoders.