# Tema 18: Obtención, calidad y limpieza de lecturas

## Estructura de la práctica:

1. Instalación de programas
2. Obtención de los datos de secuenciación 
3. Análisis de calidad de datos de secuenciación Illumina
4. Basecalling de datos de secuenciación Nanopore
5. Análisis de calidad de datos de secuenciación Nanopore

## Metodología:

## 1. Instalación de programas

### Instalación de guppy

Guppy es un programa de basecalling y análisis de datos de secuenciación de nanoporos desarrollado por Oxford Nanopore Technologies.

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

### Instalacion de dorado

Dorado es un programa de basecalling y análisis de datos de secuenciación de nanoporos desarrollado por Oxford Nanopore Technologies.

```bash
wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-0.9.1-linux-x64.tar.gz

tar xvfz dorado-0.9.1-linux-x64.tar.gz

cd dorado-0.9.1-linux-x64

cd bin

sudo ln -s /home/ins_user/genomics/software/dorado-0.9.1-linux-x64/bin/dorado /usr/local/bin/dorado

dorado --help
```

### Instalación de POD5

Permite convertir fast5 a pod5 antes del basecalling

```bash
pip install pod5 
```

> **Comentario:** POD5 es un formato de archivo desarrollado por Oxford Nanopore Technologies para almacenar datos de secuenciación de nanoporos. Es un formato más eficiente y comprimido que FAST5

### Instalar los siguientes programas en un ambiente de conda

```bash
conda create -n quality -c bioconda fastqc nanofilt nanoplot multiqc porechop trim-galore trimmomatic seqkit samtools bedtools
```

> **Comentario:** 
> - `conda create`: Este es el comando base para crear un nuevo entorno virtual con conda.
> - `-n quality`: Esta opción especifica el nombre del nuevo entorno virtual, que será "quality". Los entornos virtuales permiten tener diferentes versiones de software instaladas sin que interfieran entre sí.
> - `-c bioconda`: Esta opción le dice a conda que busque los paquetes a instalar en el canal "bioconda". Bioconda es un canal de conda especializado en software bioinformático, lo que garantiza que las herramientas estén optimizadas para análisis de datos biológicos.
> - `fastqc`: Instala FastQC, una herramienta para el control de calidad de datos de secuenciación de alto rendimiento (NGS).
> - `nanofilt`: Instala NanoFilt, una herramienta para filtrar datos de secuenciación de Oxford Nanopore Technologies (ONT) basándose en la longitud y la calidad de las lecturas.
> - `nanoplot`: Instala NanoPlot, una herramienta para visualizar datos de secuenciación de ONT.
> - `multiqc`: Instala MultiQC, una herramienta que agrega los resultados de múltiples herramientas de control de calidad en un único informe HTML.
> - `porechop`: Instala Porechop, una herramienta para recortar adaptadores y detectar lecturas de concatenación en datos de secuenciación de ONT.
> - `trim-galore`: Instala Trim Galore!, un envoltorio (wrapper) que combina Cutadapt y FastQC para recortar adaptadores y realizar control de calidad en datos de secuenciación de alto rendimiento (NGS).
> - `trimmomatic`: Instala Trimmomatic, otra herramienta popular para recortar adaptadores y realizar control de calidad en datos de secuenciación de alto rendimiento (NGS).
> - `seqkit`: Instala seqkit, una caja de herramientas rápida y multiplataforma para manipular archivos de secuencia FASTA/Q.
> - `samtools`: Instala samtools, una suite de herramientas para manipular archivos SAM/BAM (formatos para almacenar datos de alineamiento de secuencias).
> - `bedtools`: Instala bedtools, una suite de utilidades para realizar operaciones con archivos BED (formatos para definir regiones genómicas).

## 2. Obtención de los datos de secuenciación 

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

## 3. Analisis de calidad de datos de secuenciación Illumina

