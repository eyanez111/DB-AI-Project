# 📊 Social Trend Intelligence Platform

This project is a scalable, AI-powered social media trend intelligence platform that scrapes and analyzes social data to **predict emerging trends** in real time. It leverages Kubernetes (K8s) for orchestration, Dgraph for flexible and high-performance graph data storage, and integrates custom AI models for pattern recognition and forecasting.

## 🎯 Project Goal

The goal is to provide **marketing teams, agencies, and social media strategists** with a real-time, data-driven insight engine to detect:

- Viral content before it trends
- Influencer-network propagation patterns
- Shifting sentiment across regions and topics
- Hashtag and topic correlation over time

## 🧠 Core Components

### 1. **AI + ML Engine**
- NLP for topic extraction, sentiment analysis
- Time-series models to forecast emerging trends
- Graph-based ranking of influence propagation

### 2. **Social Scrapers**
- Scrapers for platforms like X (Twitter), Reddit, TikTok (public endpoints or APIs)
- Deployed as K8s CronJobs or Jobs with concurrency control
- Output: raw JSON into event pipelines or directly into Dgraph

### 3. **Dgraph (Graph Database)**
- Stores:
  - Users, posts, hashtags, entities, and relationships
  - Edge types like mentions, replies, reposts, trends
- Benefits:
  - Flexible schema for evolving data
  - Fast traversal for influence trees and topic correlation

### 4. **Kubernetes**
- All components (scrapers, workers, frontend) run as pods
- Deployed on [Civo](https://www.civo.com/) cluster
- Load-balanced via NGINX Ingress, HTTPS via cert-manager
- Volume mounting for large-scale ingestion (e.g., Hetzner 8TB)

---

## 🏗 Architecture Overview

