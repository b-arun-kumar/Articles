# Migrate from SVN to GIT

In this article we will learn how to convert an existing SVN repository into a new GIT repository while preserving the history.

We need to install the following software:

- git

- svn

- git-svn

We need a Bash shell to execute the migration steps.  
If you are using Windows, use Git Bash for a Bash shell.

# Migration Steps

1. Create GIT authors.txt file

2. Clone SVN repository as a GIT repository

3. Convert svn:ignore to .gitignore

4. Convert SVN tags and branches into GIT tags and branches

5. Push local GIT repository to a remote GIT repository

# Create GIT authors.txt file

1. List authors of the SVN repository:

   Command:

       svn log -q http://url.of.svn.repo | awk -F '|' '/^r/ {sub("^ ", "", $2); sub(" $", "", $2); print $2" = "$2" <"$2">"}' | sort -u

2. Convert SVN authors list into GIT authors list:

   > SVN author format:  
   > username = username \<username\>
   >
   > GIT author format:  
   > username = Full Name \<Email\>

   Example:

   SVN authors list:  
   `kumarar = kumarar <kumarar>`  
   `oldempl = oldempl <oldempl>`

   Converted GIT authors list:  
   `kumarar = Arun Kumar <arun.kumar@company.com>`  
   `oldempl = Old Employee <old.employee@unknown.com>`

3. Create empty authors.txt file.

4. Copy the GIT authors list into this file.

Points to note:

- Convert all authors from SVN format to GIT format.

- Do not remove any authors from the list.

- If some authors have left the organization and their names and emails are unknown, give them dummy identities like shown above.

- The authors.txt file must be UTF8 encoded and CRLF EOL.

- Do not add any extra spaces and extra lines in the authors.txt file.

# Clone SVN repository as a GIT repository

Command:

    git svn clone --stdlayout --no-metadata --authors-file="./authors.txt" http://url.of.svn.repo "NewGitRepoName"

This command converts the SVN repository at  http://url.of.svn.repo into a new GIT repository named "NewGitRepoName". This new GIT repository is created locally in the directory where this command has been executed.

Use --stdlayout option only if the SVN repository follows the standard trunk, tags and branches layout.

If the SVN repository doesn't follow the standard layout and has multiple projects in it, use the --trunk option:

    git svn clone --trunk=G1 --no-metadata --authors-file="./authors.txt" http://url.of.svn.repo "G1_Git"
    
    git svn clone --trunk=G2 --no-metadata --authors-file="./authors.txt" http://url.of.svn.repo "G2_Git"

The project at http://url.of.svn.repo/G1 will be converted into "G1_Git" repository.

The project at http://url.of.svn.repo/G2 will be converted into "G2_Git" repository.

# Convert svn:ignore to .gitignore

Enter into the converted GIT repository.

    git svn show-ignore > .gitignore
    
    git add .gitignore
    
    git commit -m 'Converted svn:ignore properties to .gitignore file'

These commands convert the svn:ignore properties into a .gitignore file and commit the file into the GIT repository.

# Convert SVN tags and branches into GIT tags and branches

Follow these steps only if tags and branches from the SVN repository were cloned:

1. Enter into the converted GIT repository.

2. View all branches:

       git branch -a

3. Convert all the SVN tags into Git tags:

       for t in $(git for-each-ref --format='%(refname:short)' refs/remotes/tags); do git tag ${t/tags\//} $t && git branch -D -r $t; done

4. Convert remote SVN branches into local GIT branches:

       for b in $(git for-each-ref --format='%(refname:short)' refs/remotes); do git branch $b refs/remotes/$b && git branch -D -r $b; done

5. Delete trunk branch as we will use the “master” branch instead:

       git branch -D origin/trunk

6. Verify branches:

       git branch -a

# Push local GIT repository to a remote GIT repository

1. Create a new empty remote GIT repository.

2. Enter into the local converted GIT repository.

3. Set remote:

   Command:

       git remote add origin https://url.of.new.remote.git.repo

4. Push master branch to remote:

       git push -u origin master

5. If tags and branches were cloned and converted, push them to remote:

       git push --all origin
       
       git push --tags origin

Migration from SVN to GIT is now complete.  
The new remote GIT repository is ready for use.

----------

# Points to consider while planning migration

1. git svn operation supports many useful commands and options. These can be utilized to customize the migration. Few useful options: -r, --ignore-paths, --include-paths.

2. When you do a git clone you get the entire history of the repo. Therefore it is recommended not to version large files(zip, tar, libraries, binaries, etc). If you must have large files in your repository, then it is advisable to use Git LFS.

3. SVN Externals aren't supported by git svn and therefore they aren't migrated from the SVN repo to the GIT repo. If your SVN repo has Externals, you can setup Submodules in GIT. Submodules is GIT's equivalent to SVN's Externals.

4. Line endings are handled differently in SVN and GIT. By default, SVN doesn't pay any attention to the type of EOL markers used in your files. In GIT, core.autocrlf setting and .gitattributes file influence the EOL markers.
