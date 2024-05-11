
Using the AIRR-C Reference Sets with Mixcr
=======================================================

Introduction
------------

Mixcr is a widely-used tool for IG/TR analysis - they have their own references,
which they distribute with the program. If you would like to use the AIRR-C Reference
Sets, follow the tutorial below!

Installation
---------------------------------------------------------

Mixcr installation requires that you have a license.

.. code-block:: bash

    wget https://github.com/milaboratory/mixcr/releases/download/v4.6.0/mixcr-4.6.0.zip
    unzip mixcr-4.6.0.zip
    ## Move to bin or add directory to your path
    export BIN=${pwd}
    mixcr activate-license


Downloading and processing AIRR-C Reference Sets
--------------------------------------------------------

.. code-block:: bash

    pip install biopython
    mkdir airrc_refs
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGH_VDJ/published/ex > airrc_refs/human_VDJ.fasta
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGKappa_VJ/published/ex >> airrc_refs/human_IGKVJ.fast
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGLambda_VJ/published/ex >> airrc_refs/human_IGLVJ.fasta


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

.. code-block:: bash

    mixcr buildLibrary --debug --v-genes-from-fasta airrc_refs/v-genes.IGH.fasta --v-gene-feature VRegion --j-genes-from-fasta airrc_refs/j-genes.IGH.fasta --d-genes-from-fasta airrc_refs/d-genes.IGH.fasta --c-genes-from-species human --chain IGH --taxon-id 9606 --species human airrc-IGH.json.gz -f
    mixcr buildLibrary --debug --v-genes-from-fasta airrc_refs/v-genes.IGK.fasta --v-gene-feature VRegion --j-genes-from-fasta airrc_refs/j-genes.IGK.fasta --c-genes-from-species human --chain IGK --taxon-id 9606 --species human airrc-IGK.json.gz -f
    mixcr buildLibrary --debug --v-genes-from-fasta airrc_refs/v-genes.IGL.fasta --v-gene-feature VRegion --j-genes-from-fasta airrc_refs/j-genes.IGL.fasta --c-genes-from-species human --chain IGL --taxon-id 9606 --species human airrc-IGL.json.gz -f
    mixcr mergeLibrary airrc-IGH.json.gz airrc-IGK.json.gz airrc-IGL.json.gz airrc.json.gz


Test
.....

.. code-block:: bash

    wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR595/003/ERR5952573/ERR5952573_1.fastq.gz
    wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR595/003/ERR5952573/ERR5952573_2.fastq.gz
    mixcr analyze takara-human-rna-bcr-umi-smartseq \
        ERR5952573_1.fastq.gz \
        ERR5952573_2.fastq.gz \
        results_default



