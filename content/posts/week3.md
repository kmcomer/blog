+++
title = 'Week 3: June 10-14'
date = 2024-06-17T10:13:16-04:00
draft = false
+++

Later after reading my week 2 blog post, I realized I incorrectly said that my week 2 goal was to implement the next 3 codecs. (While I was writing my proposal, I had written that as my week 2 goal in one draft.) My week 2 goal was actually to test the turbo parallel block, write QA codes, and document the new blocks. Adding, testing, and documenting the BCH, RS, and RA codecs was the Weeks 3-4 (June 10-21) goal. Nevertheless, I am still behind on both. (Thank you Marcus for your encouraging reply last week.)

During week 3, I continued working on the Turbo parallel blocks. Currently, I have a working Turbo encoder using AFF3CT's default configuration: RSC sub-encoders and LTE standard. The LTE standard sets the Interleaver type and the polynomials that define the RSC code. I am working on expanding the encoder options to include different types of interleavers and allow more parameter customization.

I have been struggling to implement the Turbo decoder block. I am working with my mentors to figure this out, and then I will add some options for customization. Additionally, for both blocks, I need to update some of the methods that the new blocks inherit from the FEC_API class.

While I continue to make frustratingly slow progress, I want to thank my mentors, Andrej and Daniel, for being patient with me and providing help. By the end of this week (Week 4), I want to have a functional Turbo codec with QA codes. Week 5's goal is to simulate the new blocks with the BER curve generator and to fix bugs. I deliberately built next week in to catch up if I was behind. While I may not be completely caught up by the end of next week, I hope to make quicker progress once I have the first codec completed.
