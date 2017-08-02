# Help me with...Gensim!

This notebook is meant to provide some useful how-to to help a new user approaching to the LDA model implemented in gensim library.

    # import LdaModel
    from gensim.models import LdaModel

    # useful variables for following examples.
    sample_list_of_words = ['a', 'b', 'c', 'd']
    sample_id2word = {0: 'a', 1: 'b', 2: 'c', 3: 'd'}

    # sample documents
    sample_1_documents = ['t a c c a', 'b a b f b d']
    # sample tf matrix related to sample_1_documents
    sample_1_tf_matrix = sparse.csr_matrix([[1, 2, 2, 0, 0, 0], [0, 1, 0, 3, 1, 1]])
    # array of column names
    sample_1_features = ['t', 'a', 'c', 'b', 'f', 'd']

## The Corpus data structure

Gensim models take in input an optiized implementation of a sparse matrix for streaming. The class is called Corpus. It is possible to convert a dense or sparse matrix to a Corpus.
Additional information is available at: https://radimrehurek.com/gensim/matutils.html

### Dense numpy.ndarray to Corpus

The Dense2Corpus function takes in input two parameter:
* dense, a numpy.ndarray matrix
* documents_columns, a boolean that should be True if in the dense matrix documents are represented as columns and terms are rows, False if documents are rows and terms are columns. Default is True. !! IMPORTANT


        def print_corpus(corpus):
            for i in corpus:
                print(i)

        from gensim.matutils import Dense2Corpus
        import numpy

        sample_ndarray = numpy.array([[1,2,0], [4,0,6]])
        corpus = Dense2Corpus(sample_ndarray, documents_columns=False)
        print_corpus(corpus)


Output:


    [(0, 1.0), (1, 2.0)]
    [(0, 4.0), (2, 6.0)]


### Scipy sparse matrix to Corpus

!! IMPORTANT Similarly to Dense2Corpus, pay attention to the second parameter documents_columns


    from scipy.sparse import csr_matrix
    from gensim.matutils import Sparse2Corpus

    sample_sparse_matrix = csr_matrix(sample_ndarray)
    print('Sample csr matrix')
    print(sample_sparse_matrix)
    corpus = Sparse2Corpus(sample_sparse_matrix, documents_columns=False)
    print()
    print('Transformed corpus')
    print_corpus(corpus)


Output:


    Sample csr matrix
      (0, 0)	1
      (0, 1)	2
      (1, 0)	4
      (1, 2)	6

    Transformed corpus
    [(0, 1), (1, 2)]
    [(0, 4), (2, 6)]



## Features names list to id2word

id2word is the dictionary taken as input by the LdaModel constructor and used to provide a human-friendly description of extracted topics.


    print('List of features (words)')
    print(sample_list_of_words)
    id2word = {k:v for k,v in enumerate(sample_list_of_words)}

    print('Resulting id2word')
    print(id2word)


Output:


    List of features (words)
    ['a', 'b', 'c', 'd']
    Resulting id2word
    {0: 'a', 1: 'b', 2: 'c', 3: 'd'}
    Map a new document to existent model


Conversion:

Take the id2word of the model (we use a sample one, but if you have the model: model.id2word).
Note that the id2word represents the list of features recognized by the model.


    print('id2word', sample_id2word, '\n')

    print('Transform to a word2id vector')
    word2id = {id2word[k]:k for k in sample_id2word.keys()}
    print('word2id', word2id, '\n')

    print('Consider the following new documents: ')
    print(sample_1_documents, '\n')
    print('and the corresponding tf matrix and features list: ')
    print(sample_1_tf_matrix.todense())
    print('features', sample_1_features, '\n')

    bag_of_words = [None] * sample_1_tf_matrix.shape[0]
    # for shortness
    m = sample_1_tf_matrix
    f = sample_1_features

    for doc_index in range(sample_1_tf_matrix.shape[0]):
        doc = m.getrow(doc_index)
        rows, cols = doc.nonzero()

        bag_of_words[doc_index] = [(word2id[f[col]], m[doc_index,col]) for col in cols if f[col] in word2id.keys()]

    print('Output bag of words, unknown letters (features) are ignored by the model.')
    print(bag_of_words)


Output:


    id2word {0: 'a', 1: 'b', 2: 'c', 3: 'd'}

    Transform to a word2id vector
    word2id {'d': 3, 'c': 2, 'a': 0, 'b': 1}

    Consider the following new documents:
    ['t a c c a', 'b a b f b d']

    and the corresponding tf matrix and features list:
    [[1 2 2 0 0 0]
     [0 1 0 3 1 1]]
    features ['t', 'a', 'c', 'b', 'f', 'd']

    Output bag of words, unknown letters (features) are ignored by the model.
    [[(0, 2), (2, 2)], [(0, 1), (1, 3), (3, 1)]]


Now its possible to query the model with model[bag_of_words] and retrieve the associated topics.



