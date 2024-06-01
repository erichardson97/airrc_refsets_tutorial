
MiXCR
=======================================================

Introduction
------------

MiXCR is an extremely widely-used tool for IG/TR analysis - MiXCR has its own references,
which are distributed with the program.

MiXCR has many presets and options for specifying how alignment is performed for different library preparation and
sequencing methods. This tutorial reflects one example, Takara's SMART-seq for BCR with UMIs.

Installation
---------------------------------------------------------

Mixcr installation requires that you have a license. Please refer to their website
for all details on installation. Briefly:

.. code-block:: bash

    wget https://github.com/milaboratory/mixcr/releases/download/v4.6.0/mixcr-4.6.0.zip
    unzip mixcr-4.6.0.zip
    ## Move to binary path or add directory to your path
    export PATH=$PATH:$PWD
    mixcr activate-license

Downloading and processing AIRR-C Reference Sets
--------------------------------------------------------

The following code block demonstrates how to download the human AIRR-C Reference Sets.

.. code-block:: bash

    mkdir airrc_refs
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGH_VDJ/published/ungapped_ex > airrc_refs/human_VDJ.fasta
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGKappa_VJ/published/ungapped_ex >> airrc_refs/human_IGKVJ.fasta
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGLambda_VJ/published/ungapped_ex >> airrc_refs/human_IGLVJ.fasta

The following code block is an example of how one can extract the V, D and J genes individually from the resultant FASTA files
(pip install Biopython first).

.. code-block:: python

    from Bio import SeqIO
    seqs = dict((p.id, str(p.seq)) for p in SeqIO.parse('airrc_refs/human_VDJ.fasta', 'fasta'))
    for gene in ['V', 'D', 'J']:
      with open(f'airrc_refs/{gene.lower()}-genes.IGH.fasta', 'w') as k:
        [k.write(f'>{seq_id}\n' + seqs[seq_id] + '\n') for seq_id in seqs if seq_id[3] == gene]

    seqs = dict((p.id, str(p.seq)) for p in SeqIO.parse('airrc_refs/human_IGKVJ.fasta', 'fasta'))
    for gene in ['V', 'D', 'J']:
      with open(f'airrc_refs/{gene.lower()}-genes.IGK.fasta', 'w') as k:
        [k.write(f'>{seq_id}\n' + seqs[seq_id] + '\n') for seq_id in seqs if seq_id[3] == gene]

    seqs = dict((p.id, str(p.seq)) for p in SeqIO.parse('airrc_refs/human_IGLVJ.fasta', 'fasta'))
    for gene in ['V', 'D', 'J']:
      with open(f'airrc_refs/{gene.lower()}-genes.IGL.fasta', 'w') as k:
        [k.write(f'>{seq_id}\n' + seqs[seq_id] + '\n') for seq_id in seqs if seq_id[3] == gene]


MiXCR has a nice functionality for building a custom reference or "library". Again, please refer to MiXCR's documentation and the specific protocol you used
to verify that this is optimal in your case.

.. code-block:: bash

    mixcr buildLibrary --debug --v-genes-from-fasta airrc_refs/v-genes.IGH.fasta --v-gene-feature VRegion --j-genes-from-fasta airrc_refs/j-genes.IGH.fasta --d-genes-from-fasta airrc_refs/d-genes.IGH.fasta --c-genes-from-species human --chain IGH --taxon-id 9606 --species human airrc-IGH.json.gz -f
    mixcr buildLibrary --debug --v-genes-from-fasta airrc_refs/v-genes.IGK.fasta --v-gene-feature VRegion --j-genes-from-fasta airrc_refs/j-genes.IGK.fasta --c-genes-from-species human --chain IGK --taxon-id 9606 --species human airrc-IGK.json.gz -f
    mixcr buildLibrary --debug --v-genes-from-fasta airrc_refs/v-genes.IGL.fasta --v-gene-feature VRegion --j-genes-from-fasta airrc_refs/j-genes.IGL.fasta --c-genes-from-species human --chain IGL --taxon-id 9606 --species human airrc-IGL.json.gz -f
    mixcr mergeLibrary airrc-IGH.json.gz airrc-IGK.json.gz airrc-IGL.json.gz airrc.json.gz


Test
.....

To test your installation, we'll use example data as referenced by MiXCR:

.. code-block:: bash

    wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR595/003/ERR5952573/ERR5952573_1.fastq.gz
    wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR595/003/ERR5952573/ERR5952573_2.fastq.gz

This would be their default using the Takara preset:

.. code-block:: bash

    mixcr analyze takara-human-rna-bcr-umi-smartseq \
        ERR5952573_1.fastq.gz \
        ERR5952573_2.fastq.gz \
        results_default

To use our custom reference set, we have to split up the nice "analyze" preset into its constituent
parts:

.. code-block:: bash

    mixcr align --preset takara-human-rna-bcr-umi-smartseq ERR5952573_1.fastq.gz \
    ERR5952573_2.fastq.gz airrc_alignments.vdjca --species human --library airrc -f

    mixcr refineTagsAndSort \
        airrc_alignments.vdjca \
        airrc_alignments.refined.vdjca

    mixcr assemble airrc_alignments.refined.vdjca airrc_clones.clns


