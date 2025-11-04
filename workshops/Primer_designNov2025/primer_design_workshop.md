# Primer Design Workshop: Step-by-Step Tutorial
# Molecular Barcoding Primer Design for Multiplex PCR

## Workshop Overview
This tutorial guides you through designing primer sets for molecular barcoding applications.
You'll learn to download sequences, process them, align them, and assess primers for specificity and coverage.

**Duration**: 3-4 hours  
**Expected Outputs**: Primer sets (TSV/FASTA), coverage summaries, entropy plots, alignments
**System Requirements**:
- Linux/Mac/Windows with WSL
- Miniconda installed: /docs/getting-started/miniconda/install#to-download-an-older-version
- Basic command-line knowledge
- **For Apple Silicon Macs**: Rosetta 2 installed (run `softwareupdate --install-rosetta`)

## Prerequisites
Before starting, install these tools via conda/mamba:

### Installation of biocanda packages (make sure to install miniconda first: /docs/getting-started/miniconda/install#to-download-an-older-version)
```bash
# Required tools
conda install -c bioconda mafft trimal cd-hit seqkit entrez-direct emboss primer3
# Optional tools (for advanced features)
conda install -c bioconda obitools ecoprimers  # For taxonomy-based primer design
```
### mbc-prime installation
```bash
# mbc-prime: Download from https://github.com/thackl/mbc-prime
## download workflow
git clone https://github.com/thackl/mbc-prime
cd mbc-prime
## install the dependencies, either with conda/mamba
mamba create -n mbc-prime python=3.11
mamba activate mbc-prime
mamba env update --file env.yaml
## add tool path to profile
echo 'export PATH="$PATH:/path/to/your/mbc-prime"' >> ~/.zprofile
### bash profile
echo 'export PATH="$PATH:/path/to/your/mbc-prime"' >> ~/.bash_profile
## run the tool
mbc-prime -h
```
### PrimerProspector Installation

⚠️ **Important Note for Apple Silicon Mac Users**: PrimerProspector installation is more complex on Apple Silicon Macs due to Python 2.7 requirements and the need for x86_64 architecture emulation via Rosetta 2. If you have an Apple Silicon Mac (M1/M2/M3 chips), please follow the special instructions below. For other systems (Linux, WSL, Intel Macs, Windows), use the standard installation.

```bash
# primerprospector: Installation in isolated environment (works on macOS, Linux/WSL, Windows)
# Note: PrimerProspector requires Python 2.7, so we create a separate environment

#### Standard Installation (Linux/WSL, Intel Macs, Windows):
# Step 1: Create environment (direct installation)
mamba create -n primerprospector python=2.7 numpy matplotlib hdf5 h5py -c conda-forge

# Step 2: Activate environment
mamba activate primerprospector

# If conda environment creation fails, manual installation:
# Download and extract
curl -L -O https://sourceforge.net/projects/pprospector/files/pprospector-1.0.1.tar.gz
tar -xzf pprospector-1.0.1.tar.gz
# Navigate to the top-level directory
cd pprospector-1.0.1
# Install dependencies
mamba install cogent
mamba install hdf5 h5py -c conda-forge
pip install Sphinx
# Install in the environment
python setup.py install --install-scripts=$CONDA_PREFIX/bin/

#### Special Installation for Apple Silicon Macs:
# Step 1: Install Rosetta 2 (required for x86_64 emulation)
softwareupdate --install-rosetta

# Step 2: Create environment (uses Rosetta emulation for x86_64 packages)
CONDA_SUBDIR=osx-64 mamba create -n primerprospector python=2.7 numpy matplotlib hdf5 h5py -c conda-forge

# Step 3: Activate environment
mamba activate primerprospector

# Step 4: Verify the environment is using x86_64 architecture via Rosetta
python -c "import platform; print('Architecture:', platform.machine())"
# Should output: Architecture: x86_64

# If conda environment creation fails, manual installation:
# Download and extract
curl -L -O https://sourceforge.net/projects/pprospector/files/pprospector-1.0.1.tar.gz
tar -xzf pprospector-1.0.1.tar.gz
# Navigate to the top-level directory
cd pprospector-1.0.1
# Install dependencies
mamba install cogent
mamba install hdf5 h5py -c conda-forge
pip install Sphinx
# Install in the environment
python setup.py install --install-scripts=$CONDA_PREFIX/bin/

# Common steps for both platforms:
# Test the installation
analyze_primers.py -h

# Important notes
# - Always activate the environment before using: mamba activate primerprospector
# - On Apple Silicon Macs, Rosetta 2 emulates x86_64 code for Python 2.7 compatibility
# - On Linux/WSL or Intel Macs, Python 2.7 runs natively
# - Keep this environment separate from other environments
```
### AliView: Download from http://www.ormbunkar.se/aliview/

