---
layout: post
title: "Planet Defense"
date: 2019-11-28
image: /assets/images/planet_defense_preview.jpg
categories:
  - featured
  - university project
links:
  - name: Demo Video
    url: https://www.youtube.com/watch?v=oTESaVKJJFk
    color: blue
---

## Gameplay Video

<iframe width="1024" height="576" src="https://www.youtube.com/embed/oTESaVKJJFk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Idea and Team

<!--excerpt.start-->

Planet Defense is a tower defense game in which the player places the towers on the surface of a planet instead of a 2d plane. The enemies (asteroids) fly from any direction towards the planet. The project was realized as part of a bachelor module in a team consisting of 3 people.

## My role

 * concept
 * paper prototype
 * balancing
 * Planet mesh generation
 * Tower Placement
 * Tower Interaction
 * Life System

<!--excerpt.end-->

## Technical Chalenges

One of the biggest technical challenges I've worked on in this project was the upgrade and sell system. My first idea was to write an Upgrade and Sell MonoBehavior. I quickly realized that I could abstract some of the functionality of both components into a super class caled TowerAction. But there was another problem, if an designer wants to add sell and upgrade action to a tower he had to add an component for each of them, resulting in quite a lot of component. My Idea was to write a TowerActionProvider Monobehavior with the coresponding SellActionProvider and UpgradeActionProvider classes inherating from TowerActionProvider. These classes provide TowerActions. Therefore an designer just has to add a script for selling and a script for upgradeing. The UI system querys for all TowerActionProviders of a GameObject and adds a Button for every TowerAction.

## Design Chalenges

One of the biggest design challenges was that opponents flew in from all directions and the player needed to keep track of them. Therefor we've added a prediction visualized by a line pointing in the direction where an asteroid is headed in. Now you could see in which direction an asteorite is headed in, but it was quite overwhelming for the test players processing all this spacial data. In retropective it would make more sense restricting ourself to an 2D plane. But than it would be just another tower defense game.

Another Challenge was the ballancing. We've used 2 different approaches for balancing, statistical and playtesting. I've supplied the data for the statistical balancing. For a tower like a projectile shooting tower its very easy to calculate the damage per second that the tower inflics by hand on paper. But if you want to calculate the damage inflicted by the lightning tower for example where shoots can jump to neighbouring enemys, you realize that this is much harder. So I have to bring in the big guns. I've written a programm that simulate an idialized enviroment of our game with the intend to caclulate an factor with which the damage per second of the lightning tower has to be multiplied.  