# JSON for Modern C++ (No UTF-8 Validation Fork)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository is a modified version of [nlohmann/json](https://github.com/nlohmann/json) (based on v3.10.5). 

## 💡 Why this fork?

If you are developing C++ applications on **Windows**, you are probably familiar with the encoding nightmare. The Windows console and many legacy APIs use ANSI (e.g., GBK/CP936), while modern libraries strictly enforce UTF-8.

The official `nlohmann/json` strictly validates UTF-8. If you try to parse or dump GBK strings (or raw binary data), it will immediately throw a `parse_error.101` or `type_error.316`.

**This modified version completely bypasses UTF-8 validation**, saving Windows console developers from encoding hell. It allows you to work seamlessly with GBK, ANSI, or any raw invalid bytes.

## 🛠️ What's Changed?

1. **Parsing (`scan_string`)**:
   - Removed UTF-8 multi-byte range validation.
   - Removed control character strictness (0x00-0x1F are preserved raw).
   - Removed surrogate pair validation (CESU-8 style raw bytes are preserved).
2. **Dumping (`dump_escaped`)**:
   - Simplified byte-by-byte processing. It only escapes JSON syntax characters (`"`, `\`, and control chars).
   - All other bytes (including `>= 0x80` GBK/ANSI bytes) are output as-is.
   - Completely eliminates `type_error.316` exceptions.

## 🚀 Usage

Just drop the `json.hpp` file into your project and use it as usual:

```cpp
#include <iostream>
#include "json.hpp"

using json = nlohmann::json;

int main() {
    // 1. Parsing GBK strings (e.g., "中文" in GBK is "\xD6\xD0\xCE\xC4")
    // This would normally throw a parse_error in the official version.
    std::string gbk_json = "\"\xD6\xD0\xCE\xC4\"";
    json j = json::parse(gbk_json);
    
    // 2. Dumping raw bytes
    // This would normally throw a type_error.316.
    std::string output = j.dump();
    
    // Output directly to Windows Console (which is usually GBK)
    std::cout << output << std::endl; 
    
    return 0;
}
