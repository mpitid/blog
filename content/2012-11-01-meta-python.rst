
:title: meta-python
:date: 2012-11-01
:tags: python, meta-programming


One of the great things about Python is the meta-programming facilities.
I quite enjoy abusing language features that I like, and this post
describes two such examples I've grown fond of. While their place in
production code might be questionable, I have found them quite handy in
various quick scripts.

I will present two examples, both of which rely on decorators
[#decorators]_. The first example provides a way to automatically
`memoize`_ the results of a function. Assuming the function has no
side-effects, calling it with the same argument list should produce the
same result, thus saving computation time at the expense of space. Such
a decorator can be defined as follows:


.. code-block:: python

  class Memoized(object):
      def __init__(self, function):
          self.function = function
          self.mem = {}
      def __call__(self, *args, **kwargs):
          key = args, tuple(kwargs.iteritems())
          if key in self.mem:
              return self.mem[key]
          value = self.function(*args, **kwargs)
          self.mem[key] = value
          return value

Note that the above code never deletes stored values, which can lead to
long running programs running out of memory.

Now it is very simple to use memoization without modifying a function,
by merely adding the decorator before its definition (even if the source
code is not available, any function object can be wrapped in this class
at runtime). The following example converts a function which counts the
number of times a word appears in a document to the memoized equivalent:

.. code-block:: python

  @Memoized
  def occurences(word, words):
      return sum(w == word for w in words)

The second example stems from my fascination with the Python ``yield``
statement. Quite often I find myself writing small functions which need
to return a materialised sequence (e.g. a list or dictionary), but not
small enough to nicely fit into say, a list comprehension. To avoid the
tedious boilerplate of initializing the sequence, appending elements to
it, and then returning the result, I've resorted to the following
scheme: I use a composition decorator, which takes the type of sequence
as argument, and applies it to the result of a generator function.

The end result, slightly generalised, is the following:

.. code-block:: python

  def compose(f1, f2):
      assert callable(f1) and callable(f2)
      def composition(*args, **kwargs):
          return f1(f2(*args, **kwargs))
      return composition
  
  class Compose(object):
      def __init__(self, *functions):
          self.functions = functions
      def __call__(self, function):
          return reduce(compose, self.functions + (function,))

And here is how it could be used on a higher order function which
applies arbitrary user functions to dictionary keys and/or values:

.. code-block:: python

  def identity(x): return x
  
  # Original
  def dmap1(d, keys=identity, values=identity):
      return dict((keys(key), values(value)) for key, value in d.iteritems())
  
  # Amended
  @Compose(dict)
  def dmap2(d, keys=identity, values=identity):
      for key, value in d.iteritems():
          yield keys(key), values(value)

The previous example was hardly an improvement in terms of code size,
but some more elaborate examples are hopefully more motivating:

.. code-block:: python

  @Compose(tuple)
  def take(n, items):
      assert n >= 0
      for i, item in enumerate(items):
          if i >= n:
              break
          yield item
  
  @Compose(set)
  def words_of(dictionary):
      """Return the set of words in a hunspell dictionary file."""
      with open(dictionary) as f:
          f.readline() # Skip word count
          for line in f:
              yield line.split()[0].split('/')[0].lower()

The composition decorator is quite general, and can be applied to
different scenarios as well. In addition, multiple decorators can be
chained together. For example, to produce a *sorted* sequence of
*unique* tokens in a collection of documents, one could write the
following:

.. code-block:: python

  import re

  def tokenize(data):
      for token in re.split('[\s:;!?-]+', data):
          token = token.lower().strip("""()[]{},."'""")
          if token:
              yield token
  
  @Compose(sorted, set)
  def all_words(documents):
      """Retrieve the set of unique words in a document, after tokenization."""
      for document in documents.itervalues():
          for word in tokenize(document):
              yield word

Using decorators in this manner has proven quite helpful in prototypes
and rapidly changing code.

.. _memoize: https://en.wikipedia.org/wiki/Memoization

.. [#decorators] For a nice introduction to decorators I recommend the series of tutorials by Bruce Eckel (part `1`__, `2`__, and `3`__).

__ http://www.artima.com/weblogs/viewpost.jsp?thread=240808
__ http://www.artima.com/weblogs/viewpost.jsp?thread=240845
__ http://www.artima.com/weblogs/viewpost.jsp?thread=241209

