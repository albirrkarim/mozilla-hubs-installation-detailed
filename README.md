# Introduction

My journey installing mozilla hubs, im new on compleks project like this. so im confused of course. and finaly i can running mozilla hubs on my macbook air m1.
I want to share with you how to do.

This is about running mozilla hubs on locally. this is detailed version, step by step what i do.
Give me star on this repository for supporting me to always update this.

Before we start lets see the requirement.

# Requirement:

### Hardware:
- at least 8GB of RAM
- recomended using fast CPU

### Software

- Node js installed. when im install this hubs i use v16

### Knowledge
I assume you already know 

- Javascript
- React js
- Webpack dev server

I will update this soon

# Overview

![System Overview](https://user-images.githubusercontent.com/29292018/153311792-6da76dc0-db31-4936-ba8b-806690e2cc11.jpeg)

The image above i make with [figma](figma.com) you can read more description on [documentation](https://hubs.mozilla.com/docs/system-overview.html)

# Installation

## Hubs

In this [repo](https://github.com/mozilla/hubs) contains the hubs client and hubs admin (hubs/admin)

This is hubs client

<img width="1552" alt="image" src="https://user-images.githubusercontent.com/29292018/153312304-345861e6-7f9d-403c-9814-bd2814869761.png">


This is hubs admin

<img width="1552" alt="Screen Shot 2022-02-10 at 07 15 16" src="https://user-images.githubusercontent.com/29292018/153312480-948849c3-3268-44db-bac2-07e4a434cab5.png">


Run this and it will start on localhost:8080

`
git clone https://github.com/mozilla/hubs.git
cd hubs
npm ci
npm run local
`


## Reticulum

[Its](https://github.com/mozilla/reticulum) a backend server that using elixir and phoenix. 

requirement:

Installing elixir and erlang (Elixir 1.12 and erlang version 23)
You can installing those with follow [this tutorial](https://www.pluralsight.com/guides/installing-elixir-erlang-with-asdf)

**Becareful about the version of elixir and erlang.**

Installing ansible
You can use `pip` to install. take a look on this [tutorial](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#from-pip)


I will update this soon

