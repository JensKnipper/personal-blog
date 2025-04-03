---
layout: post
title: Documentation as Code with AsciiDoctor, GitLab CI, and GitLab Pages
author: jens_knipper
date: '2025-04-03 01:00:00'
description: Writing documentation should be as low-key as possible to remove the pain from the process. One way to achieve this is through Documentation as Code. Setting it up and configuring it may take some time, but once it's done, you can focus on creating the content itself.
categories: Gitlab CI, Gitlab Pages, AsciiDoc
---
As a developer I love good documentation, but writing documentation myself is often cumbersome and does not feel rewarding, especially if no one reads or uses it. The reason: documentation spread over multiple tools where no one actually knows where to find it. 

The solution is using Documentation As Code. In projects where you use GitLab anyway you can write your documentation along your code in AsciiDoc, generate it to HTML using AsciiDoctor in Gitlab CI and publish it to Gitlab Pages for everyone to read.

## GitLab Pages

To tell GitLab that you want to upload static content and publish it as a website, the [`pages` attribute](https://docs.gitlab.com/ci/yaml/#pages) is used.  
After setting this attribute's value to `true`, all the content in the `public` folder is published as Gitlab Pages.

Each time you generate the pipeline the public folder will be empty again, so you don't have to care about cleaning up old pages. Just fill it and only the latest pages generated will be uploaded.

## GitLab CI

To setup a pipeline you need to make changes to the `.gitlab-ci.yml` file located in the root of your repository. The setup is fairly simple. Just make the necessary changes, commit and push them and your page is set up. 
As described before you need to set the `pages` attribute.  
To create html pages from AsciiDoc we take the [AsciiDoctor docker image](https://github.com/asciidoctor/docker-asciidoctor). Using it makes the [AsciiDoctor CLI](https://docs.asciidoctor.org/asciidoctor/latest/cli/man1/asciidoctor/) available.  
We use it in the `scripts` part.

```
stages:
  - deploy

deploy_pages:
  image: asciidoctor/docker-asciidoctor:1.84
  pages: true
  stage: deploy
  script:
    - |
      asciidoctor \
      --attribute data-uri \
      --attribute toc=left \
      --attribute sectnums \
      --attribute reproducible \
      --attribute icons=font \
      --attribute source-highlighter=Pygments \
      --attribute experimental \
      --require asciidoctor-diagram \
      --destination-dir public \
      --source-dir src 'doc/**/*.adoc'
  rules:
    - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH && $CI_PIPELINE_SOURCE != "merge_request_event"
      changes:
        - doc/**/*
        - .gitlab-ci.yml
```

An explanation of the [attributes](https://docs.asciidoctor.org/asciidoc/latest/attributes/document-attributes-ref/) used can be found below:
* `--attribute data-uri`: graphics as data-uri; self contained file
* `--attribute toc=left`: display table of contents on left side
* `--attribute sectnums`: display section numbers
* `--attribute reproducible`: no last updated timestamp
* `--attribute icons=font`: use font based icons
* `--attribute source-highlighter=Pygments`: set syntax highlighter
* `--attribute experimental`: add UI elements for buttons and key bindings
* `--require asciidoctor-diagram`: load diagram library to use diagrams
* `--destination-dir public`: set 'public' as output folder
* `--source-dir src 'doc/**/*.adoc'`: define input files

To keep our build times as few, and as short as possible we only want to deploy new pages in case something changes on the main branch.  
We can even optimize it further and tell GitLab to only run the generation if the documentation itself changes or something in the process specification.  
These are specified in the `rules` section of the `deploy_pages` job.

## Generating pages

The process is defined to take all `.adoc` files to and generate pages from it. So let's create a file named `index.adoc` in the `doc` folder.  
We name it index, so an `index.html` page is generated which will automatically be called once you call URL of your pages.  
An example might look like this:

```
= Your documentation

This is your documentation

== Another Header

[plantuml]
----
@startuml

skin rose

state Open
state Closed

[*] --> Open
Open --> Closed
Closed --> [*]
@enduml
----

[source,java]
----
void main() {
  println("Hello World");
}
----
```

## Results

After committing and pushing your file the pipeline will run. As soon as the pipeline finished (you can watch it in Gitlab in `Build > Pipelines`) you can find the link to your pages website at `Deploy > Pages`.  
The result should look like this:
![Generation Results](/assets/img/gitlab-asciidoc-documentation-as-code-result.png)
If you want to take a closer look at the generated page, you can go to the [pages I generated](https://gitlab-asciidoc-documentation-as-code-352c33.gitlab.io/).

You can see the whole code on [GitLab](https://gitlab.com/jensknipper/gitlab-asciidoc-documentation-as-code). Feel free to copy it and adjust it to your own needs.