# PlasmidUnroll

PlasmidUnroll detects tandem copies within an assembled circular plasmid. It reports the evidence as JSON and can write a new assembly FASTA with the selected contig represented as a single copy.

Choose a contig known to be circular from the assembly report or other evidence. A genuine plasmid multimer can have the same sequence pattern as an assembly duplication.

## Exact repeats

Use Python 3.9 or later. The script finds the shortest exact unit that tiles the entire sequence, with a default minimum unit length of 500 bp.

```bash
./plasmid-unroll assembly.fasta contig_27
./plasmid-unroll assembly.fasta contig_27 --output assembly.single-copy.fasta
```

The first command reports the candidate. The second writes a new FASTA if the check passes. Other records retain their headers and sequences. The normalized ID defaults to `contig_27.monomer` and can be set with `--output-id`.

## Imperfect repeats

Put MUMmer4's `nucmer` and `show-coords` on `PATH`. The alignment modes were tested with MUMmer4 4.0.1.

```bash
./plasmid-unroll assembly.fasta contig_24 --self-align
./plasmid-unroll assembly.fasta contig_24 --unit single-copy-guide.fasta
./plasmid-unroll assembly.fasta contig_24 --unit single-copy-guide.fasta \
  --output assembly.contig24-single-copy.fasta
```

- `--self-align` reports a same-strand terminal match and its start and end shifts for review.
- `--unit` uses a supported single-copy guide in the same orientation. Alignments must tile the entire contig in order and include at least two complete copies. The script retains the best-matching complete span of the guide's length from the input contig. The default minimum identity is 98%.
- Coordinates in the JSON report are 1-based and inclusive. Original assembly graph and statistics files still describe the original assembly.

Run `./plasmid-unroll --help` for the full command options.
