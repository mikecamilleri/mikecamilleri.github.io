---
title: "Home Platform"
date: 2016-07-29
license: cc-by-sa
---

I've suddenly come into an abundance of free time. I'm taking this opportunity to finally get started on a software project that I've been planning for some time. Originally I just wanted to write a home automation platform, and this will serve that function, but I've realized that the same basic platform could do so much beyond home automation. 

There are several well developed, fully functional, and open source home automation projects out there already. Home Assistant and openHAB are two of the more prominent ones, and they both look to be very good at what they do. Both allow consumers to easily integrate various proprietary and open devices into a user friendly platform that allows for both automated control and manual remote control of the attached devices. There are of course many proprietary hubs that work similarly. 

My goals are somewhat different. Yes, I want to create a home automation platform that integrates proprietary and homebrew devices and external data, and allows for rule-based automations. More that that though, I want to create a general purpose data and automation platform for the professional or hobbyist software developer. 

An example use case that doesn't quite fit the "home automation" category is watching Twitter or scraping a website for arbitrary information and notifying me when certain events occur. I might want to receive a text message when there is a major crime near my home or where I work. I might want to integrate with a personal fitness tracker, or a small GPS tracker attached to easily stolen devices. 

Because one of my goals isn't to make a system for the typical consumer, I'm able to forgo working on things like a GUI for device setup and instead focus on functionality.

I'm planning to write this in Python 3 for two primary reasons: (1) I really want to sharpen my Python skills. It's a language that I used in school but haven't done much with since. (2) Python has an excellent standard library and a great number of quality third party modules, including one that implements Z-Wave, a commonly used networking technology in home automation. 

The architecture is going to be very modularized to facilitate easy extension of the platform and replacement of components.

Each device or third party service will integrate with the platform via an _integration_. The first integration I plan to write will be the _REST API for Devices_ which will facilitate communication with homebrew devices. Later, I plan to add a integrations for SMS text messages, Twitter, a general purpose web scraper, and home automation protocols such as Z-Wave. 

The integrations will send messages in a standard format to and receive them from the _message bus_. Messages will represent other _events_ or _commands_. The message bus will be the core of the system. 

On the other side of the message bus will be _modules_ (not sure exactly what they will be called yet, but "modules" works for now). The first module that I will build will be a simple logging component that writes to a database. If we are collecting all of these data, we might as well store them to use in future visualizations or possibly an artificial intelligence or machine learning module. 

The most important module will be the _state manager_. The state manager will perform the functions that most people think of when they think of home automation -- reading incoming data, applying a set of rules, and taking actions. `IF Mike is home AND it is daytime THEN maintain the temperature between 68 and 72 degrees`. So far, I'm conceptualizing the state manager as operating by simply comparing actual state to desired state, both of which will be updated by incoming data. 

Other important modules will include interfaces to interact with the system including a REST API and a CLI, and a simple triggering system. The triggering system will facilitate actions that don't need to rely on the state manager. For example, `IF there is a natural disaster THEN send Mike a text message`. This sort of thing might be able to be triggered by the state manager on state change as well. 

I'm pretty excited about this project because it gives me an opportunity work with a variety of tools to build an application that will both handle home automaton functions and serve as a central hub for data from other projects.
