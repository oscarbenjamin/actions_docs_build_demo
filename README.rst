Demo using GitHub Actions to push docs to another repo
======================================================

This is a demonstration of how to create a GitHub repo that uses GitHub
Actions to build docs with Sphinx and then push the docs to another repo that
is histed via GitHub pages. I will try to list all steps needed to get this
set up from scratch with empty repos. I am creating this as test before
migrating the SymPy repo docs build from Travis to GitHub Actions.

Setup a repo that builds with sphinx
------------------------------------

Create the main code repo::

  $ mkdir actions_docs_build_demo
  $ cd actions_docs_build_demo
  $ git init


Add a doc folder and initialise sphinx::

  $ mkdir doc
  $ cd doc
  $ sphinx-quickstart

I chose yes for `Separate source and build directories`.

Edit ``doc/source/index.rst`` and insert whatever test you want to apper in
the docs. Then still in the doc directory run::

  $ make html


Add the ``html/build`` directory to ``.gitignore`` and then commit
everything::

  $ cd ..
  $ echo 'doc/build' > .gitignore
  $ git add -A .
  $ git commit -m 'doc: initial doc setup'

That builds the docs and you can open ``doc/build/html/index.html`` in a
browser.

Setup a repo for the docs
-------------------------

Create the docs repo::

  $ mkdir actions_docs_build_demo_docs
  $ cd actions_docs_build_demo_docs
  $ git init

Copy in the docs built from the code repo::

  $ cp -r ../actions_docs_build_demo/doc/build/html/* .
  $ git add -A .
  $ git commit -m 'initial doc import'

Now go to GitHub, create two new empty repos and push both up e.g.::

  $ cd actions_doc_build_demo
  $ git remote add origin https://github.com/oscarbenjamin/actions_docs_build_demo.git
  $ git push -u origin master

Have the docs build on GitHub Actions
-------------------------------------

Next we want to set something up that will build the docs on Actions as a test
of each PR and also the master branch. Actions is automatically enabled on any
repository and runs as soon as you add workflow files. Create a workflow
folder and add a workflow to build the docs::

  $ mkdir -p .github/workflows

Now create a file ``.github/workflows/builddocs.yml`` with this inside::

  name: Run tests

  # Run for each push to a PR and also when a PR is merged to master.
  on: [pull_request, push]

  jobs:

    # Docs build job:
    build-docs:
      name: Build the docs
      runs-on: ubuntu-latest

      steps:
        - uses: actions/checkout@v2
        - uses: actions/setup-python@v2
          with:
            python-version: '3.x'
        - run: pip install sphinx
        - working-directory: ./doc
          run: make html
