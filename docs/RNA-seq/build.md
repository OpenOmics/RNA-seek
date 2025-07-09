# <code>rna-seek <b>build</b></code>

## 1. About 
The `rna-seek` executable is composed of several inter-related sub commands. Please see `rna-seek -h` for all available options.

This part of the documentation describes options and concepts for <code>rna-seek <b>build</b></code> sub command in more detail. With minimal configuration, the **`build`** sub command enables you to build new reference files for the rna-seek run pipeline.    

Setting up the RNA-seek build pipeline is fast and easy! In its most basic form, <code>rna-seek <b>build</b></code> only has *five required inputs*.

## 2. Synopsis
```text
$ rna-seek build [--help] \
             [--shared-resources SHARED_RESOURCES] [--small-genome] \
             [--dry-run] [--singularity-cache SINGULARITY_CACHE] \
             [--sif-cache SIF_CACHE] [--tmp-dir TMP_DIR] \
             --ref-fa REF_FA \
             --ref-name REF_NAME \
             --ref-gtf REF_GTF \
             --gtf-ver GTF_VER \
             --output OUTPUT
```

The synopsis for each command shows its parameters and their usage. Optional parameters are shown in square brackets.

A user **must** provide the genomic sequence of the reference's assembly in FASTA format via `--ref-fa` argument, an alias for the reference genome via `--ref-name` argument, a gene annotation for the reference assembly via `--ref-gtf` argument, an alias or version for the gene annotation via the ` --gtf-ver` argument, and an output directory to store the built reference files via `--output` argument. If you are running the pipeline outside of Biowulf, you will need to additionally provide the the following options: `--shared-resources`, `--tmp-dir`. More information about each of these options can be found below.

