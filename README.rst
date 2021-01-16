Demo using GitHub Actions to push docs to another repo
======================================================

This is a demonstration of how to create a GitHub repo that uses GitHub
Actions to build docs with Sphinx and then push the docs to another repo that
is histed via GitHub pages. I will try to list all steps needed to get this
set up from scratch with empty repos. I am creating this as test before
migrating the SymPy repo docs build from Travis to GitHub Actions.

Setup a repo that builds with sphinx
------------------------------------

Create the main code repo:

  $ mkdir actions_docs_build_demo
  $ cd actions_docs_build_demo
  $ git init


Add a doc folder and initialise sphinx:
```console
$ mkdir doc
$ cd doc
$ sphinx-quickstart
````
I chose yes for `Separate source and build directories`.

Edit ``doc/source/index.rst`` and insert whatever test you want to apper in
the docs. Then still in the doc directory run:
```console
$ make html
```

Add the ``html/build`` directory to ``.gitignore`` and then commit everything:
```console
$ cd ..
$ echo 'doc/build' > .gitignore
$ git add -A .
$ git commit -m 'doc: initial doc setup'
```

That builds the docs and you can open ``doc/build/html/index.html`` in a
browser.

Setup a repo for the docs
-------------------------

Create the docs repo:
```console
$ mkdir actions_docs_build_demo_docs
$ cd actions_docs_build_demo_docs
$ git init
```
Copy in the docs built from the code repo:
```console
$ cp -r ../actions_docs_build_demo/doc/build/html/* .
$ git add -A .
$ git commit -m 'initial doc import'
```

Now go to GitHub, create two new empty repos and push both up.
```console
$ cd actions_doc_build_demo
$ git remote add origin https://github.com/oscarbenjamin/actions_docs_build_demo.git
$ git push -u origin master
```
