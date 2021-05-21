
# Embedded Understanding
Research project carried out by Thyge Enggaard under the supervision of Sune Lehmann and Morten Axel Pedersen. 

<!-- https://ecotrust-canada.github.io/markdown-toc/ -->

- [Studying contested meaning](#studying-contested-meaning)
- [Methodological approaches](#methodological-approaches)
- [Embedding choices and validation](#embedding-choices-and-validation)
  * [Corpus preprocessing](#corpus-preprocessing)
  * [Embedding parameters](#embedding-parameters)
  * [Stochatic embeddings](#stochatic-embeddings)
- [Empirical analysis so far](#empirical-analysis-so-far)
  * [Data](#data)
  * [Pre-processing of comments](#pre-processing-of-comments)
  * [Embeddings](#embeddings)
  * [Alignment](#alignment)
  * [Embedding properties and alignment](#embedding-properties-and-alignment)
    + [Original embeddings](#original-embeddings)
    + [Normalized embeddings](#normalized-embeddings)
  * [Stratified distance](#stratified-distance)


# Studying contested meaning
Apparently, ~500m tweets are tweeted on Twitter per day, and ~2m comments are made on Reddit per day. For social scienties interested in studying what is being said (and debated, and misunderstood, and...) this pose a significant challenge to methods based on reading a corpus end-to-end.   

One option is to initially enter the corpus through a focus on particular words, that the researcher expects to be central for what is taking place. Local semantic networks or embedding projections might then help the researcher get a sense of which other words the targeted word is being used in relation to.

But is it possible to formalize certain properties of such 'interesting' words and identify them accordingly? That is, as opposed to specifying certain words in advance, is it possible to specify certain criteria that characterizes what makes them interesting? 

In this project, I will attempt to formalize computational methods, that can identify words, that are understood most differently between two corpora, although they apparently seem to refer to the same. More specifically, I will attempt to utilize word embeddings to elicit differences in how words are used, and subsequently explore whether this can be interpreted as differences in understanding.

Polysemy (many-sign, many meanings or significations) is sometimes 'reduced' to a set of disjoint contexts (e.g. apple reffering to either a fruit or a company). But even for one of these options (e.g. apple as a company), how different people understand what apple is might be very polysemic. This does not refer to ones sentiment towards apple (whether one supports, barely knows of or hates apple products, although it might be related), but to the very way in which apple is understood by the subject using the word.

One might e.g. expect that what apple is on r/Capitalism is different from what it is on e.g. r/Socialism or r/Technology. Such difference might at first not seem to hinder debates and discussions - they do to some degree all agree on what apple points out or refers to (what it denotes). But these differences might nonetheless represent what could tentatively perhaps be called dis-understanding; situations in which the 'real' reference seem agreed upon, yet what that reference is, is contested.

Literature/concepts, that might be relevant:
* Floating signifier (Levi-Strauss)
* Heteroglossia (Bakhtin)


# Methodological approaches

The idea I have pursued so far is a 'direct, global' comparison of two static embeddings:
* Train two static embedding, on each of two corpora 
* Align the embeddings, by rotating one of the embeddings to best match the other (orthogonal transformation)
* Identify words with highest and lowest aligned distance

I have also considered the following alternatives:
1. Indirect, static comparison of two embedding (i.e. measures that do not require alignment)
    1. Indirect, global comparison: For each word, calculate its distance to all other words in each embedding and compare distances (e.g. avg. difference)
    1. Indirect, local comparison: For each word, compare the local neighborhoods, e.g. the overlap among the N nearest neighbors in each of the embeddings
1. Dynamic embedding of single corpus: 
    1. Obtain a representation of each token of each word type in the combined corpus, from a pretrained, dynamic embedding (e.g. from BERT, perhaps with finetuning)
    1. Cluster the word representations for each type (to capture disjoint context such as apple (fruit) and apple (company). That is, each word is now split up into multiple, 'disjoint' senses.
    1. For each cluster (word sense), calculate a measure of dispersion - highly dispersed clusters would be candidate words
    1. One advantage, is that after this, one could make a targeted local projection based on the word sense and not just the word type.
    1. One disadvantage is that the dispersion might not relate to between-corpora differences, but could be due to within-corpus effects. This would still be interesting, particular if I can somehow couple this to authors/subreddits.  
1. Dynamic embedding of two corpora:    
    1. Similar to first three steps for the dynamic embedding of single corpus - clusters obtained across corpora, to ensure they are shared
    1. Splitting by corpora, each token is then assigned to the nearest cluster
    1. Instead of calculating cluster dispersion, this would for each cluster (each sense of each word) then measure the distance between the clusters in the two corpora.
    
Unfortunately (in terms of being new) / encouraging (in terms of being feasible and promising), something along the lines of the dynamic options above has was published last year: [Analysing Lexical Semantic Change with Contextualised Word Representations](https://www.aclweb.org/anthology/2020.acl-main.365/) 


# Embedding choices and validation

The embedding of a corpus is an attempt to represent (semantic) similarities between words based on the distribution (co-occurence) of words in the corpus. Here, I will briefly elaborate on three aspects of embeddings, and how validation might look, when the focus is to represent a particular corpus as well as possible (as opposed to say generalize the embedding to be usable in different settings):
1. Corpus preprocessing prior to embedding
1. Deciding embedding parameters
1. Assessing the effect of the (potentially) stochastic nature of embeddings   

## Corpus preprocessing
Embedding the 'raw' corpus can pose various technical challenges (e.g. rare words, large vocabulary). Many pre-processing steps can be applied, e.g. removing words deemed 'unimportant' (often so called stop words), lemmatizing/stemming words, removing words with few characters and/or removing numbers.

While some choices might be analytically motivated (such as splitting words based on their part-of-speech or grammatical tense or voice), this will often not resolve all these choices.      


## Embedding parameters
Training word embeddings require making choices in two areas:
1. Embedding architecture: There are many ways to obtain embeddings. At a high-level, I distinguish between embeddings that are static (e.g. (SVD-)PPMI, SGNS, Fasttext, Glove) and contextual/dynamic (e.g. BERT). Within each of these, many other considerations apply, such as whether the embedding should consider subword-information (e.g. Fasttext) or not (e.g. SGNS).
1. Hyper parameters for the chosen architecture: Training word embeddings requires determining multiple hyper parameters in advance.
    1. Architecture parameters: The model for transforming words to vectors often involve several parameters. A common one is the window-size - how close (in number of tokens) must to word-types be in order to count as associated.
    1. Learning paramters: In addition, the learning procedure entails hyper parameters, such the number of epochs (the number of times the corpus is ingested) or the learning rate (how 'aggressively' the model updates its internal parameters).

Ideally, choices regarding preprocessing (that is not analytically motivated) and embedding paramters should be made based on an understanding of how each decision affects the properties of the embedding for the particular corpus and research question. To the best of my (limited) knowledge, there is however still significant uncertainty about many of these relations, as well as how they vary with the corpus. 

Hence, these choices are often made:
* Based on how well they have worked in other applications. This raises the question of how appropriate they are for a given corpus. As many of these observations are made by the Natural Languace Processing (NLP) community, which in turn (to my current understanding) often work on fairly large corpora, one might in particular worry whether the size of the corpora changes these results.      
* By applying different options on the given corpus and comparing the results. This raises questions of computational feasibility, as the number of joint paramter configurations is often beyond what can be applied in reasonable time.  

## Stochatic embeddings
In addition to these explicit choices (often made implicit through default settings), word embeddings are stochastic - several of the steps involved rely on randomization (e.g. the initial allocation of word vectors, sampling of word pairs in a batch - this is not the case for PPMI embeddings). This leads to instability - applying the same set of choices multiple times will likely not lead to identical embeddings.

The most obvious approach to assessing how this stochasticity affects the 'results' is to make multiple embeddings (with corpus preprocessing and embedding parameters fixed). Beyond computational limits, this then raises questions of how to compare embeddings (obtain results for each, or integrate multiple embeddings into one?) as well as how to proceed, if one is interested in multiple results.   


# Empirical analysis so far


## Data
I currently apply my ideas to two corpuses, consisting of (almost) all comments in the subreddit r/Republican and r/Democrats.

## Pre-processing of comments
Currently, I preprocess as follows: 
* Remove comments that display as removed by the user
* Remove a few obvious bot comments
* Replace urls and usernames with a REM-token
* Remove 'r/' from subreddit names
* Replace underscore with space    
* Replace '!', '?' and '\n' (newline) with a dot ('.')
* Replace characters, that are neither letter or digit with space
* Remove multiple spaces
* Remove multiple dots
* Tokenize the text (currently using the crazy-tokenizer developed for reddit, based on spaCY - https://redditscore.readthedocs.io/en/master/tokenizing.html)
* Parse the tokenized text for lemma and part-of-speech (POS) (currently using Stanza - https://stanfordnlp.github.io/stanza/)
* Lemmatize and lowercase words
* Replace stop words and single-character words a REM-token     
* Append the POS-tag to each word ('dog' transformed to 'dog_NOUN')

Further, I subset the vocabulary, by dropping words that appear less than 20 times.
    
TO DO
* Find a principled way to filter out bot comments
    * Option \#1: Identify groups of comments with high similarity (e.g. TF-IDF) and remove if they are from bots --> not feasible 

  
## Embeddings
I have obtained the following embeddings:
* SVD: Singular-value decomposition of the positive pointwise mutual information matrix
* W2V: Skip-gram with negative sampling (SGNS-version of word2vec), based on the Gensim-library
    * Selected parameters: size=300, window=5, min_count=20, negative(samples)=20 
* FT: FastText embedding, similar to W2V but embeds character n-grams to obtain full word embedding.
    * Selected parameters: Same as above, min_n(character n-gram length)=3, max_n=6
    
TO DO:
* Validate embedding stability
    * For hyperparameter setting: ?
    * For stochasticity: Run multiple embeddings, align and take average?

## Alignment
To align the embeddings, I identify a matrix, R, that satisfies the following two criteria:
1. R should preserve the structure of the embedding, in the sense that the length of vectors and angle between them should be unaffected. Mathematically, this imply that R should be an orthogonal matrix
1. Transforming the embedding of r/Democrats by R should minimize the Frobenius distance between the embedding of r/Republicans and the transformed r/Democrats embedding. 

Finding the matrix R* that meets these criteria is known as the Orthogonal Procrustes problem, and has the following solution:
* Let E_r and E_d denote the unrotated republican and democratic embeddings respectively
* Let U, V.T be the matrixes obtained from the svd of the product of the embeddings, i.e. E_d.T x E_r = U x S x V.T
* R* has the analytical solution R* = U x V.T 

TO DO
* Consider using the same distance measure when training the embedding and aligning the rotation 

## Embedding properties and alignment

### Original embeddings
    
The plot below shows the relation between word frequency (x-axis), word vector length (y-axis) and aligned distance (color - black indicate words that only appear in one of the two embeddings and hence can't be aligned) for the different embeddings:
![Figure](./Figures/freq_length_dist.png)

In general, aligned distance correlates highly with vector length (Pearson correlation between 0.76 and 0.94) and to some extent with  frequency (absolute Pearson correlation between 0.36 and 0.45 - correlation is positive for SVD and negative for W2V and FT, I don't know why this difference occur).

Given these correlations, the words with highest aligned distance are infrequent words, which seems unlikely to be usefull in identifying words with contested meaning.

### Normalized embeddings

To avoid this, and in line with standard use in many NLP-settings, I normalize the embeddings, such that all words vectors have unit length. The squared Frobenius distance between the embeddings is now equal to 2 x (1 - the inner product), which is also equal to two times the cosine distance. Hence the Frobenius distance effectively compares direction of vectors, with two words being farther apart, the more different their aligned directions are.

The plot below shows the relation between aligned distance and word frequency for normalized embeddings.
![Figure](./Figures/normalized_freq_dist.png)

For W2V and FT, the correlation is still aparent (Pearson correlation between -0.41 and -0.49, a little higher than for the original embedding), while for the SVD-PPMI embedding, the correlation seem only to be present for low-frequency words (Pearson correlation -0.07, significantly lower than -0.36 in the original embedding).

Hence, it sees the undesirable relation between frequency and aligned distance is not removed by normalizing vectors.

Potential mechanisms, that might play a role:
1. Frequency and centrality is correlated (centrality is equivalent to vector length) - confirmed
    1. There 'wider' the use of a word is (i.e. the broader its set of context word is), the more the word is used (i.e. higher frequency).
    1. Also, the 'wider' the use of a word is, the more central the word is in the embedding (i.e. the shorter the length of demeaned vectors), as its position becomes a (weighted) average of its position in the different contexts.
1. The normalized embedding is still not uniformly distributed in direction - confirmed     
    1. If vector directions where evenly distributed over the 'direction space', the mean of the normalized vectors should be at the origin, and distances to the embedding center should be tightly distributed around 1.
    1. This is not the case, and further, it seems that embedding centrality still correlates with frequency. It hence seem that frequency is also 'encoded' in direction.
    1. It might be possible to 'remove' this direction prior to aligning embeddings, e.g. by identifying a frequency direction (the hyperplane that maximizes the variance of projects words frequency, similar to PCA), identifying the normal plane to this direction and projection the embedding onto this normal plane ('loosing' one dimension of the embedding) 
    1. Perhaps the degree of 'direction uniformity' is a relevant criteria to consider, when determining when to stop the embedding procedure? Perhaps it makes sense to explicitly include it in the loss function?
1. The rotation alignment correlates centrality and aligned distance for non-uniform distributions - disproved
    1. Based on simulated 2D-data (see plot below), this does not seem to be the case, even in the case of a non-uniformly distributed embedding:

![Figure](./Figures/simul_freq_centrality.png)

## Stratified distance

Given the difficulties in removing the correlation between frequency and aligned distance, I instead control for it based on a simple linear regression. Given the linear relation between aligned distance and log of count as evident above, I regress aligned distance on the logarithm of the average count across the two subreddits, and partition the words into groups based on the percentiles of the prediced distance. The tables below show the nouns (incl. proper nouns) with the highest and lowest aligned distances for each of these groups, as well as the corresponding aligned distance (columns indicate the corresponding percentile and number of words in each group).

Highest aligned distance:
![Figure](./Figures/high_stratified_distance.png)

Lowest aligned distance:
![Figure](./Figures/low_stratified_distance.png)
