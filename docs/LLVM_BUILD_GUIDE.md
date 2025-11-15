# Rebuild TVM with LLVM Support

## Issue
Model conversion fails with: `Cannot find global function tvm.codegen.llvm.GetDefaultTargetTriple`

## Cause
TVM was built without LLVM support, which is required for CPU model conversion.

## Solution: Install LLVM and Rebuild

### Step 1: Install LLVM (Administrator PowerShell)
```powershell
choco install llvm -y
```

**Expected Duration:** 2-5 minutes

### Step 2: Close and Reopen PowerShell
Close your current PowerShell and open a new Administrator PowerShell to refresh environment variables.

### Step 3: Verify LLVM Installation
```powershell
llvm-config --version
```

Should show LLVM version (e.g., 18.1.8)

### Step 4: Clean Previous TVM Build
```powershell
cd C:\activedevelopment\mlc-llm-repo\3rdparty\tvm\build-windows
Remove-Item -Path * -Recurse -Force
```

### Step 5: Configure TVM with LLVM
```powershell
cmake .. -G "Visual Studio 17 2022" -A x64 -DCMAKE_BUILD_TYPE=Release -DUSE_LLVM=ON -DUSE_CUDA=OFF -DUSE_OPENCL=OFF -DUSE_VULKAN=OFF -DUSE_METAL=OFF
```

### Step 6: Build TVM (takes ~1 hour)
```powershell
cmake --build . --config Release --parallel 8
```

**Expected Duration:** 45-90 minutes (longer than before due to LLVM)

### Step 7: Copy New DLLs
```powershell
cd C:\activedevelopment\mlc-llm-repo
Copy-Item -Path .\3rdparty\tvm\build-windows\Release\*.dll -Destination .\3rdparty\tvm\python\tvm\ -Force
```

### Step 8: Verify TVM Works
```powershell
cd C:\activedevelopment\mlc-llm-repo
.\mlc-venv\Scripts\Activate.ps1
python -c "import tvm; print('TVM version:', tvm.__version__)"
```

### Step 9: Retry Model Conversion
Now run the conversion commands from Commands.md:
```powershell
python -m mlc_llm convert_weight .\models\Qwen2.5-3B-Instruct --quantization q4f16_1 -o .\dist\Qwen2.5-3B-Instruct-q4f16_1-MLC
```

## Alternative (Faster but Limited)
If you don't want to rebuild with LLVM, you could try using pre-quantized models from Hugging Face, but Qwen 2.5 3B may not be available in MLC format.

## Total Time Required
- LLVM Install: 5 minutes
- TVM Rebuild: 60-90 minutes
- Model Conversion: 15-35 minutes
- **Total: ~2 hours**