```bash
cd ~/genomics

mkdir quality

cd quality

mkdir illumina

cd illumina

conda activate quality

fastqc -t 2 /home/ins_user/genomics/raw_data/*.fastq.gz -o .
```

> **Comentario:** 
> - `-t 2`: Esta opción especifica el número de hilos (threads) que FastQC debe utilizar. Al usar múltiples hilos, el programa puede procesar los datos más rápidamente. En este caso, se están usando 2 hilos.
> - `/data/2024_2/genome/illumina/*.fastq.gz`: Esta parte del comando indica la ubicación de los archivos que FastQC debe analizar.
> - `*.fastq.gz`: Es un comodín que selecciona todos los archivos en ese directorio que terminan con ".fastq.gz". Esto significa que FastQC analizará todos los archivos FASTQ comprimidos en ese directorio. Los archivos fastq.gz son el formato de archivos donde se guardan las lecturas de las secuencias de ADN.
> - `-o .`: Esta opción define el directorio de salida. El punto "." representa el directorio actual. Esto significa que los informes HTML generados por FastQC se guardarán en el mismo directorio donde se ejecuta el comando.

```bash
multiqc -o raw_illumina .
```
> **Comentario:**
> - `-o raw_illumina`: Esta opción especifica el directorio de salida donde se guardará el informe HTML generado por MultiQC.
> - `.`: Representa el directorio actual. Esto le dice a MultiQC que busque archivos de resultados (los que ha generado FastQC por ejemplo) dentro del directorio en el que estás ejecutando el comando. MultiQC buscará automáticamente archivos de salida de las herramientas de control de calidad compatibles que se encuentren en el directorio actual y sus subdirectorios.

### Limpieza con trim galore

```bash
cd ~/genomics

mkdir trimming

cd trimming

mkdir illumina

cd illumina

mkdir trim_galore

cd trim_galore

trim_galore --quality 30 --length 50 --phred33 --cores 2 --fastqc --paired /home/ins_user/genomics/raw_data/T4_S1_L001_R1_001.fastq.gz /home/ins_user/genomics/raw_data/T4_S1_L001_R2_001.fastq.gz
```

> **Comentario:**
> - `--quality 30`: Esta opción especifica el umbral de calidad para el recorte de bases. Las bases con una calidad inferior a 30 (en escala Phred33) serán recortadas del final de las lecturas.
> - `--length 50`: Esta opción establece la longitud mínima de las lecturas después del recorte. Las lecturas que sean más cortas que 50 bases serán descartadas.
> - `--phred33`: Esta opción indica que los datos de calidad de las bases están en la escala Phred33, que es la escala más común utilizada en la secuenciación Illumina.
> - `--cores 2`: Esta opción especifica el número de núcleos de procesamiento que Trim Galore! debe utilizar. En este caso, se están utilizando 2 núcleos para acelerar el procesamiento.
> - `--fastqc`: Esta opción le indica a Trim Galore! que ejecute FastQC automáticamente después del recorte para generar informes de control de calidad de los datos recortados.
> - `-paired`: Esta opción indica que los datos son pareados (paired-end), lo que significa que las lecturas vienen en pares (R1 y R2).
> - `/home/ins_user/genomics/raw_data/T4_S1_L001_R1_001.fastq.gz`: Esta es la ruta del archivo FASTQ comprimido que contiene las lecturas R1 (la primera lectura del par).
> - `/home/ins_user/genomics/raw_data/T4_S1_L001_R2_001.fastq.gz`: Esta es la ruta del archivo FASTQ comprimido que contiene las lecturas R2 (la segunda lectura del par).

```bash
multiqc -o trimming_trim_galore .
```

### Limpieza con trimmomatic

```bash
cd ~/genomics/trimming/illumina

mkdir trimmomatic

cd trimmomatic
```

> **Comentario:** Crear el archivo NexteraPE.fa

