# CXR-S SDK Overview

_Source: https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/ (Chinese, fetched 2026-05-29)._

The CXR-S SDK is the on-device development toolkit running on YodaOS-Sprite, enabling developers to create standalone applications directly on Rokid Glasses. It provides access to the data channel and establishes two-way communication with the CXR-M SDK on mobile, supporting custom protocols and command transmission. With CXR-S, developers can deeply leverage hardware resources, unlock device potential, and deliver low-latency smart experiences.

## Device Connection Management

Developers can use the Rokid CXR-S SDK to monitor the connection/disconnection status of mobile devices (Android/iOS) in real time and obtain the health status of ARTC data transmission.

## Message Communication

Developers can use the Rokid CXR-S SDK to subscribe to command messages sent from the mobile end and send structured data (Caps) or binary streams (`byte[]`) to the mobile end, enabling bidirectional communication.
