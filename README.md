# git-deployment
A simple guide to easy deployment using git

**Deployment**. The final stage of launching that big idea you've worked on for the past couple months. Depending on the stack in use, it can be a breeze or **a real nightmare**. This guide is to help with the latter.

### Prerequisites
In order to follow this guide comforatably you need to know a few things:
- Basic understanding of linux commands
- Git

### Guide
The ususal procedure for deploying a web application on server where continuous deployment is not involved usually involves the following:

- ssh into the server
- navigate to the app directory
- do a git pull/clone
- restart services (server etc)

Pretty easy, right? Yeah, until you have to do it a dozen times.
![Too much work](https://i.gifer.com/Rui9.gif "Too much work")

We are programmers. We are lazy. We are don't like repeating ourselves. Also, check [Joel's Spolky's blog](https://www.joelonsoftware.com/2000/08/09/the-joel-test-12-steps-to-better-code/), especially **question 2**. Let me just leave it that.

Now let's get started.

Ssh in to the server where you are hosting(plan to host your code)
```
ssh USERNAME@HOST
```

We need to initialize an empty git repo, so run the following commands:
```
mkdir -p ~/app.git
cd ~/app.git
git init --bare # no source files, only version control

```
After running the above commands you directory should appear as follows:
```
branches  config  description  HEAD  hooks  info  objects  refs
```

Now navigate into the hooks directory. This where we'll be creating our hook.  [Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) are basically scripts which are run after particular git events. If you view the directory's contents, you'll see some samples. The hook we are interested in is the **post-receive** hook, which is triggered the moment the repo successfully receives files from push.

Open the file with your favourite text editor:
```
nano post-receive
```
and the post following contents
```
#!/usr/bin/env bash

TARGET="/root/backend"
GIT_DIR="/root/backend.git"
BRANCH="staging"

while read oldrev newrev ref
do
        # only checking out the master (or whatever branch you would like to deploy)
        if [[ $ref = refs/heads/"$BRANCH" ]];
        then
                echo "Ref $ref received. Deploying ${BRANCH} branch..."
                GIT_WORK_TREE=$TARGET git checkout -f $BRANCH
                git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f
                cd $TARGET
                ## Build, restart servers
                echo "Done."
        else
                echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
        fi
done
```

The above script basically does a few things:
- It first checks the current branch, in this case, `staging`
- It checks out a new branch under a new directory, which is $TARGET in this case
- Once checkout is successful, it navigates to that directory and does the necessary build up steps

Remember to create your $TARGET directory

Save the contents and make the file executable:
```
chmod +x post-receive
```
In summary, the script waits for events from the staging branch, copies the necessary files and does the necessary setup.
The advantage of using git hooks is that you have the power of git at your disposal. So you don't have to worry about which files to replace, or which files to delete.

Finally, on your client machine, you need to add your server as a git remote. This can simply be done with the following command:
```
git remote add staging ssh://<username@<server ip>:/path/app.git

```
Now you can push with the following:
```
git push staging
```
And your changes are deployed. With **one command**. Feels good, doesn't?

![Feels good](https://myspecificcarbohydratediet.files.wordpress.com/2014/10/10157891.gif "I kinda feel good")

Since, it's a bash script, creativity is at your disposal. You can add tests, before server setup to make sure code is working as expected. I personally prefer using a CI tool to handle the tests. So I usually push to the remote with CI first to make everything works then push to the server.

Happy deployment!!
