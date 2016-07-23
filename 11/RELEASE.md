Work in Progress for doing the 11 release
```
# Change this each release
export SAKAI_VERSION=11.0

#This should be static
export SAKAI_SNAPSHOT_VERSION=11-SNAPSHOT

git clone git@github.com:sakaiproject/sakai.git sakai-source-release
# Or  git pull --rebase  if you're in an already existing directory
cd sakai-source-release
git fetch
git checkout 11.x

# Now that you're in the 11 branch, setup the repo to release!
cd master
#First run the plugin to process and set the version
mvn versions:set -DnewVersion=${SAKAI_VERSION} -DgenerateBackupPoms=false
#Then fix up the SNAPSHOT in the properties
sed -i -e "s/${SAKAI_SNAPSHOT_VERSION}/${SAKAI_VERSION}/" pom.xml
cd ..

#Build Sakai and the packs
mvn clean install -Ppack-bin -Dmaven.test.skip=true

#Release all the needed binaries to the repo
mvn deploy -Dsakai-release=true -Dmaven.test.skip=true -DskipLocalStaging=true

#Now do the necessary commits to git with the tag if everything's completed successfully so far
git commit -a -m "Releasing Sakai ${SAKAI_VERSION}"
git tag -a ${SAKAI_VERSION} -m "Tagging Sakai version ${SAKAI_VERSION}"

cd master
mvn versions:set -DnewVersion=${SAKAI_SNAPSHOT_VERSION} -DgenerateBackupPoms=false
git commit -a -m "Switching Sakai back to ${SAKAI_SNAPSHOT_VERSION}"
```

Now if everythings okay (examine the git log, check sonatype and release the artifacts there at https://oss.sonatype.org/index.html#welcome), push it!

```
git push origin ${SAKAI_VERSION}
git push origin 11.x

```

## Deploying binaries and javadocs

Afterward you should generate the javadocs upload all of the artifacts.

You have to have an alias in .ssh/config to the sakai static release directory for the command below to work. Otherwise set one up or have something comparable.

Because this hits the server multiple times you might also want to use an app like keychain or ssh-agent.

First make the directory you'll need

`ssh sakaistatic "mkdir -p ~/public_html/release/${SAKAI_RELEASE}/artifacts"` 

Then you can copy the files over

`cd pack ; find . -name "*sakai-*" | xargs -I {} scp {} sakaistatic:~/public_html/release/${SAKAI_RELEASE}/artifacts; cd ..`

Finally run this in the top level directory to generate and aggregate the java docs

`mvn javadoc:aggregate`
And then upload them to the release directory (Make sure it's empty)

`rsync -r target/site/apidocs sakaistatic:~/public_html/release/${SAKAI_RELEASE}/`


* And that should be it! Close the Jira, check the release page for working links and send out the release notes! *

These have only been run a few times, so hopefully they work for you. Will update this next release cycle!