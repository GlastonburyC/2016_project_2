k=32

# Report run time and memory usage
export SHELL=zsh -opipefail
export REPORTTIME=1
export TIMEFMT=time user=%U system=%S elapsed=%E cpu=%P memory=%M job=%J

all: \
	results/200k/k$k/hsapiens-scaffolds.fac.tsv \
	results/200k.fac.tsv

.DELETE_ON_ERROR:
.SECONDARY:

# Download the complete data.
data/30CJCAAXX_4.fq.gz:
	mkdir -p $(@D)
	curl -o $@ ftp://ftp.bcgsc.ca/public/sjackman/$(@F)

# Download a subset of the data.
data/200k.fq.gz:
	mkdir -p $(@D)
	curl -o $@ ftp://ftp.bcgsc.ca/public/sjackman/$(@F)

# Take a subset of the data.
data/400k.fq.gz: data/30CJCAAXX_4.fq.gz
	gunzip -c $< | head -n1600000 | gzip >$@

# Unzip the data.
%.fq: %.fq.gz
	gunzip -c $< >$@

# Assemble the data with ABySS.
results/200k/k$k/hsapiens-scaffolds.fa: data/200k.fq
	mkdir -p $(@D)
	abyss-pe -C $(@D) name=hsapiens k=$k in=$(realpath $<)

# Calculate the assembly contiguity metrics.
%.fac.tsv: %.fa
	abyss-fac $< >$@

# Concatenate multiple TSV files.
results/200k.fac.tsv: \
		results/200k/k*/hsapiens-scaffolds.fac.tsv \
		results/200k/k$k/hsapiens-scaffolds.fac.tsv
	mlr --tsvlite cat $^ >$@
	datamash -H max N50 <$@
	head -n1 $@; grep $$(datamash -H max N50 <$@ | tail -n1) $@
