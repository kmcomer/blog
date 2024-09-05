+++
title = 'Final /blog post: project overview and conclusions'
date = 2024-08-29T06:40:42-04:00
+++

I worked on a Forward Error Correction (FEC) project in GNU Radio for Google Summer of Code 2024. I started the project at the end of May, and I'm wrapping up now, at the beginning of September. Throughout the project, I've made /blog posts describing my progress (see the homepage of this website), though I did not post updates for the last couple of weeks, August 19-30 -- apologies.

I would like to thank my mentors, noc0lour and Daniel Estévez, for their help on the project this summer. They were patient with me from the start, as I struggled to set up the environment and link AFF3CT, and they let me extend the deadline by a couple of weeks so I could get more done on the project.

This project has been a great introduction to open-source coding. There are many unfinished parts, but there are complete sections and working, hopefully useful, code, and I intend to keep working on it as I find the time.

## Background
FEC limits communication-chain data errors by detecting and correcting bit errors. In GNU Radio, the gr-fec in-tree module contains pairs of encoder/decoders (codecs) that implement different FEC schemes. 

The gr-fec module contains several FEC codecs, including convolutional coding (CC), Low-Density Parity Check (LDPC), Polar, repetition, Turbo Product Codes (TPC), and Consultative Committee for Space Data Systems (CCSDS). The gr-fec codec blocks inherit from the generic_encoder and generic_decoder classes within the FEC API, which handle input and output. Each encoder or decoder's constructor and generic_work function define processing for the specific codec.

There are many common FEC schemes not included in gr-fec that are in other open-source codebases. AFF3CT is a command-line communication chain simulation tool built on an extensive open-source library. Available codecs include most of those in gr-fec, plus Bose, Ray-Chaudhuri, and Hocquenghem (BCH), Repeat-Accumulate (RA), Reed-Solomon (RS), Recursive Systematic Convolutional (RSC), and Turbo parallel.

My goal was to create new FEC blocks using the AFF3CT library code, prioritizing codecs not in gr-fec but expanding to all AFF3CT codecs. Additionally, I aimed to create a testbench to analyze and compare the performance of existing and new codecs within GNU Radio.

## Deliverables
In my GSOC proposal, I listed 4 deliverables for the project:
1. Add Turbo Parallel, BCH, RS, and RA blocks using AFF3CT library
2. Develop a testbench to compare throughput and error rate for multiple FEC blocks
3. Add polar, LDPC, TPC, and repetition blocks using AFF3CT library
4. Compare AFF3CT-based and pre-existing FEC implementations using testbench

I partially met deliverables 1 and 3. The Turbo and BCH blocks are functional, but the remaining blocks are in various states. I created encoder/decoder blocks and QA codes for each codec, so a contributor can start by modifying the blocks, at a minimum. I will describe the status of each codec in detail later in this post.

