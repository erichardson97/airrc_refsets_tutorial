
IgBLAST stand-alone
=======================================================

Introduction
------------

IgBLAST developers have made strides to include AIRR-C Reference Sets for users and
you can easily specify this with the dropdown button on the IgBLAST webserver.
For further information about using IgBLAST, refer to their guide.

Installation
---------------------------------------------------------

Assuming you don't already have an IgBLAST installation, download IgBLAST and
set your $IGDATA variable so that your interpreter knows where to find the internal_data directory.

.. code-block:: bash

    wget https://ftp.ncbi.nih.gov/blast/executables/igblast/release/LATEST/ncbi-igblast-1.22.0-x64-linux.tar.gz
    tar -xvzf ncbi-igblast-1.22.0-x64-linux.tar.gz
    export IGDATA=${pwd}/ncbi-igblast-1.22.0

To follow the remainder of the steps, you will also need to install receptor_utils.

.. code-block:: bash

    pip install receptor_utils

Downloading the AIRR-C Reference Sets
---------------------------------------------------------

When you initially set up IgBLAST (if it wasn't just now!), you will have
downloaded particular germline databases for the program which you will have supplied
to IgBLAST via its -germline_db_V, -germline_db_J and -germline_db_D arguments. To
use AIRR-C Reference Sets, we need to download the AIRR-C versions of these BLAST databases.
The developers of IgBLAST kindly make these available via the ftp server - or you can build
your own.

In addition to these databases, IgBLAST has its own data directory called "internal_data",
which you are warned not to touch except for making your own custom species directories, as
well as "auxiliary_data". To use the AIRR-C Reference Sets with full IgBLAST capabilities (i.e.
CDR annotation), you will need to create two file types using the receptor_utils package.

These are:

1. .ndm file: the ndm file contains CDR1 + CDR2 coordinates for the V genes in your reference.
2. .aux file: the aux file contains CDR3 coordinates for the J genes in your reference.

You can supply the paths to these new files when you run IgBLAST as a standalone using the
"-custom_internal_data" and "-auxiliary_data" flags.

Creating the .ndm file
----------------------
The .ndm file contains the CDR/FWR annotations for the V gene database.
The best way to do this is using the AIRR-formatted JSON file retrieved by the OGRDB API.
Retrieve for all three loci, run receptor_util’s make_ndm executable, and combine into a single .ndm file.

.. code-block:: bash

    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGH_VDJ/published/airr_ex > human_VDJ.json
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGKappa_VJ/published/airr_ex > human_kappa.json
    curl https://ogrdb.airr-community.org/api/germline/set/Human/IGLambda_VJ/published/airr_ex > human_lambda.json


    make_igblast_ndm human_VDJ.json VH human_vdj.ndm.imgt
    make_igblast_ndm human_kappa.json VL human_vkappa.ndm.imgt
    make_igblast_ndm human_lambda.json VL human_vlambda.ndm.imgt

    cat human_vdj.ndm.imgt > airrc_human.ndm.imgt
    cat human_vkappa.ndm.imgt >> airrc_human.ndm.imgt
    cat human_vlambda.ndm.imgt >> airrc_human.ndm.imgt

    mkdir $IGDATA/internal_data/airrc_human
    mv airrc_human.ndm.imgt $IGDATA/internal_data/airrc_human

Creating the .aux file
----------------------

.. code-block:: bash

    annotate_j /content/ncbi-igblast-1.22.0/database/airrc_human/human_gl_J.fasta /content/ncbi-igblast-1.22.0/optional_file/airrc_human_gl.aux


Testing
-------

Let's test this using test data courtesy of the Immcantation team.

.. code-block:: bash

    curl https://zenodo.org/records/10046916/files/input.fasta?download=1 > example_data.fasta
    head -n 51 example_data.fasta > example_data_mini.fasta

    igblastn -germline_db_V $IGDATA/database/airrc_human/airr_c_human_ig.V \
            -germline_db_J $IGDATA/database/airrc_human/airr_c_human_ig.J \
                -germline_db_D $IGDATA/database/airrc_human/airr_c_human_igh.D \
                -custom_internal_data $IGDATA/internal_data/airrc_human/airrc_human.ndm.imgt \
        -query example_data_mini.fasta -auxiliary_data $IGDATA/optional_file/airrc_human_gl.aux \
            -show_translation -outfmt '7 std qseq sseq btop' > test_output.fmt7


