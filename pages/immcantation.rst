
Change-O
=======================================================

Introduction
------------

`Change-O`_ is another extremely popular toolkit for BCR repertoire analysis. There is currently less
flexibility for inclusion of AIRR-C Reference Sets in Change-O.
As of June 2024, if you want to use Change-O without amending the source code, you will unfortunately need to replace the "Human" data in the reference you have downloaded during set-up with
AIRR-C Reference Sets.
Change-O uses IgBLAST for gene assignment, so there is substantial overlap with the standalone tutorial.


Installation
------------

Please refer to Change-O's documentation for comprehensive information about `installation`_.
You will need to pip install Presto/Change-O and also will need the associated scripts for germline
database download.

.. code-block:: bash

    pip install presto changeo
    git clone https://bitbucket.org/kleinstein/immcantation/src/master/scripts/
    export PATH=$PATH:$(pwd)/scripts/scripts

You will also need to install IgBLAST:

.. code-block:: bash

    wget https://ftp.ncbi.nih.gov/blast/executables/igblast/release/LATEST/ncbi-igblast-1.22.0-x64-linux.tar.gz
    tar -xzvf ncbi-igblast-1.22.0-x64-linux.tar.gz
    export IGDATA=${pwd}/ncbi-igblast-1.22.0

And receptor_utils:

.. code-block:: bash
    pip install receptor_utils


Downloading the default reference sets
---------------------------------------------------------

Immcantation requires a reference of C genes. This locus is not included in the AIRR-C Reference
Set.

.. code-block::

    mkdir $IGDATA/database
    mkdir $IGDATA/database/human
    fetch_imgtdb.sh -o imgt_database
    clean_imgtdb.py imgt_database/human/constant/imgt_human_IGHC.fasta imgt_database/human/constant/imgt_human_IGHC.fasta
    clean_imgtdb.py imgt_database/human/constant/imgt_human_IGKC.fasta imgt_database/human/constant/imgt_human_IGKC.fasta
    clean_imgtdb.py imgt_database/human/constant/imgt_human_IGLC.fasta imgt_database/human/constant/imgt_human_IGLC.fasta
    cat imgt_database/human/constant/imgt_human_IGHC.fasta imgt_database/human/constant/imgt_human_IGLC.fasta imgt_database/human/constant/imgt_human_IGKC.fasta > database/human/imgt_human_ig_c.fasta


Downloading the AIRR-C Reference Sets
---------------------------------------------------------

The next step is downloading and formatting the AIRR-C Reference Sets. This is **in place of**
running the set up scripts (fetch_igblastdb.sh, fetch_imgtdb.sh) if you are following the Immcantation IgBLAST set up.
We apologize for the deletion of a subdirectory of IgBLAST's internal_data directory which is prohibited
in the IgBLAST set-up instructions.

.. code-block:: bash

    mkdir $IGDATA/database
    mkdir $IGDATA/database/human
    rm -rf $IGDATA/internal_data/human
    mkdir $IGDATA/internal_data/human

    ##These are the AIRR-formatted JSON files, most convenient for BLAST database construction.
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGH_VDJ/published/gapped_ex > human_VDJ.fasta
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGKappa_VJ/published/gapped_ex >> human_VDJ.fasta
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGLambda_VJ/published/gapped_ex >> human_VDJ.fasta

    ##These are the AIRR-formatted JSON files, required to build the ndms files.
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGH_VDJ/published/airr_ex > human_VDJ.json
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGKappa_VJ/published/airr_ex > human_kappa.json
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGLambda_VJ/published/airr_ex > human_lambda.json


Having downloaded all of the OGRDB files into a single FASTA file, we then need to split them up by locus, e.g. in Python.


.. code-block:: python

    import receptor_utils
    seqs = receptor_utils.simple_bio_seq.read_fasta('human_VDJ.fasta')
    for gene in ['V', 'D', 'J']:
      with open(f'database/human/human_gl_{gene}.fasta', 'w') as k:
        [k.write(f'>{seq_id}\n' + seqs[seq_id] + '\n') for seq_id in seqs if seq_id[3] == gene]


We then need to make all of our BLAST databases and accessory files (.ndm and .aux).

.. code-block:: bash

    IGDATA/bin/makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/human_gl_V.fasta -out human_V
    IGDATA/bin/makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/human_gl_V.fasta -out imgt_human_ig_v
    IGDATA/bin/makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/human_gl_D.fasta -out imgt_human_ig_d
    IGDATA/bin/makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/human_gl_J.fasta -out imgt_human_ig_j

    mv imgt_human_ig* $IGDATA/database
    mv human_V* $IGDATA/internal_data/human

    annotate_j database/human/human_gl_J.fasta $IGDATA/optional_file/human_gl.aux

    make_igblast_ndm human_VDJ.json VH human_vdj.ndm.imgt
    make_igblast_ndm human_kappa.json VL human_vkappa.ndm.imgt
    make_igblast_ndm human_lambda.json VL human_vlambda.ndm.imgt
    cat human_vdj.ndm.imgt > airrc_human.ndm.imgt
    cat human_vkappa.ndm.imgt >> airrc_human.ndm.imgt
    cat human_vlambda.ndm.imgt >> airrc_human.ndm.imgt

    mv airrc_human.ndm.imgt $IGDATA/internal_data/human/human.ndm.imgt


.. _Change-O: https://changeo.readthedocs.io/en/stable/
.. _installation: https://changeo.readthedocs.io/en/stable/install.html
