<img src="https://i.imgur.com/KnmY1Yy.png" height=200 align="center">

# PGL-GRT

PlantGeneticsLab - Genotype Retrieving Toolkit<br />
[![图片alt](https://i.imgur.com/8tW8TDV.png)](https://www.dropbox.com/s/r4gs2fefexew27o/PlantGenetics.jar?raw=1)


###### Mar 13, 2019

### Authors
Daxing Xu (dxxu@genetics.ac.cn)<sup>1,2</sup>, Jing Wang (jing.wang@genetics.ac.cn)<sup>1</sup>, 
Changbin Yin (yinchangbin@genetics.ac.cn)<sup>1</sup>, Fei Lu (flu@genetics.ac.cn)<sup>1,3</sup>
<br /><br />
### Affiliations
<sup>1 </sup>State Key Laboratory of Plant Cell and Chromosome Engineering , Institute of Genetics and Developmental Biology, The Innovative Academy of Seed Design, Chinese Academy of Sciences

<sup>2 </sup>University of Chinese Academy of Sciences

<sup>3 </sup>CAS-JIC Centre of Excellence for Plant and Microbial Science (CEPAMS), Shanghai Institutes for Biological Sciences, Chinese Academy of Sciences
<br /><br />
### Overview
Genotype retrieving toolkit (GRT) includes a set of utilities to analyze data derived from the two-enzyme genotyping-by-sequencing (GBS) approach<sup>[1](#ref)</sup>. It has been designed to achieve a goal of being accurate, robust, and efficient while conducting SNP calling and genotyping.<br />
Different from previous GBS pipelines<sup>[2,3](#ref)</sup>,  GRT has 3 features:

1. GRT supports longer sequencing tags and paired-end mapping to further increase mapping accuracy. A GRT tag by default is a 192 bp, paired-end sequence, which is aligned to the reference genome in the paired-end mode in BWA-MEM. It suits well to deal with complex genomes (e.g. wheat).
2. GRT is designed to have decentralized data collection and centralized data processing, to achieve genotyping consistency between various breeding programs. GRT uses large sequencing data sets from diverse samples to build a genetic variant database of a species, then genotypes of samples to be examined can be “retrieved” from the database. Consistent genotype can be generated from breeding program to breeding program, from generation to generation.
3. GRT is efficient for genotyping, because it scans through a genetic variant database of a species instead of aligning reads for genotype calling.<br /><br />
<div style="text-align: justify">There are two major modules in GRT, the database building module and the genotyping module as shown below. </div><br />
<div style="text-align: justify">The first module is used to build a variants database of a species from sequencing data of large amounts of samples, which can scale up to more than 100,000. By default, a 192 bp, paired-end sequence is considered as a GRT tag. Tags from all samples are merged into a tag database (DB). By aligning tags in the DB, SNP calling and allele calling are performed to add allele information to the tags in the DB. Combining tag data of each sample, genotyping can be done to generate the raw genotype. Since spurious SNPs derived from sequencing error and misalignment persists in the genotype data set, costumed genetic filters (MAF, segregation test, LD test, etc.) can be applied to filter out those spurious calls. The SNPs of validated genotype data can be used to filter and finalize the tag database for the genotyping module.</div><br />
<div style="text-align: justify">The second module is used to assign genotypes for GBS sequenced samples. By using tags as queries and scanning through the database to retrieve allele information, It can generate consistent genotype across breeding programs and generations.</div><br /><br />

![pipeline of GRT](https://i.imgur.com/YrCZX9u.png)<br /><br />


GRT is written in Java, and packed with JDK 8. Hence, it can run on Linux, Unix, Mac-OS, and Windows systems with Java 8 or later versions installed.<br /><br />
<a name="options">
### Options</a>
Options|Description|value
-------|-------|------
**-m**|Analysis mode|pf, mt, at, cs, rs, ca, cg, fd, rg <br />
**-w**|Working directory, where sub-directories are created for analysis<br />|./
**-b**|The barcode file, where sample barcoding information is stored<br />|/users/.../barcodefile.txt
**-f**|The libraryFastqMap file, where corresponding fastq files can be found for each flowcell_lane_library-index combination<br />|/users/.../libraryFastqMap.txt
**-ef**|Recognition sequence of restriction enzyme in R1, e.g GGATCC<br />|GGATCC
**-er**|Recognition sequence of restriction enzyme in R2, e.g CCGG<br />|CCGG
**-t**|Number of threads. The default value is 32. The actual number of running threads is less than the number of cores regardless of the input value, but -1 means the number of all available cores<br />|32
**-g**|The reference genome of the species. The indexing files should be included in the same directory of the reference genome<br />|/users/.../iwgscV1.fa.gz
**-bwa**|The path of bwa executable file, e.g /Users/Software/bwa-0.7.15/bwa<br />|/users/.../bwa
**-mc**|The minimum read count of tag in database. The default value is 3<br />|3
**-mca**|The minimum read count of an alternative allele. The default value is 10<br />|10
**-mq**|The minimum read mapping quality for SNP calling and allele calling.<br /> The default value is 30<br />|30
**-ml**|The maximum range of paired-end read mapping (The insert fragment size of DNA). The default value is 1000<br />|1000
**-md**|The maximum divergence between a tag and the reference genome, which is a quality control in SNP calling. The default value is 7<br />|7
**-it**|The tag identify threshold. While searching the tag DB, query tag having more mismatch than the value is not considered as a match. The default value is 3|3

<br /><br />
### Utilities
For module 1, you should follow orderly [***Parsing fastqs***](#pf), [***Merging tags***](#mt), [***Aligning tags***](#at), [***Calling SNPs***](#cs), [***RemoveLowCountSNP***](#rlcs), [***Calling alleles***](#ca), [***Calling genotype***](#cg), [***Filtering database***](#fd), then a SNP variance DB of a species can be builded, which covered almost all of the diversity of this species. Hence, you can get varience information of any other samples by performing module 2, even it was not used for constructing the DB.<br /><br />
For module 2,  you should follow orderly [***Parsing fastqs***](#pf), [***Retrieving genotype***](#rg).<br /><br /><br />
<a name="pf">
#### ***Parsing fastqs***</a>
After getting your fastq file (clean data) of individual library using two-enzyme GBS method,  you should prepare the following two  input files, including barcode file  and libraryFastqMap file. In the barcold file, you see for example,<br />

Library-name|Well-ID|Library-index|EF-Barcode|ER-Barcode|Sample-ID|Taxon-name
-------------|------|-------------|----------|----------|---------|----------
library-01|A01|ATCACG|AGGC|GCAG|W0001|taxon1
library-01|A02|ATCACG|GTCC|AACG|W0002|taxon2
library-01|A03|ATCACG|GCGA|GTCA|W0003|taxon3
library-01|A04|ATCACG|TGCT|CTGG|W0004|taxon4
...|
...|
<br />
<div style="text-align: justify">Although the above barcode file is the standard file format, you should pay attention to that a modified barcode file, not the standard barcode file, was used in this GRT pipeline. In the modified barcode file, you see for example,</div><br />

Taxon-name|Flowcell-ID|Lane|Library-index|Well-ID|EF-Barcode|ER-Barcode|Sample-ID
-------------|------|-------------|----------|----------|---------|--------|--------
 taxon1|H5FVTDSXX|4|ATCACG|A01|AGGC|GCAG|W0001
 taxon2|H5FVTDSXX|4|ATCACG|A02|GTCC|AACG|W0002
 taxon3|H5FVTDSXX|4|ATCACG|A03|GCGA|GTCA|W0003
 taxon4|H5FVTDSXX|4|ATCACG|A04|TGCT|CTGG|W0004
 ...|
 ...|
 <br />
<div style="text-align: justify">Here are reasons. In the standard barcode file, the library-name should be unique among all the libraries. But we usually need to test the quality of all our libraries based on a small sequencing data volume, so a library may be sequenced twice generating the repeated Library-name. In the modified barcode file, the Library-name was replaced by the Flowcell-ID_Lane. This will distinguish different batches of the same library. Because if a library is sequenced twice, it will have different Flowcell-ID_Lane (Attention, a library can’t be sequenced twice in the same lane of the same flowcell !!!). </div><br />
<div style="text-align: justify">The Barcode file can have duplicate samples of a Taxon-name, but this sample will be unique in the combination of Flowcell-ID_Lane_Library-Index_Well-ID. The Sample-ID will not be used to downstream analysis, so you can use any name if you want.</div><br />
<div style="text-align: justify">Another input file is a libraryFastqMap file, where each fastq file can be found for each Flowcell-ID_Lane_Library-Index combination.The Flowcell-ID_Lane_Library-Index can be used to determine a specific library even if it has been sequenced twice.  In the libraryFastqMap, you see for example, </div><br />

Flowcell-ID|Lane|Library-index|R1Path|R2Path
-----------|-----|------------|------|------
H5FVTDSXX|3|ATCACG|/Users/.../library-01_R1.clean.fq.gz|/Users/.../library-01_R2.clean.fq.gz
H5FVTDSXX|4|GGCTAC|/Users/.../library-02_R1.clean.fq.gz|/Users/.../library-02_R2.clean.fq.gz
HNF5WCCXY|2|CTTGTA|/Users/.../library-03_R1.clean.fq.gz|/Users/.../library-03_R2.clean.fq.gz
HNF5WCCXY|2|AGTTCC|/Users/.../library-04_R1.clean.fq.gz|/Users/.../library-04_R2.clean.fq.gz
...|
...|

 If you have prepared these two files successfully, then you can perform following command.  <br />
 ~~~
 nohup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m pf  -w ./  -b /users/.../barcodefile.txt  -f /users/.../libraryFastqMap.txt  -ef GGATCC  -er CCGG  -t 8   >./pfLog.txt &
 ~~~
This command will generate many compressed binary files such as taxon1_H5FVTDSXX_4_ATCACG_A01.tas and taxon2_H5FVTDSXX_4_ATCACG_A02.tas in directory ./tagsBySample. Meanwhile, a __<font face="fjalla one" size=4>pfLog.txt</font>__ file containing basic information about [***Parsing fastq***](#pf) will be generated in current working directory.<br /><br />
__<font face="fjalla one" size=3>-Xms400g</font>__ means setting the initial and minimum heap size. __<font face="fjalla one" size=3>-Xmx400g</font>__ means setting the maximum heap size.<br /> It is recommended to set the maximum heap size to equivalent to the minimum heap size in order to minimize the garbage collection. When set __<font face="fjalla one" size=3>-Xms400g</font>__ and __<font face="fjalla one" size=3>-Xmx400g</font>__, 1T size data consistent with 2.5T bases can run successful in our testing progress. You can choose proper value of Xms and Xmx according to your data size. __<font face="fjalla one" size=3>[-jar](#options) /users/.../PlanGenetics.jar</font>__ is used to specify the absolute path of GRT software. You should specify ***Parsing fastqs*** analysis model using option __<font face="fjalla one" size=3>[-m](#options) pf</font>__. Other options please refer to above.<br /><br /><br />
<a name="mt">
#### ***Merging tags***</a>
Only when you accomplish ***Parsing fastqs***, can you run this step. This step does not require any input files, just perform following command.<br />
~~~
nohup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m mt  -w ./  -mc 3  >./mtLog.txt &
~~~
This command will generate a compressed binary tag.tas file which is a  predecessor of the database. You can find this file in ./tagsLibrary/ directory. <br /><br />
It is worth noting that ***Merging tags*** analysis mode is performed by using __<font face="fjalla one" size=3>[-m](#options) mt</font>__ option. And the option of [-mc](#options) is 3 by default. To understand the mc options, first you need to understand the tag.<br /><br />
A tag is a pair of double-end sequencing reads where barcode sequence have been removed. Both paired reads have been shortened to 96 bp in order to remove the bases at the 3’ end which have the low base quality value and to compress the the paired reads so as to reduce the memory consumption (Fig. 1). And the two tags are equal only when the two reads of one tag are the same as the bases of the two reads of the other tag. So one tag can have many duplicate. The minimum duplicate number of one tag is 3 by default, which means every tag at least having 3 duplicate in our wheat variance database. It is up to you to adjust this value by using __<font face="fjalla one" size=3>[-mc](#options)</font>__ option.<br /><br />

<p align="middle"><img src="https://i.imgur.com/Mb5ZZIk.png" width=700></p>
<br /><br />

<a name="at">

#### ***Aligning tags***</a>
After getting your predecessor of the DB, you can run this step. <br />
~~~
nohup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m at  -w ./  -g /users/.../iwgscV1.fa.gz  -bwa /users/.../bwa  -t 8  >./atLog.txt &
~~~
You will get a compressed file tag.sam.gz in ./alignment directory. Additionally, two fastq files containing non-redundant reads from tag.tas file are in the same directory. The value of __<font face="fjalla one" size=3>[-t](#options)</font>__ option can be changed according to the number of logical CPU. <br />

It is worth noting that bwa must be compiled and the reference genome must be indexed before be used. You can refer to the manual of bwa in [http://bio-bwa.sourceforge.net/bwa.shtml](https://www.google.com).
<br /><br /><br />
<a name="cs">
#### ***Calling SNPs***</a>
When you get the tag.sam.gz and two fastq files, you can perform this step. <br />
~~~
nohup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m cs  -w ./  -md 5  -mq 30  -ml 1000   >./csLog.txt &
~~~
This command will update database by adding SNP information in tag.tas file. Meanwhile, a binary rawSNP.bin file representing the whole SNP information on all chromosomes will be generated in ./tagsLibrary/ directory.<br /><br /><br />
<a name="rlcs">
#### ***RemoveLowCountSNP***</a>
After adding SNPs information in DB, you can run this step.<br />
~~~
nohup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m rs  -w ./  -mca 10   >./rsLog.txt &
~~~
If an alternative allele have less than 10 reads count, then it will be removed by adding option __<font face="fjalla one" size=3>[-mca](#options) 10</font>__. It worth noting that __<font face="fjalla one" size=3>[-mca](#options)</font>__ option is different from __<font face="fjalla one" size=3>[-mc](#options)</font>__ option. The value of __<font face="fjalla one" size=3>[-mca](#options)</font>__ option equal 10 meaning that any alternative allele must have at least 10 reads count. Otherwise, it will be filtered out from the binary rawSNP.bin file.<br /><br /><br />
<a name="ca">
#### ***Calling alleles***</a>
When you finished last step, you can run this step. <br />
~~~
nohup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m ca  -w ./  -mq 30   -ml 1000   >./caLog.txt &
~~~
This command will also update the DB by adding tag alleles information in tag.tas file.<br /><br /><br />
<a name="cg">
#### ***Calling genotype***</a>
After finishing adding alleles in the DB, you can run this step.<br />
~~~
nphup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m cg  -w ./  -t 8  -it 3  >./cgLog.txt &
~~~
The VCF files of all chromosomes will be generated in ./rawGenptype/genotype/ directory. Meanwhile a small sample file of the DB will be generated in ./tagsLibrary/tag.tas.txt. You can open this file by any text editor.<br /><br /><br />
<a name="fd">
#### ***Filtering database***</a>
When you finished last step, you can run this step.<br />
~~~
nohup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m fd  -w ./  >./fdLog.txt &
~~~
This command will filter the DB and the rawSNP.bin file according to a VCF file which have high quality genotype information confirmed by your experiment.<br /><br /><br />
<a name="rg">
#### ***Retrieving genotype***</a>
When you finished building the DB and filtering it successfully, you can perform ***Parsing fastqs***（please refer above [***Parsing fastqs***](#pf)）and ***Retrieving genotype*** (please refer to following instructions) orderly to get genotype information of your individual sample.<br />
~~~
nohup  java  -Xms400g  -Xmx400g  -jar /users/.../PlanGenetics.jar  -m rg  -w ./  >./rgLog.txt &
~~~
<a name="ref">
<br />
 
### How to cite
 Our paper is coming soon
 <br /><br />
### References</a><font size=3>
1.	Poland, J. A. et al. Development of High-Density Genetic Maps for Barley and Wheat Using a Novel Two-Enzyme Genotyping-by-Sequencing Approach. *PLoS ONE* 7, e32253 (2012).<br />(https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0032253)<br />
2.	Glaubitz, J. C. et al. TASSEL-GBS: A High Capacity Genotyping by Sequencing Analysis Pipeline. *PLoS ONE* 9, e90346 (2014).<br />(https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0090346)<br />
3.	Lu, F. et al. Switchgrass Genomic Diversity, Ploidy, and Evolution: Novel Insights from a Network-Based SNP Discovery Protocol. *PLoS Genet*. 9, e1003215 (2013).</font><br />(https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1003215)<br />

