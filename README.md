DeepDist
====

Training deep belief networks requires extensive data and computation. DeepDist accelerates the training by distributing stochastic gradient descent for data stored on HDFS / Spark via a simple Python interface.

See: [DeepDist](http://deepdist.com)

Quick start:
----

    from deepdist import DeepDist
    from gensim.models.word2vec import Word2Vec
    from pyspark import SparkContext
 
    sc = SparkContext()
    corpus = sc.textFile('enwiki').map(lambda s: s.split())
 
    def gradient(model, sentences):  # executes on workers
        syn0, syn1 = model.syn0.copy(), model.syn1.copy()
        model.train(sentences)
        return {'syn0': model.syn0 - syn01, 'syn1': model.syn1 - syn1}
 
    def descent(model, update):      # executes on master
        model.syn0 += update['syn0']
        model.syn1 += update['syn1']
 
    with DeepDist(Word2Vec(corpus.collect()) as dd:
 
        dd.train(corpus, gradient, descent)
        print dd.model.most_similar(positive=['woman', 'king'], negative=['man'])

How does it work?
----

DeepDist implements a Sandblaster-like stochastic gradient descent. It start a master model server (on port 5000). On each data node, DeepDist fetches the model from the server, and then calls gradient(). After computing the gradient for each RDD partition, gradient updates are send the the server. On the server, the master model is then updated by descent().

![Alt text](http://deepdist.com/images/deepdistdesign.png)