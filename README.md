# Tema 18: Obtención, calidad y limpieza de lecturas

## Estructura de la práctica:

1. Instalación de programas
2. Análisis de calidad del secuenciamiento Illumina
3. Limpieza de los archivos FASTQ de Illumina
4. Basecalling de los archivos FAST5
5. Basecalling de los archivos POD5
6. Análisis de calidad del secuenciamiento Nanopore 
7. Limpieza de los archivos FASTQ de Nanopore 

## Programas bioinformáticos:

- dorado : [Pagina web](https://github.com/nanoporetech/dorado) 
- fastqc : [Pagina web](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
- guppy : [Pagina web](https://community.nanoporetech.com/downloads/)
- nanofilt : [Pagina web](https://github.com/wdecoster/nanofilt)
- nanoplot : [Pagina web](https://github.com/wdecoster/NanoPlot)
- multiqc : [Pagina web](https://github.com/ewels/MultiQC)
- porechop : [Pagina web](https://github.com/rrwick/Porechop)
- trimgalore : [Pagina web](https://github.com/FelixKrueger/TrimGalore)
- trimmomatic : [Pagina web](http://www.usadellab.org/cms/?page=trimmomatic)

## Metodología:

## 1. Instalación de programas

### Instalación de guppy

Guppy es un programa de basecalling y análisis de datos de secuenciación de nanoporos desarrollado por Oxford Nanopore Technologies.

```bash
cd

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

sudo ln -s /home/ins_user/software/dorado-0.9.1-linux-x64/bin/dorado /usr/local/bin/dorado

dorado --help
```

### Instalación de POD5

Permite convertir fast5 a pod5 antes del basecalling

> **Comentario:** POD5 es un formato de archivo desarrollado por Oxford Nanopore Technologies para almacenar datos de secuenciación de nanoporos. Es un formato más eficiente y comprimido que FAST5.

```bash
pip install pod5 
```

### Instalar los siguientes programas en un ambiente de conda

```bash
conda create -n quality -c bioconda fastqc nanofilt nanoplot multiqc porechop trim-galore trimmomatic

conda activate quality
```

## 2. Análisis de calidad del secuenciamiento Illumina

```bash
$ cd
$ mkdir illumina
$ cd illumina
$ mkdir quality
$ cd quality
$ conda activate nanopore_01
$ fastqc -t 10 /data/2024_2/genome/illumina/*.fastq.gz -o .
```

### 2. Limpieza de los archivos FASTQ de Illumina (40 minutos)

```bash
$ cd ~/illumina/
$ mkdir trim
$ cd trim
$ mkdir trim_galore
$ cd trim_galore
$ trim_galore --quality 30 --length 50 --phred33 --cores 4 --fastqc --paired /data/2024_2/genome/illumina/CAT_R1.fastq.gz /data/2024_2/genome/illumina/CAT_R2.fastq.gz
```

```bash
$ cd ~/b00_genome/illumina/trim/
$ mkdir trimmomatic
$ cd trimmomatic
$ nano NexteraPE.fa
```

```plaintext
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
$ trimmomatic PE /data/2024_2/genome/illumina/CAT_R1.fastq.gz /data/2024_2/genome/illumina/CAT_R2.fastq.gz CAT_R1.trim.fastq.gz CAT_R1.unpaired.fastq.gz CAT_R2.trim.fastq.gz CAT_R2.unpaired.fastq.gz ILLUMINACLIP:NexteraPE.fa:2:30:10 SLIDINGWINDOW:4:30 MINLEN:50 -threads 10
$ fastqc -t 10 *.trim.fastq.gz -o .
```

## 3. Basecalling de los archivos FAST5 (40 minutos)

```bash
$ cd
$ mkdir b00_genome
$ cd b00_genome
$ mkdir basecalling
$ cd basecalling
$ nvidia-smi --query-gpu=name --format=csv,noheader
$ watch -d -n 0.5 nvidia-smi
$ guppy_basecaller -i /data/2024_2/genome/fast5/barcode00/ -s barcode00 -c dna_r10.4_e8.1_fast.cfg -x 'cuda:0' --num_callers 4 --gpu_runners_per_device 8
```



> **Comentario:** Esta línea de código instala la biblioteca de Python `pod5`, que se utiliza para interactuar con archivos POD5.

### Ejecutar POD5

```bash
pod5 convert fast5 fast5/*.fast5 --output converted.pod5
```

> **Comentario:** Esta línea de código utiliza `pod5` para convertir todos los archivos FAST5 en el directorio `fast5` a un único archivo POD5 llamado `converted.pod5`.

### Descargar modelos con dorado

```bash
mkdir DB
cd DB
mkdir dorado_models
cd dorado_models
dorado download
mkdir sup fast hac stereo
mv *_sup@* sup
mv *_fast@* fast
mv *_hac@* hac
mv *_stereo@* stereo/
# Ejemplo de creación de enlace simbólico
ln -s /media/prosopis/E/LGBB/Server/rodrigo.suarez/Endofitos_ramas/dorado_models/sup /usr/local/bin/sup
```

> **Comentario:** Estas líneas de código descargan los modelos basecaller de dorado y los organizan en directorios.

### Realizamos el llamado de las bases

```bash
dorado basecaller sup --kit-name SQK-16S024 --min-qscore 10 --barcode-both-ends converted.pod5 > calls.bam
dorado demux --kit-name SQK-16S024 --output-dir barcodes --emit-summary calls.bam
cd barcodes
```

> **Comentario:** 
> - `--kit-name SQK-16S024`: Especifica el nombre del kit de secuenciación utilizado.
> - `--min-qscore 10`: Establece un umbral de calidad mínimo de 10 para las lecturas.
> - `--barcode-both-ends`: Indica que los códigos de barras están presentes en ambos extremos de las lecturas.
> - `> calls.bam`: Redirige la salida a un archivo BAM llamado `calls.bam`.
> - `--output-dir barcodes`: Especifica el directorio de salida para los archivos demultiplexados.
> - `--emit-summary`: Genera un resumen de la ejecución de secuenciación.

```
cat barcoding_summary.txt | head

filename	read_id	barcode
converted.pod5	0117390a-08d3-4548-ad37-bbdea086f54b	SQK-16S024_barcode02
converted.pod5	017d7593-d3df-4c37-941d-2fbb343f47cd	SQK-16S024_barcode02
converted.pod5	0214a6b5-f1ea-418d-b0f4-2573f31012d8	SQK-16S024_barcode02
converted.pod5	0355a06e-60bc-438d-8bb6-acadc76b3de4	SQK-16S024_barcode02
converted.pod5	022ed867-498d-478f-bd99-5546117b2552	SQK-16S024_barcode02
converted.pod5	02fb5dcf-65da-4123-a894-58fb866e887f	SQK-16S024_barcode02
converted.pod5	01e37d3d-e728-47c5-96b5-de7c3640e6bc	SQK-16S024_barcode02
converted.pod5	02c9c525-d8b5-47c1-a5dd-6e88efc21387	SQK-16S024_barcode02
converted.pod5	01f60196-800e-4dd0-b513-853c6a583ae3	SQK-16S024_barcode02
```

#### Resumen de la secuenciación 

```bash
for file in *.bam; do prefix="${file%.bam}"; dorado summary "$file" > "${prefix}_summary.tsv"; done
```

> **Comentario:** Genera un resumen para cada archivo BAM en el directorio y lo guarda en un archivo TSV.

```
cat SQK-16S024_barcode02_summary.tsv | head

filename	read_id	run_id	channel	mux	start_time	duration	template_start	template_duration	sequence_length_template	mean_qscore_template	barcode
converted.pod5	0117390a-08d3-4548-ad37-bbdea086f54b	8371d699ccb2363baf11e06bf47184641336067a	28	1	19382.4	4.21525	19382.4	4.21525	1343	12.3944	SQK-16S024_barcode02
converted.pod5	017d7593-d3df-4c37-941d-2fbb343f47cd	8371d699ccb2363baf11e06bf47184641336067a	364	4	19455.5	4.32825	19455.5	4.32825	1459	13.7667	SQK-16S024_barcode02
converted.pod5	0214a6b5-f1ea-418d-b0f4-2573f31012d8	8371d699ccb2363baf11e06bf47184641336067a	165	3	2941.15	4.40425	2941.15	4.40425	1425	10.4715	SQK-16S024_barcode02
converted.pod5	0355a06e-60bc-438d-8bb6-acadc76b3de4	8371d699ccb2363baf11e06bf47184641336067a	476	4	20144	2.22075	20144	2.22075	621	13.4812	SQK-16S024_barcode02
converted.pod5	022ed867-498d-478f-bd99-5546117b2552	8371d699ccb2363baf11e06bf47184641336067a	224	1	6611.75	3.621	6611.75	3.621	1301	10.6204	SQK-16S024_barcode02
converted.pod5	02fb5dcf-65da-4123-a894-58fb866e887f	8371d699ccb2363baf11e06bf47184641336067a	101	2	17048	4.478	17048	4.478	1350	11.4253	SQK-16S024_barcode02
converted.pod5	01e37d3d-e728-47c5-96b5-de7c3640e6bc	8371d699ccb2363baf11e06bf47184641336067a	440	4	7330.72	3.80175	7330.72	3.80175	1446	11.2083	SQK-16S024_barcode02
converted.pod5	02c9c525-d8b5-47c1-a5dd-6e88efc21387	8371d699ccb2363baf11e06bf47184641336067a	468	4	17689.1	4.78125	17689.1	4.78125	1373	12.9938	SQK-16S024_barcode02
converted.pod5	01f60196-800e-4dd0-b513-853c6a583ae3	8371d699ccb2363baf11e06bf47184641336067a	232	2	11831.6	2.98875	11831.6	2.98875	1061	11.7921	SQK-16S024_barcode02
```

### Convertimos de bam a fastq

```bash
for file in *.bam; do prefix="${file%.bam}"; samtools sort -n "$file" -o "${prefix}_sorted.bam"; done
for file in *_sorted.bam; do prefix="${file%.bam}"; bedtools bamtofastq -i "${prefix}.bam" -fq "${prefix}.fastq"; done
seqkit stats -a -j 4 *_sorted.fastq > stats_sorted_fastq.txt
```

> **Comentario:** 
> - `samtools sort -n`: Ordena los archivos BAM por nombre de lectura.
> - `bedtools bamtofastq`: Convierte los archivos BAM ordenados a formato FASTQ.

```bash
file                               format  type  num_seqs      sum_len  min_len  avg_len  max_len       Q1       Q2     Q3  sum_gap    N50  Q20(%)  Q30(%)
SQK-16S024_barcode01_sorted.fastq  FASTQ   DNA      2,232    2,812,246        2    1,260    3,808    1,230    1,405  1,455        0  1,421   50.76    8.48
SQK-16S024_barcode02_sorted.fastq  FASTQ   DNA      1,626    1,903,444        4  1,170.6    1,953      980    1,387  1,452        0  1,413   50.53    8.58
SQK-16S024_barcode03_sorted.fastq  FASTQ   DNA     12,802   15,342,892        7  1,198.5    2,877    1,077    1,394  1,445        0  1,408   50.64    7.79
SQK-16S024_barcode04_sorted.fastq  FASTQ   DNA      8,550    9,958,508        9  1,164.7    2,091    1,029    1,387  1,436        0  1,402    50.3     7.5
SQK-16S024_barcode05_sorted.fastq  FASTQ   DNA        678      871,012       59  1,284.7    1,937    1,333    1,404  1,455        0  1,419   49.97    7.33
SQK-16S024_barcode06_sorted.fastq  FASTQ   DNA      3,192    4,003,420        3  1,254.2    2,250    1,306    1,407  1,452        0  1,421   51.25    7.43
SQK-16S024_barcode07_sorted.fastq  FASTQ   DNA     12,476   14,802,626        4  1,186.5    2,838    1,053    1,395  1,448        0  1,409   50.54    7.45
SQK-16S024_barcode08_sorted.fastq  FASTQ   DNA        312      437,058       28  1,400.8    1,861  1,389.5    1,421  1,454        0  1,424   50.44    8.41
SQK-16S024_barcode09_sorted.fastq  FASTQ   DNA     22,938   18,677,366        2    814.3    1,993      346      873  1,335        0  1,254   51.16    7.87
SQK-16S024_barcode10_sorted.fastq  FASTQ   DNA     17,742   19,764,174        8    1,114    2,654      717    1,308  1,409        0  1,391   50.81    8.19
SQK-16S024_barcode11_sorted.fastq  FASTQ   DNA     13,662   14,534,452        1  1,063.9    2,029      838    1,192  1,384        0  1,337   50.87    7.85
SQK-16S024_barcode12_sorted.fastq  FASTQ   DNA     11,866   13,408,054       10    1,130    2,012      944    1,335  1,399        0  1,377   50.37     7.7
SQK-16S024_barcode13_sorted.fastq  FASTQ   DNA     13,526   15,222,272        1  1,125.4    2,696      914    1,326  1,398        0  1,374   50.65    7.65
SQK-16S024_barcode14_sorted.fastq  FASTQ   DNA     16,278   15,979,112        2    981.6    2,320      504    1,126  1,392        0  1,359   50.59    8.19
SQK-16S024_barcode15_sorted.fastq  FASTQ   DNA     14,378   16,015,288        2  1,113.9    2,440      831    1,338  1,402        0  1,388   50.57    7.33
SQK-16S024_barcode16_sorted.fastq  FASTQ   DNA     14,146   15,581,836        2  1,101.5    2,109      814    1,300  1,400        0  1,379   50.65     7.9
SQK-16S024_barcode17_sorted.fastq  FASTQ   DNA     17,988   18,836,740        2  1,047.2    3,057      624  1,304.5  1,399        0  1,380   50.24    7.07
SQK-16S024_barcode18_sorted.fastq  FASTQ   DNA     16,030   16,605,426        2  1,035.9    2,088      537    1,287  1,399        0  1,386   50.48    7.85
unclassified_sorted.fastq          FASTQ   DNA    474,168  648,009,092        7  1,366.6    4,216    1,393    1,405  1,421        0  1,406   53.45    8.42
```

## 4. Análisis de calidad del secuenciamiento Nanopore (30 minutos)

```bash
$ cd ~/b00_genome/
$ mkdir raw_data
$ cd raw_data
$ cat /data/2024_2/genome/fastq/barcode00/pass/*.fastq > b00_sup.fastq
```

```bash
$ cd ~/b00_genome/
$ mkdir quality
$ cd quality
$ mkdir pycoqc
$ cd pycoqc
$ conda activate nanopore_00
$ pycoQC -f /data/2024_2/genome/fastq/barcode00/sequencing_summary.txt --html_outfile b00_sup_pycoqc.html
```

```bash
$ cd ~/b00_genome/quality/
$ mkdir fastqc
$ cd fastqc
$ conda activate nanopore_01
$ fastqc -t 20 /home/alumnoZZ/b00_genome/raw_data/b00_sup.fastq -o .
```

```bash
$ cd ~/b00_genome/quality/
$ mkdir nanoplot
$ cd nanoplot
$ NanoPlot -t 20 --fastq /home/alumnoZZ/b00_genome/raw_data/b00_sup.fastq -p b00_sup_raw_ -o b00_sup_raw --maxlength 100000000
```

## 5. Limpieza de los archivos FASTQ de Nanopore (80 minutos)

```bash
$ cd ~/b00_genome/
$ mkdir trim
$ cd trim
$ mkdir porechop
$ cd porechop
$ porechop -t 20 -i /home/alumnoZZ/b00_genome/raw_data/b00_sup.fastq -o b00_sup_porechop.fastq.gz
```

```bash
$ cd ~/b00_genome/quality/nanoplot/
$ NanoPlot -t 20 --fastq /home/alumnoZZ/b00_genome/trim/porechop/b00_sup_porechop.fastq.gz -p b00_sup_porechop_ -o b00_sup_porechop --maxlength 100000000
```

```bash
$ cd ~/b00_genome/trim/
$ mkdir nanofilt
$ cd nanofilt
$ gunzip -c /home/alumnoZZ/b00_genome/trim/porechop/b00_sup_porechop.fastq.gz | NanoFilt -q 10 --length 1000 | gzip > b00_sup_nanofilt.fastq.gz
```

```bash
$ cd ~/b00_genome/quality/nanoplot/
$ NanoPlot -t 20 --fastq /home/alumnoZZ/b00_genome/trim/nanofilt/b00_sup_nanofilt.fastq.gz -p b00_sup_nanofilt_ -o b00_sup_nanofilt --maxlength 100000000
```

## 6. Aplicación de las herramientas bioinformáticas (80 minutos)

- Realizar el análisis de calidad de los archivos FASTQ de su respectivo barcode que fueron obtenidos con un basecalling en configuración sup.
- Utilizar el index sup para diferenciarlos del análisis de calidad realizado previamente.
- Los datos están localizados en la siguiente ruta: `/data/2024_2/genome/fastq/barcode00/pass/`

## Resultados:

- Valores de calidad antes y después de la limpieza de los archivos FASTQ obtenidos en el basecalling en configuración SUP (porechop + nanofilt + nanoplot)

## Conclusiones:

- Indicar si la limpieza de las lecturas obtenidas con la configuración SUP fue necesaria y explicar que características de las lecturas mejoraron.
