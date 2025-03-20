---
layout: post
title: 'How to structure your wiki (own/company)'
date: 2024-11-18
tags: kubernetes howto devops
subtitle: 'There are different ways to build up a a wiki'
---

https://diataxis.fr/
https://news.ycombinator.com/item?id=36287809

- kind of organization

  - contentbased/wiki way: flat with interlinks
  - hierachical design: strict top down with predefined paths and parent/child => folder structure

- wiki type

  - manual way: with a big toc
  - process description / sequence / procedure: build pages to describe specific processes with links to other wiki pages

- folder structure

  - one folder per department/team
    - development
    - operating
    - fincance
  - divide by main/subpoints
    - tools => each Tool gets one page/folder
    - product => everything to your companys product
    - customer => everything to your company customers
    - tipps and tricks => small helpful snippets
  - personal
    - !Inbox
      - home.md
      - dumps.md
      - todo.md
    - wiki
      - bookmarks
      - code/snippets
      - copy paste
      - knowledgebase
      - logs
    - projects

- for whom is the documentation

  - user => how to use it, what are api calls..
  - admin => how to install, troubleshoot..
  - architecture => how is the system constructed, why was a certain tech choosen..

- purpose

  - practical
    - tutorials (learning oriented)
    - how-to guides (problem/goal oriented)
  - theoretical
    - technical reference (understanding/explanation oriented)
    - explanation (information/reference oriented)

- devevlop documentation
  - in the code
  - the issue tracker
  - the version control
