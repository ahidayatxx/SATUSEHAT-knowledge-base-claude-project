# Claude Desktop Configuration

## MCP Server Configuration
Tambahkan konfigurasi berikut ke file `claude_desktop_config.json`:

- [Tavily MCP](https://github.com/tavily-ai/tavily-mcp)
- [Brave MCP](https://github.com/brave/brave-search-mcp-server)

```json
{
  "mcpServers": {
    "satusehat-tavily": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-tavily"
      ],
      "env": {
        "TAVILY_API_KEY": "tvly-YOUR_TAVILY_API_KEY_HERE"
      }
    },
    "satusehat-brave": {
      "command": "npx", 
      "args": [
        "-y",
        "@modelcontextprotocol/server-brave-search"
      ],
      "env": {
        "BRAVE_API_KEY": "YOUR_BRAVE_API_KEY_HERE"
      }
    }
  }
}
```

--- 

# Petunjuk Instalasi MCP pada Claude Desktop

Berikut adalah panduan lengkap untuk mengonfigurasi **Model Context Protocol (MCP)** pada Claude Desktop:

## 1. Persiapan Awal

Pastikan Anda telah mengunduh dan menginstal **Claude Desktop** versi terbaru. Jika sudah terinstal, periksa pembaruan dengan mengklik menu Claude dan pilih "Check for Updates..."[1]

**Persyaratan Sistem:**
- Node.js (untuk menjalankan server MCP berbasis JavaScript)
- Verifikasi dengan perintah: `node --version`[1]

## 2. Mengakses File Konfigurasi

### Melalui Interface Claude Desktop:
1. Buka Claude Desktop
2. Klik **"Claude" > "Settings..."** di menu atas[2]
3. Pilih **"Developer" > "Edit Config"**[2]
4. Claude Desktop akan otomatis membuka lokasi file `claude_desktop_config.json`

### Lokasi Manual File Konfigurasi:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`[1][2]
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`[2][1]

## 3. Konfigurasi MCP Server

Buka file `claude_desktop_config.json` dengan editor teks apapun dan gunakan format berikut:

### Format Dasar Konfigurasi:

```json
{
  "mcpServers": {
    "<nama_server_mcp>": {
      "command": "<perintah_startup_server>",
      "args": [
        // Tambahkan argumen untuk perintah startup di sini
      ],
      "env": {
        // Jika diperlukan, set environment variables sebagai key-value pairs
        "<ENV_Key>": "<ENV_Value>"
      }
    }
  }
}
```

### Contoh Konfigurasi Filesystem Server:

**Untuk macOS/Linux:**
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/Users/username/Downloads"
      ]
    }
  }
}
```

**Untuk Windows:**
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx", 
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "C:\\Users\\username\\Desktop",
        "C:\\Users\\username\\Downloads"
      ]
    }
  }
}
```

### Contoh Konfigurasi PostgreSQL Server:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y", 
        "@modelcontextprotocol/server-postgres",
        "postgresql://localhost/mydb"
      ]
    }
  }
}
```

## 4. Restart Claude Desktop

Setelah menyimpan perubahan pada file `claude_desktop_config.json`:

1. **Tutup Claude Desktop sepenuhnya**
2. **Buka kembali Claude Desktop**
3. Perubahan akan diterapkan dan MCP server akan terkoneksi[2]

## 5. Verifikasi Koneksi

Setelah restart, Anda dapat memverifikasi koneksi MCP dengan cara:

- Cari **ikon palu** di bagian bawah kanan kotak input[1]
- Klik ikon tersebut untuk melihat tools yang tersedia dari MCP server yang telah dikonfigurasi[2]

## 6. Troubleshooting

### Jika Server Tidak Muncul:

1. **Restart Claude Desktop** secara menyeluruh[1]
2. **Periksa sintaks JSON** pada file `claude_desktop_config.json`[1]
3. **Pastikan path file** yang disertakan valid dan menggunakan absolute path, bukan relative path[1]
4. **Cek logs** untuk melihat mengapa server tidak terkoneksi[1]

### Lokasi Log Files:

- **macOS:** `~/Library/Logs/Claude`[1]
- **Windows:** `%APPDATA%\Claude\logs`[1]

### Perintah untuk Melihat Logs:

**macOS/Linux:**
```bash
tail -n 20 -f ~/Library/Logs/Claude/mcp*.log
```

**Windows:**
```cmd
type "%APPDATA%\Claude\logs\mcp*.log"
```

## 7. Keamanan

⚠️ **Penting:** Claude Desktop akan menjalankan perintah dalam file konfigurasi dengan izin akun pengguna Anda dan akses ke file lokal. Hanya tambahkan perintah jika Anda memahami dan mempercayai sumbernya.[1]

## Catatan Tambahan

- Ganti `username` dengan nama pengguna komputer Anda yang sebenarnya[1]
- Path yang ditentukan harus mengarah ke direktori valid yang ingin Anda akses melalui Claude[1]
- Informasi JSON yang diperlukan biasanya disertakan dalam README atau dokumentasi server MCP yang ingin Anda gunakan[2]

Dengan mengikuti langkah-langkah ini, Anda dapat successfully mengkonfigurasi MCP server pada Claude Desktop dan mulai menggunakan fitur-fitur tambahan yang disediakan oleh server tersebut.

[1](https://modelcontextprotocol.info/docs/quickstart/user/)
[2](https://en.bioerrorlog.work/entry/connect-claude-desktop-to-mcp-server)
[3](https://modelcontextprotocol.io/quickstart/user)
[4](https://www.anthropic.com/engineering/desktop-extensions)
[5](https://support.anthropic.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop)
[6](https://dev.to/suzuki0430/the-easiest-way-to-set-up-mcp-with-claude-desktop-and-docker-desktop-5o)
[7](https://mcp-framework.com/docs/server-configuration/)
[8](https://claudedesktopconfiggenerator.com)
[9](https://gofastmcp.com/integrations/claude-desktop)
[10](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)
[11](https://appwrite.io/docs/tooling/mcp/claude)
[12](https://modelcontextprotocol.io/quickstart/server)
[13](https://github.com/orgs/modelcontextprotocol/discussions/325)
[14](https://developers.make.com/mcp-server/make-cloud-mcp-server/usage-with-claude-desktop)
[15](https://docs.browsermcp.io/setup-server)
[16](https://scottspence.com/posts/configuring-mcp-tools-in-claude-code)
[17](https://thegraph.com/docs/en/ai-suite/subgraph-mcp/claude/)
[18](https://windsurf.com/university/tutorials/configuring-first-mcp-server)
[19](https://gofastmcp.com/integrations/mcp-json-configuration)
[20](https://www.jetbrains.com/help/ai-assistant/configure-an-mcp-server.html)