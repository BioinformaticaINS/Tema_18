# Tema 18: Obtención, calidad y limpieza de lecturas

## Introducción

En este tema, exploraremos el proceso de obtención, evaluación de calidad y limpieza de datos de secuenciación, tanto para tecnologías de secuenciación de **Illumina** como de **Nanopore**. Estas tecnologías son ampliamente utilizadas en genómica y bioinformática, y cada una tiene sus propias características y desafíos en cuanto al manejo de datos. El objetivo es garantizar que los datos de secuenciación sean de alta calidad antes de proceder con análisis posteriores, como el ensamblaje de genomas o la identificación de variantes.

---

## Estructura de la práctica:

1. **Instalación de programas**: Herramientas necesarias para el análisis de datos de secuenciación.
2. **Obtención de los datos de secuenciación**: Descarga de datos crudos de Illumina y Nanopore.
3. **Análisis de calidad de datos de secuenciación Illumina**: Evaluación de la calidad de las lecturas y limpieza de datos.
4. **Basecalling de datos de secuenciación Nanopore**: Conversión de señales eléctricas a secuencias de ADN.
5. **Análisis de calidad de datos de secuenciación Nanopore**: Evaluación de la calidad de las lecturas y filtrado de datos.

---

## Metodología:

### 1. Instalación de programas

#### Instalación de **Guppy**

**Guppy** es una herramienta desarrollada por Oxford Nanopore Technologies (ONT) para realizar el **basecalling**, es decir, convertir las señales eléctricas generadas por los nanoporos en secuencias de ADN. También incluye funcionalidades para el análisis de datos.

```bash
cd
mkdir genomics
cd genomics
mkdir software
cd software
pip install gdown
gdown https://drive.google.com/uc?id=1eGGqEJUsHj5rzc5S4LGDvhj0ov6lsr-3
sudo dpkg -i ont_guppy_cpu_6.5.7-1~focal_amd64.deb 
guppy_basecaller -h
```

> **Nota:** Guppy es una herramienta esencial para el análisis de datos de Nanopore, ya que permite procesar los datos crudos (en formato FAST5 o POD5) y generar secuencias de ADN en formato FASTQ.

---

#### Instalación de **Dorado**

**Dorado** es otra herramienta de ONT para el basecalling y análisis de datos de Nanopore. Es más reciente que Guppy y ofrece mejoras en velocidad y precisión.

```bash
wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-0.9.1-linux-x64.tar.gz
tar xvfz dorado-0.9.1-linux-x64.tar.gz
cd dorado-0.9.1-linux-x64/bin
sudo ln -s /home/ins_user/genomics/software/dorado-0.9.1-linux-x64/bin/dorado /usr/local/bin/dorado
dorado --help
```

> **Nota:** Dorado es especialmente útil para el procesamiento de grandes volúmenes de datos y ofrece opciones avanzadas para la desmultiplexación y el filtrado de lecturas.

---

#### Instalación de **POD5**

**POD5** es un formato de archivo desarrollado por ONT para almacenar datos de secuenciación de nanoporos. Es más eficiente en términos de almacenamiento y velocidad en comparación con el formato FAST5.

```bash
pip install pod5
```

> **Nota:** POD5 es el formato recomendado para el almacenamiento de datos crudos de Nanopore, ya que permite una compresión más eficiente y un acceso más rápido a los datos.

---

#### Instalación de herramientas en un ambiente de Conda

Para facilitar la instalación y gestión de las herramientas de análisis de calidad, utilizaremos un entorno de Conda. Este entorno incluye herramientas como **FastQC**, **NanoFilt**, **MultiQC**, entre otras.

```bash
conda create -n quality -c bioconda fastqc nanofilt nanoplot multiqc porechop trim-galore trimmomatic seqkit samtools bedtools
```

> **Nota:** 
> - **FastQC**: Herramienta para el control de calidad de datos de secuenciación.
> - **NanoFilt**: Filtra lecturas de Nanopore basándose en la calidad y longitud.
> - **MultiQC**: Agrega resultados de múltiples herramientas de calidad en un único informe.
> - **Porechop**: Recorta adaptadores en lecturas de Nanopore.
> - **Trim Galore!**: Recorta adaptadores y realiza control de calidad en datos de Illumina.
> - **Trimmomatic**: Otra herramienta para el recorte de adaptadores en datos de Illumina.
> - **Seqkit**: Herramienta para manipular archivos FASTA/Q.
> - **Samtools**: Suite para manipular archivos SAM/BAM.
> - **Bedtools**: Suite para operaciones con archivos BED.

