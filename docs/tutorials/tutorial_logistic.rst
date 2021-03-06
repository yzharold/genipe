
.. contents:: Quick navigation
   :depth: 2


Logistic regression
====================

Logistic regressions are commonly used to perform genome-wide association
studies. It is possible to perform such an analysis using imputation data
(dosage format), where each imputed genotypes varies between 0 and 2
(inclusively). A value close to 0 means that a homozygous genotype of the most
frequent allele is the most probable. A value close to 2 means that a
homozygous genotype of the rare allele is the most probable. Finally, a value
close to 1 means that a heterozygous genotype is the most probable.

We suppose that you have followed the main :ref:`genipe-tut-page`. The
following command will create the working directory for this tutorial.

.. code-block:: bash

   mkdir -p $HOME/genipe_tutorial/logistic


.. _logit-tut-input-files:

Input files
------------

Imputed genotypes
^^^^^^^^^^^^^^^^^^

After running the :py:mod:`genipe` pipeline, the imputed genotypes files will
have the ``.impute2`` or ``.impute2.gz`` extension Those files will be located
in the ``final_impute2`` directories of each chromosomes. There should be one
*impute2* file per chromosome (see the
:ref:`genipe-tut-output-files-final_impute2` section in the main
:ref:`genipe-tut-page`). These files consist of the imputed genotypes required
to perform the analysis.

The general structure of the file contains the following columns (which are
space delimited): the chromosome, the name of the marker, its position and its
two alleles. The subsequent columns correspond to the probabilities of each
genotype (hence, there are three columns per sample). The first value
correspond to the probability of being homozygous of the first allele. The
second value correspond to the probability of being heterozygous. Finally, the
third value correspond to the probability of being homozygous of the second
allele. The following example shows two lines of the *impute2* file.

.. code-block:: text

    21 rs376366718:10000302:A:G 10000302 A G 0.986 0.014 0 1 0 0 1 0 0 ...
    21 21:10002805:C:T 10002805 C T 0.254 0.736 0.010 0.810 0.188 0.002 0.800 0.195 0.005 ...


Samples file
^^^^^^^^^^^^^

This file is generated by :py:mod:`genipe` and has the ``.sample`` extension.
There should be one sample file per chromosome (see the
:ref:`genipe-tut-output-files-final_impute2` section in the main
:ref:`genipe-tut-page`). These files greatly resembles the *Plink* ``fam``
file. Specifically, it contains the samples that are included in the *impute2*
file (with **the same order**). It is needed to correctly interpret the sample
described by the *impute2* file. The format is as follow:

.. code-block:: text

   ID_1 ID_2 missing father mother sex plink_pheno
   0 0 0 D D D B
   1341 NA06985 0 0 0 2 -9
   1341 NA06991 0 NA06993 NA06985 2 -9
   1341 NA06993 0 0 0 1 -9
   ...

The first two rows are part of the format and should be as is.

.. warning::

   The column ``ID_2`` should contain **unique** sample identification numbers,
   since the analysis will only consider the ``ID_2`` (which correspond to the
   sample ID in the *Plink* file) to correctly match the samples and the
   imputed genotypes.


Phenotype file
^^^^^^^^^^^^^^^

This file describes the phenotype and variables used to perform the logistic
regression. The file is *tab* separated and contains one row per sample, one
column per phenotype/variable.

The following is an example of a phenotype file:

.. code-block:: text

   SampleID	Pheno2	Age	Var1	Gender
   NA06985	1	53	48.01043142060001	2
   NA06993	1	47	23.7615117523	1
   NA06994	0	48	20.2946857226	1
   ...

We provide a *dummy* phenotype file (where values, except for ``Gender``, were
randomly generated). The following command should download the phenotype file.

.. code-block:: bash

   cd $HOME/genipe_tutorial/logistic

   wget http://pgxcentre.github.io/genipe/_static/tutorial/phenotypes_logistic.txt.bz2
   bunzip2 phenotypes_logistic.txt.bz2

