
Change-O
=======================================================

Introduction
------------

`Change-O`_ is an extremely popular toolkit for BCR repertoire analysis. As of June 2024, there is currently less
flexibility for inclusion of AIRR-C Reference Sets in Change-O - if you want to use Change-O without amending the source
code, you will unfortunately need to replace the "Human" data in the reference you have downloaded during set-up with
AIRR-C Reference Sets. If you don't like this, we would recommend changing the Change-O source code to allow a custom
species, though this is clearly more involved. Change-O uses IgBLAST for gene assignment, so there is substantial overlap
with the standalone tutorial.


Installation
------------

Please refer to Change-O's documentation for comprehensive information about `installation`_.
You will need to pip install Presto/Change-O and also will need the associated scripts for germline
database download, as OGRDB does not contain IGC genes - you will need to use the default.

.. code-block:: bash

    pip install presto changeo
    git clone https://bitbucket.org/kleinstein/immcantation/src/master/scripts/
    export PATH=$PATH:$PWD/scripts/scripts

You will also need to install IgBLAST:

.. code-block:: bash

    wget https://ftp.ncbi.nih.gov/blast/executables/igblast/release/LATEST/ncbi-igblast-1.22.0-x64-linux.tar.gz
    tar -xzvf ncbi-igblast-1.22.0-x64-linux.tar.gz
    export IGDATA=${PWD}/ncbi-igblast-1.22.0
    export PATH=$PATH:$IGDATA/bin

And receptor_utils:

.. code-block:: bash

    pip install receptor_utils


Downloading the default reference sets
---------------------------------------------------------

Immcantation requires a reference of C genes. These genes are not included in the AIRR-C Reference
Set.

.. code-block::

    mkdir $IGDATA/database
    mkdir $IGDATA/database/human
    fetch_imgtdb.sh -o imgt_database
    clean_imgtdb.py imgt_database/human/constant/imgt_human_IGHC.fasta imgt_database/human/constant/imgt_human_IGHC.fasta
    clean_imgtdb.py imgt_database/human/constant/imgt_human_IGKC.fasta imgt_database/human/constant/imgt_human_IGKC.fasta
    clean_imgtdb.py imgt_database/human/constant/imgt_human_IGLC.fasta imgt_database/human/constant/imgt_human_IGLC.fasta
    cat imgt_database/human/constant/imgt_human_IGHC.fasta imgt_database/human/constant/imgt_human_IGLC.fasta imgt_database/human/constant/imgt_human_IGKC.fasta > $IGDATA/database/human/imgt_human_ig_c.fasta


Downloading the AIRR-C Reference Sets
---------------------------------------------------------

The next step is downloading and formatting the AIRR-C Reference Sets. This is **in place of**
running the set up scripts (fetch_igblastdb.sh, fetch_imgtdb.sh) if you are following the Immcantation IgBLAST set up.
We apologize for the deletion of a subdirectory of IgBLAST's internal_data directory which is prohibited
in the IgBLAST set-up instructions.

.. code-block:: bash

    rm -rf $IGDATA/internal_data/human
    mkdir $IGDATA/internal_data/human
    mkdir airrc_files

    ##These are the AIRR-formatted FASTA files, most convenient for BLAST database construction.
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGH_VDJ/published/gapped_ex > airrc_files/human_VDJ.fasta
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGKappa_VJ/published/gapped_ex >> airrc_files/human_VDJ.fasta
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGLambda_VJ/published/gapped_ex >> airrc_files/human_VDJ.fasta

    ##These are the AIRR-formatted JSON files, required to build the ndms files.
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGH_VDJ/published/airr_ex > airrc_files/human_VDJ.json
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGKappa_VJ/published/airr_ex > airrc_files/human_kappa.json
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGLambda_VJ/published/airr_ex > airrc_files/human_lambda.json


Having downloaded all of the OGRDB files into a single FASTA file, we then need to split them up by locus, e.g. in Python.


.. code-block:: python

    import receptor_utils
    import os

    seqs = receptor_utils.simple_bio_seq.read_fasta('airrc_files/human_VDJ.fasta')
    for gene in ['V', 'D', 'J']:
      with open(f'{os.environ["IGDATA"]}/database/human/human_gl_{gene}.fasta', 'w') as k:
        [k.write(f'>{seq_id}\n' + seqs[seq_id] + '\n') for seq_id in seqs if seq_id[3] == gene]


We then need to make all of our BLAST databases and accessory files (.ndm and .aux) and the IGV BLAST
databases that go in the internal data directory (that we daringly deleted).

.. code-block:: bash

    makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/human_gl_V.fasta -out $IGDATA/internal_data/human/human_V
    makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/human_gl_V.fasta -out $IGDATA/database/imgt_human_ig_v
    makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/human_gl_D.fasta -out $IGDATA/database/imgt_human_ig_d
    makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/human_gl_J.fasta -out $IGDATA/database/imgt_human_ig_j
    makeblastdb -parse_seqids -dbtype nucl -in $IGDATA/database/human/imgt_human_ig_c.fasta -out $IGDATA/database/imgt_human_ig_c

    annotate_j $IGDATA/database/human/human_gl_J.fasta $IGDATA/optional_file/human_gl.aux

    make_igblast_ndm airrc_files/human_VDJ.json VH airrc_files/human_vdj.ndm.imgt
    make_igblast_ndm airrc_files/human_kappa.json VL airrc_files/human_vkappa.ndm.imgt
    make_igblast_ndm airrc_files/human_lambda.json VL airrc_files/human_vlambda.ndm.imgt
    cat airrc_files/human_vdj.ndm.imgt > airrc_files/airrc_human.ndm.imgt
    cat airrc_files/human_vkappa.ndm.imgt >> airrc_files/airrc_human.ndm.imgt
    cat airrc_files/human_vlambda.ndm.imgt >> airrc_files/airrc_human.ndm.imgt
    mv airrc_files/airrc_human.ndm.imgt $IGDATA/internal_data/human/human.ndm.imgt

Testing
-------

Let's test this, again using test data courtesy of the Immcantation team.

.. code-block:: bash

    curl https://zenodo.org/records/10046916/files/input.fasta?download=1 > example_data.fasta
    head -n 51 example_data.fasta > example_data_mini.fasta
    AssignGenes.py igblast -s example_data_mini.fasta -b $IGDATA --organism human --loci ig --format airr

.. _Change-O: https://changeo.readthedocs.io/en/stable/
.. _installation: https://changeo.readthedocs.io/en/stable/install.html