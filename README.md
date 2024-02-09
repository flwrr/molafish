<img src="https://i.imgur.com/anHs9JC.png" width=60% height=60%>

## Introduction
Molafish is a project to convert the Sunfish chess engine into a bitboard-based engine capitalizing on Python's bitwise operators and advanced bitboard techniques. Through simplicity and readability, Sunfish offers an excellent entry point to chess programming. Molafish aims to build on this by exploring more complex chess programming concepts through a series of easily-traced updates, beginning with the original Sunfish code as a foundation.

## See Results
Detailed performance evaluations between each minor update and the original sunfish engine can be found on [sunfish-like](https://github.com/flwrr/sunfish-like_tester). The testing repo also includes a suite of tools developed to ensure consistent outputs between Molafish and Sunfish in move generation, evaluation, and search.

## Roadmap
**Core Changes**<br>
- ✅  Convert board representation to bitboards using [Little-Endian File-Rank Mapping](https://www.chessprogramming.org/Square_Mapping_Considerations#Little-Endian_File-Rank_Mapping)<br>
- ✅  Move to a non-rotating board (Significant bottleneck: Molafish would need to rotate 15 bitboards)<br>
- ✅  Apply bitwise operations anywhere applicable<br>
- ✅  Precompute moves using bitboards using [Avoiding Rotated Bitboards with Direct Lookup](https://www.researchgate.net/publication/1888178_Avoiding_Rotated_Bitboards_with_Direct_Lookup)<br>
- 🔲  Universal Chess Interface, [UCI](https://en.wikipedia.org/wiki/Universal_Chess_Interface) integration<br>

**Long-term**
- Implement parts of or all of the code in C.
- Advanced evaluation and search techniques
- Numpy integration
- Lichchess bot
## Bitboards

<pre>
    String               Bitboard
    r n b q k b n r      1 1 1 1 1 1 1 1
    p p p p p p p p      1 1 1 1 1 1 1 1
    . . . . . . . .      0 0 0 0 0 0 0 0
    . . . . . . . .      0 0 0 0 0 0 0 0
    . . . . . . . .      0 0 0 0 0 0 0 0
    . . . . . . . .      0 0 0 0 0 0 0 0
    P P P P P P P P      1 1 1 1 1 1 1 1
    R N B Q K B N R      1 1 1 1 1 1 1 1
</pre>

Board representation is a fundamental aspect to any chess program which significantly influences its design and efficiency.

The original Sunfish engine represents its board with a single string of 120 characters, where each element either corresponds to a square and its occupying piece, or an empty space. The empty spaces provide a padding which Sunfish uses to prevent pieces from 'wrapping around' the board during move generation and simplifying the logic needed to check for legal moves. 

Move generation is further simplified by Sunfish rotating the board after each move, meaning all moves are generated and executed from the white perspective. Sunfish's string board representation allows this rotation to be very simple: `self.board[::-1].swapcase()`.

```python
# Sunfish board
A1, H1, A8, H8 = 91, 98, 21, 28
initial = (
    "         \n"  #   0 -  9
    "         \n"  #  10 - 19
    " rnbqkbnr\n"  #  20 - 29
    " pppppppp\n"  #  30 - 39
    " ........\n"  #  40 - 49
    " ........\n"  #  50 - 59
    " ........\n"  #  60 - 69
    " ........\n"  #  70 - 79
    " PPPPPPPP\n"  #  80 - 89
    " RNBQKBNR\n"  #  90 - 99
    "         \n"  # 100 -109
    "         \n"  # 110 -119
)
```

Molafish represents its board with 14 64-bit integers, where each bit on each bitboard represents a square on the board. Although a less intuitive technique to represent the board, bitboards can leverage 64-bit processing capabilities of modern processors, manipulating 64 bit quantities through bitwise operations. 

For instance, to advance all white pawns one square from their starting position in the initial board below, we only need to use: `initial[0] <<= 8`. This moves all pawn bits to the left by 8, effectively moving them from rank 2 to rank 3 by wrapping them up the board.

```python
# Molafish board
A1, H1, A8, H8 = 0x1, 0x80, (0x1 << 8*7), (0x80 << 8*7)
initial = (
    0b11111111 << 8,        #  [0] White Pawn
    0b10000001,             #  [1] White Rook
    0b01000010,             #  [2] White Knight
    0b00100100,             #  [3] White Bishop
    0b00001000,             #  [4] White Queen
    0b00010000,             #  [5] White King
    0b11111111 << (8 * 6),  #  [6] Black Pawn
    0b10000001 << (8 * 7),  #  [7] Black Rook
    0b01000010 << (8 * 7),  #  [8] Black Knight
    0b00100100 << (8 * 7),  #  [9] Black Bishop
    0b00001000 << (8 * 7),  # [10] Black Queen
    0b00010000 << (8 * 7),  # [11] Black King
    0xffff,                 # [12] All White Pieces
    0xffff000000000000,     # [13] All Black Pieces
    0xffff00000000ffff      # [14] All Pieces
)
```

## Run Molafish (WIP)
The play interface can currently be built and run, but will only execute two moves.<br>
Molafish uses a UCI protocol and terminal interface refactored from sunfish.

The simplest way to run Molafish is through the "fancy" terminal interface: `tools/fancy.py -cmd ./molafish.py`<br>
This project needs python >= 3.7. Also install the required packages with `pip install -r requirements.txt` <br>
It communicates through UCI protocol by the command `python3 -u uci.py`.

#### Interface from terminal play
<img src="https://i.imgur.com/V0NNHCo.png" width=40% height=40%>

## Molafish?

Molafish is a play on sunfish, and a joke on the project's goal of taking a compact, elegant engine and converting it to one with much more code and complexity. It refers to the [Ocean sunfish (*Mola mola*)](https://en.wikipedia.org/wiki/Ocean_sunfish).

## Credit

[Sunfish](https://github.com/thomasahle/sunfish)

[Stockfish](https://github.com/official-stockfish/Stockfish) 

[chessprogramming.org](https://www.chessprogramming.org/Main_Page)

# License

[GNU GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)
