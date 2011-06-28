.. _processing_18S_data:


Processing 18S data
-------------------

This tutorial explains how to use the **QIIME** pipeline to analyze 18S, or mixed 18S/16S, data.  While most of the steps are identical to the standard 16S pipeline described in the overview `tutorial <tutorial.html>`_, reference data that include eukaryotic sequences may be required for OTU picking, taxonomic assignments, and template-based alignment building.

To this end, a QIIME-compatible version of the `Silva <http://www.arb-silva.de/>`_ 104 release, which contains data for all three domains of life, is available here: http://www.arb-silva.de/download/archive/qiime/.  The `Qiime_files_r104.zip` archive contains several files that will be utilized in this tutorial, and can be used for analyzing 18S datasets.

A sample fasta file containing sequences from archaea, bacteria, and eukaryotes can be found in the /qiime_tutorial/18S_tutorial_files/ which will be used along with the Silva reference set in this tutorial.

Each of the following steps are described in this tutorial (the remaining analyses are identical to those described in the general 16S `tutorial <tutorial.html>`_):

* Picking Operational Taxonomic Units (OTUs)
* Assigning OTUs to taxonomic identity
* Separating OTU tables according to domain
* Generating and filtering an alignment to construct a phylogenetic tre

.. _pickotus:

Making OTU tables for 18S datasets
==================================

Here we will cover picking OTUs.  Using the default settings with `pick_otus.py <../scripts/pick_otus.html>`_ is no different than with 16S data.  Sequences will be clustered at 97% identity and new clusters will be allowed.  However, one can utilize a reference based approach with the Silva dataset listed above that discards sequences that do not cluster with the reference set.  The advantage to using this approach is that one can use the reference taxonomic assignments and tree rather than generating new ones, however, a large portion of sequences may be discarded.

To pick OTUs against the Silva reference data set and discard any clusters that do not match, use the following command: ::

	pick_otus.py -i 18S_tutorial_sample_seqs.fna -m uclust_ref -C -r QIIME_files/rep_set/silva_104_rep_set.fasta -o uclust_ref_picked_otus/

This will generate a `18S_tutorial_sample_seqs_otus.txt` OTU mapping file in the uclust_ref_picked_otus folder.  An OTU table with taxonomy strings from the reference dataset can be built directly.  To do so now, use the following command: ::

	make_otu_table.py -i uclust_ref_picked_otus/18S_tutorial_sample_seqs_otus.txt -t QIIME_files/taxonomy_mapping/Silva_taxa_mapping_104set_97_otus.txt -o otu_table_uclust_ref.txt

which will generate an OTU table, as shown below:

.. note::

	* # QIIME v1.2.1-dev OTU table
	* #OTU ID	AB019754	AB032231	AB294260	AB294267	AF391990	AY642706	AY854217	DQ521781	DQ809034	DQ889882	EF018454	EF100339	EU284615	Consensus Lineage
	* 1347	0	1	0	0	0	0	0	0	0	0	0	0	0	Eukaryota;Parabasalia;Trichonymphida;Teranymphidae;Eucomonympha;;Eucomonympha
	* 3838	0	0	0	0	0	0	0	0	0	0	0	0	1	Archaea;Crenarchaeota;Soil
	* 5518	0	0	0	1	0	0	0	0	0	0	0	0	0	Archaea;Crenarchaeota;AK31;uncultured
	* 12442	0	0	0	0	1	0	0	0	0	0	0	0	0	Archaea;Crenarchaeota;Miscellaneous
	* 13457	0	0	0	0	0	0	0	0	0	1	0	0	0	Bacteria;Proteobacteria;Alphaproteobacteria;Rhodospirillales;Rhodospirillaceae;uncultured;uncultured
	* 25010	0	0	0	0	0	0	1	0	0	0	0	0	0	Eukaryota;Metazoa;Nematoda;Chromadorea;Desmodorida;Desmodoridae;Spiriniinae;Spirinia;;Spirinia
	* 25076	0	0	0	0	0	1	0	0	0	0	0	0	0	Eukaryota;Fungi;Chytridiomycota;environmental
	* 38221	0	0	0	0	0	0	0	0	0	0	0	1	0	Eukaryota;environmental
	* 39680	0	0	0	0	0	0	0	0	0	0	1	0	0	Bacteria;Proteobacteria;Deltaproteobacteria;Myxococcales;Sorangiineae;Polyangiaceae;Sorangium;uncultured
	* 43905	0	0	1	0	0	0	0	0	0	0	0	0	0	Archaea;Crenarchaeota;South
	* 78139	0	0	0	0	0	0	0	0	1	0	0	0	0	Bacteria;Bacteroidetes;Bacteroidia;Bacteroidales;Bacteroidaceae;Bacteroides;uncultured
	* 88700	1	0	0	0	0	0	0	0	0	0	0	0	0	Archaea;Euryarchaeota;Thermoplasmata;Marine
	* 96301	0	0	0	0	0	0	0	1	0	0	0	0	0	Archaea;Euryarchaeota;Thermoplasmata;Thermoplasmatales;Marine


