# Tema 18: Obtención, calidad y limpieza de lecturas

## Basecalling de los datos de secuenciamiento y visualización de la calidad y limpieza de los archivos FASTQ

### Logro de la Sesión: Al finalizar la sesión, el estudiante realiza el basecalling de los archivos FAST5 y la limpieza de archivos FASTQ con herramientas bioinformáticas.

## Estructura de la práctica:

1. Análisis de calidad del secuenciamiento Illumina
2. Limpieza de los archivos FASTQ de Illumina
3. Basecalling de los archivos FAST5
4. Análisis de calidad del secuenciamiento Nanopore 
5. Limpieza de los archivos FASTQ de Nanopore 

## Programas bioinformáticos:

- **fastqc v0.12.1**: [Descargar](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
- **guppy v6.5.7**: [Descargar](https://community.nanoporetech.com/downloads/)
- **nanofilt v2.8.0**: [Descargar](https://github.com/wdecoster/nanofilt)
- **nanoplot v1.41.6**: [Descargar](https://github.com/wdecoster/NanoPlot)
- **pycoqc v2.5.2**: [Descargar](https://github.com/a-slide/pycoQC)
- **porechop v0.2.4**: [Descargar](https://github.com/rrwick/Porechop)
- **trimgalore v0.6.10**: [Descargar](https://github.com/FelixKrueger/TrimGalore)
- **trimmomatic v0.39**: [Descargar](http://www.usadellab.org/cms/?page=trimmomatic)

## Metodología:

### 1. Análisis de calidad del secuenciamiento Illumina (30 minutos)

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

### 3. Basecalling de los archivos FAST5 (40 minutos)

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

### 4. Análisis de calidad del secuenciamiento Nanopore (30 minutos)

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

### 5. Limpieza de los archivos FASTQ de Nanopore (80 minutos)

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

### 6. Aplicación de las herramientas bioinformáticas (80 minutos)

- Realizar el análisis de calidad de los archivos FASTQ de su respectivo barcode que fueron obtenidos con un basecalling en configuración sup.
- Utilizar el index sup para diferenciarlos del análisis de calidad realizado previamente.
- Los datos están localizados en la siguiente ruta: `/data/2024_2/genome/fastq/barcode00/pass/`

## Resultados:

- Valores de calidad antes y después de la limpieza de los archivos FASTQ obtenidos en el basecalling en configuración SUP (porechop + nanofilt + nanoplot)

## Conclusiones:

- Indicar si la limpieza de las lecturas obtenidas con la configuración SUP fue necesaria y explicar que características de las lecturas mejoraron.
