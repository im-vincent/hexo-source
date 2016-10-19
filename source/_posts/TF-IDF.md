title: TF-IDF
date: 2015-09-24 14:57:45
tags: [python,spark]
---
### **ER as Text Similarity - Weighted Bag-of-Words using TF-IDF**
#### Bag-of-words comparisons are not very good when all tokens are treated the same: some tokens are more important than others. Weights give us a way to specify which tokens to favor. With weights, when we compare documents, instead of counting common tokens, we sum up the weights of common tokens. A good heuristic for assigning weights is called "Term-Frequency/Inverse-Document-Frequency," or [TF-IDF][tfidf] for short.
#### **TF**
#### TF rewards tokens that appear many times in the same document. It is computed as the frequency of a token in a document, that is, if document *d* contains 100 tokens and token *t* appears in *d* 5 times, then the TF weight of *t* in *d* is *5/100 = 1/20*. The intuition for TF is that if a word occurs often in a document, then it is more important to the meaning of the document.
#### **IDF**
#### IDF rewards tokens that are rare overall in a dataset. The intuition is that it is more significant if two documents share a rare word than a common one. IDF weight for a token, *t*, in a set of documents, *U*, is computed as follows:
* #### Let *N* be the total number of documents in *U*
* #### Find *n(t)*, the number of documents in *U* that contain *t*
* #### Then *IDF(t) = N/n(t)*.
#### Note that *n(t)/N* is the frequency of *t* in *U*, and *N/n(t)* is the inverse frequency.
> #### **Note on terminology**: Sometimes token weights depend on the document the token belongs to, that is, the same token may have a different weight when it's found in different documents.  We call these weights *local* weights.  TF is an example of a local weight, because it depends on the length of the source.  On the other hand, some token weights only depend on the token, and are the same everywhere that token is found.  We call these weights *global*, and IDF is one such weight.
#### **TF-IDF**
#### Finally, to bring it all together, the total TF-IDF weight for a token in a document is the product of its TF and IDF weights.
[tfidf]: https://en.wikipedia.org/wiki/Tf%E2%80%93idf

### **Implement a TF function**
#### Implement `tf(tokens)` that takes a list of tokens and returns a Python [dictionary](https://docs.python.org/2/tutorial/datastructures.html#dictionaries) mapping tokens to TF weights.
#### The steps your function should perform are:
* #### Create an empty Python dictionary
* #### For each of the tokens in the input `tokens` list, count 1 for each occurance and add the token to the dictionary
* #### For each of the tokens in the dictionary, divide the token's count by the total number of tokens in the input `tokens` list

```
def tf(tokens):
    """ Compute TF
    Args:
        tokens (list of str): input list of tokens from tokenize
    Returns:
        dictionary: a dictionary of tokens to its TF values
    """

    return {token:tokens.count(token)/float(len(tokens)) for token in tokens}
    #词频 (TF) 是一词语出现的次数除以该文件的总词语数

print tf(tokenize(quickbrownfox)) # Should give { 'quick': 0.1666 ... }

```

### **Implement an IDFs function**
#### Implement `idfs` that assigns an IDF weight to every unique token in an RDD called `corpus`. The function should return an pair RDD where the `key` is the unique token and value is the IDF weight for the token.
#### Recall that the IDF weight for a token, *t*, in a set of documents, *U*, is computed as follows:
* #### Let *N* be the total number of documents in *U*.
* #### Find *n(t)*, the number of documents in *U* that contain *t*.
* #### Then *IDF(t) = N/n(t)*.
#### The steps your function should perform are:
* #### Calculate *N*. Think about how you can calculate *N* from the input RDD.
* #### Create an RDD (*not a pair RDD*) containing the unique tokens from each document in the input `corpus`. For each document, you should only include a token once, *even if it appears multiple times in that document.*
* #### For each of the unique tokens, count how many times it appears in the document and then compute the IDF for that token: *N/n(t)*
#### Use your `idfs` to compute the IDF weights for all tokens in `corpusRDD` (the combined small datasets).
#### How many unique tokens are there?

```
def idfs(corpus):
    """ Compute IDF
    Args:
        corpus (RDD): input corpus
    Returns:
        RDD: a RDD of (token, IDF value)
    """
    N = corpus.count()
    #总得词数
    uniqueTokens = corpus.map(lambda (k,v): set(v))
    #去除每行里重复的词
    tokenCountPairTuple = uniqueTokens.flatMap(lambda x: [(t,1) for t in x])  #[('rom', 1)]
    tokenSumPairTuple = tokenCountPairTuple.reduceByKey(lambda a,b:a+b) #[('aided', 1), ('precise', 4)]
    #计算词所出现的次数
    return (tokenSumPairTuple.map(lambda (k,v): (k,N/float(v))).cache())
    #IDF(t) = N/n(t)

idfsSmall = idfs(amazonRecToToken.union(googleRecToToken))
uniqueTokenCount = idfsSmall.count()

print 'There are %s unique tokens in the small datasets.' % uniqueTokenCount
```
### **Implement a TF-IDF function**
#### Use your `tf` function to implement a `tfidf(tokens, idfs)` function that takes a list of tokens from a document and a Python dictionary of IDF weights and returns a Python dictionary mapping individual tokens to total TF-IDF weights.
#### The steps your function should perform are:
* #### Calculate the token frequencies (TF) for `tokens`
* #### Create a Python dictionary where each token maps to the token's frequency times the token's IDF weight
#### Use your `tfidf` function to compute the weights of Amazon product record 'b000hkgj8k'. To do this, we need to extract the record for the token from the tokenized small Amazon dataset and we need to convert the IDFs for the small dataset into a Python dictionary. We can do the first part, by using a `filter()` transformation to extract the matching record and a `collect()` action to return the value to the driver. For the second part, we use the [`collectAsMap()` action](http://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.RDD.collectAsMap) to return the IDFs to the driver as a Python dictionary.


```
def tfidf(tokens, idfs):
    """ Compute TF-IDF
    Args:
        tokens (list of str): input list of tokens from tokenize
        idfs (dictionary): record to IDF value
    Returns:
        dictionary: a dictionary of records to TF-IDF values
    """
    tfs = tf(tokens)
    tfIdfDict = {token:tfs[token]*idfs[token] for token in tokens}
    return tfIdfDict
```