For [human](https://www.gencodegenes.org/human/) and [mouse](https://www.gencodegenes.org/mouse/) data, we highly recommend downloading the latest available `PRI` genome assembly and corresponding gene annotation from [GENCODE](https://www.gencodegenes.org/). These reference files contain chromosomes and scaffolds sequences. 

 The build pipeline will generate a JSON file containing key, value pairs to required reference files for the <code>rna-seek <b>run</b></code> pipeline. This file will be located in the path provided to `--output`. The name of this JSON file is dependent on the values provided to `--ref-name` and `--gtf-ver` and has the following naming convention: `{OUTPUT}/{REF_NAME}_{GTF_VER}.json`. Once the build pipeline completes, this reference JSON file can be passed to the `--genome` option of <code>rna-seek <b>run</b></code>. This is how new references are built for the RNA-seek pipeline. 

Use you can always use the `-h` option for information on a specific command. 

### 2.1 Required Arguments

Each of the following arguments are required. Failure to provide a required argument will result in a non-zero exit-code.

  `--ref-fa REF_FA`  
> **Genomic FASTA file of the reference genome.**  
> *type: file*
> 
> This file represents the genome sequence of the reference assembly in FASTA format. If you are downloading this from GENCODE, you should select the *PRI* genomic FASTA file. This file will contain the primary genomic assembly (contains chromosomes and scaffolds). This input file should not be compressed. Sequence identifers in this file must match with sequence identifers in the GTF file provided to `--ref-gtf`.  
> 
> ***Example:***
> `--ref-fa GRCh38.primary_assembly.genome.fa`


---  
  `--ref-name REF_NAME`  
> **Name of the reference genome.**  
> *type: string*
> 
> Name or alias for the reference genome. This can be the common name for the reference genome. Here is a list of common examples for different model organisms: mm10, hg38, rn6, danRer11, dm6, canFam3, sacCer3, ce11. If the provided values contains one of the following sub-strings (hg19, hs37d, grch37, hg38, hs38d, grch38, mm10, grcm38), then Arriba will run with its corresponding blacklist. 
> 
> ***Example:*** `--ref-name hg38`

--- 
  `--ref-gtf REF_GTF`  
> **Gene annotation or GTF file for the reference genome.**  
> *type: file*
> 
> This file represents the reference genome's gene annotation in GTF format. If you are downloading this from GENCODE, you should select the 'PRI' GTF file. This file contains gene annotations for the primary assembly (contains chromosomes and scaffolds). This input file should not be compressed. Sequence identifers (column 1) in this file must match with sequence identifers in the FASTA file provided to `--ref-fa`.  
> ***Example:***  `--ref-gtf gencode.v36.primary_assembly.annotation.gtf`

---
  `--gtf-ver GTF_VER`  
> **Version of the gene annotation or GTF file provided.**  
> *type: string or int*
> 
> This is the version of the supplied gene annotation or GTF file. If you are using a GTF file from GENCODE, use the release number or version (i.e. *M25* for mouse or *37* for human). Visit gencodegenes.org for more details.  
> ***Example:*** `--gtf-ver 36`
  
--- 
  `--output OUTPUT`
> **Path to an output directory.**  
> *type: path*
>   
> This location is where the build pipeline will create all of its output files. If the user-provided working directory has not been initialized, it will automatically be created.  
> ***Example:*** `--output /data/$USER/refs/hg38_v36/`

### 2.2 Build Options

Each of the following arguments are optional and do not need to be provided. If you are running the pipeline outside of Biowulf, the `--shared-resources` option only needs to be provided at least once. This will ensure reference files that are shared across different genomes are downloaded locally.

  `--shared-resources SHARED_RESOURCES`  
> **Local path to shared resources.**  
> *type: path*
>
> The pipeline uses a set of shared reference files that can be re-used across reference genomes. These currently include reference files for kraken and FQScreen. These reference files can be downloaded with the build sub command's `--shared-resources`  option. With that being said, these files only need to be downloaded once. We recommend storing this files in a shared location on the filesystem that other people can access. If you are running the pipeline on Biowulf, you do NOT need to download these reference files! They already exist on the filesystem in a location that anyone can acceess; however, if you are running the pipeline on another cluster or target system, you will need to download the shared resources with the build sub command, and you will need to provide this option every time you run the pipeline. Please provide the same path that was provided to the build sub command's --shared-resources option. Again, if you are running the pipeline on Biowulf, you do NOT need to provide this option. For more information about how to download shared resources, please reference the build sub command's `--shared-resources` option.
> 
> ***Example:*** `--shared-resources /data/shared/rna-seek`

---  
  `--small-genome`            
> **Builds a small genome index.**  
> *type: boolean*
> 
> For small genomes, it is recommeded running STAR with a scaled down `--genomeSAindexNbases` value. This option runs the build pipeline in a mode where it dynamically finds the optimal value for this option using the following formula: `min(14, log2(GenomeSize)/2 - 1)`. Generally speaking, this option is not really applicable for most mammalian reference genomes, i.e. human and mouse; however, researcher working with very small reference genomes, like S. cerevisiae ~ 12Mb, should provide this option.
>
> When in doubt feel free to provide this option, as the optimal value will be found based on your input. It is also worth noting that if you are working with a prokaryotic genome, like a bacterial genome, you will run to provide the `--prokaryote` option to the [run subcommand](./RNA-seq/run/).
>
> ***Example:*** `--small-genome`

### 2.3 Orchestration Options

  `--dry-run`            
> **Dry run the build pipeline.**  
> *type: boolean*
> 
> Displays what steps in the build pipeline remain or will be run. Does not execute anything!
>
> ***Example:*** `--dry-run`

--- 
  `--singularity-cache SINGULARITY_CACHE`  
> **Overrides the $SINGULARITY_CACHEDIR environment variable.**  
> *type: path*  
> *default: `--output OUTPUT/.singularity`*
>
> Singularity will cache image layers pulled from remote registries. This ultimately speeds up the process of pull an image from DockerHub if an image layer already exists in the singularity cache directory. By default, the cache is set to the value provided to the `--output` argument. Please note that this cache cannot be shared across users. Singularity strictly enforces you own the cache directory and will return a non-zero exit code if you do not own the cache directory! See the `--sif-cache` option to create a shareable resource. 
> 
> ***Example:*** `--singularity-cache /data/$USER/.singularity`

---  
  `--sif-cache SIF_CACHE`
> **Path where a local cache of SIFs are stored.**  
> *type: path*  
>
> Uses a local cache of SIFs on the filesystem. This SIF cache can be shared across users if permissions are set correctly. If a SIF does not exist in the SIF cache, the image will be pulled from Dockerhub and a warning message will be displayed. The `rna-seek cache` subcommand can be used to create a local SIF cache. Please see `rna-seek cache` for more information. This command is extremely useful for avoiding DockerHub pull rate limits. It also remove any potential errors that could occur due to network issues or DockerHub being temporarily unavailable. We recommend running RNA-seek with this option when ever possible.
> 
> ***Example:*** `--singularity-cache /data/$USER/SIFs`
> 

---  
  `--tmp-dir TMP_DIR`   
> **Path on the file system for writing temporary files.**  
> *type: path*  
> *default: `/lscratch/$SLURM_JOBID`*
> 
> This is a path on the file system for writing temporary output files. By default, the temporary directory is set to '/lscratch/$SLURM_JOBID' for backwards compatibility with the NIH's Biowulf cluster; however, if you are running the pipeline on another cluster, this option will need to be specified. Ideally, this path should point to a dedicated location on the filesystem for writing tmp files. On many systems, this location is set to somewhere in /scratch. If you need to inject a variable into this string that should NOT be expanded, please quote this options value in single quotes. Again, if you are running the pipeline on Biowulf, you do NOT need to provide this option. 
> 
> ***Example:*** `--tmp-dir /cluster_scratch/$USER/`

### 2.4 Misc Options

Each of the following arguments are optional and do not need to be provided. 

  `-h, --help`            
> **Display Help.**  
> *type: boolean*
> 
> Shows command's synopsis, help message, and an example command
> 
> ***Example:*** `--help`

## 3. Hybrid Genomes  

If you have two GTF files, e.g. hybrid genomes (host + virus), then you need to create one genomic FASTA file and one GTF file for the hybrid genome prior to running the <code>rna-seek <b>build</b></code> command. 

We recommend creating an artifical chromosome for the non-host sequence. The sequence identifer in the FASTA file must match the sequence identifer in the GTF file (column 1). Generally speaking, since the host annotation is usually downloaded from Ensembl or GENCODE, it will be correctly formatted; however, that may not be the case for the non-host sequence!

Please ensure the non-host annotation contains the following features and/or constraints:   

 * for a given `gene` feature   
     * each `gene` entry has at least one `transcript` feature    
     * and each `transcript` entry has atleast one `exon` feature 

If not, the GTF file may need to be manually curated until these conditions are satisfied. 

Here is an example feature from a hand-curated Biotyn_probe GTF file:

```bash
Biot1   BiotynProbe gene    1   21  0.000000    +   .   gene_id "Biot1"; gene_name "Biot1"; gene_biotype "biotynlated_probe_control";
Biot1   BiotynProbe transcript  1   21  0.000000    +   .   gene_id "Biot1"; gene_name "Biot1"; gene_biotype "biotynlated_probe_control"; transcript_id "Biot1"; transcript_name "Biot1"; transcript_type "biotynlated_probe_control";
Biot1   BiotynProbe exon    1   21  0.000000    +   .   gene_id "Biot1"; gene_biotype "biotynlated_probe_control"; transcript_id "Biot1"; transcript_type "biotynlated_probe_control";
```

In this tab-delimited example above, 

 * ***line 1:*** the `gene` feature has 3 required attributes in column 9: `gene_id` and `gene_name` and `gene_biotype`
 * ***line 2:*** the `transcript` entry for the above `gene` repeats the same attributes with following required fields: `transcript_id ` and `transcript_name`
     * *Please note:* `transcript_type` is *optional*
 * ***line 3:*** the `exon` entry for the above `transcript` has 3 required attributes: `gene_id` and `transcript_id` and `gene_biotype`
     * *Please note:* `transcript_type` is *optional*

For a given gene, the combination of the `gene_id` AND `gene_name` should form a unique string. There should be no instances where two different genes share the same `gene_id` AND `gene_name`. 


## 4. Convert, patch or clean a malformed GTF file 

While building reference genomes from various sources, you may run into unexpected issues with the GTF file that was provided. The GTF file format has [evolved over the years](https://agat.readthedocs.io/en/latest/gxf.html). Each iteration of the format has its own set of features and attributes.  And while there is a basic defintion for the GTF file format, overall there is a general lack of standardization. 

Most of the issues encountered with the build pipeline can be attributed to this lack of standardization. Over the years, several tools have been [developed to convert between formats](https://github.com/NBISweden/GAAS/blob/master/annotation/knowledge/gff_to_gtf.md). [AGAT](https://agat.readthedocs.io/en/latest/index.html) is an awesome set of tools that can convert between formats and fix issues as they are encountered. 

With that being said, we have provided a universal script to fix malformed GTF files. It also has the extra benefit that it can convert between GFF and GTF formats. As so, we recommened running this script if you run into any issues. This script is also recommended over `./resources/gff3togtf.py`, which will be depreciated in the near future.

For more information about the script and its usage, please run:
```bash
# Steps for converting messy a GFF/GTF file
# into a properly formatted GTF file

# Step 0.) Pull the AGAT docker image and
#  create a SIF file.
which singularity \
  || module load singularity 2> /dev/null \
  || echo 'Error: singularity is not installed!'
SINGULARITY_CACHEDIR=${PWD}/.${USER} \
singularity pull -F \
  docker://quay.io/biocontainers/agat:1.4.2--pl5321hdfd78af_0

# Step 1.) Run AGAT todo the heavy lifting and 
#   correct any major issues. AGAT can take a
#   GTF or GFF file as input, but we recommend
#   providing a GTF file to AGAT if you have
#   access to it (especially if it comes from
#   NCBI, i.e RefSeq or Genbank).
singularity exec -B $PWD \
agat_1.4.2--pl5321hdfd78af_0.sif agat_convert_sp_gff2gtf.pl \
    --gtf_version 3 \
    --gff input.gtf \
    -o converted.gtf

# Step 2.) Finally run this script to add any
#    required metadata to the 9th column of
#    the GTF file and to fix any other issues.
#    The cleaned gtf file can be provided to
#    the build sub command of the pipeline.
module load python/3.10
./resources/clean_gtf.py \
    --gene-name-attribute gene_name \
    --input-gtf converted.gtf \
    --output-gtf cleaned.gtf
```

## 5. Example

### 5.1 Biowulf 

On Biowulf getting started with the pipeline is fast and easy! In this example, we build a mouse reference genome. 

```bash 
# Step 0.) Grab an interactive node (do not run on head node)
srun -N 1 -n 1 --time=2:00:00 -p interactive --mem=8gb  --cpus-per-task=4 --pty bash
module purge
module load singularity snakemake

# Step 1.) Dry run the Build pipeline
./rna-seek build --ref-fa GRCm39.primary_assembly.genome.fa \
              --ref-name mm39 \
              --ref-gtf gencode.vM26.annotation.gtf \
              --gtf-ver M26 \
              --output /data/$USER/refs/mm39_M26 \
              --sif-cache /data/OpenOmics/SIFs/ \
              --dry-run

# Step 2.) Build new RNA-seek reference files 
./rna-seek build --ref-fa GRCm39.primary_assembly.genome.fa \
              --ref-name mm39 \
              --ref-gtf gencode.vM26.annotation.gtf \
              --gtf-ver M26 \
              --output /data/$USER/refs/mm39_M26 \
              --sif-cache /data/OpenOmics/SIFs/ 
```

### 5.2 Generic SLURM Cluster

Running the pipeline outside of Biowulf is easy; however, there are a few extra options you must provide. Please note when running the build sub command for the first time, you will also need to provide the `--shared-resources` option. This option will download our kraken2 database and bowtie2 indices for FastQ Screen. The path provided to this option should be provided to the `--shared-resources` option of the [run](./RNA-seq/cache/) sub command. Next, you will also need to provide a path to write temporary output files via the `--tmp-dir` option. We also recommend providing a path to a SIF cache. You can cache software containers locally with the [cache](./RNA-seq/cache/) sub command. 

```bash 
# Step 0.) Grab an interactive node (do not run on head node)
srun -N 1 -n 1 --time=2:00:00 -p interactive --mem=8gb  --cpus-per-task=4 --pty bash
# Add snakemake and singularity to $PATH,
# This step may vary across clusters, you
# can reach out to a sys admin if snakemake
# and singularity are not installed.
module purge
module load singularity snakemake

# Step 1.) Dry run the Build pipeline
./rna-seek build --ref-fa GRCm39.primary_assembly.genome.fa \
              --ref-name mm39 \
              --ref-gtf gencode.vM26.annotation.gtf \
              --gtf-ver M26 \
              --output /data/$USER/refs/mm39_M26 \
              --shared-resources /data/shared/rna-seek \
              --tmp-dir /cluster_scratch/$USER/ \
              --sif-cache /data/$USER/cache \
              --dry-run

# Step 2.) Build new RNA-seek reference files 
./rna-seek build --ref-fa GRCm39.primary_assembly.genome.fa \
              --ref-name mm39 \
              --ref-gtf gencode.vM26.annotation.gtf \
              --gtf-ver M26 \
              --output /data/$USER/refs/mm39_M26 \
              --shared-resources /data/shared/rna-seek \
              --tmp-dir /cluster_scratch/$USER/ \
              --sif-cache /data/$USER/cache 
```
