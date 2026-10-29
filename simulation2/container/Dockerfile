# Container image with all tools of the read simulator (simulation2/).
# Build: docker build -t tes-aav-simulator simulation2/container
# Tool and Python package versions are pinned to the image that produced the
# published simulated datasets, so that seeded runs reproduce them.
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

RUN apt-get update && apt-get install -y --no-install-recommends \
        autoconf automake build-essential ca-certificates curl git libbz2-dev \
        libcurl4-openssl-dev liblzma-dev libncurses5-dev libssl-dev libtool \
        pkg-config python3.12 python3.12-dev python3.12-venv wget zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# Python packages (versions in constraints.txt):
COPY constraints.txt /tmp/constraints.txt
RUN python3.12 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install -c /tmp/constraints.txt setuptools wheel \
    && pip install -c /tmp/constraints.txt \
        biopython matplotlib numpy pandas pytest pyyaml scipy

# Badread (long reads), master at f38ef6f (reports v0.4.1):
RUN git clone https://github.com/rrwick/Badread.git /tmp/Badread \
    && git -C /tmp/Badread checkout f38ef6f \
    && pip install -c /tmp/constraints.txt /tmp/Badread \
    && rm -rf /tmp/Badread

# minimap2 2.30
RUN curl -fsSL https://github.com/lh3/minimap2/releases/download/v2.30/minimap2-2.30_x64-linux.tar.bz2 \
        | tar -xj -C /tmp \
    && mv /tmp/minimap2-2.30_x64-linux/minimap2 /usr/local/bin/ \
    && rm -rf /tmp/minimap2-2.30_x64-linux

# bedtools 2.31.1
RUN cd /tmp && wget -q https://github.com/arq5x/bedtools2/releases/download/v2.31.1/bedtools-2.31.1.tar.gz \
    && tar -xzf bedtools-2.31.1.tar.gz && cd bedtools2 && make -j"$(nproc)" && make install \
    && cd /tmp && rm -rf bedtools2 bedtools-2.31.1.tar.gz

# samtools 1.19.2 (installs wgsim too) and bcftools 1.19 (genomes/prepare_genomes.sh)
RUN cd /tmp && wget -q https://github.com/samtools/samtools/releases/download/1.19.2/samtools-1.19.2.tar.bz2 \
    && tar -xjf samtools-1.19.2.tar.bz2 && cd samtools-1.19.2 \
    && ./configure --prefix=/usr/local && make -j"$(nproc)" && make install \
    && cd /tmp && rm -rf samtools-1.19.2 samtools-1.19.2.tar.bz2
RUN cd /tmp && wget -q https://github.com/samtools/bcftools/releases/download/1.19/bcftools-1.19.tar.bz2 \
    && tar -xjf bcftools-1.19.tar.bz2 && cd bcftools-1.19 \
    && ./configure --prefix=/usr/local && make -j"$(nproc)" && make install \
    && cd /tmp && rm -rf bcftools-1.19 bcftools-1.19.tar.bz2

# seqtk 1.5 (genomes/prepare_genomes.sh)
RUN git clone --branch v1.5 --depth 1 https://github.com/lh3/seqtk.git /tmp/seqtk \
    && make -C /tmp/seqtk -j"$(nproc)" && cp /tmp/seqtk/seqtk /usr/local/bin/ \
    && rm -rf /tmp/seqtk

# vsearch 2.30.4
RUN cd /tmp && wget -q https://github.com/torognes/vsearch/archive/v2.30.4.tar.gz \
    && tar -xzf v2.30.4.tar.gz && cd vsearch-2.30.4 && ./autogen.sh \
    && ./configure CFLAGS="-O2" CXXFLAGS="-O2" --prefix=/usr/local \
    && make ARFLAGS="cr" -j"$(nproc)" && make install \
    && cd /tmp && rm -rf vsearch-2.30.4 v2.30.4.tar.gz

# ART (art_illumina, paired-end short reads), version MountRainier 2016-06-05
RUN cd /tmp && wget -q https://www.niehs.nih.gov/research/resources/assets/docs/artbinmountrainier2016.06.05linux64.tgz \
    && tar -xzf artbinmountrainier2016.06.05linux64.tgz \
    && cp art_bin_MountRainier/art_illumina /usr/local/bin/ \
    && rm -rf art_bin_MountRainier artbinmountrainier2016.06.05linux64.tgz

RUN badread --version && minimap2 --version && bedtools --version && samtools --version | head -n 1 \
    && bcftools --version | head -n 1 && (seqtk 2>&1 | grep Version) && vsearch --version \
    && (wgsim 2>&1 | grep Version) && (art_illumina 2>&1 | grep -m 1 Version) \
    && python -c "import Bio, matplotlib, numpy, pandas, scipy, yaml"

LABEL org.opencontainers.image.description="Tools for the TES AAV read simulator (simulation2/)"

WORKDIR /data
CMD ["/bin/bash"]
