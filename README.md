# Edax

Edax is a very strong othello program. Its main features are:
- fast bitboard based & multithreaded engine.
- accurate midgame-evaluation function.
- opening book learning capability.
- text based rich interface.
- multi-protocol support to connect to graphical interfaces or play on Internet (GGS).
- multi-OS support to run under MS-Windows, Linux and Mac OS X.

## Installation
From [the release section of github](https://github.com/abulmo/edax-reversi/releases), you must 7unzip both an executable of your favorite OS, and the evaluation weights (data/eval.dat) in the same directory.
Only 64 bit executable with popcount support are provided.

## Run

### local

```sh
mkdir -p bin
cd src

# e.g. OS X sample
make pgo-build ARCH=armv8-5a COMP=clang OS=osx
cd ..
./bin/mEdax
```

### docker

```sh
docker build . -t edax
docker run --name "edax" -v "$(pwd)/:/home/edax/" -it edax

cd /home/edax/
mkdir -p bin
cd src
make build ARCH=x86-64-v3 COMP=clang OS=linux

cd ..
curl -OL https://github.com/abulmo/edax-reversi/releases/download/v4.4/eval.7z # e.g. use v4.4 eval.dat
7z x eval.7z

./bin/lEdax-x64
```

## Changes

### 2025-7-30 add a command -x <problem file's text> to solve a text-based board

Add a new command -x <problem file's text>, like -solve <problem file>. 

**What is `-solve` command?**

Format: <board_line> <side_to_move> <expected_move (optional)>  
Sample problem file content:
--XXXXX-O-XXXO--OOXXXOX-OOOOXOX---OOOOXX--OOOOX----O------------ B
O-XXXX--O-XXX---OXXXXOXOOOXOXOX---OOOXXX--OOOOX----X------------ W

See more: ./problem/**.obf

```bash
edax -eval-file ./bin/data/eval.dat -l 10 -solve ./problem/fforum-1-19.obf
                                                                             
 # | depth|score|       time   |  nodes (N)  |   N/s    | principal variation
---+------+-----+--------------+-------------+----------+---------------------
  1|   14   +18        0:00.003         93479   31159667 g8 H7 a8 A6 a4 A7 b6
  2|   14   +10        0:00.001         32852   32852000 a4 B7 a3 A2 b8 A7 g7
  3|   14   +02        0:00.003        156099   52033000 d1 G1 b8 C1 g3 A8 g2
  4|   14   +00        0:00.001         37134   37134000 h8 B6 a7 H7 g7 A5 b7
  5|   14   +32        0:00.001         14229   14229000 g8 G7 h8 G2 b2 A2 a1
  6|   14   +14        0:00.002         73073   36536500 h3 H4 h6 A7 a8 H7 h8
  7|   14   +08        0:00.001         22615   22615000 a6 C8 b7 A7 a8 B8 h8
  8|   15   +08        0:00.003        299184   99728000 E1 h7 H6 g7 H8 g8 H2
  9|   15   -08        0:00.001         66877   66877000 G7 a7 A4 h7 H1 g1 H8
 10|   15   +10        0:00.003        156204   52068000 B2 b7 G1 g8 H8 g7 H7
 11|   15   +30        0:00.001         72557   72557000 B3 a3 A6 c3 B4 pa A2
 12|   15   -08        0:00.002        132727   66363500 B7 h2 A7 a8 H1 g1 B2
 13|   16   +14        0:00.004        115791   28947750 b7 H7 h8 A8 g8 G2 a7
 14|   16   +18        0:00.004        116035   29008750 a3 B7 a4 B2 b1 G2 a1
 15|   16   +04        0:00.006        423312   70552000 g3 F1 c1 D1 b8 A8 g1
 16|   16   +24        0:00.006        272616   45436000 f8 B6 a7 C7 h1 G7 h7
 17|   16   +08        0:00.002         35515   17757500 f8 F7 g8 H3 h7 B7 b2
 18|   16   -02        0:00.003        211938   70646000 g2 B7 a8 A7 g8 H1 f1
 19|   16   +08        0:00.005        253083   50616600 b6 B5 a6 C8 b7 A7 a8
---+------+-----+--------------+-------------+----------+---------------------
./problem/fforum-1-19.obf: 2585320 nodes in  0:00.052 (cpu =  0:00.132) (49717692 nodes/s).  

```

**New command -x**

```bash
edax -l <level:1-60> -c <color:B/W> -x <puzzle text without last color char> 
```

Example:

```bash
edax -l 10 -c B -x "--XXXXX--OOOXX-O-OOOXXOX-OXOXOXXOXXXOXXX--XOXOXX-XXXOOO--OOOOO--"     
                                                  
depth:14, time: 0:00.003, nodes:94622, nps:31540667, principal:g8 H7 a8 A6 a4 A7 b6 A2 h8 A3 h1 G2 a1 B1 
```

**Build at Macbook Air M2**

```bash
cd src/
make build ARCH=armv8.4-a CC=clang OS=osx

cd ../bin/
./mEdax-armv8.4-a -l 20 -c B -x "--XXXXX-O-XXXO--OOXXXOX-OOOOXOX---OOOOXX--OOOOX----O------------"

```

## Document

```sh
cd src
doxygen
open ../doc/html/index.html
```

## version 4.6
version 4.6 is an evolution of version 4.4 that tried to incorporate changes made by Toshihiko Okuhara in version 4.5.3 and :
 - keep the code encapsulated: I revert many pieces of code from version 4.5.3 with manually inlined code.
 - remove assembly code (intrinsics are good enough)
 - make some changes easily reversible with a macro switch (USE_SIMD, USE_SOLID, etc.)
 - remove buggy code and/or buggy file path.
 - disable code (#if 0) that I found too slow on my cpu.
 - make soft CRC32c behave the same as the hardware CRC32c (version 4.5.3 is buggy here).
 - the code switch from c99 to c17 and use stdatomic.h threads.h (if available) stdalign.h
 - remove bench.c: most of the functions get optimized out and could not be measured.
 - support only 64 bit OSes. 

## makefile
the major change is that the ARCH options are no longer the same, as they are too many possible options to enable avx2, avx512, CRC32c, etc.
Use make -help for a list of options. 