## LdaModel

### Load model from file

!! Models are often splitted in different files when saved on disk. To retrieve a previously saved model, it is necessary to provide only the file prefix given to the save function.

Load the model:


    filepath_with_prefix = './data/gensim/example-model'
    model = LdaModel.load(filepath_with_prefix)



### Print all topics


    num_topics_to_print = 5
    num_words_per_topic = 20

    print('For each topic a pair is printed: (topic_id, topic string representation)\n')
    for i, t in enumerate(model.show_topics(num_topics=num_topics_to_print, num_words=num_words_per_topic)):
        print('#{0}\t{1}\n'.format(i, t))


For each topic a pair is printed: (topic_id, topic string representation)

Output:


    #0	(14, '0.031*"information" + 0.012*"agent" + 0.012*"service" + 0.011*"technology" + 0.010*"system" + 0.010*"based" + 0.009*"user" + 0.009*"ontology" + 0.009*"data" + 0.008*"development" + 0.008*"support" + 0.007*"issue" + 0.007*"language" + 0.007*"negotiation" + 0.007*"multimedia" + 0.006*"knowledge" + 0.006*"tool" + 0.006*"specification" + 0.006*"concept" + 0.005*"semantic"')

    #1	(24, '0.016*"method" + 0.013*"design" + 0.010*"system" + 0.008*"computing" + 0.008*"square" + 0.007*"agent" + 0.007*"decomposition" + 0.007*"based" + 0.006*"equilibrium" + 0.006*"least" + 0.006*"fluid" + 0.006*"element" + 0.006*"implementation" + 0.005*"letter" + 0.005*"learning" + 0.005*"multiplier" + 0.005*"singular" + 0.004*"mesh" + 0.004*"service" + 0.004*"handwriting"')

    #2	(21, '0.050*"query" + 0.046*"view" + 0.013*"schema" + 0.011*"maintenance" + 0.011*"warehouse" + 0.010*"relationship" + 0.010*"materialized" + 0.010*"aggregation" + 0.009*"data" + 0.009*"defect" + 0.008*"tracing" + 0.008*"system" + 0.006*"source" + 0.006*"base" + 0.005*"composite" + 0.005*"efficient" + 0.005*"point" + 0.005*"aggregate" + 0.005*"relation" + 0.004*"business"')

    #3	(35, '0.024*"graph" + 0.021*"time" + 0.019*"bound" + 0.013*"optimal" + 0.012*"number" + 0.010*"scheme" + 0.009*"size" + 0.008*"linear" + 0.007*"show" + 0.007*"code" + 0.007*"known" + 0.007*"minimum" + 0.006*"complexity" + 0.006*"dual" + 0.006*"also" + 0.006*"space" + 0.006*"node" + 0.006*"cost" + 0.005*"phase" + 0.005*"weight"')

    #4	(2, '0.030*"human" + 0.029*"user" + 0.015*"robot" + 0.012*"interface" + 0.011*"question" + 0.010*"system" + 0.009*"interaction" + 0.009*"tool" + 0.008*"navigation" + 0.008*"information" + 0.007*"environment" + 0.006*"author" + 0.006*"device" + 0.006*"personal" + 0.006*"collection" + 0.006*"service" + 0.006*"data" + 0.006*"computer" + 0.006*"different" + 0.005*"display"')


### Print one topic by id


    topic_number = 10
    number_of_words_to_print = 20


Output:


    print(model.print_topic(topic_number, number_of_words_to_print))
    0.056*"channel" + 0.017*"decoding" + 0.016*"system" + 0.016*"error" + 0.015*"code" + 0.013*"performance" + 0.012*"capacity" + 0.011*"turbo" + 0.011*"frequency" + 0.011*"fading" + 0.010*"symbol" + 0.009*"detection" + 0.009*"rate" + 0.009*"noise" + 0.008*"gaussian" + 0.008*"bound" + 0.007*"estimation" + 0.007*"simulation" + 0.007*"block" + 0.007*"bias"



### Get and print the topic assignment for a specific document

Important: feature ids should match with id2word internal dictionary, new words should be ignored.


    # Corpus format (list of lists of tuples)

    print('Query a single document')
    documents_bow = [(7,9), (2,12), (9,3)]
    assignments = model[documents_bow]

    print('Extracted assignment:')
    print(assignments)
    print('')

    print('Query with more than one document')
    documents_bow = [[(1,2), (2,12), (4,3)], [(5,3), (2,2), (8,3)]]
    multiple_assignments = model[documents_bow]
    print('Extracted assignments:')
    for i in multiple_assignments: print(i)


Output:


    Query a single document
    Extracted assignment:
    [(5, 0.36080000000000029), (49, 0.60080000000000044)]

    Query with more than one document
    Extracted assignments:
    [(25, 0.1174058544855012), (49, 0.82926081218116554)]
    [(6, 0.29928250617927882), (49, 0.59405082715405444)]
