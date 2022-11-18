# Difference Between Hubs Cloud vs Full Self-Hosted Hubs

Correct me if im wrong, As i know the different are

## Hubs Cloud

[see](https://hubs.mozilla.com/cloud)

- Only in AWS or Digital Ocean
- Partialy open source code (don't have access to reticulum) only hubs and spoke code
- Email service easy to setup (SMTP)
- Require less technical skill

## Full Self-Hosted Hubs

- Hosting anywhere
- Full access all code
- You can fully optimize the code, storage, Directly fix all bugs
- You must setting up SMTP for email verification (i don't do this i just editing reticulum code to match my needs)
- Require more technical skill (dev ops and the program it self)

## Concern

How many maximal user can interact? you must doing simulation research.

I will share little bit of my research.

Let's define the parameter what makes server can serve hubs user interaction at the same time (concurency)

### RAM

This is the estimation.

| Action                                     | Memory (MB) |
| ------------------------------------------ | ----------- |
| 27 Online together in one room. not moving | 80          |
| 1 Share Screen                             | 50          |
| 1 Moving user                              | 30          |
| 1 User Not Moving                          | 3           |

Assume one room is 25 user multiple by 10 if everybody is moving continuously is cost 250 MB

### CPU

I haven't test this

### Network

#### Bandwith
There are two kind of bandwith, Bandwith international (overseas) and local bandwith.

#### Replication
So the user will be able to access a server that is closer. thus making the access speed faster

### Best Approach so far
Making web bot that can simulate user interaction on the room. (walking, do click, etc).

Also the bot can replicate him self. so we can simulate how many user can access and interact at the same time.