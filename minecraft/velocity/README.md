# Velocity Setup

## Description

This folder contains everything to setup a Velocity Proxy on Kubernetes for a minecraft server.
Everything is designed to be setup with kustomize with the following command :

`kubectl apply -k .`

## Problem

This setup exposes Velocity proxy on port 30065 because Kubernetes can only expose ports from 30000 to 32767.
To bypass this problem we use a sidecar Docker container to redirect traffic from port 25565 to 30065 (Velocity proxy service port)

The command to do this is the following :

`docker run -d --name mc-port-forward -p 25565:25565 alpine/socat tcp-listen:25565,fork,reuseaddr tcp-connect:127.0.0.1:30065`

## Plugins

To add plugins to the Velocity proxy, just add the plugin download link to the file `plugins.txt`.

## Velocity configuration

To configure your velocity the file `velocity.toml` is here to customize the configuration.