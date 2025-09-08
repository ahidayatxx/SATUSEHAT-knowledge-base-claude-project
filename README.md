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

### Step 1: Setup Claude Desktop
1. Pastikan Tavily MCP dan Brave MCP sudah dikonfigurasi di `claude_desktop_config.json`
2. Restart Claude Desktop setelah konfigurasi
3. Verifikasi kedua MCP servers aktif

### Step 2: Create Claude Project
1. Buka Claude Desktop
2. Create New Project dengan nama "SATUSEHAT Knowledge Base"
3. Copy-paste Project Instructions di atas ke dalam project settings
4. Save dan activate project

### Step 3: Testing & Validation
Test dengan sample queries:
- **Brave Test**: "Apa itu platform SATUSEHAT secara umum?"
- **Tavily Test**: "Bagaimana implementasi Patient resource untuk rawat jalan?"
- **Mixed Test**: "Jelaskan proses authentication dan berikan contoh request"

### Step 4: Optimization
- Monitor response quality dan adjust routing jika perlu
- Update Project Instructions berdasarkan usage patterns
- Add specific URLs ke favorites untuk quick access

---

*Setup ini memberikan akses real-time ke seluruh dokumentasi SATUSEHAT tanpa maintenance overhead, dengan smart routing untuk efisiensi optimal.*

---

*Setup ini mengoptimalkan Claude Desktop untuk akses dinamis ke dokumentasi SATUSEHAT, memberikan respons yang akurat dan up-to-date tanpa perlu maintenance knowledge base manual.*