---

## Step 1: Create Working Directories

Run these commands to set up your workspace:

```bash
# Create directories for organization
mkdir -p logs uniq_fasta cdhit_out alignments entropy_plots stats primerprospector consensus primer3_input primer3_output

# Verify directories were created
ls -la
```

**What this does**: Creates a clean folder structure to keep your files organized.

---

## Step 2: Download Reference Sequences

Choose the appropriate download method based on your USE_OBITOOLS setting:

### Download Reference Sequences

```bash
# Download FASTA sequences directly from NCBI
conda activate base  # Ensure you're in the base environment for entrez-direct
which esearch  # Verify entrez-direct is available
which efetch  # Verify entrez-direct is available
# Define your target organism and gene
## The more data is available, the longer this step will take to download all the sequences
esearch -db nucleotide -query "Ophiocordyceps[Organism] OR Cordyceps[Organism] \
AND (ITS2[All Fields] OR internal transcribed spacer \
2[All Fields]) AND 0:8000[SLEN]" | efetch -format fasta > "uniq_fasta/refs.fasta"

# Check how many sequences you downloaded
grep -c "^>" uniq_fasta/refs.fasta
```

**What this does**: Retrieves reference sequences from NCBI for your target gene and organism.

---

## Step 3: Process and Filter Sequences

```bash
# Remove duplicate sequences
seqkit rmdup -s -i -o "uniq_fasta/uniq_refs.fasta" "uniq_fasta/refs.fasta"

# Cluster similar sequences (99% identity)
cd-hit-est -i "uniq_fasta/uniq_refs.fasta" -o "cdhit_out/refs_c99.fasta" -c 0.99 -M 0 -T 0

# Generate statistics
seqkit stats "cdhit_out/refs_c99.fasta" > "stats/refs.stats"
cat "stats/refs.stats"
```

**What this does**: Cleans up the data by removing duplicates and clustering similar sequences.

---

## Step 4: Align Sequences

```bash
# Align all sequences (this may take several minutes)
mafft --thread -1 --auto "cdhit_out/refs_c99.fasta" > "alignments/refs_c99.aln" 2> "logs/mafft_refs.log"
# Check MAFFT log for any issues
tail -n 50 logs/mafft_refs.log

# Trim poorly aligned regions
trimal -in "alignments/refs_c99.aln" -out "alignments/refs_c99_trimmed.aln" -automated1 -fasta
# Inspect trimmed alignment
head -n 100 "alignments/refs_c99_trimmed.aln"
```

**What this does**: Creates a multiple sequence alignment to identify conserved regions for primer design.

---

## Step 5: Plot Conservation/Entropy

```bash
# Create entropy plot to find primer binding sites
plotcon -sequence "alignments/refs_c99_trimmed.aln" -winsize 10 -graph png -goutfile "entropy_plots/refs_entropy.png"

# View the plot (on Mac/Linux)
open entropy_plots/refs_entropy.png.1.png  # or use your image viewer
```

**What this does**: Generates a plot showing sequence conservation. Low entropy = conserved regions = good primer sites.

---

## Step 6: Manual Primer Design with AliView

```bash
# Open the alignment in AliView for manual primer design
# Download AliView from: http://www.ormbunkar.se/aliview/
# Open the trimmed alignment file in AliView
```

**Tips for primer design**:
- Choose regions with high conservation (few gaps, consistent nucleotides)
- Aim for 18-25 bp primers
- Check for secondary structures (eg. websites like OligoAnalyzer)
- Ensure primers don't form dimers
- Target melting temperatures (Tm) around 55-65°C

---

## Step 7: Automated Primer Design with Consensus + Primer3

This approach creates a consensus sequence from your multiple sequence alignment and uses Primer3 to design primers with specific settings optimized for your target product size range.