---

### 2. Obtención de los datos de secuenciación

Los datos de secuenciación pueden provenir de diferentes tecnologías, como Illumina o Nanopore. En este caso, descargaremos datos de ambas tecnologías.

```bash
cd ~/genomics
mkdir raw_data
cd raw_data

### Datos de Illumina
gdown https://drive.google.com/uc?id=1hpoWBRzA2OzWbtxtvwqM-3qnauhM1ulk
gdown https://drive.google.com/uc?id=1GUHeRwpfelPAmU5b6dPoiT79iQ-QQYMP

### Datos de Nanopore
gdown https://drive.google.com/uc?id=1FyGJAS33tB-DkbAiL2195ACaWf_qIi3Z
gdown https://drive.google.com/uc?id=1FP6zi0jJmPf6n1ic0Fd3Cdytcf8gY3bk

unzip pod5.zip
unzip fast5.zip
```

> **Nota:** Los datos de Illumina suelen estar en formato FASTQ, mientras que los datos de Nanopore pueden estar en formato FAST5 o POD5. Estos últimos contienen las señales eléctricas crudas que deben ser convertidas a secuencias de ADN mediante el proceso de basecalling.

---

### 3. Análisis de calidad de datos de secuenciación Illumina

El análisis de calidad es un paso crucial para garantizar que los datos de secuenciación sean adecuados para análisis posteriores. Utilizaremos **FastQC** para evaluar la calidad de las lecturas y **MultiQC** para generar un informe consolidado.

```bash
cd ~/genomics
mkdir quality
cd quality
mkdir illumina
cd illumina
conda activate quality

fastqc -t 2 /home/ins_user/genomics/raw_data/*.fastq.gz -o .
multiqc -o raw_illumina .
```

> **Nota:** 
> - **FastQC** genera informes visuales que muestran métricas como la calidad por base, la distribución de longitudes de las lecturas y la presencia de adaptadores.
> - **MultiQC** agrega los resultados de múltiples ejecuciones de FastQC en un único informe, lo que facilita la comparación entre muestras.

---

#### Limpieza con **Trim Galore!**

**Trim Galore!** es una herramienta que combina **Cutadapt** y **FastQC** para recortar adaptadores y realizar control de calidad.

```bash
cd ~/genomics
mkdir trimming
cd trimming
mkdir illumina
cd illumina
mkdir trim_galore
cd trim_galore

trim_galore --quality 30 --length 50 --phred33 --cores 2 --fastqc --paired /home/ins_user/genomics/raw_data/T4_S1_L001_R1_001.fastq.gz /home/ins_user/genomics/raw_data/T4_S1_L001_R2_001.fastq.gz
multiqc -o trimming_trim_galore .
```

> **Nota:** 
> - **--quality 30**: Recorta bases con calidad inferior a 30.
> - **--length 50**: Descarta lecturas con menos de 50 bases.
> - **--phred33**: Especifica la escala de calidad Phred33.
> - **--paired**: Indica que los datos son pareados (paired-end).

---

#### Limpieza con **Trimmomatic**

**Trimmomatic** es otra herramienta popular para el recorte de adaptadores y el control de calidad.

```bash
cd ~/genomics/trimming/illumina
mkdir trimmomatic
cd trimmomatic

nano NexteraPE.fa
# Añadir secuencias de adaptadores

trimmomatic PE /home/ins_user/genomics/raw_data/T4_S1_L001_R1_001.fastq.gz /home/ins_user/genomics/raw_data/T4_S1_L001_R2_001.fastq.gz T4_R1.trim.fastq.gz T4_R1.unpaired.fastq.gz T4_R2.trim.fastq.gz T4_R2.unpaired.fastq.gz ILLUMINACLIP:NexteraPE.fa:2:30:10 SLIDINGWINDOW:4:30 MINLEN:50 -threads 2

fastqc -t 2 *.trim.fastq.gz -o .
multiqc -o trimming_trimmomatic .
```