I completed deliverable 2 and created a bit error rate (BER) / frame error rate (FER) testbench similar to AFF3CT's, though there are many minor improvements I would like to make to the interface. The testbench is command-line, like AFF3CT, and logs the simulation data to a text file. This file is compatible with AFF3CT's [PyBER plotting GUI](https://github.com/aff3ct/PyBER) and [online BER/FER Comparator tool](https://aff3ct.github.io/comparator.html).

I did not get to deliverable 4 but did use the testbench to compare AFF3CT's simulation tool with the finished AFF3CT-based GNU Radio blocks. I will describe those results later on in this post. Of all the loose ends of the project, this is the one that I regret most not finishing. Comparing the existing blocks to the AFF3CT-based blocks is important to evaluate the usefulness of the new blocks.

## Early progress
In the first 3 weeks, I encountered a lot of setup errors, first in building GNU Radio from source, then in linking the AFF3CT library into my project. More recently, Daniel had a couple of the same issues, so I'll describe them and their fixes here.

### Compiler errors when trying to build gr-fec_dev
#### Compiling and installing AFF3CT
For reference, I followed the [instructions on AFF3CT's User Manual](https://aff3ct.readthedocs.io/en/latest/user/installation/installation.html) and compiled and installed from source. I modified two CMake Options in `CMakeLists.txt` to compile the shared and static library.
```
option(AFF3CT_COMPILE_STATIC_LIB "Compile the static library"   ON)
option(AFF3CT_COMPILE_SHARED_LIB "Compile the shared library"   ON)
```

#### `error: ‘runtime’ has not been declared`
```
error: ‘runtime’ has not been declared
  352 |         return runtime::status_t::SUCCESS;
```
Solution:

It seems that 3 AFF3CT library files have namespace errors. In `Decoder_LDPC_BP_flooding_inter.hxx`, `Decoder_LDPC_BP_vertical_layered_inter.hxx`, and `Decoder_LDPC_BP_horizontal_layered_inter.hxx`, I changed instances of `tools::runtime_error` to `spu::tools::runtime_error` and `runtime` to `spu::runtime`.

Another solution is to comment out any instances where those files are `#include`d. If there are any statements to include the entire library (`aff3ct.hpp`), comment those out too.

#### `cpptrace is not installed`
Solution:

In `aff3ct/lib/streampu/lib/cpptrace`, I ran
```
mkdir
cd build
cmake ..
make
make install
```

## gr-fec_dev
Initially, I aimed to develop new blocks in-tree, but I had issues using gr_modtool and decided to create an out-of-tree (OOT) module called gr-fec_dev. One remaining task is to port the functional blocks in-tree.

Learning how the AFF3CT library and GNU Radio's FEC API are structured was a major challenge at the beginning of the project. The AFF3CT library is optimized for simulations, and [its library examples](https://aff3ct.readthedocs.io/en/latest/user/library/examples.html) use the full communication chain, rather than just the encoder or decoder. Recognizing the files pattern and finding the right constructor for each FEC_API object was an important step.

I started by creating blocks modeled on the Dummy codec in gr-fec, which copies the input to the output without any processing. Each encoder/decoder is a FEC_API object that inherits from the generic encoder or decoder and overrides mandatory functions including `get_output_size()`, `get_input_size()`, `rate()`, and `generic_work()`. The FEC_API object has private members for the AFF3CT encoder/decoder and any other AFF3CT tools needed in the constructor (ex. interleaver) to build the encoder/decoder or needed in `generic_work()` (ex. quantizer). In the encoder, `generic_work()` simply calls `encode` on the input and output buffers. In the decoder, `volk` multiplies the input log-likelihood ratios (LLRs) by -1. Then the data is quantized by the AFF3CT quantizer and finally decoded by the AFF3CT decoder.

I have documented all the parameters in the public header files. I included many parameters that are not implemented but are AFF3CT library options, in case someone wants to add these options in the future. If a user tries to choose an option that is not relevant / has not been coded, they will get a runtime error.

One issue I faced during the project was prioritizing my time. The AFF3CT library is capable of an overwhelming number of configurations, and I probably spent more time adding uncommon user choices than I should have.

## Testbench
The testbench is a Python script that runs flowgraph simulations to generate BER and FER data. The parameters and output mimic the AFF3CT simulation. Some of the parameters in the testbench are listed below, followed by output from an AFF3CT simulation and a testbench simulation.

| Parameter | Description                                              |
|---|------------------------------------------------------------------|
| C | Encoding/decoding scheme (Turbo, BCH, etc.)                      |
| K | Information bits per frame                                       |
| N | Encoded bits per frame. Calculated automatically for most codecs |
| F | Number of frames per message                                     |
| E | Number of error frames (stop after this many frame errors)       |
| m | Minimum Eb/N0                                                    |
| M | Maximum Eb/N0                                                    |
| S | Eb/N0 step between each simulation iteration                     |

![Screenshot of AFF3CT simulation](/blog/posts/aff3ct_sim.png)

AFF3CT command-line simulation

![Screenshot of gr-fec_dev simulation](/blog/posts/gr_testbench.png)

gr-fec_dev testbench simulation

The script first creates a log file with the simulation information. For each Eb/N0, the script generates a random message of length F and creates a communication chain flowgraph: Source -> Throttle -> Unpack -> Encoder -> BPSK -> Noise -> Decoder. After the flowgraph executes, the script counts the number of bit and frame errors. If the number of frame errors is at least E, the script saves the data to the file and moves on to the next Eb/N0 value. Otherwise, it reruns the flowgraph. After each iteration of the flowgraph, the script updates the values to the command window.

One area for improvement within the testbench is allowing more user options that may be available through the AFF3CT simulation and/or in the encoder/decoder implementations. For example, while a user could run a custom Turbo code with row-column interleaving using the GNU Radio blocks, the testbench is not equipped to handle all the necessary configuration options. Right now, it might be easier to hardcode specific options than to try to use the command-line interface.

The testbench is not currently configured to run with existing gr-fec blocks. This is important, and theoretically not difficult, but I ran out of time to do it. This would pave the way for completing deliverable 4, comparing existing gr-fec blocks to new, AFF3CT-based gr-fec_dev blocks.

The trace data for the gr-fec_dev testbench is also lacking compared to the AFF3CT simulation. It would not be difficult to add more information to the header using the user configuration and default values. When running the testbench for BCH codes, the header info does not list T, the correction power. This is an important parameter and just an example, as other information is not saved that should be. This will require some additional conditional statements to determine exactly what should be printed and saved for specific parameter choices.

A current problem with the testbench is that the number of frames per message, F, must be a power of 2. If it is not, the testbench stops with an error like 
`ValueError: operands could not be broadcast together with shapes (2176,) (2112,)`

From looking at [the code for the base class encoder](https://github.com/gnuradio/gnuradio/blob/main/gr-fec/lib/encoder_impl.cc), Daniel thinks the issue may be that `ninput_items[0] < fixed_rate_noutput_to_ninput(noutput_items)` in some calls, making the last bits in the encoder input garbage. Actually, this is likely a problem within gr-fec and not the testbench itself.

<!-- Does anyone want to build a custom Turbo code with bottom-left column-row interleaving, inter-intra SIMD BCJR X2_AVX decoding? I'm not sure. Is it configurable? Still no, actually, that's a work in progress. Will it work in GNU Radio once its configured correctly? Maybe. Underlying all those questions, is it worth the time? -->


## Block parameters, status, and results
The public header files list all the parameters for the gr-fec_dev blocks and link to the appropriate page of the AFF3CT documentation for more information.

### K
K is the number of information bits and is called frame_bits or frame_size in the constructor. K is the number of input bits to the GNU Radio encoder block (output from decoder block). 

Exception: For BCH and RS, K_info is the number of frame bits / information bits, while K itself is the number of input bits to the AFF3CT encoder (output bits from the AFF3CT decoder).

### N
N is the codeword size and is the number of output bits from the GNU Radio encoder block (input to decoder block). For most blocks, N is calculated within the constructor and is not specified by the user. 

Exception: For BCH and RS, N_cw is the codeword size, and N is the number of output bits from the AFF3CT encoder (input bits to the decoder).

### Quantizer
AFF3CT uses power 2 quantization in the decoder, which has two forms: standard and fast. quant_fixed_point_pos sets the position of the decimal point in the quantized representation. quant_saturation_pos sets the number of bits used in the fixed-point representation.

The user can also choose NO quantization. There is an unimplemented CUSTOM quantization option in AFF3CT, which may be useful to someone in the future.

The public header files list the default quantization parameters for each block, which should match the [AFF3CT documentation table](https://aff3ct.readthedocs.io/en/latest/user/simulation/parameters/quantizer/quantizer.html).


### Turbo
I spent the most time working on the Turbo, and it is the most complete. Users can configure LTE, CCSDS, or CUSTOM turbo codes. The following graphs from PyBER show there is about 0.1 dB offset between the gr-fec_dev Turbo simulation (red) and the AFF3CT simulation (yellow).

![PyBER Turbo FER](/blog/posts/turbo_fer.png)

PyBER graph of Turbo FER from GR testbench and AFF3CT simulations

![PyBER Turbo BER](/blog/posts/turbo_ber.png)

PyBER graph of Turbo BER from GR testbench and AFF3CT simulations

### BCH
The BCH implementation in GNU Radio pads the input with zeros as needed due to constrained N and K values, and it casts 8-bit values to 32-bit values due to another constraint in the AFF3CT BCH polynomial calculation. I encountered a lot of errors along the way with the BCH codec. Unfortunately, the testbench results were not great for the GR/AFF3CT comparison. I chose K and T, the correction power, for the simulation so there was no zero padding (for an even comparison with the AFF3CT simulation), but the large offset in error rate indicates there might still be an error in the blocks.

![PyBER BCH FER](/blog/posts/bch_fer.png)

PyBER graph of BCH FER from GR testbench and AFF3CT simulations

![PyBER BCH BER](/blog/posts/bch_ber.png)

PyBER graph of BCH BER from GR testbench and AFF3CT simulations

### RA
The RA codec is functional, but it seems to have bugs. In the BER and FER graphs, the RA codes performed much worse and generally poorly compared to the AFF3CT simulation. I generated the data for the yellow, 'aff3ct sim' line by running an AFF3CT simulation from the command line. The red, 'aff3ct lib ref' line is from the 'ref' folder within the AFF3CT library. I am not sure why there is a difference between this data and the simulation data, but the parameters in that simulation may have been tweaked for best performance.

![PyBER RA FER](/blog/posts/ra_fer.png)

PyBER graph of RA FER from GR testbench and AFF3CT simulations

![PyBER BCH BER](/blog/posts/ra_ber.png)

PyBER graph of RA BER from GR testbench and AFF3CT simulations

### Polar
The Polar codec is functional but does not allow much user configuration compared to what is possible with AFF3CT. The polar blocks require K, N, and sigma inputs. Ideally, users would be allow to configure frozen bit generation method (GA_ARIKAN, FIVE_G, BEC, GA, TV), noise type (Sigma, Received_optical_power, Event_probability), decoder type (SC, ASCL, SCAN, SCF, SCL), and decoder implementation (NAIVE, FAST). 

With the NAIVE decoder implementation, N must be a power of 2, but this is not a requirement when using the AFF3CT command-line simulation, for NAIVE or FAST decoder implementation. It seems that AFF3CT rounds the codeword size to the next highest power of 2.

The current combination of Polar parameters that works is GA_ARIKAN frozen bit generation, Sigma noise, SC decoder, and NAIVE implementation. Running with these parameters, the AFF3CT-based GNU Radio blocks ('gr-fec_dev' curve) performed about 0.1 dB worse than the AFF3CT simulation. I attempted to incorporate the existing Polar blocks ('gr-fec' curve) to compare with the AFF3CT-based blocks. These results were quite poor, but I may have coded these incorrectly within the testbench.

![PyBER Polar FER](/blog/posts/polar_fer.png)

PyBER graph of Polar FER from GR testbench and AFF3CT simulations

![PyBER Polar BER](/blog/posts/polar_ber.png)

PyBER graph of Polar BER from GR testbench and AFF3CT simulations

### RS
Currently, running the RS codec results in segmentation faults. RS in AFF3CT is similar to BCH. I have made many bug fixes to BCH and have not looked through the RS code carefully to make sure it does not make the same mistakes. It is possible that the RS codec is very close to working, although there could be additional bugs in the BCH codec too.

### Repetition
The repetition codec requires debugging, as running qa_rep.py leads to a segmentation fault. The error may have to do with using mismatched data types, but I have not had a chance to look in detail.

### TPC
The TPC codec is not functional and needs work. When running in the testbench, the following error occurs:

```
TypeError: ufunc 'bitwise_xor' not supported for the input types, and the inputs could 
not be safely coerced to any supported types according to the casting rule ''safe''
```

I tried to test the TPC blocks using QA codes (`qa_tpc_aff3ct.py`) but encountered an error that I think is related to the error requiring the testbench's number of frames to be a power of 2, which I described above. When running the QA script, the decoder output from the flowgraph is empty.

### LDPC
I made the least progress on LDPC codes. I added parameters based on [the AFF3CT documentation](https://aff3ct.readthedocs.io/en/latest/user/simulation/parameters/codec/ldpc/codec.html), but the blocks are non-functional.

## Ongoing issues / code that needs work
- Porting functional blocks to main tree
- Using testbench with existing FEC blocks in gr-fec
- Adding more information to testbench text file about parameters chosen
- Allowing more configuration options in the testbench
- Updating GRC files for gr-fec_dev blocks
- Supporting testbench in GRC
- Supporting other configurations in functional blocks (polar)
- Fixing non-functional blocks (repetition, TPC, LDPC, RS)
- Debugging poorly-performing blocks (BCH, RA)
- Fine-tuning working blocks for best performance (turbo, polar)
- Reducing instances of repeated code by creating functions or classes
- Adding CUSTOM quantization option

