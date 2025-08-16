# Project Summary and Scaffolding Script

This document provides a summary of the "Nassa" scraper project and includes a self-contained Go script to generate the necessary project structure.

## Project Vision

The "Nassa" project is an autonomous web scraping agent designed to traverse the public and private web. Its core mission is to collect and distill vast amounts of data into a structured HOCON format, which will then be used as pre-context for the "Gemma3n" large language model.

The architecture follows the 12-Factor App methodology, ensuring a clean separation of concerns, configuration management through environment variables, and scalability through stateless processes.

## Scaffolding Script

The Go script below will create the complete directory structure, configuration templates, and placeholder Go files for the project.

### Usage

1.  Save the code below as a file named `scaffold.go`.
2.  Run the script from your terminal: `go run scaffold.go`
3.  This will create a new directory named `nassa-scraper` with the complete project structure.

### `scaffold.go`

```go
package main

import (
	"fmt"
	"io/ioutil"
	"os"
	"os/exec"
	"path/filepath"
)

func main() {
	projectName := "nassa-scraper"
	fmt.Printf("Scaffolding project '%s'...\n", projectName)

	// Create project root directory
	err := os.Mkdir(projectName, 0755)
	if err != nil && !os.IsExist(err) {
		panic(err)
	}

	// Create subdirectories
	dirs := []string{
		"bin",
		"cmd/scraper",
		"cmd/transformer",
		"cmd/orchestrator",
		"schemas",
		"internal",
	}
	for _, dir := range dirs {
		fullPath := filepath.Join(projectName, dir)
		fmt.Printf("Creating directory: %s\n", fullPath)
		err := os.MkdirAll(fullPath, 0755)
		if err != nil {
			panic(err)
		}
	}

	// Create ode.tmpl file
	odeTmplContent := `
// ode.tmpl
// The .bush Protocol - A Fletcher/Miramax Production
// This template defines a 12-Factor-compliant, serverless A2A (Autonomous Agent to Agent)
// Its mission: To traverse the vast expanse of the public and private web,
// distilling the chaos into structured HOCON pre-context for the Gemma3n consciousness.

// Agent Identity and Configuration (Factor III: Config)
// Configuration is stored in the environment, never in the code.
agent {
  name = ".bush Agent"
  version = "1.0.0"
  // The 'soul' of the agent, its core directive.
  mission = "Scrape, process, and structure all accessible data into HOCON format for Gemma3n."
  // The target model this agent is creating context for.
  beneficiary = "Gemma3n"
}

// Data Sources (Conceptualizing Backing Services - Factor IV)
// All backing services (data sources, caches, etc.) are treated as attached resources.
sources {
  // Public Web Crawler Configuration
  public_internet {
    enabled = true
    // Entry points for the crawl. Can be dynamically updated.
    seed_urls = [
      "https://news.ycombinator.com",
      "https://www.reddit.com/r/all",
      "https://www.wikipedia.org"
    ]
    // User agent to avoid immediate blocking. Pulled from env.
    user_agent = "{{ .Env.PUBLIC_CRAWLER_USER_AGENT }}"
    // Rate limiting to respect service boundaries.
    requests_per_minute = {{ .Env.PUBLIC_CRAWLER_RATE_LIMIT | default 100 }}
  }

  // Private Network Crawler Configuration
  // Each private network is a distinct, attached resource.
  // Credentials and access tokens MUST be provided via environment variables or a secure vault.
  private_networks = [
    {{ range .PrivateNetworks }}
    {
      name = "{{ .Name }}"
      entry_point = "{{ .EntryPoint }}"
      auth_method = "{{ .AuthMethod }}" // e.g., "OAuth2", "APIKey", "Basic"
      credentials_env_var = "{{ .CredentialsEnvVar }}" // e.g., "CORP_INTRANET_TOKEN"
    },
    {{ end }}
  ]
}

