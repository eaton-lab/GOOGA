This project implements a genetic algorithm (GA) designed to take genotype data from a segregating population and use it to identify an optimal scaffold order from a perhaps not so well assembled genome.
It does this by permuting scaffold orders from the genome assembly and fitting a HMM with genotype data to estimate recombination rates and the ultimate likelihood of a particular order.  Then this is fed into the GA to search for a near optimal order.

The code runs in python and was developed for python2.7 and requires the scipy package to be installed.

To run on the test data, first download this repo.  Then you need to extract the test data:

` bunzip2 data/LVR.cross.tar.gz` 

` tar xf data/LVR.cross.tar` 

This gives several files, including the genotype files (e.g. g.F2.163.txt), which look like:

`head g.F2.163.txt
F2.163	1	0	AB
F2.163	1	100000	AB
F2.163	1	200000	AB
F2.163	1	300000	AB
F2.163	1	400000	AB
F2.163	1	500000	AB
F2.163	1	600000	AB
F2.163	1	700000	AB
F2.163	1	800000	AB
F2.163	1	900000	AB`

where the columns are line_name, chromosome, interval, genotype

Also the test data contains a list of all lines you wish to consider for ordering scaffolds, it's called `LVR.f2set.txt`, an estimate of the intitial intra-scaffold recombinantion rates called `LVR.isr.txt`, and a file of error rates called `LVR.er2.txt`.

Finally, the genetic algorithm code is called `parallel.genet.alg.final.py`. It was written to be run on a multiprocessor system and can make use of parallelism.  It takes several flags at runtime.  To get these flags simply run:
`python parallel.genet.alg.final.py --help
Usage: parallel.genet.alg.final.py marker_file [options]

Options:
  -h, --help            show this help message and exit
  -e ERROR_FILE         File with precaculated error rates -
                        default=error.rates.txt
  -f F2_FILE            File with F2s to use in analysis -
                        default=test.f2group.txt
  -i INTRASCAFF_RATES_FILE
                        File with precalculated intra-scaffold recombination
                        rates - default=intrascaff_rates.txt
  -c NCPU               Number of CPUs - default=16
  -g NGEN               Number of generations to run the GA - default=10^10
  -l ELITE              Number of elite in each generation default=3
  -t TERMINATION        Number of generations with no improvement before
                        termination - default=1000` 

NCPUs should be set according to your computer.  I was running on a 16 core machine, hence the settings.  

ELITE designates the number of contigs orders (in a genet. alg. they are called individuals) to be carried over to the next generation.  I've had good luck setting this between about 2-4.

Remember that after the 1st generation you will have POP_SIZE - ELITE novel indv since we save past results, on a machine with 10 CPUs, if ELITE=2, you may want to do a popsize of 12, because that will max out all 10 CPUs after Gen 1

You can also control how long the genet. alg. runs, using both NGEN and TERMINATION.  NGEN simply sets a max number of generations for the genetic algorithm, whereas TERMINATION allows you to tell the genetic algorithm to stop after a specified number of generations where the best scaffold order (individual) has not be improved upon.
For example, if NGEN is set to 500 and TERMINATION is set to 200, the genetic algorithm will stop at 500 generations total or 200 gen without improvement, whichever comes first.

Finally there is the per indiviudal mutation rate.  It's specified by the list given to MUTATION.  It is currently set to randomly sample between 0 and 3 mutations per indvidiual per generation.  This proved useful for chromosomes with approx. 20 scaffolds.  It may need to be increased or descreased for for different genomes.

The final input file is a starting a genomic scaffold order.  THere's an example in markers.2.LM.txt.  The genetic algorithm uses this as a staring place.

Then run the code:
`python parallel.genet.alg.final.py markers.2.LM.txt > some.output.file`

And at each generation it will output various run statistics, including all current scaffold orders and likelihoods that the algorithm is grinding away on.

At some point it will stop, and you might want to run:

`python run.stats.py some.output`

This step requires that you have the matplotlib package installed.  It gives a plot of various run statistics so you can see how it progressed.  It gives some sense of whether you need to tune things or if it was pretty successful.

You'll also want to check out the last generation of the output file.  This will contain the full likelihood of the best individual and the corresponding contig order.
