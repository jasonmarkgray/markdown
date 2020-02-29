# GitHub to Confluence automated workflow

This repository is the source of my GitHub-to-Confluence workflow. The files in "source" are the source Sphinx (Python doc format) topics. Whenever a pull request is closed, it triggers a GitHub action which publishes the new or changed topics to my Confluence repository.

The repo is here:

https://github.com/jasonmarkgray/markdown/

The workflow file is here:

https://github.com/jasonmarkgray/markdown/blob/master/.github/workflows/review-workflow.yml

And the Confluence space is here:

https://jasongray.atlassian.net/wiki/spaces/MARKDOWN/

You can login as a read-only user with username:ohpen password:sesame

The topics in this space will be updated whenever I merge a branch or PR to GitHub.

For the purpose of this exercise I have not taken much care to refine the configuration, but it could be extended to any extent. I am also assuming that the condition of PR = closed is sufficient to establish an informal workflow. 
