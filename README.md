# weathermesh

**BEWARE** this is very much work in progress!

## A modern REST API and MQTT bridge for weather station data
**weathermesh** provides a clean, well-architected API layer for accessing weather data from WeeWX and other weather station software. It bridges the gap between traditional weather station software and modern IoT/home automation systems.

## Key Features

* RESTful API - Modern HTTP/JSON API with OpenAPI documentation
* MQTT Integration - Real-time data distribution for IoT devices and home automation
* Multi-Protocol - Access data via REST, MQTT, or WebSocket
* Aggregation Support - Query across multiple stations, compute averages, min/max values
* Alert System - Temperature thresholds, weather warnings via MQTT topics
* Authentication - JWT-based auth with API key support
* Cloud Native - Containerized microservices architecture with Docker/Podman
* Observable - Built-in Prometheus metrics and structured logging

# Use Cases

* Build modern web interfaces for weather data
* Integrate weather data with home automation (Home Assistant, OpenHAB)
* Optimize solar/heat pump systems based on weather conditions
* Create multi-station weather networks
* Develop mobile apps with real-time weather updates

# Technology Stack

* Written in Go for performance and simple deployment
* MQTT broker (Mosquitto) for real-time pub/sub
* Valkey cache for fast data access
* Traefik API gateway for routing and TLS
* Docker Compose for easy deployment

weathermesh turns your weather station into a proper data mesh node, making weather data accessible to any modern application or service.
