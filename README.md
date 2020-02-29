# GitHub to Confluence automated workflow

This repository is the source of my GitHub-to-Confluence workflow. The files in /source are the source markdown topics. Whenever a push event happens on master, it triggers a GitHub workflow action which publishes the updated or new topics to my Confluence space.

The repo is here:

https://github.com/jasonmarkgray/markdown/

The workflow file is here:

https://github.com/jasonmarkgray/markdown/blob/master/.github/workflows/review-workflow.yml

And the Confluence space is here:

https://jasongray.atlassian.net/wiki/spaces/SPHINX/pages/1478643/Webhooks

You can view the pages as an anonymous user. 

For the purpose of this exercise I have not done very much work to refine the configuration, but it could be extended very easily. I am also assuming that the condition of a Git push event is sufficient to establish an informal workflow. 

The automation is based on the [Sphinx ConfluenceBuilder](https://github.com/sphinx-contrib/confluencebuilder) with the [Publish Confluence](https://github.com/marketplace/actions/publish-confluence) GitHub workflow. I have also modified it to read Markdown files instead of Sphinx.  