```bash
nano NexteraPE.fa

>PrefixNX/1
AGATGTGTATAAGAGACAG
>PrefixNX/2
AGATGTGTATAAGAGACAG
>Trans1
TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG
>Trans1_rc
CTGTCTCTTATACACATCTGACGCTGCCGACGA
>Trans2
GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG
>Trans2_rc
CTGTCTCTTATACACATCTCCGAGCCCACGAGAC
```

```bash
trimmomatic PE /home/ins_user/genomics/raw_data/T4_S1_L001_R1_001.fastq.gz /home/ins_user/genomics/raw_data/T4_S1_L001_R2_001.fastq.gz T4_R1.trim.fastq.gz T4_R1.unpaired.fastq.gz T4_R2.trim.fastq.gz T4_R2.unpaired.fastq.gz ILLUMINACLIP:NexteraPE.fa:2:30:10 SLIDINGWINDOW:4:30 MINLEN:50 -threads 2
```

> **Comentario:**
> - `PE`: Indica que los datos son pareados (paired-end).
> - `/home/ins_user/genomics/raw_data/T4_S1_L001_R1_001.fastq.gz`: Esta es la ruta del archivo FASTQ comprimido que contiene las lecturas R1 (la primera lectura del par).
> - `/home/ins_user/genomics/raw_data/T4_S1_L001_R2_001.fastq.gz`: Esta es la ruta del archivo FASTQ comprimido que contiene las lecturas R2 (la segunda lectura del par).
> - `T4_R1.trim.fastq.gz`: Archivo de salida para las lecturas R1 recortadas y emparejadas.
> - `T4_R1.unpaired.fastq.gz`: Archivo de salida para las lecturas R1 que quedaron sin par después del recorte.
> - `T4_R2.trim.fastq.gz`: Archivo de salida para las lecturas R2 recortadas y emparejadas.
> - `T4_R2.unpaired.fastq.gz`: Archivo de salida para las lecturas R2 que quedaron sin par después del recorte.
> - `ILLUMINACLIP:NexteraPE.fa:2:30:10`: Son los parámetros para el recorte de adaptadores (mismatch allowance:palindrom match threshold:simple match threshold).
> - `SLIDINGWINDOW:4:30`: Indica que se va a utilizar una ventana deslizante de 4 bases y que la calidad promedio mínima dentro de la ventana debe ser 30.
> - `MINLEN:50`: Esta opción establece la longitud mínima de las lecturas después del recorte. Las lecturas que sean más cortas que 50 bases serán descartadas.

```bash
fastqc -t 2 *.trim.fastq.gz -o .

multiqc -o trimming_trimmomatic .
```

## 4. Basecalling de datos de secuenciación Nanopore

### Basecalling de FAST5

```bash
cd ~/genomics

mkdir basecalling

cd basecalling

guppy_basecaller -i /home/ins_user/genomics/raw_data/fast5 -s fast5 -c dna_r10.4_e8.1_fast.cfg
```

> **Comentario:** 
> - `-i /home/ins_user/genomics/raw_data/fast5`: Esta opción especifica el directorio de entrada donde se encuentran los archivos de datos de secuenciación crudos (en formato FAST5). Guppy leerá todos los archivos FAST5 dentro de este directorio.
> - `-s fast5`: Esta opción especifica el directorio de salida donde se escribirán los datos convertidos. La bandera -s significa "ruta de guardado" (save path). Guppy creará un nuevo directorio llamado "fast5" dentro del directorio de trabajo actual y almacenará los archivos de salida allí.
> - `-c dna_r10.4_e8.1_fast.cfg`: Esta opción especifica el archivo de configuración que se utilizará para el "basecalling". El archivo dna_r10.4_e8.1_fast.cfg contiene ajustes optimizados para la celda de flujo R10.4 y el modelo de "basecalling" "fast" (rápido). Este modelo prioriza la velocidad sobre la máxima precisión.

