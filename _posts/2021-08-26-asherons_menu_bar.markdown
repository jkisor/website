---
layout: post
title:  "Displaying Game Server Population in MacOS menu bar"
date:   2021-08-26 00:00:00 -0400
---

# Displaying Game Server Population in MacOS menu bar

My goal with this project was to know, at a glance, how many people are connect to the game server and to look at my character stats easily from outside the game.

![image](/assets/ac_xbar_0.png)

See the code on Github: [here](https://github.com/jkisor/ac_pop_xbar/blob/main/ac_pop.5m.rb)

## About the Game

[Asheron's Call](https://en.wikipedia.org/wiki/Asheron%27s_Call) is an MMORPG that was released in 1999. It was a game with a vast world to explore alongside potentially hundreds or thousands of other adventurers. I played off and on over the years for a nostalgic trip. Unfortunately the official servers were shutdown in 2017, however, the community has worked hard to emulate the server code. The private server I visit these days ([Levistras](https://acportalstorm.com/)) is running [ACEmulator](https://github.com/ACEmulator/ACE).

## Obtaining server data

One of the many plugins players use to enhance the clientside experience is called [TreeStats](https://treestats.net/). This plugin automatically submits character and server information to TreeStats on login so that you can review information about the server and its characters. This will be the source of our data. See [TreeStats API](https://treestats.net/api).

For server population data: `https://treestats.net/player_counts-latest.json`

For character data: `https://treestats.net/Levistras/Loaf.json`

## Showing data in the MacOS menu bar

To display the data in the menu bar, we will use [xbar](https://github.com/matryer/xbar). Once installed, you can write a script (in any supported language -- I choose ruby) and it's standard output is shown in the menu bar. It has some of its own rules for links or submenus that are neat.

I've written a ruby script and placed it into xbar's plugins folder. By filename convention it's configured to refresh every 5 minutes. This script fetches the data from the json endpoints, prepares strings to display, and prints them in standard out.

![image](/assets/ac_xbar_2.png)

The menubar item shows an emoji and the server population. When clicked, it lists my characters and their levels. When a character is clicked, It opens a browser to the treestats page to see the details of the character.

Now while coding I can keep an eye on the game world and quickly review my game characters.

See the code on Github: [here](https://github.com/jkisor/ac_pop_xbar/blob/main/ac_pop.5m.rb)


