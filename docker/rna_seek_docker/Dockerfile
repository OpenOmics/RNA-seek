FROM ubuntu:20.04

# Dependencies of rna_seek
# fastqc
# multiqc
# kraken2
# krona
# fastq screen
# preseq
# picard
# rseqc
# cutadapt
# star
# rsem
# arriba
# Snakemake

############### INIT ################
# Create Container filesystem specific 
# working directory and opt directories
# to avoid collisions with the host's
# filesystem, i.e. /opt and /data
RUN mkdir -p /opt2 && mkdir -p /data2

# Set time zone to US east coast 
ENV TZ=America/New_York
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
        && echo $TZ > /etc/timezone

#################### SETUP ########################
# This section installs system packages requirements
RUN apt-get update \
    && apt-get -y upgrade \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y \
        locales \
        python3 \
        python3-pip \
        wget \
        xxd \
        git \
        unzip \
        curl\
        bowtie2 \
        perl \
        make \
        build-essential \
        cmake \
        gcc \
        g++ \
        zlibc \
        zlib1g \
        zlib1g-dev \
    && apt-get clean && apt-get purge \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
    
# Set the locale
RUN localedef -i en_US -f UTF-8 en_US.UTF-8

## install python packages
# install required tools to run rna seek
RUN ln -sf /usr/bin/python3.8 /usr/bin/python \
    && ln -sf /usr/bin/python3.8 /usr/bin/python3
RUN pip3 install --upgrade pip
RUN pip install numpy
    
# install miniconda to install other tools
# Note: pick best version that work with you processor
# (If you have intel chip use Miniconda3-latest-Linux-x86_64.sh ) 
# RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh -O /tmp/miniconda.sh
RUN /bin/bash /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh && \
    echo "export PATH=/opt/conda/bin:$PATH" > /etc/profile.d/conda.sh
ENV PATH /opt/conda/bin:$PATH

# RUN conda --version
#Update conda 
RUN conda update -n base -c defaults conda
# Add bioconda channel
RUN conda config --add channels bioconda
# Check conda version
RUN conda --version

# install fastqc using conda 
RUN conda install -c bioconda -y fastqc
# Check FastQC versions
RUN fastqc --version

#Install Multiqc 
RUN pip3 install multiqc==1.12 
    
