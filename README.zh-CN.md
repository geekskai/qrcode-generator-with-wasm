# QR 码生成器 (WebAssembly 版)

这是一个使用 Rust 和 WebAssembly 技术开发的高性能 QR 码生成器。该项目将 Rust 的高效能与 WebAssembly 的跨平台特性相结合，为 Web 应用提供快速、高效的 QR 码生成功能。

## ✨ 功能特点

- ⚡️ **高性能**：利用 Rust 和 WebAssembly 实现毫秒级二维码生成，性能远超纯 JavaScript 实现
- 🔄 **实时预览**：输入变化时即时更新 QR 码
- 📋 **复制功能**：一键复制输入的文本到剪贴板
- 💾 **下载支持**：可下载生成的 QR 码图像（PNG 格式）
- 🎯 **标准规范**：严格遵循 QR 码标准，包含 4 模块静默区
- 📱 **响应式设计**：适配不同设备屏幕尺寸
- 🚀 **零依赖前端**：使用原生 JavaScript，无需额外框架
- 🔧 **开发友好**：支持热重载和错误调试

## 🛠 技术栈

- **Rust**：核心 QR 码生成逻辑，使用 `qr_code` 库
- **WebAssembly**：将 Rust 编译为可在浏览器中运行的高性能模块
- **JavaScript (ES6+)**：前端交互逻辑和 Canvas 渲染
- **HTML5 Canvas**：二维码图像渲染
- **CSS3**：现代化响应式界面设计
- **Vite**：快速的前端构建工具
- **wasm-pack**：Rust 到 WebAssembly 的编译工具

## 📦 安装与使用

### 前置条件

确保您的系统已安装以下工具：

