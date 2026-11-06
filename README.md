# PlasmidUnroll

A circular plasmid is a loop of DNA. When software pieces together its sequence, the result can include several laps around the same loop. PlasmidUnroll looks for those back-to-back copies and can save one complete copy.

## How it works

- **Two or more complete copies.** A 10,000-letter sequence can contain two full laps of a 5,000-letter plasmid. If the copies are identical, the tool can check that repeating the first copy recreates the whole sequence. If the copies have small differences, the tool needs a trusted single-copy guide to account for the full sequence before it can save one copy.
- **One complete copy plus a fragment.** A 7,500-letter sequence can contain one full 5,000-letter lap followed by 2,500 letters from the next lap. The repeated letters can match exactly or have small differences. A match between the beginning and end gives the tool a reason to flag the sequence for review. Saving one copy in this case requires a trusted guide and an explicit option.

Start with a sequence you have evidence is circular. A real plasmid can also contain several copies of the same DNA. PlasmidUnroll reports the evidence so you can review it before choosing to save a single copy.

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
- `--unit` uses a supported single-copy guide in the same orientation. Alignments must tile the entire contig in order. By default, the script requires at least two complete copies. It retains the best-matching complete copy within the allowed length difference. The default minimum identity is 98%.
- Coordinates in the JSON report are 1-based and inclusive. Original assembly graph and statistics files still describe the original assembly.

### Small differences in copy length

By default, the retained copy must be the same length as the guide. Add `--max-length-diff-pct` to allow a complete copy to be a little shorter or longer. For example, `--max-length-diff-pct 1` allows a 6,000 bp guide to match a retained copy of 5,940–6,060 bp. The copy still has to align across the full guide and meet the identity and minimum length thresholds. PlasmidUnroll saves the sequence at its observed length and reports the difference from the guide in bases.

```bash
./plasmid-unroll assembly.fasta contig_24 --unit single-copy-guide.fasta \
  --max-length-diff-pct 1 --output assembly.contig24-single-copy.fasta
```

## Partial second copies

`--self-align` flags a same-strand match between the beginning and end that suggests fewer than two copies as `review_partial_repeat`. It also checks for exact end matches of at least 100 bp that NUCmer can omit. The estimated copy count comes from the length of the contig divided by the distance between the starts of the matching stretches.

To save one copy, supply a trusted single-copy guide and add `--allow-partial`. The guide must account for the entire sequence without gaps or overlaps between matches. There must be one full copy within the allowed length difference plus matching partial sequence at one or both ends. The output retains that full copy from the input contig.

```bash
./plasmid-unroll assembly.fasta contig_24 --unit single-copy-guide.fasta \
  --allow-partial --output assembly.contig24-single-copy.fasta
```

Run `./plasmid-unroll --help` for the full command options.
