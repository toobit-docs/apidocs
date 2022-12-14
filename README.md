# API Documentation Project

## Introduction

This is the project for the official API documents . This README will introduce how this project is structured and how to work with it.


### *Recommended way to make a change by CI*

- Just need to make changes to `./source/index.html.md` in specific branch and commit, let CI complete next two steps.

### To make a change to a specific version of the documentation

1. Checkout the target version's branch, e.g. if you want to update v1 chinese documents, checkout v1_cn

2. Make changes to source/index.html.md in this branch and commit

3. Run deploy.sh to deploy the changes to the website.

4. Confirm the result on the web page, if all look good, push the changes.

### To make a change to common files (logo, style, layout)

1. Checkout master branch

2. Make the common file changes

3. Commit and push the changes

4. Checkout each version's branch, merge/cherry-pick, then run deploy.sh

## Make Changes

There are two main types of changes: appearance and content.

### Appearance Change

* Change logo: <https://github.com/lord/slate/wiki/Changing-the-Logo>
* Customize style: <https://github.com/lord/slate/wiki/Custom-Slate-Themes>

### Content Change

* Change content markdown: <https://github.com/lord/slate/wiki/Markdown-Syntax>

## Build and Deploy the API

### Set Up Slate Build and Deploy Environment Locally

<https://github.com/lord/slate/wiki/Installing-Slate>

### Publish API Documents

<https://github.com/lord/slate/wiki/Deploying-Slate>