.. note::

   Note that the gender is encoded such that males are ``1`` and females are
   ``2``. Samples with missing gender (encoded as ``0``) will be excluded only
   if gender is in the covariable list.

.. note::

   Categorical variables should be specified using the ``--categorical``
   option.

.. warning::

   The sample identification numbers should match the ones in the sample file
   (see above). Those numbers should be unique for each sample. Only the
   samples that are **both** in the sample and phenotype files will be kept for
   analysis. The order of the samples in the phenotype file is not important.


Sites to extract (optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This file (which is optional) should contain a list of site (one identification
number per line) to keep for the analysis. This file might be the
``.good_sites`` file automatically generated by :py:mod:`genipe` (see the
:ref:`genipe-tut-output-files-final_impute2` section in the main
:ref:`genipe-tut-page`).


.. _logit-tut-execute:

Executing the analysis
-----------------------

If you followed the :ref:`genipe-tut-page`, the following commands should
execute the logistic regression analysis.

.. code-block:: bash

   cd $HOME/genipe_tutorial/logistic

   imputed-stats logistic \
       --impute2 ../genipe/chr22/final_impute2/chr22.imputed.impute2.gz \
       --sample ../genipe/chr22/final_impute2/chr22.imputed.sample \
       --pheno phenotypes_logistic.txt \
       --extract-sites ../genipe/chr22/final_impute2/chr22.imputed.good_sites \
       --nb-process 8 \
       --nb-lines 6000 \
       --gender-column Gender \
       --covar Age,Var1,Gender \
       --sample-column SampleID \
       --pheno-name Pheno2

For more information about the arguments and options, see the
:ref:`logit-tut-usage` section. For an approximation of the execution time,
refer to the :ref:`stats-exec-time` section.

.. _logit-tut-output-files:

Output files
-------------

There will be two output files: ``.logistic.dosage`` will contain the
statistics, and ``.log`` will contain the execution log.


``.logistic.dosage`` file
^^^^^^^^^^^^^^^^^^^^^^^^^^

This file contains the results from the logistic regression. It shows the
following information:

* ``chr``: the chromosome.
* ``pos``: the position on the chromosome.
* ``snp``: the name of the marker.
* ``major``: the major allele.
* ``minor``: the minor allele.
* ``maf``: the frequency of the minor allele.
* ``n``: the number of samples that were used for this marker.
* ``coef``: the coefficient.
* ``se``: the standard error.
* ``lower``: the lower value of the 95% confidence interval.
* ``upper``: the upper value of the 95% confidence interval.
* ``z``: the *z*-statistic.
* ``p``: the *p*-value.

.. note::

   By default, the statistics are computed only for markers with a minor allele
   frequency of 1% and higher. Markers with lower MAF will have ``NA`` values.
   To modify this behavior, use the ``--maf`` option.


.. _logit-tut-usage:

Usage
------

The following command will display the documentation for the logistic
regression analysis in the console:

.. code-block:: console


   $ imputed-stats logistic --help
   usage: imputed-stats logistic [-h] [-v] [--debug] --impute2 FILE --sample FILE
                                 --pheno FILE [--extract-sites FILE] [--out FILE]
                                 [--nb-process INT] [--nb-lines INT] [--chrx]
                                 [--gender-column NAME] [--scale INT]
                                 [--prob FLOAT] [--maf FLOAT] [--covar NAME]
                                 [--categorical NAME] [--missing-value NAME]
                                 [--sample-column NAME] [--interaction NAME]
                                 --pheno-name NAME

   Performs a logistic regression on imputed data using a GLM with a binomial
   distribution. This script is part of the 'genipe' package, version 1.4.0.

   optional arguments:
     -h, --help            show this help message and exit
     -v, --version         show program's version number and exit
     --debug               set the logging level to debug

   Input Files:
     --impute2 FILE        The output from IMPUTE2.
     --sample FILE         The sample file (the order should be the same as in
                           the IMPUTE2 files).
     --pheno FILE          The file containing phenotypes and co variables.
     --extract-sites FILE  A list of sites to extract for analysis (optional).

   Output Options:
     --out FILE            The prefix for the output files. [imputed_stats]

   General Options:
     --nb-process INT      The number of process to use. [1]
     --nb-lines INT        The number of line to read at a time. [1000]
     --chrx                The analysis is performed for the non pseudo-autosomal
                           region of the chromosome X (male dosage will be
                           divided by 2 to get values [0, 0.5] instead of [0, 1])
                           (males are coded as 1 and option '--gender-column'
                           should be used).
     --gender-column NAME  The name of the gender column (use to exclude samples
                           with unknown gender (i.e. not 1, male, or 2, female).
                           If gender not available, use 'None'. [Gender]

   Dosage Options:
     --scale INT           Scale dosage so that values are in [0, n] (possible
                           values are 1 (no scaling) or 2). [2]
     --prob FLOAT          The minimal probability for which a genotype should be
                           considered. [>=0.9]
     --maf FLOAT           Minor allele frequency threshold for which marker will
                           be skipped. [<0.01]

   Phenotype Options:
     --covar NAME          The co variable names (in the phenotype file),
                           separated by coma.
     --categorical NAME    The name of the variables that are categorical (note
                           that the gender is always categorical). The variables
                           are separated by coma.
     --missing-value NAME  The missing value in the phenotype file.
     --sample-column NAME  The name of the sample ID column (in the phenotype
                           file). [sample_id]
     --interaction NAME    Add an interaction between the genotype and this
                           variable.

   Logistic Regression Options:
     --pheno-name NAME     The phenotype.


.. _logit-tut-comparison:

Results comparison
-------------------

The logistic regression results from :py:mod:`genipe` and *Plink* were compared
for validity. The following figure shows the comparison for, from left to
right, the coefficients, the standard errors and the *p*-values. The *x* axis
shows the results from :py:mod:`genipe`, and the *y* axis shows the results for
*Plink*. This comparison includes 58,871 "good" imputed markers with a MAF
higher or equal to 10%, analyzed for 60 samples (*i.e* results from this
tutorial). Note that for this comparison, the **probability threshold**
(``--prob``) **was changed from 0.9 to 0** to *imitate* *Plink* analysis (see
note below for more information).

.. note::

   Only markers with minor allele frequency (MAF) higher or equal to 10% were
   compared, since markers with lower MAF might have convergence issues (*e.g.*
   all exposed samples are all cases *or* all controls). In that case, the
   coefficient is large, and the odds ratio (:math:`e^{coef}`) gets too large.

.. figure:: ../_static/images/Logistic_Diff_Prob0.png
   :align: center
   :width: 100%
   :alt: Logistic regression comparison between genipe and Plink (prob. of 0)

.. note::

   The sign of the coefficients might be different when comparing
   :py:mod:`genipe` to *Plink*, since :py:mod:`genipe` computes the statistics on
   the rare allele, while *Plink* computes them on the second (alternative)
   allele. The alternative allele might not always be the rarest.


.. note::

   By default, :py:mod:`genipe` excludes samples with a maximum probability
   lower than 0.9 (the ``--prob`` option), while *Plink* keeps all the samples
   for the analysis. In order to get the same results as *Plink*, the analysis
   must be done with a probability threshold of 0 (*i.e.* ``--prob 0``, keeping
   all imputed genotypes including those with poor quality). This is what was
   done for the previous figure.

   The following figure shows the comparison between *Plink* and
   :py:mod:`genipe` for the same analysis, but using the default probability
   threshold of 0.9 (excluding imputed genotypes with poor quality). Hence,
   58,769 markers were compared.

   .. figure:: ../_static/images/Logistic_Diff.png
      :align: center
      :width: 100%
      :alt: Logistic regression comparison between genipe and Plink

