# Principal Word Vectors

<strong>[Principal Word Vectors](http://urn.kb.se/resolve?urn=urn:nbn:se:uu:diva-353866)</strong> refer to a set of word vectors (word embeddings) that are built through performing a principal component analysis on a transformed contextual matrix (also known as co-occurrence matrix).
The codes in this repository train a set of principal word vectors from both raw and annotated corpora with different types of context including bag-of-words, position-of-words, and indexed context which enables the tool to process an arbitrary context. 
The codes are tested on `Linux` and `Mac`. The requirements are 
  * A `C` compiler (e.g., `gcc`) to compile the codes in the directory `cwvec`. There is a `Makefile` in this directory for this aim. 
  * Either `octave` or `python3` (with `numpy` and `scipy`) to run the codes in the directory `pwvec`. 

Before running the program, change your directory to `cwvec` and type `make`

```bash
cd cwvec
make
```
 The bash script `princ_wvec.sh` provides you with some parameters that can be set to train a set of word embeddings. 
Modify the parameters in this file in your desired way and type

```bash
bash princ_wvec.sh
```

We explain the main steps of `princ_wvec.sh` in such a way that you can train the embeddings step-by-step. 
The directory `cwvec` (contextual word vectors) contains codes to construct a contextual matrix. 

You should have two executable files in the directory cwvec/build
  * `cwvec`: to build a contextual matrix
  * `print_overflow`: to display overflow files created during the construction of a contextual matrix

The ELF file `cwvec` can process two types of corpus:
  * a raw corpus in which each line is a tokenised sentence
  * an annotated corpus in which each line is a word associated with its features. The words of a sentence are placed in adjacent lines, and an empty line is between sentences. 
  
  An example of a raw corpus is:
  ```
  thats a pretty picture .
  ```
  
  Each line of an annotated corpus should be in the following format:
		
  ```
  id<TAB>word_form<TAB>contexts_ids<TAB>contextual_features
  ```
where:
* 'id' is an integer stating the position of a word in a sentence. It starts at 
* 'word_form' is a word form (usually normalised). We refer to this as the current word
* 'contexts_ids' is a comma-separated list of integers referring to the ids of the contexts of the current word. Use 0 for the root of a dependency context
* 'contextual_features' is a comma-separated list of categorical features associated with the current word. Note that the symbol ',' is a reserved character used as a split character 

An example of an annotated corpus is: 
```
1	thats	4	that,NOUN,NNS,nsubj
2	a	4	a,DET,DT,det
3	pretty	4	pretty,ADJ,JJ,amod
4	picture	0	picture,NOUN,NN,root
5	.	4	.,PUNCT,.,punct
```

Note that the delimiter between the columns should be TAB. The python code in `conllu2context.py` can be used to convert a CoNLL-U format file to an annotated corpus as described above.

cwvec has the following options:

```
$ ./build/cwvec --help
options --input <file.txt>
use the following options
    --corpus-type 	'raw' corpus or 'annotated' corpus (default raw)
    --input	-i	path to input file
    --output	-o	path to output file (default input.bin or input.txt)
    --vocab		path to vocabulary file (default input.vcb)
    --feature		path to feature file (default input.feat)
    --normalize	-n	word normalization (all letters are converted to lowecase format and all sequences of digits are replaced with <num>
    --context-type	-c	the context type, bow (bag-of-word), pow (position-of-word), neighbourhood, or indexed (default bow)
    --load-vocab	load words from vocab file
    --load-features	load features from feature file
    --output-format	-f	'bin' or 'txt' output (default bin)
    --max-memory	-m	the amount of memory (in gigabyte) used for fast matrix access. (default 1.0)
    --overflow-file	overflow file prefix (default overflow)
    --min-vcount	minimum word frequency. Words with frequency smaller than min-vcount are assumed as unknown word (default 1)
    --max-vocab		maximum number of voabulary plus one used for unknown words. Set 0 for infinity. (default 0)
    --min-fcount	minimum feature frequency. Feature with frequency smaller than min-fcount are assumed as unknown feature (default 1).
    --max-feature	maximum number of features plus one used for unknown feature. Set 0 for infinity. (default 0)
    --window	-w	window size (default -1)
    --symmetric		symmetric window of size window_size)
    --print		print cooccurrence matrix on standard output
    --verbose	-v	enable verbose
    --help	-h	print this message
  ```
  
  Example of usage:
  ```bash
  ./build/cwvec --input test/dep_index.txt --corpus-type annotated -c indexed -o test/dep_index.bin -v 
  ```
  where:
  * `dep_index.txt` is an annotated corpus
  * `dep_index.bin` is the corresponding output contextual matrix

  In addition to the contextual matrix, `cwvec` generates a list of vocabulary and features in two files specified by options `--vocab` and `--feature`.  
  
  Once a contextual matrix (the output of cwvec) is built, the principal word vectors are generated by performing a PCA on that. This can be done by the codes available at the pwvec directory. There are two implmentations, one is a `python3` code and the other an `octave` code. 
If you are a python user import `princ_wvec` and contsruct an object based on the class `PrincipalWordVectors`. Set the `cooc_file` parameter to the path of the contextual matrix built by `cwvec`. 
  
 ```python
 import princ_wvec as pwvec
 princ_wvec = pwvec.PrincipalWordVector(cooc_file='../../cwvec/test/dep_index.bin', embeddings_file='../../cwvec/test/dep_index.wvec')
 ```

These are basically done in the file `pwvec/python/pwvec.py`.
 
At this stage, the embeddings are in the file `dep_index.wvec`. It should have the same number of lines as the vocabulary file `dep_index.txt.vcb`. The lines of these two files are aligned to each other. If we want a single file with both words and vectors, we should merge (`paste`) these two files. 

```bash
paste ../../cwvec/test/dep_index.txt.vcb ../../cwvec/test/dep_index.wvec |\
  awk '{printf($1) ; for (i=3;i<=NF;i++) printf(" %s", $i) ; printf("\n")}' > ../../cwvec/test/dep_index.wembed
```

The file `dep_index.wembed` should contain a list of words and vectors (word embeddings). 

# References

* Basirat, A., (2018), Principal Word Vectors, Doctoral Thesis, Uppsala University. Series: Studia Linguistica Upsaliensia, ISSN 1652-1366 ; 22
* Basirat, A., (2018), A Generalized Principal Component Analysis for Word Embedding, Basirat, A., The Seventh Swedish Language Technology Conference (SLTC), Stockholm, Sweden.


