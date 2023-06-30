## Alunas: Laura Ribeiro 202100819 e Raquel Pereira 202100818
# CodigosSinonovacula

Códigos utilizados para a replicação do artigo 
Genetic diversity and population structure of razor clam Sinonovacula constricta in Ariake Bay, Japan

#SHELL

#Ferramenta de controle de qualidade para dados de sequência de alto rendimento

conda create -n hts
conda activate hts
conda install -c bioconda fastqc
conda install -c "bioconda/label/broken" fastqc
conda install -c "bioconda/label/cf201901" fastqc

#Obter Dados

conda activate hts
conda install -c bioconda bowtie2 
conda install -c bioconda samtools  
conda install -c bioconda tablet
conda install -c bioconda bcftools
sudo apt install sra-toolkit
prefetch
prefetch DRR250449
conda create -n sra
conda activate sra
mkdir sra_seqs
cd sra_seqs
fasterq-dump DRP007238


#Análises de Bibliotecas de Representação Reduzida

# Add conda-forge channel
conda config --add channels conda-forge
# Make sure conda is up to date
conda update conda
# Try mamba (optional step)
conda install mamba
# Create and activate a new conda environment
conda create -n ipyrad
conda activate ipyrad
# Install 
conda install ipyrad nympy=1.19 -c bioconda  
mamba install ipyrad numpy=1.19 -c bioconda
#Buscar os dados 

mkdir ipyrad-assembly
cd ipyrad-assembly
sudo apt install git

#Criar um novo arquivo de parâmetros
pwd
ipyrad -n data
cat params-data.txt
nano params-data.txt
ipyrad -p params-data.txt -s 1 -c 3
ipyrad -p params-data.txt -r
ipyrad -p params-data.txt -s 2 -c 3
ipyrad -p params-data.txt -s 3 -c 3
ipyrad -p params-data.txt -s 4 -c 3
ipyrad -p params-data.txt -s 5 -c 3
ipyrad -p params-data.txt -s 6 -c 3
ipyrad -p params-data.txt -s 7 -c 3
ls data_outfiles/
ipyrad -p params-data.txt -s 7 -c 3 -f

#RaxMl Árvore filogenética
raxmlHPC-PTHREADS-SSE3 -f a -p 12345 -m GTRGAMMA -N 2 -s data.phy -n ml -T 2 -x 12345
raxmlHPC-PTHREADS-SSE3 -b 12345 -p 12345 -m GTRGAMMA -N 20 -s data.phy -n bootstrap -T 2
raxmlHPC-PTHREADS-SSE3 -f a -m GTRCAT -p 12345 -x 12345 -N 70 -s data.phy -n catfish -T 4

#Python PCA

conda install ipyrad-c conda -forge -c bioconda
conda create -n base

import ipyrad
import ipyrad.analysis as ipa 
vcffile = "/home/laura/Downloads/data.vcf"
imap = {
     "Kashima_c":["DRR250420","DRR250421","DRR250422","DRR250423","DRR250424"],
     "Nanaura_a":["DRR250425","DRR250426", "DRR250444","DRR250445"],
     "2014":["DRR250427","DRR250428","DRR250429","DRR250430","DRR250431"],
     "2016":["DRR250432","DRR250433", "DRR250434","DRR250435","DRR250436","DRR250437","DRR250438"],
     "Kashima_b":["DRR250439","DRR250440","DRR250441","DRR250442"],
     "Kashima_a":["DRR250443"],
     "Takori_a":["DRR250446","DRR250447","DRR250448","DRR250449"]
}

pca = ipa.pca(data=vcffile, imap=imap)
pca.run()
pca.draw()

imap = {
     "> 60":["DRR250420","DRR250421","DRR250422","DRR250439","DRR250440","DRR250441","DRR250442","DRR250443","DRR250444","DRR250445","DRR250447"],
     "60~75":["DRR250423","DRR250424","DRR250446"],
     "75~90":["DRR250433","DRR250434","DRR250435","DRR250436","DRR250437","DRR250448"],
     "< 90":["DRR250427","DRR250428","DRR250429","DRR250430","DRR250431","DRR250432","DRR250438","DRR250449"]
}
pca = ipa.pca(data=vcffile, imap=imap)
pca.run()
pca.draw()


#Análise de mistura

wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -f
~/miniconda3/bin/conda init bash
exit

conda config --add channels conda-forge
conda create -n structure python=3.9 r r-rcolorbrewer bioconductor-snprelate bioconductor-lfa r-devtools -c bioconda
conda activate structure  # Activate the new environement
python3 -m pip install structure_threader 

wget https://raw.githubusercontent.com/CoBiG2/RAD_Tools/6648d1ce1bc1e4c2d2e4256abdefdf53dc079b8c/vcf_parser.py
python3 vcf_parser.py --center-snp -vcf data.vcf

structure_threader run -i dataCenterSNP.vcf -o ./results_dataCenterSNP -als ~/miniconda3/envs/structure/bin/alstructure_wrapper.R -K 10 -t 3 --ind data.indfile

