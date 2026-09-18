# Spatial Discovery Pod — Interactive Carnot Engine Prototype

A lightweight, zero-headset spatial learning workstation prototype developed for **Google Gemini: Fund My Crazy 2026**.

## Overview
Traditional Class 11/12 NCERT thermodynamics often relies on static 2D diagrams and rote formula memorization. This prototype demonstrates how low-cost optical computer vision can turn abstract physics into tactile, spatial manipulation without expensive VR headsets.

## Key Features
- **3D Thermal Apparatus:** Interactive Carnot cylinder with an insulated barrel, frictionless piston, conducting base, and dynamic ideal gas particles.
- **Three Reversible Stations:** Move the cylinder across the Heat Source ($T_1$), Insulating Stand, and Heat Sink ($T_2$).
- **Live P-V Indicator Diagram:** Synchronous 2D indicator diagram tracing the four thermodynamic stages ($A \to B \to C \to D \to A$) in real time.
- **Touchless Hand Gestures:** Driven entirely in the browser via **Google MediaPipe Hands** (lateral movement shifts thermal contact; pinch gesture compresses/expands volume).

## Tech Stack
- **Three.js** (WebGL 3D Rendering)
- **Google MediaPipe Hands** (Edge Computer Vision Hand Tracking)
- **HTML5 Canvas** (Dynamic Real-Time Graphing)
- **Gemini Architecture** (Curriculum Compilation & Semantic Voice Layer Concept)

## Live Demo
Run directly in any modern browser with webcam access.
