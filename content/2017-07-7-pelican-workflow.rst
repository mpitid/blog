
:title: Pelican workflow
:date: 2017-07-07 16:00
:tags: markup, meta

Time flies --having an audience of one does not help either. Since it's
been so long since I've used my new pelican workflow, I had to
reconstruct most of it, so I'm putting it down in this post as faster
reminder just in case (although I'm sure I'll be updating frequently
*this time* :P).

First off, a few packages:

.. code-block:: shell

  aptitude install pelican python-typogrify ghp-import

The fist two are fairly straightforward while the latter is used to
automate the management of the ``gh-pages`` branch required by
`github-pages`_.

After writing new posts or updating content test locally with something
like

.. code-block:: shell

  pelican content
  cd output && python -m pelican.server 
  firefox localhost:8000

After committing and pushing the master branch, publish the output with
the following:

.. code-block:: shell

  pelican content -o output -s publishconf.py
  ghp-import output -m "new post"

Do a final check and if everything checks out, push:

.. code-block:: shell

  git checkout gh-pages && git push origin gh-pages
  git checkout master && rm -r output

For more information consult the `github-pages`_ documentation, the
`pelican publishing`_ guide and the relevant `pelican tips`_.

.. _github-pages: https://help.github.com/categories/github-pages-basics/
.. _pelican publishing: http://docs.getpelican.com/en/stable/publish.html
.. _pelican tips: http://docs.getpelican.com/en/3.6.3/tips.html#publishing-to-github

