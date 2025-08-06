# Nuxt Gin Go Module

This module is a powerful integration package designed specifically for the <mcurl name="Nuxt Gin Starter" url="https://github.com/RapboyGao/nuxt-gin-starter.git"></mcurl> project, providing core functionality support. It seamlessly combines the Nuxt frontend framework with the Gin backend framework, offering a convenient solution for full-stack development.

## Project Introduction

This module is a core component of the Nuxt Gin Starter project, providing key functionalities such as configuration management, Gin mode settings, and Vue service rendering. Through this module, you can quickly set up a separated frontend-backend application architecture while balancing development efficiency and runtime performance.

## Technology Stack

- **Backend**: Go 1.24.3, Gin 1.10.1
- **Frontend**: Nuxt.js (via reverse proxy or static file service)
- **Dependency Management**: Go Modules
- **Other Tools**: github.com/json-iterator/go (high-performance JSON processing), github.com/arduino/go-paths-helper (file path operations)

## Project Structure

```plaintext
├── .gitignore
├── .vscode/
│   └── settings.json
├── LICENSE
├── README.zh-CN.md
├── ServeVue.go       # Vue service rendering
├── getConfig.go      # Configuration management
├── getGinMode.go     # Gin mode configuration
├── go.mod            # Go dependency management
└── utils/            # Utility functions
    ├── Framework.Directory.go
    ├── Framework.Excelize.go
    ├── Framework.IPAddress.go
    ├── Framework.MapStructure.go
    ├── Framework.MultipleFiles.go
    ├── Framework.Percentage.go
    └── Framework.Regex.go
```

## Core Features

### 1. Configuration Management (getConfig.go)

Provides a unified configuration loading and management mechanism, reading configuration from `server.config.json` file and supporting automatic parsing into structs.

```go:/Users/gaoxiaoyi/Documents/nuxt-gin-go-module/getConfig.go
// Configuration struct example
type Config struct {
    GinPort  int    `json:"ginPort"`  // Gin server port
    NuxtPort int    `json:"nuxtPort"` // Nuxt application port
    BaseUrl  string `json:"baseUrl"`  // Application base URL
}

// Global configuration instance
var GetConfig Config = func() Config {
    config := Config{}
    config.Acquire()
    return config
}()
```

### 2. Gin Mode Setting (getGinMode.go)

Automatically switches Gin's running mode based on project directory structure:

- When `node_modules` directory exists, enables development mode (DebugMode)
- When `node_modules` directory does not exist, enables production mode (ReleaseMode)

```go:/Users/gaoxiaoyi/Documents/nuxt-gin-go-module/getGinMode.go
func ConfigureGinMode() {
    path := paths.New("node_modules")
    path.ToAbs()

    if path.IsDir() {
        fmt.Println("/node_modules found: using gin.DebugMode")
    } else {
        fmt.Println("/node_modules not found: using gin.ReleaseMode")
        gin.SetMode(gin.ReleaseMode)
    }
}
```

### 3. Vue Service Rendering (ServeVue.go)

Provides different Vue service methods based on Gin running mode:

- Development mode: Connect to Nuxt development server via reverse proxy
- Production mode: Directly serve static files

```go:/Users/gaoxiaoyi/Documents/nuxt-gin-go-module/ServeVue.go
func ServeVue(engine *gin.Engine) {
    // Redirect root path to application base URL
    engine.GET("", func(ctx *gin.Context) {
        ctx.Redirect(http.StatusPermanentRedirect, GetConfig.BaseUrl)
    })

    if gin.Mode() == gin.ReleaseMode {
        ServeVueProduction(engine)
    } else {
        ServeVueDevelopment(engine)
    }
}
```

### 4. Utility Functions Library (utils/)

Provides various practical utility functions, including directory operations, Excel processing, IP address handling, data structure conversion, etc.

## Usage

### Install Dependencies

```bash
cd /Users/gaoxiaoyi/Documents/nuxt-gin-go-module
# Set Go proxy if needed
export GOPROXY="https://goproxy.io,direct"
# Install dependencies
go mod tidy
```

### Integrate into Project

```go
// Import the module
import (
    "github.com/RapboyGao/nuxt-gin-go-module"
    "github.com/gin-gonic/gin"
)

func main() {
    // Configure Gin mode
    nuxtGin.ConfigureGinMode()

    // Create Gin engine
    engine := gin.Default()

    // Configure Vue service
    nuxtGin.ServeVue(engine)

    // Start the service
    engine.Run(fmt.Sprintf(":%d", nuxtGin.GetConfig.GinPort))
}
```

## Notes

1. Ensure to create a `server.config.json` file in the project root directory with necessary parameters
2. In development mode, ensure the Nuxt development server is running
3. In production mode, ensure Vue static files are correctly bundled into the specified directory
4. If you need to modify dependencies, use the `go get` command to update the `go.mod` file

## License

This project is licensed under the MIT License.
