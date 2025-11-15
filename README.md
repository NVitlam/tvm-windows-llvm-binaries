# TVM Windows x64 Binaries with LLVM Support

**Pre-built TVM libraries for Windows with LLVM codegen enabled - the missing piece for MLC-LLM model conversion on Windows.**

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Platform](https://img.shields.io/badge/Platform-Windows%20x64-blue)]()
[![TVM](https://img.shields.io/badge/TVM-Commit%20c6fb2be79-green)]()
[![LLVM](https://img.shields.io/badge/LLVM-18.1.8-orange)]()

---

## 🎯 What Is This?

This repository provides **pre-compiled TVM binaries for Windows x64 with LLVM support enabled**.

### Why Does This Exist?

**The Problem:**
- MLC-LLM's official pre-built packages for Windows **do not include LLVM support**
- Without LLVM, you **cannot convert models** (the `mlc_llm convert_weight` command fails)
- Building TVM with LLVM on Windows is **extremely difficult** and time-consuming (2-4 hours)
- Most developers give up and switch to Linux/WSL

**The Solution:**
- Use these pre-built binaries to save **hours of painful Windows C++ build hell**
- Drop-in replacement for the official TVM installation
- Enables full MLC-LLM model conversion workflow on native Windows

---

## 📦 What's Included

This package contains 4 DLL files built on Windows with LLVM 18.1.8:

| File | Size | Description |
|------|------|-------------|
| `tvm.dll` | 109 MB | **Full TVM with LLVM codegen** - Required for model conversion |
| `tvm_runtime.dll` | 2.5 MB | Lightweight runtime (no LLVM) - For inference only |
| `tvm_ffi.dll` | 1.4 MB | Foreign Function Interface - Python/C++ interop layer |
| `tvm_ffi_testing.dll` | 582 KB | Testing utilities (optional) |

**Build Configuration:**
- ✅ LLVM Support: **ENABLED** (v18.1.8)
- ✅ MSVC Compiler: Visual Studio 2022
- ✅ Build Type: Release (optimized)
- ✅ Target: Windows x64 only
- ❌ CUDA: Disabled (CPU-only for model conversion)
- ❌ OpenCL/Vulkan: Disabled (not needed for conversion)

---

## 🚀 Quick Start

### Prerequisites

1. **Windows 10/11 (64-bit)**
2. **Visual C++ Redistributable 2015-2022 (x64)**
   - Download: https://aka.ms/vs/17/release/vc_redist.x64.exe
   - Required for MSVC-built DLLs
3. **Python 3.11+** with virtual environment
4. **16GB+ RAM** (for model conversion)

### Installation Steps

#### Step 1: Set Up Python Environment

```powershell
# Create virtual environment
python -m venv mlc-venv
.\mlc-venv\Scripts\Activate.ps1

# Install PyTorch (CPU version)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu

# Install transformers and dependencies
pip install transformers accelerate
```

#### Step 2: Clone MLC-LLM Repository

```powershell
# Clone with submodules
git clone --recursive https://github.com/mlc-ai/mlc-llm.git
cd mlc-llm

# Update submodules
git submodule update --init --recursive
```

#### Step 3: Replace TVM DLLs with Pre-built Binaries

```powershell
# Download this repository
git clone https://github.com/YOUR_USERNAME/tvm-windows-llvm-binaries.git

# Copy DLLs to TVM Python package directory
# Location: mlc-llm\3rdparty\tvm\python\tvm\
Copy-Item tvm-windows-llvm-binaries\bin\*.dll mlc-llm\3rdparty\tvm\python\tvm\ -Force

# Verify files are copied
ls mlc-llm\3rdparty\tvm\python\tvm\tvm.dll
# Should show tvm.dll (~109 MB)
```

#### Step 4: Install TVM Python Package

```powershell
cd mlc-llm\3rdparty\tvm\python

# Install TVM in development mode
pip install -e .

# Verify installation
python -c "import tvm; print('TVM version:', tvm.__version__)"
```

#### Step 5: Install MLC-LLM

```powershell
cd ..\..\..\  # Back to mlc-llm root

# Install MLC-LLM
pip install -e . --no-build-isolation

# Verify installation
python -c "import mlc_llm; print('MLC-LLM imported successfully')"
```

#### Step 6: Test Model Conversion

```powershell
# Download a test model (example: Qwen 2.5 3B)
mkdir models
cd models
git lfs clone https://huggingface.co/Qwen/Qwen2.5-3B-Instruct
cd ..

# Convert model weights to MLC format
python -m mlc_llm convert_weight \
  .\models\Qwen2.5-3B-Instruct \
  --quantization q4f16_1 \
  -o .\dist\Qwen2.5-3B-Instruct-q4f16_1-MLC

# Generate MLC chat config
python -m mlc_llm gen_config \
  .\models\Qwen2.5-3B-Instruct \
  --quantization q4f16_1 \
  --conv-template qwen2_5 \
  -o .\dist\Qwen2.5-3B-Instruct-q4f16_1-MLC
```

**Expected Results:**
- ✅ Conversion completes without errors
- ✅ Model size reduced from ~6GB to ~2GB (q4f16_1 quantization)
- ✅ No "Cannot find global function tvm.codegen.llvm.GetDefaultTargetTriple" error

---

## 🔍 Use Cases

### What You CAN Do With These Binaries

✅ **Convert models to MLC format** (`mlc_llm convert_weight`)
✅ **Generate MLC chat configs** (`mlc_llm gen_config`)
✅ **Quantize models** (q4f16_1, q4f16_0, q3f16_1, etc.)
✅ **Run TVM Python package** on Windows
✅ **Compile models for CPU inference**
✅ **LLVM IR code generation**

### What You CANNOT Do

❌ GPU acceleration (CUDA/OpenCL/Vulkan disabled)
❌ Android/ARM compilation (this is Windows x64 only)
❌ Metal acceleration (macOS only)

**Note:** For Android deployment, you need to build TVM separately for ARM64 with GPU support.

---

## 📖 Detailed Usage Guide

### Use Case 1: Converting Models for Android/Mobile

If you're building an Android app with MLC-LLM:

1. **On Windows (using these binaries):**
   - Convert model weights: `mlc_llm convert_weight`
   - Generate config: `mlc_llm gen_config`
   - Output: Quantized model files (~2GB for 3B model)

2. **On Android device:**
   - Use a separate TVM build for Android ARM64 with GPU support
   - Load the converted model files
   - Run inference with Vulkan/OpenCL acceleration

**This workflow allows you to do heavy model conversion on Windows, then deploy to mobile.**

### Use Case 2: Testing Model Conversion Locally

```powershell
# Test conversion with a small model first
python -m mlc_llm convert_weight \
  .\models\your-model \
  --quantization q4f16_1 \
  -o .\dist\your-model-q4f16_1-MLC

# Check output
ls .\dist\your-model-q4f16_1-MLC
# Should see: params_shard_*.bin, mlc-chat-config.json, tokenizer files
```

### Use Case 3: Quantization Experiments

Try different quantization levels:

```powershell
# 4-bit with fp16 activations (balanced)
python -m mlc_llm convert_weight model --quantization q4f16_1 -o dist/model-q4f16_1

# 3-bit (smaller, faster, lower quality)
python -m mlc_llm convert_weight model --quantization q3f16_1 -o dist/model-q3f16_1

# 4-bit variant (faster)
python -m mlc_llm convert_weight model --quantization q4f16_0 -o dist/model-q4f16_0
```

---

## 🛠️ Troubleshooting

### Error: "The code execution cannot proceed because MSVCP140.dll was not found"

**Solution:** Install Visual C++ Redistributable

```powershell
# Download and install from Microsoft
https://aka.ms/vs/17/release/vc_redist.x64.exe
```

### Error: "Cannot find global function tvm.codegen.llvm.GetDefaultTargetTriple"

**Cause:** You're still using the official TVM without LLVM support.

**Solution:** Make sure you copied the DLLs correctly:

```powershell
# Check tvm.dll size - should be ~109MB (with LLVM)
ls mlc-llm\3rdparty\tvm\python\tvm\tvm.dll

# If it's smaller (~2MB), you have the runtime-only version
# Re-copy from this repository
```

### Error: "Out of memory" during conversion

**Solution:**
- Close other applications
- Model conversion needs 16GB+ RAM
- Try a smaller model first
- Reduce system memory usage

### Error: "ImportError: cannot import name 'tvm' from 'tvm'"

**Solution:**

```powershell
# Reinstall TVM package
cd mlc-llm\3rdparty\tvm\python
pip install -e . --force-reinstall --no-deps
```

### Conversion is Very Slow

**Expected Times:**
- Small models (1-3B): 10-20 minutes
- Medium models (7-13B): 30-60 minutes
- Large models (30B+): 1-3 hours

**CPU-only builds are slower than GPU builds - this is normal.**

---

## 🏗️ How These Binaries Were Built

These binaries were compiled from source with the following process:

### Build Environment
- **OS:** Windows 11 x64
- **Compiler:** Microsoft Visual Studio 2022 (MSVC v143)
- **CMake:** 4.1.2
- **LLVM:** 18.1.8 (pre-built SDK from GitHub releases)
- **vcpkg:** Used for libxml2 dependency

### Build Steps (for reference)

<details>
<summary>Click to expand full build instructions</summary>

#### 1. Install Prerequisites

```powershell
# Install Visual Studio 2022 Build Tools
# Download from: https://visualstudio.microsoft.com/downloads/

# Install LLVM 18.1.8
# Download from: https://github.com/llvm/llvm-project/releases/tag/llvmorg-18.1.8
# Extract to C:\LLVM-Dev\

# Install vcpkg
git clone https://github.com/Microsoft/vcpkg.git C:\vcpkg
cd C:\vcpkg
.\bootstrap-vcpkg.bat

# Install libxml2 (MSVC-compiled)
.\vcpkg install libxml2:x64-windows-static
```

#### 2. Clone and Prepare TVM

```powershell
git clone --recursive https://github.com/mlc-ai/mlc-llm.git
cd mlc-llm\3rdparty\tvm

# Fix CMake version compatibility issues
# Edit: 3rdparty\tokenizers-cpp\msgpack\CMakeLists.txt
# Change line 1: CMAKE_MINIMUM_REQUIRED (VERSION 3.5 FATAL_ERROR)

# Edit: 3rdparty\tokenizers-cpp\sentencepiece\CMakeLists.txt
# Change line 15: cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
```

#### 3. Configure TVM Build

```powershell
mkdir build-windows
cd build-windows

cmake .. `
  -G "Visual Studio 17 2022" `
  -A x64 `
  -DCMAKE_BUILD_TYPE=Release `
  -DUSE_LLVM=ON `
  -DLLVM_DIR="C:/LLVM-Dev/lib/cmake/llvm" `
  -DUSE_CUDA=OFF `
  -DUSE_OPENCL=OFF `
  -DUSE_VULKAN=OFF `
  -DUSE_METAL=OFF
```

#### 4. Build TVM (takes 60-90 minutes)

```powershell
cmake --build . --config Release --parallel 8
```

#### 5. Output Files

```
build-windows\
├── Release\
│   ├── tvm.dll (109 MB)
│   └── tvm_runtime.dll (2.5 MB)
└── lib\
    ├── tvm_ffi.dll (1.4 MB)
    └── tvm_ffi_testing.dll (582 KB)
```

</details>

**Build Time:** ~2-3 hours (first time)
**Difficulty:** High (Windows C++ builds are complex)
**Pain Level:** 🔥🔥🔥🔥🔥 (saved you this nightmare!)

---

## 📚 Additional Resources

### Official Documentation
- **MLC-LLM Docs:** https://llm.mlc.ai/docs/
- **TVM Docs:** https://tvm.apache.org/docs/
- **MLC-LLM GitHub:** https://github.com/mlc-ai/mlc-llm
- **TVM GitHub:** https://github.com/apache/tvm

### Community Support
- **MLC-LLM Discord:** https://discord.gg/9Xpy2HGBuD
- **TVM Discuss:** https://discuss.tvm.apache.org/

### Related Projects
- **llama.cpp:** Alternative for GGUF model inference
- **GGUF-to-MLC Converter:** (if available)
- **HuggingFace Model Hub:** https://huggingface.co/models

---

## 🤝 Contributing

Found a bug or have improvements? Please open an issue or PR!

### Reporting Issues

Please include:
- Windows version (run `winver`)
- Python version (`python --version`)
- Error message (full traceback)
- Steps to reproduce

### Building Newer Versions

Want to build a newer TVM version yourself?

1. Check the build instructions in `docs/BUILD_PROCESS.md`
2. Update LLVM version if needed
3. Follow the CMake configuration steps
4. Submit a PR with updated binaries

---

## 📄 License

**Apache License 2.0** (same as TVM and MLC-LLM)

These binaries are compiled from official TVM source code with no modifications except build configuration.

- TVM: https://github.com/apache/tvm (Apache 2.0)
- MLC-LLM: https://github.com/mlc-ai/mlc-llm (Apache 2.0)
- LLVM: https://llvm.org/ (Apache 2.0 with LLVM Exceptions)

---

## ⚠️ Disclaimer

These binaries are provided **as-is** without warranty.

- Built on Windows 11 x64 with Visual Studio 2022
- Tested with Python 3.11 and MLC-LLM commit 7b15b196
- May not work with all configurations
- Always test with your specific setup

**For production use, consider building from source to match your exact environment.**

---

## 🙏 Acknowledgments

- **Apache TVM Team:** For the amazing tensor compiler
- **MLC-LLM Team:** For making LLM deployment accessible
- **LLVM Project:** For the compiler infrastructure
- **Open Source Community:** For helping debug Windows build issues

---

## 📊 Stats

- **Total Build Time Invested:** ~8 hours (including troubleshooting)
- **Time Saved Per User:** ~2-4 hours
- **Community Benefit:** Hopefully hundreds of developers!

---

## 🔗 Quick Links

- 📥 [Download Latest Release](https://github.com/YOUR_USERNAME/tvm-windows-llvm-binaries/releases)
- 📖 [Full Build Documentation](docs/BUILD_PROCESS.md)
- 🐛 [Report Issues](https://github.com/YOUR_USERNAME/tvm-windows-llvm-binaries/issues)
- 💬 [Discussions](https://github.com/YOUR_USERNAME/tvm-windows-llvm-binaries/discussions)

---

**Made with ❤️ and lots of ☕ to save you from Windows build hell.**

*Star ⭐ this repo if it saved you time!*
