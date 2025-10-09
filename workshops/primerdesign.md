# Workshop: Designing Primer Sets for Molecular Barcoding

Wednesday, Nov 5. 1-4pm

## Overview
Hands-on workshop on designing primer sets for multiplex PCR (amplifying multiple fragments of a gene or multiple genes in the same reaction). Uses bioinformatics tools for sequence processing and primer design. Expect a duration of 3-4 hours and potential follow-ups.
## Objectives
* Master primer design in barcoding applications.
* Create overlapping primer sets to amplify full gene using degraded DNA template.
* Evaluate specificity across taxa (e.g., families/orders).
## Prerequisites
* For Windows PC, a Linux sub system, see instructions here: https://learn.microsoft.com/en-us/windows/wsl/install
 
* Laptop with miniconda:
  * How to install miniconda in you Mac/Linux system: https://www.anaconda.com/docs/getting-started/miniconda/install#quickstart-install-instructions
  * Check installion in the terminal by typing conda --version
 
* ###	Tools you can pre-install with conda: 
  *	[mafft](https://anaconda.org/bioconda/mafft)
  *	[trimal](https://anaconda.org/bioconda/trimal)
  *	[obitools](https://anaconda.org/bioconda/obitools)
  *	[PrimerProspector](https://anaconda.org/bioconda/primerprospector)
  *	[cd-hit](https://anaconda.org/bioconda/cd-hit)
  *	[seqkit](https://anaconda.org/bioconda/seqkit)
  *	[ecoPrimers](https://anaconda.org/bioconda/ecoprimers)
  *	[entrez-direct](https://anaconda.org/bioconda/entrez-direct)
  *	[emboss](https://anaconda.org/bioconda/emboss)
* ###	Other tools you can install (not available through conda):
  *	[mbc-prime](https://github.com/thackl/mbc-prime)
  *	[AliView](http://www.ormbunkar.se/aliview/#DOWNLOAD)
*	Basic command-line bioinformatics.
*	Target gene sequences from NCBI (e.g., COI for Insecta) - We'll show how to automatically download reference sequences using Entrez tools.

### Potential Agenda (~3-4 hours)
1.	Intro to molecular barcoding & multiplex primer design (20 min).
2.	Reference sequence retrieval, filtering, clustering, alignment (20 min).
3.	Primer design with AliView, assessment with ecoPrimers/mbc-prime and PrimerProspector (120 min).
4.	Wrap-up & Q&A (15 min).

### Hands-on Exercises
*	Download/filter/deduplicate/cluster target gene data.
*	Align, trim, plot entropy and visualize reference sequences to find potential primer sequences.
*	Assess primers for efficiency, specificity and coverage across targeted taxa.
### Expected Outputs
*	Primer sets (TSV/FASTA) for target gene/barcode.
*	Coverage summaries for broader taxa.
*	Entropy plots, alignments.




