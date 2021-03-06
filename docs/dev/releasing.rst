.. doctest-skip-all

****************************
Package Release Instructions
****************************

This document outlines the steps for releasing Photutils to PyPI. This
process currently requires admin-level access to the Photutils GitHub
repository, as it relies on the ability to commit to master directly. It
also requires a PyPI account with admin-level access for Photutils.

These instructions assume the name of the git remote for the repo is
called ``upstream``.

#. Ensure Travis-CI and any other continuous integration is passing
   for the branch you are going to release. Also, ensure that Read the
   Docs builds are passing.

#. Locally run the tests using ``tox`` to do thorough tests in isolated
   environments::

        tox -e test-alldeps -- --remote-data=any
        tox -e build_docs
        tox -e linkcheck

#. Update the ``CHANGES.rst`` file to make sure that all the changes are
   listed and update the release date, which should currently be set to
   ``unreleased``, to the current date in ``yyyy-mm-dd`` format. Then
   commit the changes::

        git add CHANGES.rst
        git commit -m'Finalizing changelog for version <X.Y.Z version>'

#. Remove any untracked files (WARNING: this will permanently remove any
   files that have not been previously committed, so make sure that you
   don't need to keep any of these files)::

        git clean -dfx

#. Make sure the source distribution doesn't inherit limited permissions
   from your default umask::

        umask 0022
        chmod -R a+Xr .

#. Update the package version number to the version you’re about to
   release by creating an annotated git tag (optionally signing with the
   ``-s`` option)::

        git tag -a <X.Y.Z version> -m'<X.Y.Z version>'
        git show <X.Y.Z version>  # show the tag
        git tag  # show all tags

#. Check out the release commit::

        git checkout <X.Y.Z version>

#. Generate the source distribution tar file by first making sure the
   `pep517 <https://pypi.org/project/pep517/>`_ package is installed and
   up to date::

        pip install pep517 --upgrade

   then creating the source distribution with::

        python -m pep517.build --source .

#. Run tests on the generated source distribution by going inside the
   ``dist`` directory, expanding the tar file, going inside the expanded
   directory, and running the tests with::

        cd dist
        tar xvfz <file>.tar.gz
        cd <file>
        tox -e test-alldeps -- --remote-data=any
        tox -e build_docs

   Optionally, install and test the source distribution in a virtual
   environment::

        <install and activate virtual environment>
        pip install -e .[all,test]
        pytest --remote-data=any

   or::

        <install and activate virtual environment>
        pip install ./<file>.tar.gz[all,test]
        cd <any-directory-outside-of-photutils-source>
        python
        >>> import photutils
        >>> photutils.__version__
        >>> photutils.test(remote_data=True)

#. Go back to the package root directory and remove the generated files
   with::

        git clean -dfx

#. Generate the source distribution and upload it to PyPI::

        python -m pep517.build --source .
        twine check dist/*
        twine upload dist/*

   Check that the entry on PyPI is correct, and that the tarfile is
   present.

#. Go back to the master branch::

    git checkout master

#. Push the released tag to the upstream repo::

        git push upstream <X.Y.Z>

#. Add a new section to ``CHANGES.rst`` for the next x.y.z version,
   with a single entry ``No changes yet``, e.g.,::

       x.y.z (unreleased)
       ------------------

       - No changes yet

    Then commit the changes and push to the upstream repo::

        git add CHANGES.rst
        git commit -m'Add version <x.y.z next_version> to the changelog'
        git push upstream master

#. Create a GitHub Release
   (https://github.com/astropy/photutils/releases) by clicking on
   "Draft a new release", select the tag of the released version, add
   a release title with the released version, and add a description
   of "See ```CHANGES.rst``` for release notes.". Then click "Publish
   release". This step will trigger an automatic update of the package
   on Zenodo (see below).

#. Close the GitHub Milestone
   (https://github.com/astropy/photutils/milestones) for the released
   version and open a new Milestone for the next release.

#. Go to Read the Docs
   (https://readthedocs.org/projects/photutils/versions/) and check that
   the "stable" docs correspond to the new released version. Deactivate
   any older released versions (i.e., uncheck "Active").

#. Check that Zenodo is updated with the released version
   (https://doi.org/10.5281/zenodo.596036). Zenodo is already configured
   to automatically update with a new published GitHub Release (see
   above).

#. After the release, the conda-forge bot (``regro-cf-autotick-bot``)
   will automatically create a pull request on
   https://github.com/conda-forge/photutils-feedstock. The ``meta.yaml``
   recipe may need to be edited with updated dependencies. Modify (if
   necessary), review, and merge the PR to create the conda-forge
   package (https://anaconda.org/conda-forge/photutils). The Astropy
   conda channel (https://anaconda.org/astropy/photutils) will
   automatically mirror the package from conda-forge.

#. Build wheels and upload them to PyPI. The
   Photutils wheels are currently built using
   https://github.com/larrybradley/photutils-wheel-forge. Once the
   wheels have been built, they are uploaded as artifacts in Azure
   Pipelines. Download the wheels from Azure Pipelines and upload them
   to PyPI::

        python get_wheels.py
        twine upload wheelhouse/*.whl
