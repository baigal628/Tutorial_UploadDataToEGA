# How to upload data to EGA?


## What is [EGA](https://ega-archive.org/)?

The European Genome-phenome Archive (EGA) is a service for permanent archiving and sharing of personally identifiable genetic, phenotypic, and clinical data generated for the purposes of biomedical research projects or in the context of research-focused healthcare systems (From EGA Website).

## Prepare documents before getting a submission account

* Fill out the submission form https://ega-archive.org/submission-form.php.
* Form a DAC ([Data Access Committee](https://ega-archive.org/submission/data_access_committee)) and fill out Data processing agreement. This file needs to be signed by both your institution and EGA.
* Email the above file to EGA and they will provide you an account with login info.
* Fill out DAA ([Data Access Agreement](https://ega-archive.org/submission/dac/documentation)).

## Data encryption

Before uploading data to EGA, you should encype your data locally.

### Download data encryptor from EGA:
```{bash}
$ wget https://ega-archive.org/files/EgaCryptor.zip

$ unzip EgaCryptor.zip
```


### Encrype data:
```{bash}
# Here I am using scRNAseq count matrix as an exmple.

$ tar -czvf ./data/Pool01_1_outs.tar.gz ./data/EGA/GEXdata/Pool01_1/outs/

$ java -jar EGA-Cryptor-2.0.0/ega-cryptor-2.0.0.jar -t {threads} \
    -i ./data/Pool01_1_outs.tar.gz \
    -o ./data/EGA/encryped/

# After the encryption, you will see three outputs.

$ ls ./data/EGA/encryped/

Pool01_1_outs.tar.gz.gpg
Pool01_1_outs.tar.gz.gpg.md5
Pool01_1_outs.tar.gz.md5
```

## Upload data to EGA

I use Aspera ascp command line program to upload data. You can also use FTP for uploading files. More details are under: https://ega-archive.org/submission/tools/ftp-aspera

### Download [Aspera CLI](https://github.com/IBM/aspera-cli)
```{bash}
$ wget https://d3gcli72yxqn2z.cloudfront.net/downloads/connect/latest/bin/ibm-aspera-connect_4.2.6.393_linux_x86_64.tar.gz

$ tar xvzf ibm-aspera-connect_4.2.6.393_linux_x86_64.tar.gz

# For my computer, it is installed under home directory:
~/.aspera/connect/bin/ascp
```

### Upload encryped data to EGA
```{bash}
# For a single file:
$ ASPERA_SCP_PASS={password} ~/.aspera/connect/bin/ascp  -P33001  -O33001 -QT -m3000M -L- ./data/EGA/encryped/Pool01_1_outs.tar.gz.gpg ega-box-{id}@fasp.ega.ebi.ac.uk:/.

# For a directory
$ ASPERA_SCP_PASS={password} ~/.aspera/connect/bin/ascp  -P33001  -O33001 -QT -m3000M -L- ./data/EGA/encryped/* ega-box-{id}@fasp.ega.ebi.ac.uk:/.
```

## Register sample metadata

You can either type the sample information one by one in the submission protal or upload a csv file to the portal for large data sets.

### Prepare sample csv file
Here is an example format of how this csv file should look like
```
$ head sample_meta.csv

title,alias,description,subjectId,bioSampleId,caseOrControl,gender,organismPart,cellLine,region, phenotype
scRNAseq,Pool01_1,tumor,001,,case,female,brain,,cortex,relapsed
scRNAseq,Pool01_2,normal,002,,control,male,brain,,cortex,wt
.....
```

## Link data with registered sample metadata
After sample registration, you will want to link the data uploaded to EGA with registered samples. You can do this step manually by clicking on the sample and data item on EGA submission portal. However, you will never want to repeat this many times for large data sets. Here is an example of csv file, where you can link the .bam data items with samples easily. Uploading this file to the submission portal will automatically link the bam files and their md5 checksums to the registered samples.

Here is an example format of how this csv file should look like:
```
$ head sample_bam.csv

Sample alias,BAM File,Checksum,Unencrypted checksum
Pool01_1,Pool01_1.bam,,
Pool01_2,Pool01_2.bam,,
......
```

## Final notes:
* So far, I have not found out a way of linking analysis objects with sample meta via uploading csv file. The only way seems to be manually clicking in the protal. Because of that, I recommend puutting all the analysis data in to one folder, compress to the tar format and upload to the portal.