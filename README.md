# SATUSEHAT Knowledge Base - Claude Project Setup

## Project Overview
Proyek ini mengonfigurasi Claude Desktop untuk mengakses dokumentasi platform SATUSEHAT secara dinamis menggunakan MCP (Model Context Protocol), menggantikan metode scraping tradisional.

## Prerequisites
- Claude Desktop terinstall
- Tavily MCP Server configured
- Brave MCP Server configured
- API keys untuk kedua services

---

# Implementation Guide

## 1. Implementation Steps

### Step 1: Install Claude Desktop
Lihat panduan lengkap di: [CLAUDE_DESKTOP_INSTALLATION.md](https://github.com/ahidayatxx/SATUSEHAT-knowledge-base-claude-project/blob/master/CLAUDE_DESKTOP_INSTALLATION.md)

### Step 2: Setup Claude Desktop MCP
Lihat panduan lengkap di: [DESKTOP_CONFIGURATION.md](https://github.com/ahidayatxx/SATUSEHAT-knowledge-base-claude-project/blob/master/DESKTOP_CONFIGURATION.md)

### Step 3: Create Claude Project
Lihat panduan lengkap di: [PROJECT_SETUP.md](https://github.com/ahidayatxx/SATUSEHAT-knowledge-base-claude-project/blob/master/PROJECT_SETUP.md)

### Step 4: Testing & Validation
Test dengan sample queries:
- **Brave Test**: "Apa itu platform SATUSEHAT secara umum?"
- **Tavily Test**: "Bagaimana implementasi Patient resource untuk rawat jalan?"
- **Mixed Test**: "Jelaskan proses authentication dan berikan contoh request"

### Step 5: Optimization
- Monitor response quality dan adjust routing jika perlu
- Update Project Instructions berdasarkan usage patterns
- Add specific URLs ke favorites untuk quick access

---

*Setup ini memberikan akses real-time ke seluruh dokumentasi SATUSEHAT tanpa maintenance overhead, dengan smart routing untuk efisiensi optimal.*

---

*Setup ini mengoptimalkan Claude Desktop untuk akses dinamis ke dokumentasi SATUSEHAT, memberikan respons yang akurat dan up-to-date tanpa perlu maintenance knowledge base manual.*