# Helix VCON Telco Bot

A Helix-powered application that provides natural language interface for call analysis using VCON standard.
This project helps you deploy a Helix app that interfaces with a VCON (Voice Conversation) server, allowing you to analyze call data using natural language queries like:

1. Show me calls between Frank Smith and Obi Johnson
2. Display conversations about project timelines
3. Find calls from specific dates or times

## Features
- Natural language queries for call data
- Real-time call analysis
- Structured conversation data using VCON format
- Integration with Together AI

## Prerequisites
- Docker and Docker Compose
- Go 1.19 or later
- Together AI API key
- Helix CLI

## Quick Start

1. **Install Helix**
```
cd /opt
sudo mkdir helix
sudo chown $USER:$USER helix
cd helix
curl -O https://raw.githubusercontent.com/helixml/helix/main/install.sh
chmod +x install.sh
./install.sh

```

2. **Start Helix**

```
cd /opt/helix
docker compose up
```

3. **Create VCON Server**
```
mkdir -p ~/vcon-server
cd vcon-server
```

**Create main.go**
```
nano main.go
```

```
import (
    "encoding/json"
    "log"
    "net/http"
    "time"
)

type Party struct {
    Name string `json:"name"`
    Tel  string `json:"tel"`
}

type DialogEntry struct {
    Timestamp time.Time `json:"timestamp"`
    Speaker   string    `json:"speaker"`
    Text      string    `json:"text"`
}

type VCON struct {
    UUID      string        `json:"uuid"`
    CreatedAt time.Time    `json:"created_at"`
    Subject   string       `json:"subject"`
    Parties   []Party      `json:"parties"`
    Dialog    []DialogEntry `json:"dialog"`
}

var calls = []VCON{
    {
        UUID:      "call-001",
        CreatedAt: time.Date(2024, 2, 7, 14, 30, 0, 0, time.UTC),
        Subject:   "Project Timeline Discussion",
        Parties: []Party{
            {Name: "Frank Smith", Tel: "123-456-7890"},
            {Name: "Obi Johnson", Tel: "098-765-4321"},
        },
        Dialog: []DialogEntry{
            {
                Timestamp: time.Date(2024, 2, 7, 14, 30, 0, 0, time.UTC),
                Speaker:   "Frank Smith",
                Text:     "Hi Obi, let's discuss the project timeline.",
            },
            {
                Timestamp: time.Date(2024, 2, 7, 14, 31, 0, 0, time.UTC),
                Speaker:   "Obi Johnson",
                Text:     "Sure Frank, I've reviewed the milestones.",
            },
        },
    },
}

func main() {
    http.HandleFunc("/vcon", getAllVcons)
    http.HandleFunc("/vcons/search", searchVcons)
    
    log.Printf("Starting VCON server on :8005")
    log.Fatal(http.ListenAndServe("0.0.0.0:8005", nil))
}

func getAllVcons(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("Access-Control-Allow-Origin", "*")
    json.NewEncoder(w).Encode(calls)
}

func searchVcons(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("Access-Control-Allow-Origin", "*")
    
    name := r.URL.Query().Get("name")
    tel := r.URL.Query().Get("tel")
    
    var results []VCON
    for _, call := range calls {
        for _, party := range call.Parties {
            if (name != "" && party.Name == name) || 
               (tel != "" && party.Tel == tel) {
                results = append(results, call)
                break
            }
        }
    }
    
    json.NewEncoder(w).Encode(results)
}
```
```
go run main.go
```

4. **Configure Environment**
Copy .env.example to .env and add your Together AI key.

## Usage

Access Helix UI at http://localhost:8080 and try these queries:

- "Show me calls with Frank Smith"
- "What was discussed in the project timeline?"
- "List all calls from February 7th"

## Project Structurehelix-vcon-telco/

├── .env.example
├── .gitignore
├── README.md
├── helix.yaml
└── vcon-server/
└── main.go

## Contributing
1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a new Pull Request

## License
MIT