### Basecalling de POD5

#### Obtención de archivos POD5
```bash
cd ~/genomics/basecalling

mkdir pod5

cd pod5

conda deactivate

pod5 convert fast5 /home/ins_user/genomics/raw_data/fast5/*.fast5 --output converted.pod5
```

> **Comentario:** Esta línea de código utiliza `pod5` para convertir todos los archivos FAST5 en el directorio `fast5` a un único archivo POD5 llamado `converted.pod5`

#### Basecalling y desmultiplexación

```bash
dorado basecaller fast --kit-name SQK-NBD114-24 --min-qscore 8 --barcode-both-ends converted.pod5 > calls.bam
```

> **Comentario:** 
> - `basecaller fast`: Esto invoca el programa Dorado en modo "basecaller" (llamador de bases) y utiliza el modo "fast" (rápido). El modo rápido prioriza la velocidad sobre la precisión.
> - `--kit-name SQK-NBD114-24`: Especifica el nombre del kit de secuenciación utilizado.
> - `--min-qscore 8`: Establece un umbral de calidad mínimo de 8 para las lecturas.
> - `--barcode-both-ends`: Indica que los códigos de barras están presentes en ambos extremos de las lecturas.
> - `> calls.bam`: Redirige la salida a un archivo BAM llamado `calls.bam`.

```bash
dorado demux --kit-name SQK-NBD114-24 --output-dir barcodes --emit-summary calls.bam
```

> **Comentario:** 
> - `demux`: Invoca la función de desmultiplexación de la herramienta Dorado.
> - `--kit-name SQK-NBD114-24`: Indica el kit de secuenciación que se usó. Dorado necesita esta información para saber qué códigos de barras buscar y cómo interpretarlos.
> - `--output-dir barcodes`: Especifica que los archivos de salida se guarden en un directorio llamado "barcodes". Dentro de este directorio, se creará un archivo BAM para cada código de barras, conteniendo las lecturas correspondientes.
> - `--emit-summary`: Indica que se genere un archivo de resumen que contenga información sobre la clasificación de las lecturas por código de barras.
> - `calls.bam`: Es el archivo de entrada, que en este caso es el archivo BAM generado por el comando dorado basecaller que vimos anteriormente.

```bash
cd barcodes
```

> **Comentario:** 
> - `--kit-name SQK-16S024`: Especifica el nombre del kit de secuenciación utilizado.
> - `--min-qscore 8`: Establece un umbral de calidad mínimo de 8 para las lecturas.
> - `--barcode-both-ends`: Indica que los códigos de barras están presentes en ambos extremos de las lecturas.
> - `> calls.bam`: Redirige la salida a un archivo BAM llamado `calls.bam`.
> - `--output-dir barcodes`: Especifica el directorio de salida para los archivos demultiplexados.
> - `--emit-summary`: Genera un resumen de la ejecución de secuenciación.

```
cat barcoding_summary.txt | head

filename	read_id	barcode
converted.pod5	79575555-aded-46c6-9f18-e9556fbd576d	SQK-NBD114-24_barcode15
converted.pod5	00870608-57ee-497a-a738-b31d4876c161	unclassified
converted.pod5	0054a804-cf1f-4218-9877-f490288b5675	unclassified
converted.pod5	0046d92e-d735-4f25-9fb3-6b80ef88a3fd	unclassified
converted.pod5	032006e4-6d5a-4e8e-a1f7-e6f695d732ad	unclassified
converted.pod5	03d484a9-6644-4856-bf4e-ddf071e2a52e	unclassified
converted.pod5	013562b4-3582-4bf5-852d-11297ed2b35f	unclassified
converted.pod5	037de90d-c40a-4ae4-8b7b-887350c6c219	unclassified
converted.pod5	043aa25b-75b0-4363-ab12-3c3fe75dcf89	unclassified
```

#### Resumen de la secuenciación 

