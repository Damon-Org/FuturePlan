# Damon Music futures

## Table of Contents

 - [Short](#short)
 - [Audio Server](#audio-server)
    - [Current Implementation](#current-implementation)
    - [The Problems](#the-problems)
    - [Possible solution](#possible-solution)
    - [Future](#future)

## Short

 - Create our own audio server implementation
 - Have a "central" point where services "announce" themselves
 - Create a Chrome Extension that can forward tracks to the bot
 - Create a mobile app that hooks into your voice assistant
 - Web interface with integrated search tools
 - Secure Certificate Distribution

****

## Audio Server

The end goal is to have our own audio server implementation, ideally something compiled to native machine instructions that does not require a VM (dotnet, Java, etc..)

### Current Implementation

We're using [LavaLink](https://github.com/freyacodes/Lavalink) for our audioservers, although not perfect it offers a lot of customisation and ways to bypass YouTube's ratelimiting. Eventually a server will get its entire IPv6 address space ratelimited and we have to destroy the server and get a new address space by creating a new VPS with our provider, no way of getting around this.

### The Problems

Most of this deployment is automated, except for that the music bot can in no way possible know when a new audio server is available, we would have to create a way of "announcing" our audio servers to the network as well as "retire" ratelimited audio servers.

### Possible solution

The audio servers already run in a dockerized environment, we could make a wrapping service that reads the statistics of the audio server through a shared network, that way it can announce this new audio server's IP and password and eventually retire the VPS once the IPv6 address space is exhausted.

### Future

Later on we can implement our own audio server that integrates the above mentioned solution in one package.

****

## Socket Server

This server will handle communication between all services, it needs to be robust and fast. Crashes can not be tolerated neither can this one be slow, ideally multithreaded and decentralized.

### Current Environment

A websocket server exists but it has certain flaws making it inneficient, prone to failure and so on.

This server is hosted on a domestic IP which changes randomly over time, when this server changes address, services connected to this address will silently fail and think they're still connected.

There is a ping/pong system in place that should prevent this behaviour but has not been implemented yet to surrounding services.

If a message is directed to multiple receivers these message need to be manually collected by the original service that sent out this message.

It uses a secure websocket server that relies on Letsencrypt certificates, these have to be manually generated as of now.

### Ideal Solution

Having a TCP socket server that is fast and uses multiprocessing, this is by design awful to work with. How do you keep track of connected clients, how would you send a message to clients that are connected on a different thread than the current on receiving or sending the message, etc...
NodeJS has a [clustering](https://nodejs.org/api/cluster.html) module which has a build in way of communication between child processes, as an alternative we can use Redis to make this communication more efficient and less reliant on this one way of communication. We can still use clustering to use one shared socket and make efficient use of a multicore environment.
This server should also be very secure as it's a centralized attack vector.

****

## Certificate Distribution

Currently I have to manually generate certificates and move these into active services and restore these, this is really annoying and recurring work that can be automated.

I made a proof of concept that can automatically refresh certificates through Digital Ocean, there only needs to be a way to securely get these certificates from the machine and put them on other services...

### Methods

Services should be able to get certificates from this central server is several ways, through streaming it over a secure socket or downloading it to local disk.
