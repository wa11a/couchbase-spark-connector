# Contributing

We've decided to use Gerrit for our code review system, making it
easier for all of us to contribute with code and comments.

There are some prerequisites before you can start contributing, so be sure to start with the following:

  1. Visit [http://review.couchbase.org(http://review.couchbase.org) and "Register" for an account
  2. Review [our Contributor Licence Agreement (CLA)](http://review.couchbase.org/static/individual_agreement.html)
  3. Agree to CLA by visiting [http://review.couchbase.org/#/settings/agreements](http://review.couchbase.org/#/settings/agreements)
     Otherwise, you won't be able to push changes to Gerrit (instead getting a "`Upload denied for project`" message).
  4. If you do not receive an email, please contact us
  5. Have a look at current changes in review in the spark connector area [http://review.couchbase.org/#/q/status:open+project:couchbase-spark-connector,n,z](http://review.couchbase.org/#/q/status:open+project:couchbase-spark-connector,n,z)
  6. Join us on IRC at #libcouchbase on Freenode :-)

General information on contributing to Couchbase projects can be found on the [website](http://developer.couchbase.com/open-source-projects#how-to-contribute-code) and in the [wiki](http://www.couchbase.com/wiki/display/couchbase/Contributing+Changes).

## Preparing for Contribution
Gerrit needs a little bit of setup in the repository of the project you are planning to contribute to.

> **Tip**: [Here](https://www.mediawiki.org/wiki/Gerrit/Tutorial) is an extensive tutorial on how to work with git and
Gerrit. It includes explanations about a [`git-review`](https://github.com/openstack-infra/git-review) CLI tool that can
be used to work with Gerrit instead of the bare git commands described in the following sections.

You should already have created a Gerrit account and associated it with a SSH key, as well as having signed the Contributor Licence Agreement (CLA, see introduction).

You should also have performed a `git clone` of the project from Github, so we'll assume you're in the project's directory, `couchbase-spark-connector/`.

First you need to add a remote for Gerrit (replace `YOUR_SSH_USERNAME` with the relevant value from your ssh config):

```bash
git remote add gerrit ssh://YOUR_SSH_USERNAME@review.couchbase.org:29418/couchbase-spark-connector.git
```

You'll also need to install a commit hook script that will generate a new Gerrit Change-Id each time you make a brand new commit:

```bash
cd .git/hook
wget http://review.couchbase.com/tools/hooks/commit-msg
chmod +x commit-msg
```

Keep in mind that in Gerrit (unlike Github's Pull Requests), all iterations of a particular change **must** take place in a single commit.
This is done by using `git commit --amend` (or rebasing and squashing) each time you make alterations to your change.
The discussion takes place inside of Gerrit's UI (it will provide you the URL to the change when pushing), and the history
of the change is internally maintained by Gerrit so you can actually compare each revision despite having used --amend.

## Making the Change
The previous step is a one-time configuration of the repository. The following must be done for each new contribution.

Before you start coding, you should usually open an issue in the [Couchbase bug tracker](https://issues.couchbase.com/projects/SPARKC/)
(JIRA). That may avoid unnecessary effort, for example if the change you are planning to make is going to be
obsolete because of a modification we've already planned.

In order to test the change you'll need a running local couchbase cluster with user 'Administrator' and password 'password'.
you'll also need 2 buckets: 'travel-sample', 'default'
you'll also need a query service and a search service

you can create the cluster on docker using the following commands:


```bash
# installing couchbase on docker:
docker run -d --name db-spark-connector -p 8091-8096:8091-8096 -p 11210-11211:11210-11211 couchbase

# set the services
docker exec db-spark-connector bash -c 'curl -X POST -d services=kv,index,n1ql,fts,cbas,eventing http://Administrator:password@localhost:8091/node/controller/setupServices'

# initialize node
curl  -u Administrator:password -v -X POST http://127.0.0.1:8091/nodes/self/controller/settings \
  -d 'path=%2Fopt%2Fcouchbase%2Fvar%2Flib%2Fcouchbase%2Fdata& \
  index_path=%2Fopt%2Fcouchbase%2Fvar%2Flib%2Fcouchbase%2Fdata& \
  cbas_path=%2Fmnt%2Fd1&cbas_path=%2Fmnt%2Fd2&cbas_path=%2Fmnt%2Fd3'

# create the cluster
docker exec db-spark-connector bash -c 'curl -X POST -d name=travel-sample -d ramQuotaMB=100 -d authType=sasl  -d flushEnabled=1  -d replicaNumber=0 http://Administrator:password@localhost:8091/pools/default/buckets'
docker exec db-spark-connector bash -c 'curl -X POST -d name=default -d ramQuotaMB=100 -d authType=sasl  -d flushEnabled=1  -d replicaNumber=0 http://Administrator:password@localhost:8091/pools/default/buckets'

# setup administrator user and password
curl -u Administrator:password -v -X POST http://127.0.0.1:8091/settings/web -d password=password -d username=Administrator -d port=8091
docker exec db-spark-connector bash -c 'curl -X PUT --data "name=Administrator&roles=cluster_admin&password=password" -H "Content-Type: application/x-www-form-urlencoded" http://Administrator:password@127.0.0.1:8091/settings/rbac/users/local/Administrator'

```

This will also give the change an issue number that you can reference in the commit.

To prepare a patch for the `master` branch, start from it and preferably create a new branch for your patch:

```bash
# this will be kept local, so you can use a very simple branch name
git checkout master -b myPatch
# or a fancier branch name :-)
git checkout master -b contribs/SPARKC-XXX-myPatch
```

Make your changes and do the first commit, it will be issued a Change-Id:

```
git commit
```

Your preferred text editor should open for you to fill in the commit message. Note that we use a template for commit
messages that looks like this:

```txt
##IDEAL SIZE FOR COMMIT TITLE (50char)###########
#<TICKET-NR>: Commit Short Info, goes in Changelog

#Motivation
#----------
## UNCOMMENT THE ABOVE IF THE SECTION IS RELEVANT

##IDEAL SIZE FOR BODY (72char)#########################################
# A few lines about why this change is needed at all. Probably a bug
# that has shown up or why the enhancement, or new feature makes sense.

#Modifications
#-------------
## UNCOMMENT THE ABOVE IF THE SECTION IS RELEVANT

##IDEAL SIZE FOR BODY (72char)#########################################
# Explicit info about what has changed, more comments on the code that
# has changed and explicitly state breaking changes or things that
# impact users.

#Result
#------
## UNCOMMENT THE ABOVE IF THE SECTION IS RELEVANT

##IDEAL SIZE FOR BODY (72char)#########################################
# What’s the outcome of the change? For example if there is a
# performance optimization adding before/after numbers here make sense.
```

>**`TICKET-NR`** is the Jira issue id you have created, eg. `SPARKC-123`.
>
> If you want to verify that the Change-Id was correctly generated, look at the end of the commit message in `git log`,
> it should appear there.

Example of a very short commit message that follows this template:

```txt
SPARKC-123: Fix MadeUpClass array out of bounds

Motivation
----------
The MadeUpClass uses an array of 128 ints internally, but sometimes we
query it with an index of 256...

Modifications
-------------
Use Math.min to limit the index we query with.

Added a unit test to avoid regressions on this bug in the future.

Result
------
No more ArrayIndexOutOfBoundsException. This bug is now tested against.
```

> Notice the IDEAL SIZE lines in the template have the recommended maximum length for the section.
>
> Notice as well that in the above we uncommented the section headers and filled something in for each section.

If you want to add modifications or you come back to the change later (eg. because of discussions in Gerrit), **don't
forget to use `--amend`** (and don't modify the last lines where Gerrit metadata are appended, especially the Change-Id):

```bash
# to also edit the commit message:
git commit --amend
# if the commit message is already good:
git commit --amend --no-edit
```

Finally, to upload the change to Gerrit (as a new Changeset) or a later modification (as a new Patchset inside the
Changeset), push to the Gerrit remote's special branch:

```bash
git push gerrit HEAD:refs/for/master
```

> **Note:** As stated earlier there is a CLI tool, [`git-review`](https://github.com/openstack-infra/git-review)
> from OpenStack for dealing with Gerrit-specific commands. The above would be replaced by `git review -R`.
>
> And it generates the Change-Id, so no need to set up the commit hook manually.

Gerrit should answer with the URL to your Changeset, where you can call for reviewers (for the Connector, `Michael Nitschinger`).

To mark a changeset as ready for review (you are confident the change is complete with code and tests, and you have
executed all unit tests and preferably all integration tests), you can use the "Reply..." button, top right and give
"`Verified`" a note of +1.

Reviewers will be notified and will look at your change, replying with a "`Code-Review`" mark between -2 and +2.
If <= 0, there should be comments visible in the bottom list, starting a discussion to improve the change until a later
patchset can receive a `Code-Review +2` and be merged in.

## Troubleshooting
### "Upload denied for project"
If you get the following error while attempting to push your first change to Gerrit:
```
fatal: Upload denied for project 'couchbase-spark-connector'
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
```

Make sure that you have accepted the CLA as described in step 3. of the intro. Also please check your ssh configuration.

This applies both when using bare git (`git push gerrit HEAD:refs/for/master`) or git-review ( `git review -R`).

## Final Note
Finally, feel free to reach out to the maintainers over the forums, IRC or email ([sdk_dev@couchbase.com](mailto:sdk_dev@couchbase.com))
if you have further questions on contributing or get stuck along the way. **We love contributions and want to help you
get your change over the finish line - and you mentioned in the release notes!**