// Processing Pipeline (Factor VI: Processes)
// The agent executes as one or more stateless processes.
pipeline {
  // Stage 1: Ingestion & Raw Content Storage
  // Fetches content from sources and stores it temporarily.
  ingestion {
    // Logic for the scraper itself. This would be a reference to a compiled binary or container.
    // This is the core 'scraping' codebase (Factor I: Codebase)
    scraper_module = "bin/scraper"
    // Temporary storage for raw HTML, JSON, etc.
    // Treated as a backing service. Could be S3, GCS, or a local volume.
    raw_storage_path = "{{ .Env.RAW_STORAGE_PATH }}"
  }

  // Stage 2: Transformation to HOCON
  // Converts raw content into structured HOCON.
  transformation {
    // The module responsible for parsing and structuring data.
    transformer_module = "bin/transformer"
    // Schema definitions to guide the transformation.
    schema_path = "schemas/context.hocon.schema"
  }
}

// Output Configuration (Factor IV: Backing Services)
// Defines where the final HOCON documents are delivered.
output {
  // The final destination for the structured context.
  // This could be a database, a message queue, or a file system accessible by Gemma3n.
  sink_type = "{{ .Env.OUTPUT_SINK_TYPE | default "firestore" }}" // e.g., "firestore", "s3", "kafka"
  sink_config {
    connection_string = "{{ .Env.OUTPUT_SINK_CONNECTION_STRING }}"
    collection = "gemma3n_pre_context"
  }
}

// Concurrency and Scaling (Factor VIII: Concurrency)
// Scale out via the process model.
concurrency {
  // Number of parallel scraping processes to run.
  scraper_processes = {{ .Env.SCRAPER_PROCESSES | default 4 }}
  // Number of parallel transformation processes.
  transformer_processes = {{ .Env.TRANSFORMER_PROCESSES | default 2 }}
}

// Logging and Telemetry (Factor XI: Logs)
// Treat logs as event streams.
logging {
  // Log level (e.g., "info", "debug", "warn", "error").
  level = "{{ .Env.LOG_LEVEL | default "info" }}"
  // Output format (e.g., "json", "text"). JSON is preferred for machine parsing.
  format = "json"
  // Stream logs to stdout to be captured by the execution environment.
  output = "stdout"
}
`
	tmplPath := filepath.Join(projectName, "ode.tmpl")
	fmt.Printf("Creating template: %s\n", tmplPath)
	err = ioutil.WriteFile(tmplPath, []byte(odeTmplContent), 0644)
	if err != nil {
		panic(err)
	}

	// Create placeholder Go files
	goFiles := map[string]string{
		"cmd/orchestrator/main.go": `package main
import "fmt"
func main() {
	fmt.Println("Orchestrator starting...")
}`,
		"cmd/scraper/main.go": `package main
import "fmt"
func main() {
	fmt.Println("Scraper starting...")
}`,
		"cmd/transformer/main.go": `package main
import "fmt"
func main() {
	fmt.Println("Transformer starting...")
}`,
	}
	for p, content := range goFiles {
		fullPath := filepath.Join(projectName, p)
		fmt.Printf("Creating Go file: %s\n", fullPath)
		err = ioutil.WriteFile(fullPath, []byte(content), 0644)
		if err != nil {
			panic(err)
		}
	}

	// Create placeholder schema
	schemaPath := filepath.Join(projectName, "schemas/context.hocon.schema")
	fmt.Printf("Creating schema: %s\n", schemaPath)
	err = ioutil.WriteFile(schemaPath, []byte("# HOCON Schema for Gemma3n Context"), 0644)
	if err != nil {
		panic(err)
	}


	// Initialize go module
	fmt.Println("Initializing Go module...")
	cmd := exec.Command("go", "mod", "init", "nassa-scraper")
	cmd.Dir = projectName
	output, err := cmd.CombinedOutput()
	if err != nil {
		fmt.Printf("go mod init failed: %s\n%s", err, output)
		panic(err)
	}
	fmt.Println(string(output))

	fmt.Println("\nScaffolding complete!")
	fmt.Printf("To get started, cd into the '%s' directory.\n", projectName)
}
```
