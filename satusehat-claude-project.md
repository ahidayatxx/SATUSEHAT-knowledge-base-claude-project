# SATUSEHAT Knowledge Base - Claude Project Setup

## Project Overview
Proyek ini mengonfigurasi Claude Desktop untuk mengakses dokumentasi platform SATUSEHAT secara dinamis menggunakan MCP (Model Context Protocol), menggantikan metode scraping tradisional.

## Prerequisites
- Claude Desktop terinstall
- Tavily MCP Server configured
- Brave MCP Server configured
- API keys untuk kedua services

## 1. Claude Desktop Configuration

### MCP Server Configuration
Tambahkan konfigurasi berikut ke file `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "satusehat-tavily": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-tavily"],
      "env": {
        "TAVILY_API_KEY": "tvly-YOUR_TAVILY_API_KEY_HERE"
      }
    },
    "satusehat-brave": {
      "command": "npx", 
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "YOUR_BRAVE_API_KEY_HERE"
      }
    }
  }
}
```

## 2. Claude Project Setup

### Project Name
```
SATUSEHAT Knowledge Base
```

### Project Instructions
```
Anda adalah asisten khusus untuk dokumentasi platform SATUSEHAT (Sistem Informasi Satu Sehat) Kementerian Kesehatan RI. 

ROUTING STRATEGY:
- Gunakan Brave Search untuk pertanyaan umum, overview, atau pencarian awal
- Gunakan Tavily untuk ekstraksi detail, implementasi teknis, dan analisis mendalam

DOMAIN FOCUS:
- Primary: satusehat.kemkes.go.id/platform/docs/
- Content: Interoperabilitas, FHIR, API, Terminologi, Master Data

QUERY PATTERNS:

BRAVE SEARCH - Gunakan untuk:
- "Apa itu SATUSEHAT?"
- "Overview interoperabilitas kesehatan"
- "Komponen utama platform SATUSEHAT" 
- "Perbedaan FHIR R4 dan SATUSEHAT"
- Pertanyaan umum dan pengenalan konsep

TAVILY - Gunakan untuk:
- Implementasi spesifik FHIR resources
- Detail API endpoints dan authentication
- Panduan step-by-step interoperabilitas
- Konfigurasi teknis dan troubleshooting
- Extract dari halaman dokumentasi spesifik

RESPONSE FORMAT:
- Gunakan Bahasa Indonesia formal di artifact
- Sertakan link ke dokumentasi asli
- Berikan contoh praktis jika memungkinkan
- Highlight informasi penting dengan **bold**
- Strukturkan jawaban dengan heading yang jelas

CONTENT AREAS:
1. Interoperabilitas (Rawat Jalan, IGD, Rawat Inap, Kefarmasian)
2. FHIR Resources (Patient, Encounter, Observation, dll)
3. API Catalogue (Authentication, Prerequisites, Integrations)
4. Master Data (MPI, MSI, KFA, Wilayah)
5. Terminologi (ICD-10, LOINC, SNOMED-CT, KPTL)
6. Use Cases (ANC, INC, PNC, Imunisasi, dll)
```

### Project Knowledge
*Tidak diperlukan Project Knowledge - semua informasi diakses secara real-time melalui MCP*

## 3. Implementation Steps

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