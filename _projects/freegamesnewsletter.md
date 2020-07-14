---
title: "Free Games Newsletter"
excerpt: "Subscribe to be notified when a game is free for a limited time on Steam"
date: 2020-07-13T23:01:30-03:00
link: https://www.freegamesnewsletter.tech/
header:
    image: /assets/images/projects/freegamesnewsletter-site.png
    teaser: /assets/images/projects/freegamesnewsletter-teaser.png
---

I started this project after noticing that I was missing a lot of games when they became free-to-keep for a limited time on Steam. Steam does not publish on its front page when a game is free (like Epic Games does, for example), so you have to check periodically in places like Reddit or [steamdb](https://steamdb.info/sales) to avoid missing these free games.

Having this in mind, I started thinking on a way to always be informed when a game is free-to-keep in Steam. At the same time, the COVID-19 pandemic gave me a lot of free time to work on. I decided that I wanted to create something that I could show to other people and they would like to use it.

The objective was then to create something that other people could use easily (it needed to be better than just saying people "hey just remember to check this specific site every day").

# Solution

This led to the creation of the Free Games Newsletter. The concept is that people check their email frequently, and most of the time a notification is shown when you receive a new email. So it would be natural to see the newsletter almost immediately when you receive it, and **only** when you receive it.

This is important because there are times when there are no free games for a long period. Before, you would need to check Reddit or steamdb every single day to guarantee you wouldn't lose an offer. With Free Games Newsletter, you receive an email only when a game is free.

# How

## Finding the games: Crawling the Steam library
The process of finding free games is the backbone of this project. There would be no Free Games Newsletter if I couldn't get the games that are free, right?
Initially, I implemented a simple scrapping program that would crawl steamdb.info/sales looking for free games. This was very simple because they were doing the heavy lifting for me. I just needed to find games on that page that were tagged _free-to-keep_.

However, there were several problems with this implementation. First, their information is not up-to-date 100% of the time. Most times a game that was tagged _free-to-keep_ on their site was not free for several hours, even days, and new free games were not updated instantly. So there was a risk that my crawler would find games that were no longer free, and missed games that became free recently. This was bad because **sometimes a game is free for less than 24 hours**. Second, it was illegal. Their _robots.txt_ file states that the sales page should not be crawled. I couldn't release my service to the public using their platform as my source.

With this in mind, I tossed out that implementation and started over. This time, I would crawl the Steam library directly. To create a crawler that searches free-to-keep games in their library once a day, I used [Scrapy](https://scrapy.org/), which is an open-source project for creating bots to crawl web pages.

I had some difficulties with the library, starting with the "infinite scroll" feature on the Steam page. Scrapy works with static pages, it means that every information that comes after the page ended loading would not be crawled by Scrapy. I ended up solving this by finding a hidden parameter in the page that removes the infinite scroll and use the old "go to next page". Another option was to crawl directly through the API that the infinite scroll uses to fetch data, but I decided that my solution was easier to implement.

In the process of learning and implementing this solution, I found an inconsistency in Scrapy docs and reported it, leading to a fix that will benefit other people.

With this, I had a crawler that finds all free-to-keep games on Steam.

{% include figure image_path="/assets/images/projects/freegamesnewsletter-newsletter.png" alt="Image of the email subscribers receive when a game becomes free-to-keep" caption="Image of the email subscribers receive when a game becomes free-to-keep" %}

## Getting the subscribers: freegamesnewsletter.tech

A newsletter is useless without subscribers. So I needed to create a web page for my users to subscribe to the newsletter. This was made using Nodejs. I already had some familiarity with Nodejs, and it became an opportunity to train my skills with a real project.

The implementation was made using the [Express](https://expressjs.com/) framework. I learned a lot about the framework's best practices during the development process. Also, I had to take care of securing my user's inputs to avoid compromising my application. This was interesting and made me learn a lot about techniques to avoid malicious user inputs. I also had the opportunity to learn about _async/await_ as a way to fix [callback hell](https://blog.avenuecode.com/callback-hell-promises-and-async/await).

To deploy the website, I went after a domain for it, having to configure things like the DNS, and configured an Nginx server to serve the page. On top of that, the site uses SSL, guaranteeing a secure connection.

When users subscribe, they receive a confirmation email. This decision was made to avoid people registering non-existent emails, reducing costs. Also, this guarantees that only users who want to receive my emails are subscribed, reducing the odds that my emails are flagged as spam.

Finally, the subscribers are stored in a NoSQL database, which allowed me to learn more about this type of database.

{% include figure image_path="/assets/images/projects/freegamesnewsletter-double-opt-in.png" alt="Free Games Newsletter uses double opt-in to confirm subscribers" caption="Free Games Newsletter uses double opt-in to confirm subscribers" %}

## Sending the emails: microservices
Initially, both the site and the newsletter application were in charge of sending emails. The newsletter application sent the emails with the free games, and the site backend sent subscription confirmation emails. 

That is ok for a small system, as there are no scalability issues or a huge codebase to be concerned about. Nonetheless, I implemented a microservices approach to the project, for training purposes (and for fun, obviously).

I decoupled the mailing implementation from both the newsletter and the site and created a separate Python application, responsible for sending emails. I had to decide how would the other services communicate with the mailing application. The options I came with were exposing a REST API or using a message broker. I chose RabbitMQ message broker, for learning purposes.

It means that the code is now easier to maintain, and I'm ready to change my mailing provider when the user base increases.

## Deploying: Docker
With the increasing number of applications in my project, I decide to use Docker to manage its deployment. All application images are built using Dockerfiles, and the entire stack is managed using docker-compose.

This was the first time I used Docker in a real situation, and after a brief period of struggle (and some short online courses), I am now comfortable using Docker for all my projects.