# Install Kraken 2
#RUN mkdir -p /kraken2
#RUN wget -qO- https://github.com/DerrickWood/kraken2/archive/refs/tags/v2.1.3.tar.gz | tar xz -C /kraken2 --strip-components=1
#RUN cd /kraken2 && \
#    ./install_kraken2.sh /usr/local/bin/
#ENV PATH=/usr/local/bin:/kraken2/bin:${PATH}
WORKDIR /opt2
RUN wget https://github.com/DerrickWood/kraken2/archive/v2.1.3.tar.gz && \
	tar -xzf v2.1.3.tar.gz && \
	rm v2.1.3.tar.gz && \
	cd kraken2-2.1.3 && \
 	./install_kraken2.sh . && \
	chmod a+rX /opt2/kraken2-2.1.3/*
# Add kraken2 to PATH
ENV PATH="/opt2/kraken2-2.1.3":$PATH

#install krona
# Get latest krona release (v2.8.1): https://github.com/marbl/Krona/releases
# Build Krona Taxonomy Databases
# ktImportTaxonomy relies on NCBI taxonomy: https://github.com/marbl/Krona/wiki/Installing 
RUN wget https://github.com/marbl/Krona/archive/refs/tags/v2.8.1.tar.gz && \
    tar -xzf v2.8.1.tar.gz && \
   rm v2.8.1.tar.gz && \ 
   cd Krona-2.8.1/KronaTools && \
   ./install.pl  && \
   mkdir -p /opt2/Krona-2.8.1/KronaTools/taxonomy && \
   ./updateTaxonomy.sh
# RUN ktImportTaxonomy

#(If above code doesnot work you this one for krona)
#RUN conda install -c bioconda -y krona RUN wget https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz && \
#    mv taxdump.tar.gz /opt/conda/opt/krona/taxonomy/. && \ 
#    ktUpdateTaxonomy.sh --build-only

# Download FastQ Screen:
RUN wget -O fastq_screen.tar.gz https://www.bioinformatics.babraham.ac.uk/projects/fastq_screen/fastq_screen_v0.13.0.tar.gz && \
    tar -xvf fastq_screen.tar.gz && \
    rm fastq_screen.tar.gz && \
    mv fastq_screen_v0.13.0 /usr/local/fastq_screen && \
    rm -rf /var/lib/apt/lists/*
# Add FastQ Screen to PATH
ENV PATH="/usr/local/fastq_screen:${PATH}"

# Download and install Preseq
RUN wget https://github.com/smithlabcode/preseq/releases/download/v3.2.0/preseq-3.2.0.tar.gz && \
    tar -xzf preseq-3.2.0.tar.gz && \
    rm preseq-3.2.0.tar.gz && \
    cd preseq-3.2.0 && \
    ./configure --prefix=/usr/local/preseq && \
    make && \
    make install && \
    cd .. && \
    rm -rf preseq-3.2.0

# Add Preseq to PATH
ENV PATH="/usr/local/preseq/bin:${PATH}"

# Create a directory to store Picard
# Set the working directory
WORKDIR /usr/local/picard

# Download Picard jar file
RUN wget -O picard.jar https://github.com/broadinstitute/picard/releases/download/2.25.6/picard.jar

# Set up an environment variable as a shortcut
ENV PICARD /usr/local/picard/picard.jar
# Clean up
WORKDIR /
# Test installation
# RUN java -jar $PICARD -h 

# Clone the RSeQC repository
RUN git clone --branch fresh https://github.com/MonashBioinformaticsPlatform/RSeQC.git
# Set the working directory
WORKDIR /RSeQC
# Install RSeQC
RUN python setup.py install
# Set the working directory back to root
WORKDIR /
RUN mv /RSeQC /usr/local

#Install cutadapt
RUN pip install cutadapt
#check versions
RUN cutadapt --version

#Install STAR 
WORKDIR /opt2
RUN wget https://github.com/alexdobin/STAR/archive/2.7.0f.tar.gz && \
	tar -xzf 2.7.0f.tar.gz && \
	cd /opt2/STAR-2.7.0f/source && \
	make STAR
RUN rm 2.7.0f.tar.gz
ENV PATH="/opt2/STAR-2.7.0f/source":$PATH

# Install RSEM
WORKDIR /opt2
RUN wget https://github.com/deweylab/RSEM/archive/v1.3.3.tar.gz \
    && tar -xzf v1.3.3.tar.gz \
    && cd RSEM-1.3.3 \
    && make \
    && make install
    
# RUN rsem-prepare-reference --help
# Set environment variables
ENV PATH="/opt2/RSEM-1.3.3:${PATH}"

#RUN rsem-prepare-reference --help

# clean up unnecessary files after installation
RUN rm -rf /opt2/v1.3.3.tar.gz

# Set the working directory
WORKDIR /

# Install Arriba
WORKDIR /opt2
RUN wget https://github.com/suhrig/arriba/releases/download/v2.4.0/arriba_v2.4.0.tar.gz
RUN tar -xzf arriba_v2.4.0.tar.gz
RUN cd arriba_v2.4.0 && make 
RUN rm arriba_v2.4.0.tar.gz
ENV PATH="/opt2/arriba_v2.4.0:${PATH}"

#Install snakemake 8.8.0
# Install Mamba to install snakemake

Run conda install -n base -c conda-forge mamba
# Create a new empty environment called "snakemake"
RUN conda create --name snakemake
# Activate the environment and install Snakemake
RUN echo "source /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate snakemake" >> ~/.bashrc && \
    /bin/bash -c "source ~/.bashrc && mamba install -n snakemake -c conda-forge -c bioconda snakemake"

# Set the default command to activate the snakemake environment
CMD ["/bin/bash", "-c", "source /opt/conda/etc/profile.d/conda.sh && conda activate snakemake && /bin/bash"]

# Add Dockerfile and export env variables
ADD Dockerfile /opt2/Dockerfile
RUN chmod -R a+rX /opt2
ENV PATH="/opt2:$PATH"



    