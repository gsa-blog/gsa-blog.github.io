---
layout: post
title: "To 'p' or not to 'p' (that is our question)"
subtitle: '--On GSA and the p-value controversy--'
author: "Antonio Mora"
header-style: text
lang: en
tags:
  - GSA
  - Statistics
  - p-value
  - null hypothesis
  - effect size
  - confidence interval
  - Topconfects
  - QuSAGE
  - R
---

Like many of you, I am not a statistician but a “p-value user”. Computing p-values has become as common to me as dividing numbers or extracting their square root; however, just as division and square roots, there is usually a machine doing it for me: For some of us, “p-value users”, the p-values are still a little bit of a black box. That is why reading the “news” of p-value computation being under scrutiny is both interesting and concerning.

The following post is a summary of what I have learned about the p-value controversy and how is affecting my field of interest: bioinformatics, especially differential expression and gene set analysis.

### 1. A p-what?:
Let’s start by clarifying any possible confusion. Ask yourself “What is a p-value?”... Take your time until you have a good answer… Now continue… If you answered that (a) the p-value is the probability that the null hypothesis is true, or (b) it tells us if there is an effect (or a difference) in the experiment, or (c) the probability that some observations are produced by chance alone, thanks for trying, but YOU ARE WRONG! However, you are not alone in this: A review of 791 articles from 5 journals found that 51% of them wrongly interpreted the meaning of p-values and significance, usually assuming that non-significance means no effect [1].

p-values are definitively not magic numbers that decide what is true and what is not, or what is different and what is not. The technical definition of a p-value is “the probability of obtaining an effect at least as extreme as the one in your sample data, assuming the truth of the null hypothesis”. In other words, the p-value evaluates how likely are your data assuming a true null hypothesis; it is not about how likely is your hypothesis [2].

Let’s dig deeper on that definition: The null hypothesis (H0) is the statement that says that there is no relationship between two phenomena, or no difference between two groups. Usually, we want to reject H0 because we expect to find differences. Such a hypothesis corresponds to the entire population, but our data is made of samples. The samples can have differences due to sampling procedures, so we may ask how likely is that the samples reflect the population. In this context, a p-value of 0.01 indicates that, if H0 was true (if there is no difference between the groups), we still could have the observed differences/effect (or more) in 1% of the studies due to sampling error. Therefore, such a small p-value tells us that our data is actually unlikely assuming a true null. However, the p-value can’t tell you if (a) H0 is true but your sample is just unusual, or (b) H0 is false. It isn’t right to assume that small p-values imply (b) [2].

Misunderstanding the meaning of the p-values is the source of their misuse, and their misuse is the reason why some statisticians are calling for their retirement.

