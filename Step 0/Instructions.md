Hi [Name],

For the first task, I want you to set up a simple development and deployment workflow.

I’ll give you:

SSH access to a fresh Ubuntu VM
Access to a GitHub repository containing a small Next.js app
The domain/subdomain to use

Your task is to:

SSH into the VM
Install Coolify on the VM. https://coolify.io/self-hosted
Connect Coolify to the GitHub repository.
Deploy the app "Demo nextJS app" (available here: https://github.com/CrazedBySerenity/demo-nextjs-app) and make it available over HTTPS.
Clone the "Demo nextJS app" locally and download the required dependancies (You might need docker desktop if you're not on linux for example)
Download Cursor and set up an account using the link I sent you.
Create a new feature branch in git
Set your new feature branch as the target branch in coolify
Confirm that changes merged into your feature branch deploy automatically.
Create a small content change to the app. Choose something small you think make sense based on the apps current state
Open a clear pull request with:
A short summary
What you tested
Before and after screenshots
Merge your own PR and confirm the updated version deploys successfully.
Write deployment documentation explaining what you did, good enough so that a future new developer could follow your instructions to set up the project without use of AI. This is the most important part! Documenting your work so that other developers can continue where you left off, approximately 30% of your time spent on Step 0 should be spent on this!
Please put the deployment documentation in THIS REPO, not the nextJS demo app

Please use AI tools or online documentation as you see fit. You are responsible for understanding and testing anything they suggest. Please remember: Do not commit or share passwords, tokens, SSH keys, or other secrets!

Please send me:

A short update when you begin
An update when the first deployment is live
The PR when it is ready for review
A clear blocker message if you are stuck for more than about 45 minutes, including what happened, what you checked, and what you tried

The task is successful when:

Coolify and the app are accessible over HTTPS
The app deploys automatically from main
The PR is focused, documented, and passes the available checks
The merged change appears on the live site
Another developer could understand the deployment from your notes
No secrets or unrelated changes are included

The goal is not speed. I’m mainly looking at how independently you work, how you debug problems, how you communicate, and the quality of your Git and deployment workflow.