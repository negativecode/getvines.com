---
layout: docs
title: Vines Cloud - Overview
---
# Vines Cloud Overview

The Vines Cloud service includes a real-time XMPP messaging system as well as a RESTful API for storage. Several apps can be hosted under a single Vines Cloud account, each with their own isolated message channels and database. This is particularly useful for running testing and production environments for a single app.

## Publish/Subscribe XMPP Channels

Apps connect to the Vines Cloud XMPP server to exchange real-time messages with each other over pubsub channels.  For example, a game might create a channel called <em>game-1</em> to which the app subscribes in order to exchange moves and scores between two players.

This is a bit different than mobile push messages available for iOS and Android devices.  While those services are designed for relatively infrequent alerts and notifications, Vines Cloud delivers your messages instantly, within a few milliseconds of sending them.  Vines is designed for real-time interaction between devices.

We'll have a quick start guide ready for the private beta. Stay tuned!

## RESTful Database API

The database storage service in Vines Cloud allows apps to save JSON data to the cloud without worrying about configuring complex server software. All database features (create, update, delete, and query) are exposed with a RESTful web service API.

Check out the [REST API developer guide](/docs/rest) for more details.

<p>
  <a id="register" href="https://secure.getvines.com/beta">Beta sign up</a>
</p>