```bash
# Create consensus sequence from trimmed alignment using EMBOSS cons
# identity=10 (minimum number of identical residues), plurality=0.8 (minimum fraction)
mkdir -p consensus

cons -sequence "alignments/refs_c99_trimmed.aln" -outseq "consensus/consensus_refs.fasta" \
     -identity 10 -plurality 0.8 -name "consensus_refs"

# Verify consensus was created
cat "consensus/consensus_refs.fasta"

# Create Primer3 input file with your settings
mkdir -p primer3_input primer3_output

# Here you can adjust settings as needed (eg. product size range, number of primers, GC clamp, melting temp...)
cat > "primer3_input/consensus_refs.p3in" << EOF
SEQUENCE_ID=consensus_refs
SEQUENCE_TEMPLATE=$(grep -v '^>' consensus/consensus_refs.fasta | tr -d '\n')
PRIMER_PICK_LEFT_PRIMER=1
PRIMER_PICK_INTERNAL_OLIGO=0
PRIMER_PICK_RIGHT_PRIMER=1
PRIMER_OPT_SIZE=20
PRIMER_MIN_SIZE=18
PRIMER_MAX_SIZE=22
PRIMER_PRODUCT_SIZE_RANGE=1500-3100
PRIMER_MIN_TM=55.0
PRIMER_OPT_TM=60.0
PRIMER_MAX_TM=65.0
PRIMER_GC_CLAMP=1
PRIMER_NUM_RETURN=3
PRIMER_EXPLAIN_FLAG=1
=
EOF

# Run Primer3
primer3_core < "primer3_input/consensus_refs.p3in" > "primer3_output/consensus_refs.p3out"

# View results
echo "Primer3 results:"
cat "primer3_output/consensus_refs.p3out" | grep -A 20 "PRIMER_LEFT_0_SEQUENCE"

# Extract primer sequences for further analysis
echo "Primer Pairs (Forward = LEFT, Reverse = RIGHT):"
for i in {0..2}; do
  left=$(grep "PRIMER_LEFT_${i}_SEQUENCE" "primer3_output/consensus_refs.p3out" | cut -d'=' -f2)
  right=$(grep "PRIMER_RIGHT_${i}_SEQUENCE" "primer3_output/consensus_refs.p3out" | cut -d'=' -f2)
  size=$(grep "PRIMER_PAIR_${i}_PRODUCT_SIZE" "primer3_output/consensus_refs.p3out" | cut -d'=' -f2)
  left_tm=$(grep "PRIMER_LEFT_${i}_TM" "primer3_output/consensus_refs.p3out" | cut -d'=' -f2)
  right_tm=$(grep "PRIMER_RIGHT_${i}_TM" "primer3_output/consensus_refs.p3out" | cut -d'=' -f2)
  left_gc=$(grep "PRIMER_LEFT_${i}_GC_PERCENT" "primer3_output/consensus_refs.p3out" | cut -d'=' -f2)
  right_gc=$(grep "PRIMER_RIGHT_${i}_GC_PERCENT" "primer3_output/consensus_refs.p3out" | cut -d'=' -f2)
  if [ ! -z "$left" ] && [ ! -z "$right" ]; then
    echo "Pair ${i}:"
    echo "  Forward: ${left} | Tm: ${left_tm}°C | GC%: ${left_gc}%"
    echo "  Reverse: ${right} | Tm: ${right_tm}°C | GC%: ${right_gc}%"
    echo "  Product size: ${size} bp"
    echo ""
  fi
done
```

