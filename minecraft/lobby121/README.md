# Lobby Setup

## Description

This folder contains everything to setup a default lobby on Kubernetes for a minecraft server.
Everything is designed to be setup with kustomize with the following command :

`kubectl apply -k .`

## Plugins

To add plugins to the Velocity proxy, just add the plugin download link to the file `plugins.txt`.

## Environment configuration

To configure the server you can modify the file `.lobby-env` with the values needed.
For more informations see [the official documentation](https://docker-minecraft-server.readthedocs.io/en/latest/variables/)