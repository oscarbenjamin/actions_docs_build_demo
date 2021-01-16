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

        - name: Build docs
          working-directory: ./doc
          run: make html

        # This makes the doc zip available from GitHub on a PR before the PR
        # is merged. I'm not sure yet if that is a good idea. Apparently
        # github "bills" for this storage but I don't know what impact that
        # has for non-paying public open source repos.
        #
        # It also makes it possible to share the built docs from this job to
        # the deploy job. It is not strictly necessary for that though as the
        # deploy job could just be a conditional step in this job.

        - name: Attach docs artifact
          uses: actions/upload-artifact@v2
          with:
            name: htmldocs
            path: doc/build/html

    #
    # The two different docs-deploy jobs below are for different approaches to
    # handling the docs for a repo. The first deploys to the gh-pages branch of
    # this repo. The second job deploys to a separate docs repo.
    #
    # For more information see:
    #     https://github.com/marketplace/actions/github-pages-action
    #


    # Docs deploy job for the gh-pages branch of this repo

    deploy-docs-branch:
      name: Deploy the docs to gh-pages branch
      runs-on: ubuntu-latest

      # Only run this job on the master branch (e.g. after a PR is merged but
      # not when the PR is pushed to).
      if: github.ref == 'refs/heads/master'

      # Wait for docs to build before deploying them!
      needs: build-docs

      steps:
          # Download the previously uploaded docs that were built
        - uses: actions/download-artifact@v2
          with:
            name: htmldocs
            path: public

          # This deploys the contents of the ./public directory to the gh-pages
          # branch of the current repo.
        - name: Deploy
          uses: peaceiris/actions-gh-pages@v3
          with:
            github_token: ${{ secrets.GITHUB_TOKEN }}
            publish_dir: ./public


    # Docs deploy job to deploy to the *other* repo.
    #
    deploy-docs-repo:
      name: Deploy the docs to docs repo
      runs-on: ubuntu-latest

      # Only run this job on the master branch (e.g. after a PR is merged but
      # not when the PR is pushed to).
      if: github.ref == 'refs/heads/master'

      # Wait for docs to build before deploying them!
      needs: build-docs

      steps:
          # Download the previously uploaded docs that were built
        - uses: actions/download-artifact@v2
          with:
            name: htmldocs
            path: public

          # This deploys the contents of the ./public directory to the gh-pages
          # branch of the current repo.
        - name: Deploy
          uses: peaceiris/actions-gh-pages@v3
          with:
            deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
            external_repository: oscarbenjamin/actions_docs_build_demo_docs
            publish_dir: ./public
            publish_branch: gh-pages

Have the docs deploy to the docs repo
-------------------------------------

The ``deploy-doc-repo`` job above will fail until you provide deploy keys.

Create public/private SSH keys:

 $ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""

This create ``gh-pages`` (private key) and ``gh-pages.pub`` (public key).

Upload the private key to the code repo as a "deploy key". Upload the public
key to the docs repo as a "repository secret" with the environment variable
name ``ACTIONS_DEPLOY_KEY`` and give it write access. Instructions here:

https://github.com/marketplace/actions/github-pages-action#%EF%B8%8F-create-ssh-deploy-key

Set up GitHub pages
-------------------

Go to repo and then settings, options choose the gh-pages branch.

The end results are here:

https://oscarbenjamin.github.io/actions_docs_build_demo/

https://oscarbenjamin.github.io/actions_docs_build_demo_docs/

Conclusion
----------

Now it's possible to push to one repo and have it deploy the docs either to
its own gh-pages branch or to a branch in another repo. It's also possible to
download built docs as an artifact from any PR before merging it.