([https://reactome.org/PathwayBrowser/](https://reactome.org/PathwayBrowser/)) 
![A pathway from WikiPathways](/img/wikip.png)

_A pathway from WikiPathways_

### 2. p all over the place:
An ontology can be defined as a formal system for describing knowledge, which includes the terms in a certain field, their precise definition, and conceptual relationships among them (such as "is-a" or "part-of"). There are several biological ontologies, including the "Gene Ontology" (GO) (12), "Disease Ontology" (DO) (13), "Human Phenotype Ontology" (14), and many others, which can be discovered using ontology search services such as the "Ontology Lookup Service" (OLS) ([https://www.ebi.ac.uk/ols/index](https://www.ebi.ac.uk/ols/index)). When ontology terms are annotated with gene information, such as in GO, they become the most commonly used resource for GSA.

The Gene Ontology (GO) is a set of biological terms/concepts/classes that can be applied to genes, gene products or gene functions, as well as the relationships between these concepts. It is composed of: A dictionary (containing the definition of the terms involved in molecular and cell biology), an ontology (a formal system to describe the terms), and a map between genes and GO terms (GO annotation). GO terms are hierarchically organized, from more general terms to more specific terms. At the top of the hierarchy, there are three basic ontologies: biological process (pathways and larger processes), cellular component (localization of gene products), and molecular function. Each term in the GO tree may have multiple parents or multiple children, and each child is related to its parents by relationships such as "is-a", "part-of", "regulates", "occurs-in", or "capable-of". For example, "cellular process" (child) is-a "biological process" (parent). GO annotations are descriptions of the function and localization of specific gene products by creating links between gene names and GO terms. They can be created manually by curators or automatically. GO has a very detailed code for the different types of evidence recorded, ranging from EXP (inferred from experiment) to IC (inferred by curator) and IEA (inferred from electronic annotation). The detailed list of GO evidence codes can be found in their website ([www.geneontology.org/page/guide-go-evidence-codes](www.geneontology.org/page/guide-go-evidence-codes)).

![The Finger Ontology](/img/finger.PNG)

_Ontologies 101_

### Gene-set databases:
A "gene set" is a simpler and more general concept. Any list of genes that can be grouped for any reason can be defined as a gene set. For example, a group of genes differentially expressed (DE) in a given disease. An annotated ontology (such as GO), the list of genes in a pathway, or a group of phenotypically-related genes, may be gene sets as well.

Some examples of gene set databases are "GeneSetDB" (15), "MSigDB" (16) and "PAGED" (17). GeneSetDB contains several pathway databases (including Reactome and Wikipathways), disease/phenotype databases, drug/chemical databases, gene regulation databases, and the GO. MSigDB contains a series of sub-collections, such as: Positional gene sets (gene set per human chromosome and cytogenetic band), curated gene sets (chemical and genetic perturbations, canonical pathways, Biocarta, KEGG, Reactome), motif gene sets (miRNA targets, TF targets), computational gene sets (cancer gene neighborhoods and modules), GO, oncogenic signatures, and immunologic signatures.

### Functional Association Networks:
Functional Association Networks (FAN) are networks that integrate different types of source data such as pathway, co-expression, genetic interaction, protein interaction, and others. The nodes in such networks represent genes or gene products, while the edges represent the probability or degree of belief that those genes are functionally related. Some examples are: FunCoup (18) and GeneMania (19). FANs are mainly used as a reference dataset for a new class of gene set analysis methods (including NEA, EnrichNet, CrossTalkZ, NetPEA, and BinoX) that relies on network interaction.

### Unstructured sources of knowledge:
Biomedical literature may contain more pathway information than any existing biological database. However, reading the literature is a task that would take humans a prohibitive amount of time. Therefore, we need methodologies to automatically extract gene-set knowledge from the literature. "Text mining" tools have been developed for many years (20) and they are out of the scope of this review, but it would be interesting to discuss at least one example. MELODI ([http://www.melodi.biocompute.org.uk/](http://www.melodi.biocompute.org.uk/)) is a text mining tool that extracts mechanisms of disease based on subject-predicate-object triples from SemMedDB (Semantic Medline Database) (21). Using a database of triples for the biomedical literature, MELODI is able to link information from different papers and reconstruct the pathways for us. The text mining approach (MELODI+MedSemDB) is less taken into account nowadays, but the growth of the literature and the improvement of the text mining tools may lead us to consider it as another important source of gene set annotation.

### R databases:
Several of the above-mentioned pathway, ontology, and gene-set databases do exist as R packages, making it easier to integrate with the different R GSA methods. Examples of R databases are KEGGREST, reactome.db, and others.

KEGGREST allows access to the KEGG REST API. Some basic commands:
```R
library(KEGGREST)
query <- keggGet("hsa:10458")
query[[1]]
query[[1]]$DEFINITION
query[[1]]$PATHWAY

query2<- keggGet("hsa05130")
query2[[1]]
query2[[1]]$NAME
query2[[1]]$GENE

query3 <- keggGet("hsa05130", "image")
library(png)
writePNG(query3, "imagepath.png")
```

reactome.db is an R dataset with Reactome information. Some basic commands:
```R
library(reactome.db)
xx <- as.list(reactomeEXTID2PATHID)
xx['100']

yy <- as.list(reactomePATHID2EXTID)
yy['R-HSA-114608']

zz <- as.list(reactomePATHID2NAME)
zz['R-HSA-114608']
```

GeneSetDB can be found in the Bioconductor package EGSEAdata. Some basic commands:
```R
library(EGSEAdata)
egsea.data()
gsetdb.human[[3]]
```

MSigDB can be found in multiple R packages. However, RData files can be downloaded from here: [http://bioinf.wehi.edu.au/software/MSigDB/](http://bioinf.wehi.edu.au/software/MSigDB/) Among them, the “c2” dataset is one of the most used.

Two Bioconductor packages with ontology data (but not the gene annotation) are GO.db and DO.db. GO.db is a Bioconductor annotation package with Gene Ontology information.  For example:
```R
library(GO.db)
xx <- as.list(GOTERM)
xx[['GO:0000001']]
```

DO.db is a Bioconductor annotation package with Disease Ontology information.  For example:
```R
library(DO.db)
DOTERM[['DOID:0014667']]
```

However, in order to get the GO annotation, you would need a package like biomaRt:
```R
library(biomaRt)
ensembl = useMart("ensembl", dataset="hsapiens_gene_ensembl")
genes <- getBM (attributes = c('hgnc_symbol', 'ensembl_transcript_id', 'go_id'), filters = 'go', values = 'GO:0000002', mart = ensembl)
```

### Adding your own gene sets:
Finally, if you have your own gene sets, you may build R gene set objects with them and share them with the scientific community. Building an RData object would be enough, but it would be even better if you can store them in the same format used by the previous databases, which can be recognized by some GSA tools. In addition, tools for building gene sets have also been developed; see, for example, GSEABase ([http://bioconductor.org/packages/release/bioc/vignettes/GSEABase/inst/doc/GSEABase.pdf](http://bioconductor.org/packages/release/bioc/vignettes/GSEABase/inst/doc/GSEABase.pdf)).

### Discussion and thoughts:
As I said in the beginning, sometimes you may want to go beyond the databases offered by the GSA tool that you are using. Choosing a new database resource depends on which kind of information are you looking for (specific pathways? functions in the “Biological Process” Gene Ontology? specific disease association? all of them?). In general, we must keep in mind that pathways contain structural information than gene sets lack, and gene sets contain much more information than just pathways. In addition, in pathways, some genes can be up-regulated and some down-regulated at a given time, while gene sets may change in a coordinated fashion (either all genes up-regulated or down-regulated in a given disease, for example). It has been already discussed that some methodologies were designed to detect coordinated changes, and therefore researchers should be aware of that when choosing a pair of a GSA method and an annotated database (22). In general, over-representation analysis (ORA) and functional class scoring (FCS) methods work with gene set databases (including pathways), pathway databases are necessary for pathway topology based (PT) methods, while FANs are necessary for network interaction (NI) methods.

In any case, it is always important to take into account the trustworthiness and number of gene sets in the database, if they are open or proprietary, and how well they link to advanced GSA and visualization software. For that reason, even though "Pathguide" (23) lists "702 biological pathway related resources and molecular interaction related resources" ([http://www.pathguide.org](http://www.pathguide.org)), the truth is that just a few resources are frequently used. 

Current databases are highly incomplete and, therefore, before thinking about more sophisticated GSA analytical methods, it is wise to pay attention to having the right type of gene sets, in a good amount/coverage, and with a good quality. It has been said (in the context of machine learning) that “More data beats clever algorithms” and “Better data beats more data” ([https://www.azquotes.com/quote/1020712](https://www.azquotes.com/quote/1020712)). In the functional interpretation of biomedical data, we could also claim that no GSA method will give us the biological insights we are looking for if we don’t pay enough attention to the amount and the quality of the information contained in the annotation databases that we are using.

### References
1.	Ogata H, Goto S, Sato K, Fujibuchi W, Bono H, Kanehisa M. KEGG: Kyoto Encyclopedia of Genes and Genomes. Nucleic Acids Res. 1999;27(1):29-34.
2.	Fabregat A, Jupe S, Matthews L, Sidiropoulos K, Gillespie M, Garapati P, et al. The Reactome Pathway Knowledgebase. Nucleic Acids Res. 2017.
3.	Pico AR, Kelder T, van Iersel MP, Hanspers K, Conklin BR, Evelo C. WikiPathways: pathway editing for the people. PLoS Biol. 2008;6(7):e184.
4.	Cerami EG, Gross BE, Demir E, Rodchenkov I, Babur O, Anwar N, et al. Pathway Commons, a web resource for biological pathway data. Nucleic Acids Res. 2011;39(Database issue):D685-90.
5.	Soh D, Dong D, Guo Y, Wong L. Consistency, comprehensiveness, and compatibility of pathway databases. BMC Bioinformatics. 2010;11:449.
6.	Stobbe MD, Houten SM, Jansen GA, van Kampen AH, Moerland PD. Critical assessment of human metabolic pathway databases: a stepping stone for future integration. BMC Syst Biol. 2011;5:165.
7.	Altman T, Travers M, Kothari A, Caspi R, Karp PD. A systematic comparison of the MetaCyc and KEGG pathway databases. BMC Bioinformatics. 2013;14:112.
8.	Sales G, Calura E, Cavalieri D, Romualdi C. graphite - a Bioconductor package to convert pathway topology to gene network. BMC Bioinformatics. 2012;13:20.
9.	Bauer-Mehren A, Furlong LI, Sanz F. Pathway databases and tools for their exploitation: benefits, current limitations and challenges. Mol Syst Biol. 2009;5:290.
10.	Wittig U, De Beuckelaer A. Analysis and comparison of metabolic pathway databases. Brief Bioinform. 2001;2(2):126-42.
11.	Chowdhury S, Sarkar RR. Comparison of human cell signaling pathway databases--evolution, drawbacks and challenges. Database (Oxford). 2015;2015.
12.	Ashburner M, Ball CA, Blake JA, Botstein D, Butler H, Cherry JM, et al. Gene ontology: tool for the unification of biology. The Gene Ontology Consortium. Nat Genet. 2000;25(1):25-9.
13.	Schriml LM, Arze C, Nadendla S, Chang YW, Mazaitis M, Felix V, et al. Disease Ontology: a backbone for disease semantic integration. Nucleic Acids Res. 2012;40(Database issue):D940-6.
14.	Groza T, Kohler S, Moldenhauer D, Vasilevsky N, Baynam G, Zemojtel T, et al. The Human Phenotype Ontology: Semantic Unification of Common and Rare Disease. Am J Hum Genet. 2015;97(1):111-24.
15.	Araki H, Knapp C, Tsai P, Print C. GeneSetDB: A comprehensive meta-database, statistical and visualisation framework for gene set analysis. FEBS Open Bio. 2012;2:76-82.
16.	Liberzon A, Subramanian A, Pinchback R, Thorvaldsdottir H, Tamayo P, Mesirov JP. Molecular signatures database (MSigDB) 3.0. Bioinformatics. 2011;27(12):1739-40.
17.	Huang H, Wu X, Sonachalam M, Mandape SN, Pandey R, MacDorman KF, et al. PAGED: a pathway and gene-set enrichment database to enable molecular phenotype discoveries. BMC Bioinformatics. 2012;13 Suppl 15:S2.
18.	Alexeyenko A, Sonnhammer EL. Global networks of functional coupling in eukaryotes from comprehensive data integration. Genome Res. 2009;19(6):1107-16.
19.	Mostafavi S, Ray D, Warde-Farley D, Grouios C, Morris Q. GeneMANIA: a real-time multiple association network integration algorithm for predicting gene function. Genome Biol. 2008;9 Suppl 1:S4.
20.	Raychaudhuri S, Schutze H, Altman RB. Using text analysis to identify functionally coherent gene groups. Genome Res. 2002;12(10):1582-90.
21.	Kilicoglu H, Shin D, Fiszman M, Rosemblat G, Rindflesch TC. SemMedDB: a PubMed-scale repository of biomedical semantic predications. Bioinformatics. 2012;28(23):3158-60.
22.	Tarca AL, Bhatti G, Romero R. A comparison of gene set analysis methods in terms of sensitivity, prioritization and specificity. PLoS One. 2013;8(11):e79217.
23.	Bader GD, Cary MP, Sander C. Pathguide: a pathway resource list. Nucleic Acids Res. 2006;34(Database issue):D504-6.