```bash
for file in *.bam; do prefix="${file%.bam}"; dorado summary "$file" > "${prefix}_summary.tsv"; done
```

> **Comentario:** Genera un resumen para cada archivo BAM en el directorio y lo guarda en un archivo TSV.

```
cat bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode13_summary.tsv | head

filename	read_id	run_id	channel	mux	start_time	duration	template_start	template_duration	sequence_length_template	mean_qscore_template	barcode
converted.pod5	05886056-e7c4-4c7f-9f61-a1f025b940a4	bcf4b7732185c1a3353d1b4fe80266cd3ac60162	31	1	9069.31	1.7564	9069.31	1.7564	678	10.5307	SQK-NBD114-24_barcode13
converted.pod5	a87c7559-6f22-463c-b288-a281fa7b0b41	bcf4b7732185c1a3353d1b4fe80266cd3ac60162	66	1	9083.14	0.9576	9083.14	0.9576	219	11.1093	SQK-NBD114-24_barcode13
converted.pod5	44432ea9-99d6-4f46-aba7-82a288b6c9ff	bcf4b7732185c1a3353d1b4fe80266cd3ac60162	132	3	8941.51	1.6908	8941.51	1.6908	621	9.64351	SQK-NBD114-24_barcode13
converted.pod5	af970a9d-239a-4623-80a3-7e26485f66ae	bcf4b7732185c1a3353d1b4fe80266cd3ac60162	328	3	9135.62	3.1966	9135.62	3.1966	1267	12.403	SQK-NBD114-24_barcode13
converted.pod5	9a105f99-ef53-43bb-9e26-321139cd8bad	bcf4b7732185c1a3353d1b4fe80266cd3ac60162	434	4	9016.32	1.05	9016.32	1.05	219	8.39137	SQK-NBD114-24_barcode13
```

#### Conversión de bam a fastq

```bash
conda activate quality

for file in *.bam; do prefix="${file%.bam}"; samtools sort -n "$file" -o "${prefix}_sorted.bam"; done

for file in *_sorted.bam; do prefix="${file%.bam}"; bedtools bamtofastq -i "${prefix}.bam" -fq "${prefix}.fastq"; done

seqkit stats -a -j 4 *_sorted.fastq > stats_sorted_fastq.txt
```

> **Comentario:** 
> - `samtools sort -n`: Ordena los archivos BAM por nombre de lectura.
> - `bedtools bamtofastq`: Convierte los archivos BAM ordenados a formato FASTQ.
> - `seqkit stats`: Calcula las estadísticas de archivos FASTQ que han sido previamente ordenados.

```bash
cat barcoding_summary.txt | head

file                                                                           format  type  num_seqs    sum_len  min_len  avg_len  max_len     Q1     Q2     Q3  sum_gap    N50  N50_num  Q20(%)  Q30(%)  AvgQual  GC(%)  sum_n
bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode13_sorted.fastq  FASTQ   DNA         10      6,008      219    600.8    1,267    219    621    678        0    678        2   45.21   17.71    11.17  35.92      0
bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode14_sorted.fastq  FASTQ   DNA      5,854  6,073,564       16  1,037.5   20,665    445    700  1,179        0  1,405      481   47.28    18.5    11.38  45.46      0
bcf4b7732185c1a3353d1b4fe80266cd3ac60162_SQK-NBD114-24_barcode15_sorted.fastq  FASTQ   DNA          2      2,900    1,450    1,450    1,450  1,450  1,450  1,450        0  1,450        1   51.93   23.24    12.08     52      0
bcf4b7732185c1a3353d1b4fe80266cd3ac60162_unclassified_sorted.fastq             FASTQ   DNA      2,156  2,567,210       28  1,190.7   26,377    585    859  1,351        0  1,439      223   47.34    18.8    11.34  45.45      0
```

## 5. Análisis de calidad del secuenciamiento Nanopore 