> **Nota:** 
> - **ILLUMINACLIP**: Recorta adaptadores específicos.
> - **SLIDINGWINDOW**: Recorta lecturas utilizando una ventana deslizante.
> - **MINLEN**: Descarta lecturas con menos de 50 bases.

---

### 4. Basecalling de datos de secuenciación Nanopore

El **basecalling** es el proceso de convertir las señales eléctricas generadas por los nanoporos en secuencias de ADN. Utilizaremos **Guppy** y **Dorado** para este propósito.

#### Basecalling con **Guppy**

```bash
cd ~/genomics
mkdir basecalling
cd basecalling

guppy_basecaller -i /home/ins_user/genomics/raw_data/fast5 -s fast5 -c dna_r10.4_e8.1_fast.cfg
```

> **Nota:** 
> - **-i**: Directorio de entrada con archivos FAST5.
> - **-s**: Directorio de salida para los archivos procesados.
> - **-c**: Archivo de configuración para el modelo de basecalling.

---

#### Basecalling con **Dorado**

**Dorado** es una alternativa más reciente y eficiente para el basecalling.

```bash
cd ~/genomics/basecalling
mkdir pod5
cd pod5

pod5 convert fast5 /home/ins_user/genomics/raw_data/fast5/*.fast5 --output converted.pod5
dorado basecaller fast --kit-name SQK-NBD114-24 --min-qscore 8 --barcode-both-ends converted.pod5 > calls.bam
dorado demux --kit-name SQK-NBD114-24 --output-dir barcodes --emit-summary calls.bam
```

> **Nota:** 
> - **pod5 convert**: Convierte archivos FAST5 a POD5.
> - **dorado basecaller**: Realiza el basecalling y desmultiplexación.
> - **dorado demux**: Separa las lecturas por código de barras.

---

### 5. Análisis de calidad de datos de secuenciación Nanopore

El análisis de calidad para datos de Nanopore es similar al de Illumina, pero con herramientas específicas como **NanoPlot** y **NanoFilt**.

#### Visualización de la calidad con **NanoPlot**

```bash
cd ~/genomics/quality
mkdir nanopore
cd nanopore

NanoPlot -t 2 --fastq /home/ins_user/genomics/raw_data/nanopore/b01_fast.fastq -p b01_fast_raw_ -o b01_fast_raw --maxlength 100000
```

> **Nota:** 
> - **NanoPlot** genera gráficos que muestran la distribución de longitudes, la calidad de las lecturas y otros parámetros importantes.

---

#### Eliminación de adaptadores con **Porechop**

```bash
cd ~/genomics/trimming
mkdir nanopore
cd nanopore
mkdir porechop
cd porechop

porechop -t 2 -i /home/ins_user/genomics/raw_data/nanopore/b01_fast.fastq -o b01_fast_porechop.fastq.gz
```

> **Nota:** 
> - **Porechop** recorta adaptadores y separa lecturas concatenadas.

---

#### Filtrado de lecturas con **NanoFilt**

```bash
cd ~/genomics/trimming/nanopore
mkdir nanofilt
cd nanofilt

gunzip -c /home/ins_user/genomics/trimming/nanopore/porechop/b01_fast_porechop.fastq.gz | NanoFilt -q 9 --length 1000 | gzip > b01_fast_nanofilt.fastq.gz
```

> **Nota:** 
> - **NanoFilt** filtra lecturas basándose en la calidad y longitud.

---

#### Visualización de la calidad después del filtrado

```bash
cd ~/genomics/quality/nanopore

NanoPlot -t 2 --fastq /home/ins_user/genomics/trimming/nanopore/nanofilt/b01_fast_nanofilt.fastq.gz -p b00_fast_filt_ -o b01_fast_filt --maxlength 100000
```

> **Nota:** Este paso permite verificar que el filtrado ha mejorado la calidad de las lecturas.

---

## Conclusión

El proceso de obtención, evaluación de calidad y limpieza de datos de secuenciación es fundamental para garantizar que los análisis posteriores sean precisos y confiables. Tanto para datos de Illumina como de Nanopore, es crucial utilizar herramientas específicas que permitan identificar y corregir problemas comunes, como la presencia de adaptadores o lecturas de baja calidad. Con un flujo de trabajo bien estructurado, es posible obtener datos de alta calidad que sean adecuados para análisis genómicos avanzados.
