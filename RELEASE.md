<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

# Release

**PyDolphinScheduler** office release is in [ASF Distribution Directory](#release-to-apache-distribution),
but we also have a [PyPi](#release-to-pypi) repository for Python package distribution for convenience.

## Prepare

* Remove `sphinx-multiversion` dependency in `setup.py`, we still can not fix this issue
  [Distribute tarball and wheel error with direct dependency](https://github.com/apache/dolphinscheduler/issues/12238)
* Change `version_ext` about the dolphinscheduler version current support, the syntax is respect [pep-044](https://peps.python.org/pep-0440/#version-specifiers)
* Run all test locally, `tox -e local-ci && tox -e local-integrate-test`, after you start dolphinscheduler to
  pass `local-integrate-test`

## Build and Sign Package

We use [build](https://pypi.org/project/build/) to build package, and [twine](https://pypi.org/project/twine/) to
upload package to PyPi. You could first install and upgrade them by:

```shell
# Install or upgrade dependencies
python3 -m pip install --upgrade pip build twine

# Change version
# For macOS
sed -i '' "s/__version__ = \".*\"/__version__ = \"${VERSION}\"/" src/pydolphinscheduler/__init__.py
# For Linux
sed -i "s/__version__ = \".*\"/__version__ = \"${VERSION}\"/" src/pydolphinscheduler/__init__.py
git commit -am "Release v${VERSION}"

# Add Tag
VERSION=<VERSION>  # The version of the package you want to release, e.g. 1.2.3
REMOTE=<REMOTE>  # The git remote name, we usually use `origin` or `remote`
git tag -a "${VERSION}" -m "Release v${VERSION}"
git push "${REMOTE}" --tags

# Build and sign according to the Apache requirements
python setup.py clean && python setup.py asdist
```

## Create Draft Release

Go to [GitHub release page](https://github.com/apache/dolphinscheduler-sdk-python/releases) and create a release
based to the specific tag, and set it as pre-release.

## Release to Apache Distribution

### To Apache SVN

```shell
svn co https://dist.apache.org/repos/dist/dev/dolphinscheduler/ release/dolphinscheduler
mkdir -p release/dolphinscheduler/python/"${VERSION}"
cp dolphinscheduler-python-src-"${VERSION}"* release/dolphinscheduler/python/"${VERSION}"

cd release/dolphinscheduler && svn add python && svn commit python -m "Release Apache DolphinScheduler-SDK-Python version ${VERSION}"
```

### Vote Mail

Create a new vote in dev@dolphinscheduler.apache.org

```text
TITLE: [VOTE] Release Apache DolphinScheduler SDK Python <VERSION>

BODY:

Hello DolphinScheduler Community,

This is a call for vote to release Apache DolphinScheduler SDK Python version <VERSION>

Release notes: https://github.com/apache/dolphinscheduler-sdk-python/releases/tag/<VERSION>

The release candidates: https://dist.apache.org/repos/dist/dev/dolphinscheduler/python/<VERSION>/

Git tag for the release: https://github.com/apache/dolphinscheduler-sdk-python/tree/<VERSION>

Release Commit ID: https://github.com/apache/dolphinscheduler-sdk-python/commit/<commit-SHA>

Keys to verify the Release Candidate: https://dist.apache.org/repos/dist/dev/dolphinscheduler/KEYS

The vote will be open for at least 72 hours or until necessary number of votes are reached.

Please vote accordingly:

[ ] +1 approve
[ ] +0 no opinion
[ ] -1 disapprove with the reason

Checklist for reference:

[ ] Download links are valid.
[ ] Checksums and PGP signatures are valid.
[ ] Source code artifacts have correct names matching the current release.
[ ] LICENSE and NOTICE files are correct for each DolphinScheduler repo.
[ ] All files have license headers if necessary.
[ ] No compiled archives bundled in source archive.
```

### Check Release Candidate

- Artifacts in staging repository are published with `.asc`, and `sha512` files.
- `LICENSE` and `NOTICE` are in source codes and distribution package.
- Check `sha512` and `gpg` signature with following command:

```shell
VERSION=<VERSION>  # The version of the package you want to release, e.g. 1.2.3
# Download source code
curl https://dist.apache.org/repos/dist/dev/dolphinscheduler/python/"${VERSION}"/dolphinscheduler-python-src-"${VERSION}".tar.gz > dolphinscheduler-python-src-"${VERSION}".tar.gz
curl https://dist.apache.org/repos/dist/dev/dolphinscheduler/python/"${VERSION}"/dolphinscheduler-python-src-"${VERSION}".tar.gz.asc > dolphinscheduler-python-src-"${VERSION}".tar.gz.asc
curl https://dist.apache.org/repos/dist/dev/dolphinscheduler/python/"${VERSION}"/dolphinscheduler-python-src-"${VERSION}".tar.gz.sha512 > dolphinscheduler-python-src-"${VERSION}".tar.gz.sha512

# Verify sha512
sha512sum --check dolphinscheduler-python-src-"${VERSION}".tar.gz.sha512
# Verify gpg signature
curl https://dist.apache.org/repos/dist/release/dolphinscheduler/KEYS > KEYS
gpg --import KEYS
gpg --verify dolphinscheduler-python-src-"${VERSION}".tar.gz.asc
```

Vote result should follow these:

- PMC vote is +1 binding, all others is +1 no binding.
- Within 72 hours, you get at least 3 (+1 binding), and have more +1 than -1. Vote pass.
- **Send the closing vote mail to announce the result**. When count the binding and no binding votes, please list the names of voters. An example like this:

    ```text
    [RESULT][VOTE] Release Apache SkyWalking Python version $VERSION
    
    72+ hours passed, we’ve got ($NUMBER) +1 bindings (and ... +1 non-bindings):
    
    (list names)
    +1 bindings:
    xxx
    ...
    
    +1 non-bindings:
    xxx
    ...
     
    Thank you for voting, I’ll continue the release process.
    ```

### Publish

- Move source codes tar balls and distributions to https://dist.apache.org/repos/dist/release/dolphinscheduler/, **you can do this only if you are a PMC member**.

    ```shell
    # Move from dev to release
    svn mv https://dist.apache.org/repos/dist/dev/dolphinscheduler/python/"${VERSION}" https://dist.apache.org/repos/dist/release/dolphinscheduler/python/"${VERSION}"
    # Delete the old version
    svn delete -m "remove old release Python SDK" https://dist.apache.org/repos/dist/release/dolphinscheduler/python/<PREVIOUS-RELEASE-VERSION>
    ```

- Update [GitHub release page](https://github.com/apache/dolphinscheduler-sdk-python/releases) and set it as formal release.
- Send ANNOUNCE email to `dev@dolphinscheduler.apache.org` and `announce@apache.org`.

    ```text
    Subject: [ANNOUNCE] Apache DolphinScheduler SDK Python $VERSION Released

    Content:

    Hi Community,

    We are glad to announce the release of Apache DolphinScheduler SDK Python $VERSION.

    Apache DolphinScheduler SDK Python is an API for Apache DolphinScheduler which allow you definition your workflow by Python code, aka workflow-as-codes.
  
    DolphinScheduler is a distributed and easy-to-extend visual workflow scheduler system,
    dedicated to solving the complex task dependencies in data processing, making the scheduler system out of the box for data processing.

    Download Links: https://dolphinscheduler.apache.org/#/en-us/download

    Release Notes: https://github.com/apache/dolphinscheduler-sdk-python/releases/tag/$VERSION

    DolphinScheduler Resources:
    - Issue: https://github.com/apache/dolphinscheduler-sdk-python/issues/
    - Mailing list: dev@dolphinscheduler.apache.org
    - Website: https://dolphinscheduler.apache.org
    - Documents: https://dolphinscheduler.apache.org/python/$VERSION
    ```

## Release to PyPi

[PyPI](https://pypi.org), Python Package Index, is a repository of software for the Python programming language.

### Release to TestPyPi

TestPyPi is a test environment of PyPi, you could release to it to test whether the package is work or not.

1. Create an account in [TestPyPi](https://test.pypi.org/account/register/).
2. Clean unrelated files in `dist` directory, and build package `python3 setup.py clean`.
3. Build package `python3 -m build`, and you will see two new files in `dist` directory, with extension
   `.tar.gz` and `.whl`.
4. Upload to TestPyPi `python3 -m twine upload --repository testpypi dist/*`.
5. Check the package in [TestPyPi](https://test.pypi.org/project/apache-dolphinscheduler/) and install it
   by `python3 -m pip install --index-url https://test.pypi.org/simple/ --no-deps apache-dolphinscheduler` to
   test whether it is work or not.

### Release to PyPi

PyPi is the official repository of Python packages, it is highly recommended [releasing package to TestPyPi](#release-to-testpypi)
first to test whether the package is correct.

1. Create an account in [PyPI](https://pypi.org/account/register/).
2. Clean unrelated files in `dist` directory, and build package `python3 setup.py clean`.
3. Build package `python3 -m build`, and you will see two new files in `dist` directory, with extension
   `.tar.gz` and `.whl`.
4. Upload to TestPyPi `python3 -m twine upload dist/*`.
5. Check the package in [PyPi](https://pypi.org/project/apache-dolphinscheduler/) and install it
   by `python3 -m pip install apache-dolphinscheduler` to install it.
