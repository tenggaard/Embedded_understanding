# Embedded Understanding

Research project carried out by Thyge Enggaard under the supervision of Sune Lehmann and Morten Axel Pedersen. 

## About this project
The central tenet of this project is that while there might be no such thing as a (completely) private language, all 'points' of understanding are somewhat unique. This is the case whether one compares two or more persons or the 'same' person at different points in chronological time. At the scale of multiple persons, groups of relatively similar understanding might arise. If this is the case, how would we be able to elicit and interpret these differences in understanding?   

In this project, I will attempt to utilize word embeddings to elicit such differences. The embedding of text in a vector space captures relations between words as they are used in a corpus. If we distinguish between direct association (e.g. the words 'blue' and 'sky' being used in succession) and structural equivalence (e.g. the word 'cat' and 'dog' being relatively interchangeable), word embeddings in particular seem to capture the latter.

By comparing embeddings from different corpora, embeddings might help us elicit which words that have the largest (and smallest) discrepancies in terms of their structural equivalences between corpora.  


## Data

To explore various ways in which such patterns might be elicited, I currently apply my ideas to two corpuses, consisting of all comments in the subreddit r/Republican and r/Democrats respectively.

## Analysis so far

### Pre-processing of comments
TO DO
* Identify groups of comments with high similarity (e.g. TF-IDF) and remove if they are from bots 

Most notably, I add the part-of-speech (POS) tag to each word. This partly helps reduce problems of polysemy (e.g. 'back' as noun ('my back hurts' and 'back' as adverb (the room is at the back of the house)), and further allow me to study whether certain word-types have larger discrepancies than others.  

  
### Embedding
Initially, I create static embeddings (each word has a fixed vector). Dynamic embeddings (e.g. BERT) would be better suited to alleviate the problem of polysemy, but comparing word-types between corpuses seem more difficult.

For now, I use skip-gram with negative sampling (word2vec) as implemented in the Gensim-library.


### Approaches to comparing corpora
Word embeddings allow for multiple ways of comparing corpora. At a broad level, two approaches can be distinguished:
1. Direct comparison: Aligning the embedded space to a shared space, in order to directly compare word
1. Indirect comparison: Compare word based on comparing features of each embedding
    1. Global comparison: For each word, compare its distance to all other words between the two embeddings
    1. Local comparison: For each word, compare its neighboorhodd, e.g. the overlap among the N nearest neighbors


### Results from direct comparison
TO DO
* Consider using the same distance measure when training the embedding and aligning the rotation 

I have applied three approaches to identify an optimal distance-preserving rotation, that yield similar results (last is the exact solution):
* Stochastic Gradient Descent (sgd): Adjust the rotation matrix according to the gradient of the mean squared euclidean distance between sampled words and find the nearest orthogonal matrix based on the SVD. 
* Pseudo-inverse (pseudo): Apply the pseudo-inverse from a sample of words to solve the equivalent of an exact rotation, take the mean of the suggested rotations matrices, and find the nearest orthogonal matrix.  
* Orthogonal Procustes Problem (svd): The problem is known as the orthogonal procrustes problem, which has the analytical solution U x V.T, where U and VT are from the SVD of the product of the two embeddings.

#### Distribution of squared euclidean distances
The two approaches generate very similar distributions of rotated distances.
![Figure 1](./Figures/Aligned_distances.png)

#### Squared euclidean distance by POS-type
There is great within-POS variation in rotated distance. Noun-parts (NOUN, ADJ, PROPN) seem a little higher, but the difference migth be insignificant.
![Figure 2](./Figures/Aligned_distances_by_POS.png)

#### Squared euclidean distance by corpus frequency
Rotated distances is greater for words that are less frequent in the corpus. This seem to be an undesirable feature. 
![Figure 3](./Figures/Aligned_distances_by_count.png)

#### Squared euclidean distance by distance to embedding center
One explanation for the relation between word frequency and aligned distance, might be the following:
* When aligning words, it might matter more for avg. aligned distance to match words closer to the embedding center than in the periphery.
* Words that appear more frequently might be placed closer to the embedding center to minimize the avg. distance to context words   

The last part seems to be the case:
![Figure 4](./Figures/Distances_to_center_by_count.png)

#### Inspecting individual words
The 50 words with the largest rotated difference are:

> ['yada_PROPN', 'vis_X', 'yadda_PROPN', 'spade_NOUN', 'dee_PROPN', 'ding_PROPN', 'usable_ADJ', 'drip_NOUN', 'pirate_NOUN', 'resubmit_VERB', 'footer_NOUN', 'blah_INTJ', 'jacobs_PROPN', 'latina_PROPN', 'gaza_PROPN', 'greet_VERB', 'boop_PROPN', 'plug_NOUN', 'rah_PROPN', 'strip_NOUN', 'mushroom_NOUN', 'duplicate_ADJ', 'cunningham_PROPN', 'patch_NOUN', 'shepard_PROPN', 'beta_NOUN', 'precision_NOUN', 'nsfw_PROPN', 'gag_NOUN', 'tat_NOUN', 'robin_PROPN', 'rig_NOUN', 'lighting_NOUN', 'buckley_PROPN', 'quadruple_VERB', 'sewer_NOUN', 'rescind_VERB', 'choir_NOUN', 'railroad_NOUN', 'gimme_PROPN', 'sort_ADJ', 'fill_NOUN', 'gifs_NOUN', 'imagery_NOUN', 'bp_PROPN', 'convo_NOUN', 'turner_PROPN', 'sterling_PROPN', 'increased_ADJ', 'gracious_ADJ']

The 50 words with the smallest rotated difference are:

> ['actually_ADV', 'probably_ADV', 'know_VERB', 'think_VERB', 'guy_NOUN', 'maybe_ADV', 'gt_INTJ', 'mean_VERB', 'thing_NOUN', 'happen_VERB', 'sure_ADJ', 'go_VERB', 'gt_CCONJ', 'say_VERB', 'simply_ADV', 'lol_PROPN', 'instead_ADV', 'believe_VERB', 'yeah_INTJ', 'people_NOUN', 'obviously_ADV', 'lot_NOUN', 'guess_VERB', 'time_NOUN', 'start_VERB', 'especially_ADV', 'right_ADV', 'problem_NOUN', 'consider_VERB', 'talk_VERB', 'idea_NOUN', 'like_SCONJ', 'exactly_ADV', 'try_VERB', 'tell_VERB', 'continue_VERB', 'well_ADJ', 'reason_NOUN', 'support_VERB', 'get_VERB', 'win_VERB', 'bad_ADJ', 'certainly_ADV', 'work_VERB', 'good_ADJ', 'help_VERB', 'literally_ADV', 'fact_NOUN', 'call_VERB', 'come_VERB']

Based on the plot above, there seems to be a small number of highly used words with relative high distance. Among words used at least 80.000 times in at least one of the subreddits, the ten words with the largest rotated distance are:

> ['wiki_NOUN', 'karma_PROPN', 'bot_NOUN', 'automatically_ADV', 'perform_VERB', 'compose_VERB', 'contact_VERB', 'moderator_NOUN', 'delete_VERB', 'comment_VERB']