**What this does**: 
- Creates a consensus sequence representing the most common nucleotides at each position
- Includes melting temperature constraints (55-65°C) and GC clamp (≥1 G/C at 3' end) for reliable PCR amplification
- Returns N primer pairs per sequence
- Output includes primer sequences, positions, and thermodynamic properties

---

## Step 8: Assess Primer Quality with PrimerProspector

```bash
# Activate the PrimerProspector environment (Python 2.7)
mamba activate primerprospector

# Prepare sequences for analysis
seqkit seq -u "cdhit_out/refs_c99.fasta" > "primerprospector/refs_upper.fasta"

# Create primers file (edit this with your designed primers)
cat > "primerprospector/Primers_refs.txt" << EOF
# Replace these with your actual primers!
# Format: PrimerName<TAB>Sequence
MyPrimer_F1	NNNNNNNNNNNNNNNNNNNN
MyPrimer_R1	NNNNNNNNNNNNNNNNNNNN
EOF

# Run primer analysis
analyze_primers.py -P "primerprospector/Primers_refs.txt" -f "primerprospector/refs_upper.fasta" -v -o "primerprospector/refs_analysis"

# View results
ls primerprospector/refs_analysis/
```

---

## Other design approach using mbc-prime (host amplification avoidance)

```bash
# 1) Prepare combined alignment (download host, combine with refs, align + trim)
mkdir mbc_output

# Download off-target sequences
esearch -db nucleotide -query "Hemiptera[Organism] AND (ITS2[All Fields] OR \
internal transcribed spacer 2[All Fields]) AND 0:8000[SLEN]" \
| efetch -format fasta > "uniq_fasta/host.fasta"

# Deduplicate
seqkit rmdup -s -i -o "uniq_fasta/uniq_host.fasta" "uniq_fasta/host.fasta"
cd-hit-est -i "uniq_fasta/uniq_host.fasta" -o "cdhit_out/host_c99.fasta" -c 0.99 -M 0 -T 0

# Combine and align divergent set with MAFFT --retree 2 (fast, rough, good enough for primer design)
cat "cdhit_out/refs_c99.fasta" "cdhit_out/host_c99.fasta" > "cdhit_out/combined_refs_host.fasta"

# Fast alignment: --retree 2 builds rough tree twice, --maxiterate 0 skips refinement
mafft --thread -1 --retree 2 --maxiterate 0 \
  "cdhit_out/combined_refs_host.fasta" > "alignments/combined_refs_host.aln" \
  2> "logs/mafft_combined.log"

# 2) Run mbc-prime on the combined trimmed alignment
mamba activate mbc-prime

# Target count = number of reference (non-host) sequences
target_count=$(grep -c '^>' "cdhit_out/refs_c99.fasta")

# Ensure output directory exists
mkdir -p mbc_output

# -s (score): the aggregated score of the primer ranging from 0-1, with 1 capturing 100% of targets while excluding 100% of other sequences.
mbc-prime -t "$target_count" -s 0.8 -v \
  "alignments/combined_refs_host.aln" > "mbc_output/refs_host_primers.tsv" 2> "mbc_output/mbc-prime_refs_host.log"
```

**What this does**: Automates primer design considering coverage and specificity.
See mbc-prime documentation to understand better the scoring system: https://github.com/thackl/mbc-prime.

Other tools like PrimerBLAST can also be used for primer design and specificity checking: https://www.ncbi.nlm.nih.gov/tools/primer-blast/

---

## Expected Outputs

After completing all steps, you should have:

- `stats/refs.stats` - Sequence statistics
- `alignments/refs_c99_trimmed.aln` - Multiple sequence alignment
- `entropy_plots/refs_entropy.png` - Conservation plot
- `consensus/consensus_refs.fasta` - Consensus sequence from alignment
- `primer3_output/consensus_refs.p3out` - Primer3 primer design results
- `primerprospector/refs_analysis/` - Primer quality assessments (if Step 10 run)
- `mbc_output/refs_host_primers.tsv` - mbc-prime results (if Step 9 run)

## Next Steps and Applications

1. **Iterate**: Design new primers based on analysis results
2. **Test in silico**: Use tools like ecoPCR to test primer specificity
3. **Wet lab validation**: Test primers in PCR reactions
4. **Multiplex design**: Combine primers for different genes/regions

## Troubleshooting

- **No sequences downloaded**: Check your ORGANISM and GENE queries
- **Alignment fails**: Reduce sequence number or use faster MAFFT options
- **Memory issues**: Use `cd-hit` with higher similarity threshold
- **Taxonomy problems**: Ensure obitools is properly installed
- **Mamba activation fails**: If you get shell initialization errors, run `mamba shell init --shell zsh --root-prefix=~/miniforge3` and restart your terminal
- **Package not found**: Some tools may not be available for Apple Silicon Macs - check alternatives like ecoPrimers
- **PrimerProspector installation issues**: 
  - Ensure Rosetta 2 is installed: `softwareupdate --install-rosetta`
  - Always activate the environment: `mamba activate primerprospector`
  - For Apple Silicon Macs, verify the environment uses x86_64: run `python -c "import platform; print('Architecture:', platform.machine())"` after activation - it should show 'x86_64'
  - CONDA_SUBDIR=osx-64 during environment creation automatically handles x86_64 emulation on Apple Silicon
  - Do not use 'mamba config set subdir' commands as subdir is auto-detected
  - If conda installation fails, install dependencies with `mamba install cogent`, `mamba install hdf5 h5py -c conda-forge`, `pip install Sphinx`, followed by `python setup.py install --install-scripts=$CONDA_PREFIX/bin/`
  - PrimerProspector requires Python 2.7 and x86_64 emulation on Apple Silicon

## Additional Resources

- [NCBI Entrez documentation](https://www.ncbi.nlm.nih.gov/books/NBK25501/)
- [OBITools tutorial](http://metabarcoding.org/obitools/doc/tutorial.html)
- [Primer design guidelines](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1636359/)

---

*This tutorial is based on bioinformatics workflows for molecular ecology and barcoding studies.*