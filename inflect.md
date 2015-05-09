---
layout: default
img: exam
img_link: http://www.flickr.com/photos/gianellbendijo/4034021658/
caption: Image by gianellbendijo (used with permission)
title: Homework 5 | Inflection
active_tab: homework
---

Inflection <span class="text-muted"></span>
==============================================================

The traditional formulations of the main problems in machine
translation, from alignment to model extraction to decoding
and through evaluation, ignore the linguistic phenomenon of
[morphology](http://en.wikipedia.org/wiki/Morphology_(linguistics)),
instead treating all words as distinct atoms. This misses
out on a number of generalizations; for example, in
alignment, it could be useful to accumulate evidence across
the various inflections of a verb such as *walk*, since
*walk*, *walked*, *walks*, and *walking* all likely have
related and overlapping translations.

There are two types of morphology:
[inflectional morphology](http://en.wikipedia.org/wiki/Inflection)
studies how words change to reflect grammatical properties,
roles, and other information, while
[derivational morphology](http://en.wikipedia.org/wiki/Derivation_(linguistics))
describes how words changes as they are adapted to different
parts of speech. Of these, inflectional morphology is the
more important modeling omission in natural language
generation tasks like machine translation, because choosing
the right form of a word is necessary to produce
grammatical output.

The inflectional morphology of English is simple. It is
mostly limited to verbs and pronouns, which reflect only a
subset of
[person](http://en.wikipedia.org/wiki/Grammatical_person),
[number](http://en.wikipedia.org/wiki/Grammatical_number),
and one of two
[cases](http://en.wikipedia.org/wiki/Grammatical_case), and
the forms overlap among the possible combinations. We can
translate into English fairly well without bothering with
morphology
([an auspicious fact for the development of field](http://cs.jhu.edu/~post/bitext/#same-language)).

However, this is not the case for many of the world's
languages. Languages such as Russian, Turkish, and Finnish
have complex case systems that can produce hundreds of
surface variations of a single
[lemma](http://en.wikipedia.org/wiki/Lemma_(psycholinguistics)). The
vast number of potential word forms creates data sparsity,
an issue that is exacerbated by the fact that
morphologically complex languages are often the ones without
much in the way of parallel data.

In this assignment, you will earn an appreciation for the
difficulties posed by morphology.  The setting is simple:
you are presented with a sequence of Czech lemmas, and your
task is to choose the correct inflected form for each of
them. You can imagine this as a translation task itself,
except with no reordering and with a bijection between the
source and target words. To support you in this task, you
are provided with a parallel training corpus containing
sentence pairs in both reduced and inflected forms, and a
default solution chooses the most probable form for each
lemma. 

Getting Started
---------------

<div class="alert alert-danger"> <b>Important!</b> The <a
  href="http://catalog.ldc.upenn.edu/LDC2006T01">data used
  in this assignment</a> is released through the <a
  href="http://ldc.upenn.edu">Linguistic Data Consortium
  (LDC)</a>, and the license agreement prohibits
  direct distribution of the data as we have done for other
  assignments. You will need a SEAS account to work on this
  assignment, and please do not remove the data from those
  servers (except to upload your output on the test
  data).</div>

Start by downloading the assignment [starter kit](http://seas.upenn.edu/~cis526/inflect.zip)
onto your eniac account.
In the `inflect` directory, type the following
to create symlinks to the training and development data (and
please observe the warning that began this section):

    cd inflect
    bash scripts/link_data.sh

Note that you must be on a Penn server to access the data.
You will then find three sets of parallel files under `data`:
training data (for building models), development test data
(for testing your model), and held-out test data (for
submitting to the [leaderboard](leaderboard.html)). Sentences
are parallel at the line level, and the words on each line
also correspond exactly across files. The parallel files
have the prefix `train`, `dtest`, and `etest`, and the
following suffixes:

- `*.lemma` contains the lemmatized version of the data. Each
  lemma can be inflected to one or more fully inflected
  forms (that may or may not share the same surface form).

- `*.tag` contains a two-character sequence denoting each
  word's part of speech

- `*.tree` contains [dependency trees](http://en.wikipedia.org/wiki/Dependency_grammar), which
  organize the words into a tree with words 
  generating their arguments. The tree format is described
  below.

- `*.form` contains the fully inflected form. Note that we
  provide `dtest.form` to you (the grading script needs it),
  but you should not look at it or build models over
  it. `etest.form` is kept unreadable and will be used to
  produce your leaderboard score.

You should use the development data (`dtest`) to test your
approaches (make sure you don't use the answers except in
the grader). When you have something that works, you should
run it on the test data (`etest.*`) and submit that
output. The `scripts/` subdirectory contains a number of
scripts, including a grader and a default implementation
that simply chooses the most likely inflection for each
word:

    # Just for fun: no inflection
    cat data/dtest.lemma | ./scripts/grade

    # Default: Choose the most likely inflection using the development data
    cat data/dtest.lemma | ./scripts/inflect | ./scripts/grade

The evaluation method is accuracy: what percentage of the
correct inflections did you choose?
    
The Challenge
-------------

Your challenge is to __improve the accuracy of the inflector
as much as possible__. The provided implementation simply
chooses the most frequent inflection computed from the 
lemma alone (with statistics gathered from the training data).

For a passing grade, it is sufficient to implement a bigram
language model of some form (conditioned on the previous
word or lemma). However, as described above, we have
provided plenty more information to you that should permit
much subtler approaches. Here are some suggestions:

* Incorporate part-of-speech tags.
* Implement a bigram language model over inflected forms.
* Implement a longer n-gram model and a custom backoff
  structure that consider shorter contexts, POS tags, the
  lemma, etc.
* Train your language model on more data, perhaps pulled
  from the web.
* Model long-distance agreement by incorporating the labeled
  dependency structure. For example, you could build a
  bigram language model that decomposes over the dependency
  tree, instead of the immediate n-gram history.
* Implement multiple approaches and take a vote on each word.

Obviously, you should feel free to pursue other ideas.
Morphology for machine translation is an understudied
problem, so it's possible you could come up with an idea
that people have not tried before!

### A note on POS tags and dependency trees

The `.pos` and `.tree` files contain parts of speech and
dependency trees for each sentence. Information about the
part-of-speech tags
[can be found here](https://ufal.mff.cuni.cz/pdt2.0/doc/manuals/en/a-layer/html/ch01s02.html).

Dependency trees are represented as follows. The tokens on
each line correspond to the words they share an index with,
and contain two pieces of information, depicted as
PARENT/LABEL. PARENT is the index of the word's parent word,
and LABEL is the label of the edge implicit between those
indices. Parent index 0 represents the root of the
tree. Each child selects its parent, but the edge direction
is from parent to child.

For example, consider the following lines, from the lemma,
POS, tree, and word files (plus an English gloss), respectively:

    třikrát`3 rychlý než-2 slovo
    Cv AA J, NN
    2/Adv 0/ExD 2/AuxC 3/ExD
    Třikrát rychlejší než slovo
    Three-times faster than-the-word

Line 3 here corresponds to the following dependency tree:

![Dependency tree](assets/img/hw5_dep.png)

To avoid duplicated work, a class is provided to you that
will read the dependency structure for you, providing direct
access to each word's head and children (if any), along with
the labels of these edges. Example usage can be found in
`scripts/inflect-tree`. For a list of analytical functions
(the edge labels),
[see this document](https://ufal.mff.cuni.cz/pdt2.0/doc/manuals/en/a-layer/html/ch03.html#s1-list-anal-func).

Ground Rules
------------

* You must work independently.
* You must turn in three things:
  1. The output of your inflector on <b>both the dev and test sets</b>
     (`data/dtest.lemma` and `data/etest.lemma`, concatenated together), uploaded using
     the command `turnin -c cis526 -p inflect inflect.txt` from any Eniag or Biglab
     machine. You can upload new output as often as you like, up until the assignment deadline.
     Your output file should have 8,714 lines.
  2. Your code, uploaded using the command `turnin -c cis526 -p inflect-code file1 file2 ...`.
     This is due 24 hours after the leaderboard closes.
     You are free to extend the code we provide or write your own in whatever
     langugage you like, but the code should be self-contained,
     self-documenting, and easy to use.
  3. A report describing the models you designed and experimented with, uploaded
     using the command `turnin -c cis526 -p inflect-report inflect-report.pdf`. This is
     due 24 hours after the leaderboard closes. Your report does not need to be
     long, but it should at minimum address the following points:
     
     * **Motivation**: Why did you choose the models you experimented with?

     * **Description of models or algorithms**: Describe mathematically or algorithmically
         what you did.
         Your descriptions should be clear enough that someone else in the class could
         implement them.

     * **Results**: You most likely experimented with various settings of any models
       you implemented.
       We want to know how you decided on the final model that you submitted for us to grade.
       What parameters did you try, and what were the results?
       Most importantly: what did you learn?

       Since we have already given you a concrete problem and dataset, you do not
       need describe these as if you were writing a full scientific paper. Instead,
       you should focus on an accurate technical description of the above items.

       Note: These reports will be made available via hyperlinks on the leaderboard.
       Therefore, **you are not required to include your real name** if you would prefer not
       to do so.
       
* You should not need any other data than what we provide. You
   are free to use any code or software you like, __except
   for those expressly intended to do morphological
   generation__.  If you want to use other part-of-speech taggers,
   syntactic or semantic parsers, machine learning
   libraries, thesauri, or any other off-the-shelf
   resources, plese feel free to do so. If you aren't sure
   whether something is permitted, ask us.

*Credits: This assignment was designed for this course by
 [Matt Post](http://cs.jhu.edu/~post/). The data used in the
 assignment comes from the
 [Prague Dependency Treebank v2.0](https://ufal.mff.cuni.cz/pdt2.0/)*
