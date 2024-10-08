# process-sff-with-mothur

## USING MOTHUR TO READ .DFF FILES

### 1. If you have homebrew installed, run this
```
brew install g++ make zlib
```


### 2. download the correct version of MOTHUR from here:
```
https://github.com/mothur/mothur/releases
```


### 3. download SILVA database and taxonomy
```
wget https://mothur.s3.us-east-2.amazonaws.com/wiki/silva.seed_v138.tgz
```


### 4. place your sff file and the SILVA files into the MOTHUR directory, then open the MOTHUR terminal


### 5. to create fasta and qual files:
```
sffinfo(sff=file.sff)
```


### 6. align to SILVA database
```
align.seqs(fasta=file.fasta, reference=silva.nr_v138.align)
```

### 7. Gets a summary
```
summary.seqs(fasta=file.align)
```

### 8. screen to make sure they fit length criteria
```
screen.seqs(fasta=file.align, start=1044, end=43116, minlength=200)
```

### 9. remove chimeric sequences
```
chimera.vsearch(fasta=file.align)
```

### 10. generate a count file
```
unique.seqs(fasta=file.align)
```

### 11. generate a count table
```
count.seqs(name=file.names, fasta=file.unique.align)
```

### 12. classify sequences to create tax file
```
classify.seqs(fasta=file.unique.align, count=file.count_table, reference=silva.nr_v138.align, taxonomy=silva.nr_v138.tax, processors=8)
```

### 13. split your sequences by taxonomic level and cluster them into OTUs
```
cluster.split(fasta=file.unique.align, count=filet.count_table, taxonomy=file.unique.taxonomy, taxlevel=4, processors=8)
```
* fasta: The refined alignment file (test.unique.pick.align).
* count: The refined count table (test.pick.count_table).
* taxonomy: The refined taxonomy file (test.unique.seed_v138.wang.pick.taxonomy).
* taxlevel=4: Specifies the taxonomic level at which to split sequences (Level 4 generally corresponds to the class level).
* processors=8: This specifies the number of processors to use for faster processing (adjust based on your system).

### 14. assign taxonomy to the OTUs
```
classify.otu(list=file.unique.pick.opti_mcc.list, count=file.pick.count_table, taxonomy=file.unique.seed_v138.wang.pick.taxonomy, label=0.03)
```
* list: The .list file generated by the clustering step, in this case, test.unique.pick.opti_mcc.list. This file contains information on the OTUs formed during clustering.
* count: The .count_table file that keeps track of the abundance of sequences, test.pick.count_table.
* taxonomy: The .taxonomy file that contains the taxonomic classification of your sequences, test.unique.seed_v138.wang.pick.taxonomy.
* label=0.03: The OTU label, which corresponds to a 3% dissimilarity threshold, typically used for species-level OTUs.


### 15. transfer to R for plotting:
```
library(ggplot2)
library(dplyr)

taxonomy_data <- read.table("file.unique.pick.opti_mcc.0.03.cons.taxonomy", header = TRUE, sep = "\t")

library(tidyr)
library(dplyr)
library(ggplot2)
library(stringr)

taxonomy_data <- taxonomy_data %>%
  separate(Taxonomy, into = c("Kingdom", "Phylum", "Class", "Order", "Family", "Genus", "Species"), sep = ";", fill = "right")

taxonomy_data <- taxonomy_data %>%
  mutate(across(Kingdom:Species, ~ str_remove(., "\\(.*\\)")))

taxonomy_data <- taxonomy_data %>%
  mutate(across(Kingdom:Species, ~ str_replace(., "unclassified", "na")))

phylum_data <- taxonomy_data %>%
  group_by(Genus) %>%
  summarise(counts = sum(Size))

ggplot(phylum_data, aes(x = Genus, y = counts, fill = Genus)) +
  geom_bar(stat = "identity") +
  scale_y_log10() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  labs(title = "genus level community makeup", x = "genus", y = "count (log adjusted)") +
  theme_bw() +
  theme(legend.position = "none", axis.text.x = element_text(angle = 90, hjust = 1))
```