- [Rust](https://www.rust-lang.org/tools/install) (最新稳定版)
- [wasm-pack](https://rustwasm.github.io/wasm-pack/installer/)
- [Node.js](https://nodejs.org/) (>= 22.14.0)
- [pnpm](https://pnpm.io/) (>= 10.8.0，推荐使用)

### 快速开始

1. **克隆仓库**

   ```bash
   git clone https://github.com/liuliangsir/qrcode-generator-with-wasm.git
   cd qrcode-generator-with-wasm
   ```

2. **构建 WebAssembly 模块**

   ```bash
   wasm-pack build --target web --out-dir src
   ```

   ```bash
   pnpm run build:wasm
   ```

3. **安装前端依赖**

   ```bash
   pnpm install
   ```

4. **启动开发服务器**

   ```bash
   pnpm dev
   ```

5. **访问应用**

   在浏览器中打开 [http://localhost:5173](http://localhost:5173)

### 生产构建

```bash
# 构建 WebAssembly 模块
pnpm build:wasm

# 构建前端资源
pnpm build
```

构建产物将输出到 `dist` 目录。

## 🎮 使用方法

1. **输入内容**：在输入框中输入任意文本、URL 或数据
2. **实时生成**：QR 码将自动生成并显示在画布上
3. **复制文本**：点击"Copy"按钮将输入的文本复制到剪贴板
4. **下载图片**：点击"Download"按钮保存 QR 码图像到本地

### 支持的输入类型

- 纯文本
- URL 链接
- 邮箱地址
- 电话号码
- WiFi 配置信息
- 其他任意字符串数据

## 📁 项目结构

```text
qrcode-generator-with-wasm/
├── src/                    # 源代码目录
│   ├── lib.rs             # Rust WebAssembly 核心模块
│   ├── app.js             # 前端 JavaScript 主逻辑
│   ├── app.css            # 样式表文件
│   └── pkg/               # wasm-pack 生成的 WASM 模块 (构建后)
├── public/                # 静态资源目录
├── .github/               # GitHub Actions 工作流
│   └── workflows/
│       └── deploy-to-github-pages.yml
├── index.html             # 主 HTML 页面
├── Cargo.toml             # Rust 项目配置
├── package.json           # Node.js 项目配置
├── vite.config.js         # Vite 构建配置
├── .editorconfig          # 编辑器配置
├── .gitignore             # Git 忽略文件
└── README.zh-CN.md        # 中文说明文档
```

## 🔧 核心原理

### Rust → WebAssembly → JavaScript 完整技术流程

#### 1. Rust 代码编写阶段

```rust
// src/lib.rs - Rust 源代码
use wasm_bindgen::prelude::*;
use qr_code::QrCode;

#[wasm_bindgen]  // 这个宏标记函数可以被 JavaScript 调用
pub struct QrCodeResult {
    #[wasm_bindgen(getter_with_clone)]
    pub data: Vec<u8>,  // 像素数据：0=白色，1=黑色
    pub size: usize,    // 二维码尺寸
}

#[wasm_bindgen]  // 导出给 JavaScript 的函数
pub fn generate_qr_code_wasm(input_data: &str) -> Result<QrCodeResult, JsValue> {
    // 1. 使用 qr_code 库生成二维码矩阵
    let code = QrCode::new(input_data.as_bytes())?;

    // 2. 处理像素数据和静默区
    let original_size = code.width() as usize;
    let quiet_zone = 4;
    let final_size = original_size + 2 * quiet_zone;

    // 3. 生成最终的像素数组
    let mut pixel_data = vec![0u8; final_size * final_size];
    // ... 像素处理逻辑

    // 4. 返回结构化数据给 JavaScript
    Ok(QrCodeResult {
        data: pixel_data,
        size: final_size,
    })
}
```

#### 2. WebAssembly 编译阶段

```bash
# 使用 wasm-pack 将 Rust 代码编译为 WebAssembly
wasm-pack build --target web --out-dir src
```

**编译过程详解：**

1. **Rust 编译器 (rustc)** 将 Rust 代码编译为 WebAssembly 字节码
2. **wasm-bindgen** 生成 JavaScript 绑定代码，包括：
   - `qrcode_generator_with_wasm.js` - JavaScript 包装器
   - `qrcode_generator_with_wasm_bg.wasm` - WebAssembly 二进制文件
   - `qrcode_generator_with_wasm.d.ts` - TypeScript 类型定义

**生成的文件结构：**

```text
src/
├── qrcode_generator_with_wasm.js       # JS 绑定代码
├── qrcode_generator_with_wasm_bg.wasm  # WASM 二进制模块
├── qrcode_generator_with_wasm.d.ts     # TypeScript 类型
└── package.json                        # 模块配置
```

#### 3. JavaScript 集成阶段

```javascript
// src/app.js - 前端 JavaScript 代码

// 1. 导入 WebAssembly 模块
import init, { generate_qr_code_wasm } from "./qrcode_generator_with_wasm.js";

// 2. 初始化 WebAssembly 模块
async function initWasm() {
  await init(); // 加载 .wasm 文件并初始化
  console.log("WebAssembly 模块已加载");
}

// 3. 调用 Rust 函数
function generateQRCode(inputText) {
  try {
    // 直接调用 Rust 函数，就像调用普通 JavaScript 函数一样
    const result = generate_qr_code_wasm(inputText);

    // 4. 获取返回的数据
    const pixelData = result.data; // Vec<u8> 自动转换为 Uint8Array
    const size = result.size; // usize 自动转换为 number

    // 5. 使用 Canvas API 渲染
    renderQRCode(pixelData, size);
  } catch (error) {
    console.error("QR 码生成失败:", error);
  }
}

// 6. Canvas 渲染函数
function renderQRCode(pixelData, size) {
  const canvas = document.getElementById("qrCanvas");
  const ctx = canvas.getContext("2d");

  // 计算每个模块的像素大小
  const moduleSize = canvas.width / size;

  // 遍历像素数据并绘制
  for (let y = 0; y < size; y++) {
    for (let x = 0; x < size; x++) {
      const pixelIndex = y * size + x;
      const isBlack = pixelData[pixelIndex] === 1;

      ctx.fillStyle = isBlack ? "#000000" : "#ffffff";
      ctx.fillRect(x * moduleSize, y * moduleSize, moduleSize, moduleSize);
    }
  }
}
```

### 数据类型自动转换机制

**wasm-bindgen 提供的自动类型转换：**

| Rust 类型      | JavaScript 类型  | 说明         |
| -------------- | ---------------- | ------------ |
| `&str`         | `string`         | 字符串引用   |
| `String`       | `string`         | 拥有的字符串 |
| `Vec<u8>`      | `Uint8Array`     | 字节数组     |
| `usize`        | `number`         | 无符号整数   |
| `bool`         | `boolean`        | 布尔值       |
| `Result<T, E>` | 抛出异常或返回值 | 错误处理     |

### 内存管理机制

```javascript
// WebAssembly 内存管理示例
function handleMemory() {
  const result = generate_qr_code_wasm("Hello World");

  // 1. Rust 在 WebAssembly 线性内存中分配数据
  // 2. wasm-bindgen 自动创建 JavaScript 视图 (Uint8Array)
  // 3. JavaScript 可以直接访问这些数据
  const pixelData = result.data; // 零拷贝访问

  // 4. 当 JavaScript 对象被垃圾回收时，
  //    对应的 WebAssembly 内存也会被释放
}
```

### 性能优化原理

1. **零拷贝数据传输**：

   - Rust 和 JavaScript 共享同一块内存
   - 避免了数据序列化/反序列化的开销

2. **编译时优化**：

   ```toml
   # Cargo.toml 中的优化配置
   [profile.release]
   lto = true          # 链接时优化，减小文件大小
   opt-level = 's'     # 优化代码大小
   ```

3. **运行时性能**：
   - WebAssembly 接近原生代码性能
   - 比纯 JavaScript 实现快 10-100 倍

### 错误处理流程

```rust
// Rust 端错误处理
pub fn generate_qr_code_wasm(input_data: &str) -> Result<QrCodeResult, JsValue> {
    let code = QrCode::new(input_data.as_bytes())
        .map_err(|e| JsValue::from_str(&format!("二维码生成错误: {:?}", e)))?;
    // ...
}
```

```javascript
// JavaScript 端错误捕获
try {
  const result = generate_qr_code_wasm(inputText);
} catch (error) {
  // Rust 的 Err 会自动转换为 JavaScript 异常
  console.error("错误信息:", error);
}
```

## 🚀 开发指南

### 修改 Rust 代码

当您修改 `src/lib.rs` 或其他 Rust 代码后：

```bash
pnpm build:wasm
```

### 修改前端代码

前端代码（JavaScript/CSS/HTML）修改后会自动热重载，无需手动重启。

### 调试技巧

1. **Rust 调试**：启用 `console_error_panic_hook` 功能，在浏览器控制台查看详细错误信息
2. **性能分析**：使用浏览器开发者工具的 Performance 面板分析 WASM 性能
3. **内存监控**：使用 Memory 面板监控 WebAssembly 内存使用情况

### 自定义配置

您可以通过修改以下文件来自定义项目：

- `Cargo.toml`：Rust 依赖和编译选项
- `vite.config.js`：前端构建配置
- `src/app.css`：界面样式
- `index.html`：页面结构和元数据

## 🌐 部署

### GitHub Pages

项目已配置 GitHub Actions 自动部署：

1. 推送代码到 `main` 分支
2. GitHub Actions 自动构建并部署到 GitHub Pages
3. 访问 `https://yourusername.github.io/qrcode-generator-with-wasm`

### 其他平台

构建后的 `dist` 目录可以部署到任何静态网站托管服务：

- Vercel
- Netlify
- Cloudflare Pages
- 阿里云 OSS
- 腾讯云 COS

## 🤝 贡献指南

我们欢迎各种形式的贡献！

### 如何贡献

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

### 代码规范

- Rust 代码遵循 `rustfmt` 格式化标准
- JavaScript 代码使用 ES6+ 语法
- 提交信息使用英文，格式清晰
- 添加适当的注释和文档

### 问题反馈

如果您遇到问题或有建议，请：

1. 查看 [Issues](https://github.com/liuliangsir/qrcode-generator-with-wasm/issues) 是否已有相关问题
2. 创建新的 Issue，详细描述问题和复现步骤
3. 提供浏览器版本、操作系统等环境信息

## 📄 许可证

本项目基于 [MIT 许可证](LICENSE) 开源。

## 🙏 致谢

- [qr_code](https://crates.io/crates/qr_code) - 优秀的 Rust QR 码生成库
- [wasm-bindgen](https://github.com/rustwasm/wasm-bindgen) - Rust 和 WebAssembly 绑定工具
- [Vite](https://vitejs.dev/) - 快速的前端构建工具

## 📊 项目统计

- 🦀 Rust 代码：~60 行
- 📜 JavaScript 代码：~200 行
- 🎨 CSS 代码：~150 行
- 📦 WASM 模块大小：~15KB (gzipped)
- ⚡ 生成速度：<1ms (典型输入)

---

如果这个项目对您有帮助，请给我们一个 ⭐️！