To generate *de novo* OTUs, use the following command: ::

	pick_otus.py -i 18S_tutorial_sample_seqs.fna -o uclust_picked_otus/

This will create a `18S_tutorial_sample_seqs_otus.txt` file in the uclust_picked_otus directory, which can be used to make a representative sequence set for downstream analysis with the following command: ::

	pick_rep_set.py -i uclust_picked_otus/18S_tutorial_sample_seqs_otus.txt -f 18S_tutorial_sample_seqs.fna -o rep_set.fna

To assign taxonomies to the *de novo* OTUs that were just generated, use the following command: ::

	assign_taxonomy.py -i rep_set.fna -o rdp_assigned_taxonomy/ -t QIIME_files/taxonomy_mapping/Silva_RDP_taxa_mapping.txt -r QIIME_files/rep_set/silva_104_rep_set.fasta


If BLAST is the preferred method of assignment, the generic taxonomic mapping file can be used instead: ::

	assign_taxonomy.py -i rep_set.fna -o blast_assigned_taxonomy/ -t QIIME_files/taxonomy_mapping/Silva_taxa_mapping_104set_97_otus.txt -r QIIME_files/rep_set/silva_104_rep_set.fasta -m blast

Finally, an OTU table can be built which includes the taxonomic assignments (in this case we will use the RDP assignments): ::

	make_otu_table.py -i uclust_picked_otus/18S_tutorial_sample_seqs_otus.txt -t rdp_assigned_taxonomy/rep_set_tax_assignments.txt -o otu_table.txt

Separating OTU Tables According to Domain
=========================================

It may be desirable to split the OTU table according to domain for mixed 16S/18S datasets.  To do this, we will use the  `split_otu_table_by_taxonomy.py` module.

We will split the OTU table generated in the last step at the domain level, 2, by using the following command: ::

	split_otu_table_by_taxonomy.py -i otu_table.txt -L 2 -o separated_otu_tables/

The output directory, separated_otu_tables, will contain an OTU table for archaea, bacteria, and eukaryotes, which can be utilized in downstream QIIME analyses just as any OTU table.

Alignments and Tree Building
============================

To build a tree utilizing the Silva 104 reference set, we will first create an alignment with the `align_seqs.py <../scripts/align_seqs.html>`_ module.  The core Silva aligned set will be used as the template.

Use the following command with the `rep_set.fna` created in the OTU picking step above: ::

	align_seqs.py -i rep_set.fna -t QIIME_files/core_aligned_set/core_Silva_aligned.fasta -o pynast_aligned/

Next, the alignment must be filtered.  For 16S datasets, a Lanemask is usually applied to remove high entropy positions.  QIIME has incorporated a dynamic entropy and gap calculation to the `filter_alignment.py <../scripts/filter_alignment.html>`_ module, which removes the need for a Lanemask.  To filter the alignment created above, use the following command: ::

	filter_alignment.py -i pynast_aligned/rep_set_aligned.fna -o pynast_aligned/ -e 0.10 -g 0.80

In this case, the 10% most variable positions and positions that are greater than 80% gaps were removed (the -e and -g parameters respectively).

Finally, a tree can be built using `make_phylogeny.py <../scripts/make_phylogeny.html>`_: ::

	make_phylogeny.py -i pynast_aligned/rep_set_aligned_pfiltered.fasta -o rep_set.tre

Trees an OTU tables created can then be utilized in the downstream QIIME analyses as seen in the `tutorial <tutorial.html#view-statistics-of-the-otu-table>`_.

Workflow Scripts
================

The Silva 104 reference set can be used in a workflow, such as `pick_otus_through_otu_table.py <../scripts/pick_otus_through_otu_table.html>`_.  It is necessary to modify the `qiime_parameters.txt` file to correctly point to the Silva reference filepaths, and to use the dynamic alignment filtering rather than the 16S Lanemask.  See the `documentation <../documentation/file_formats.html#qiime-parameters>`_ for details about the `qiime_parameters.txt` file.

Parameters that should be modified:

	* pick_otus:otu_picking_method	uclust (should be set to uclust_ref if a reference based approach is desired)
	* pick_otus:refseqs_fp (specify the filepath to the representative Silva 104 set, if reference based approach is desired)

	* align_seqs:template_fp (specify the core aligned Silva 104 fasta file path)

	* filter_alignment:lane_mask_fp (do not specify a lanemask filepath)
	* filter_alignment:allowed_gap_frac	0.999999 (set to 0.80 instead of default)
	* filter_alignment:entropy_threshold	0.10 (set to 0.10 if not already set)

	* assign_taxonomy:id_to_taxonomy_fp (specify the taxonomy mapping file path, RDP version if RDP is the method of choice)
	* assign_taxonomy:reference_seqs_fp (specify the Silva representative set file path)

