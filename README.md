# Overview
This is a bootstrapping repo for AI Factory. This repo performs the following tasks:

1. Downloads secrets from Azure Key vault
2. Creates a new Azure DevOps project
3. Creates a service connection for connecting to Azure via a service principal
4. Creates a service connection for connecting to Github via Personal Access Token
5. Creates a git repo for the project
6. Creates the following 4 pipelines in the Azure DevOps project created above:
    - Infrastructure as Code pipeline
    - Training pipeline
    - Batch scoring pipeline
    - Realtime scoring pipeline

Instructions for the setup of this repo can be found on the [getting started](/getting_started.md) page
