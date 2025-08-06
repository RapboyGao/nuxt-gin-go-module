# Nuxt Gin Go Module

这个模块是一个功能强大的集成包，专为 <mcurl name="Nuxt Gin Starter" url="https://github.com/RapboyGao/nuxt-gin-starter.git"></mcurl> 项目设计，提供核心功能支持。它无缝结合了 Nuxt 前端框架和 Gin 后端框架，为全栈开发提供了便捷的解决方案。

## 项目介绍

该模块是 Nuxt Gin Starter 项目的核心组件，提供了配置管理、Gin 模式设置和 Vue 服务渲染等关键功能。通过这个模块，可以快速搭建前后端分离的应用架构，同时兼顾开发效率和运行性能。

## 技术栈

- **后端**: Go 1.24.3, Gin 1.10.1
- **前端**: Nuxt.js (通过反向代理或静态文件服务)
- **依赖管理**: Go Modules
- **其他工具**: github.com/json-iterator/go (高性能 JSON 处理), github.com/arduino/go-paths-helper (文件路径操作)

## 项目结构

```plaintext
├── .gitignore
├── .vscode/
│   └── settings.json
├── GetConfig.go      # 配置管理
├── GetGinMode.go     # 配置Gin运行模式
├── LICENSE
├── ServeVue.go       # Vue服务渲染
├── go.mod            # Go依赖管理
└── utils/            # 工具函数
    ├── Framework.Directory.go
    ├── Framework.Excelize.go
    ├── Framework.IPAddress.go
    ├── Framework.MapStructure.go
    ├── Framework.MultipleFiles.go
    ├── Framework.Percentage.go
    └── Framework.Regex.go
```

## 核心功能

### 1. 配置管理 (Getconfig.go)

提供了统一的配置加载和管理机制，从`server.config.json`文件读取配置，并支持自动解析到结构体中。

```go:/Users/gaoxiaoyi/Documents/nuxt-gin-go-module/Getconfig.go
// 配置结构体示例
type Config struct {
    GinPort  int    `json:"ginPort"`  // Gin服务器端口
    NuxtPort int    `json:"nuxtPort"` // Nuxt应用端口
    BaseUrl  string `json:"baseUrl"`  // 应用基础URL
}

// 全局配置实例
get GetConfig Config = func() Config {
    config := Config{}
    config.Acquire()
    return config
}()
```

### 2. Gin 模式设置 (GetGinMode.go)

根据项目目录结构自动切换 Gin 的运行模式：

- 当`node_modules`目录存在时，启用开发模式(DebugMode)
- 当`node_modules`目录不存在时，启用生产模式(ReleaseMode)

```go:/Users/gaoxiaoyi/Documents/nuxt-gin-go-module/GetGinMode.go
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

### 3. Vue 服务渲染 (ServeVue.go)

根据 Gin 运行模式提供不同的 Vue 服务方式：

- 开发模式：通过反向代理连接到 Nuxt 开发服务器
- 生产模式：直接提供静态文件服务

```go:/Users/gaoxiaoyi/Documents/nuxt-gin-go-module/ServeVue.go
func ServeVue(engine *gin.Engine) {
    // 根路径重定向到应用基础URL
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

### 4. 工具函数库 (utils/)

提供了多种实用工具函数，包括目录操作、Excel 处理、IP 地址处理、数据结构转换等。

## 使用方法

### 安装依赖

```bash
cd /Users/gaoxiaoyi/Documents/nuxt-gin-go-module
# 设置Go代理(如果需要)
export GOPROXY="https://goproxy.io,direct"
# 安装依赖
go mod tidy
```

### 集成到项目中

```go
// 导入模块
import (
    "github.com/RapboyGao/nuxt-gin-go-module"
    "github.com/gin-gonic/gin"
)

func main() {
    // 配置Gin模式
    nuxtGin.ConfigureGinMode()

    // 创建Gin引擎
    engine := gin.Default()

    // 配置Vue服务
    nuxtGin.ServeVue(engine)

    // 启动服务
    engine.Run(fmt.Sprintf(":%d", nuxtGin.GetConfig.GinPort))
}
```

## 注意事项

1. 确保在项目根目录下创建`server.config.json`文件，配置必要的参数
2. 开发模式下，需要确保 Nuxt 开发服务器正在运行
3. 生产模式下，确保 Vue 静态文件已正确打包到指定目录
4. 如需修改依赖项，请使用`go get`命令更新`go.mod`文件

## 许可证

本项目使用 MIT 许可证