### Concatenado de los archivos FASTQ

```bash
cd ~/genomics/raw_data

mkdir nanopore

cd nanopore

cat /home/ins_user/genomics/basecalling/fast5/pass/*.fastq > b01_fast.fastq
```

### Visualización de la calidad

```bash
cd ~/genomics/quality

mkdir nanopore

cd nanopore

NanoPlot -t 2 --fastq /home/ins_user/genomics/raw_data/nanopore/b01_fast.fastq -p b01_fast_raw_ -o b01_fast_raw --maxlength 100000
```

> **Comentario:** 
> - `--fastq /home/ins_user/genomics/raw_data/nanopore/b01_fast.fastq`: Indica la ruta del archivo FASTQ que contiene las lecturas de secuenciación de ONT que se van a analizar.
> - `-p b01_fast_raw_`: Define el prefijo que se usará para los nombres de los archivos de salida. En este caso, todos los gráficos generados comenzarán con "b01_fast_raw_".
> - `-o b01_fast_raw`: Especifica el directorio de salida donde se guardarán los gráficos. Si el directorio no existe, NanoPlot lo creará.
> - `--maxlength 100000`: Establece la longitud máxima que se mostrará en los gráficos. Las lecturas que superen esta longitud serán truncadas en las visualizaciones. Esto ayuda a enfocar el análisis en la mayoría de las lecturas y evita que las lecturas extremadamente largas distorsionen la visualización.

### Eliminación de adaptadores

```bash
cd ~/genomics/trimming/

mkdir nanopore

cd nanopore

mkdir porechop

cd porechop

porechop -t 2 -i /home/ins_user/genomics/raw_data/nanopore/b01_fast.fastq -o b01_fast_porechop.fastq.gz
```

> **Comentario:** 
> - `-i /home/ins_user/genomics/raw_data/nanopore/b01_fast.fastq`: Esta opción indica la ruta del archivo FASTQ de entrada. Este es el archivo que contiene las lecturas de secuenciación de ONT que se van a procesar.
> - `-o b01_fast_porechop.fastq.gz`: Esta opción especifica el nombre del archivo FASTQ de salida comprimido con gzip. Este archivo contendrá las lecturas después de que Porechop haya recortado los adaptadores y separado las lecturas concatenadas.

### Eliminación de lecturas de baja calidad

```bash
cd ~/genomics/trimming/nanopore

mkdir nanofilt

cd nanofilt

gunzip -c /home/ins_user/genomics/trimming/nanopore/porechop/b01_fast_porechop.fastq.gz | NanoFilt -q 9 --length 1000 | gzip > b01_fast_nanofilt.fastq.gz
```

> **Comentario:** 
> - `gunzip -c /home/ins_user/genomics/trimming/nanopore/porechop/b01_fast_porechop.fastq.gz`: Descomprime el archivo "b01_fast_porechop.fastq.gz".
> - `NanoFilt -q 9 --length 1000`: Filtra las lecturas descomprimidas utilizando NanoFilt, manteniendo solo aquellas que tengan un puntaje de calidad mínimo de 9 y una longitud mínima de 1000 bases.
> - `gzip > b01_fast_nanofilt.fastq.gz`: Comprime las lecturas filtradas y las guarda en un nuevo archivo llamado "b01_fast_nanofilt.fastq.gz".
> - `|`: Este símbolo es una "tubería" (pipe) que conecta la salida de un comando a la entrada de otro. 

```bash
cd ~/genomics/quality/nanopore

NanoPlot -t 2 --fastq /home/ins_user/genomics/trimming/nanopore/nanofilt/b01_fast_nanofilt.fastq.gz -p b00_fast_filt_ -o b01_fast_filt --maxlength 100000
```

Hemos creado este repositorio pero como puedes ver algunos subtitulos solo contienen información de los comandos. Podrías añadirle información que ayude a un mejor entendimiento del tema.
