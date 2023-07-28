---
title: Linux 환경에서 Crypto++ 테스트
date: 2023-07-28 10:40:12 +0900
categories: [Stack, C++]
tag: [C++, Encryption, Crypto++, 암호화, AES256, GS]
---

GS 인증을 받는 과정 중 암호가 저장된 일부 설정 파일(ini, json 등)에 대한 암호화 과정이 필요했습니다.

본문은 Linux (ARMv8) 환경에서 Crypto++를 설치(빌드)하고 예제 코드를 테스트하는 과정을 기록한 글입니다.

---

### 테스트 환경
- Nvidia Jetson Xavier NX (Jetpack 4.6)
- Ubuntu 18.04 (Linux for Tegra)
- GCC 7.5.0 (C++17), CMake 3.24

### 설치 과정
먼저 Github를 통해 프로젝트를 다운 받았습니다.
```bash
git clone https://github.com/weidai11/cryptopp.git
cd cryptopp
```

Crypto++는 GNUmakefile을 통해 리눅스 빌드를 지원하고 있었습니다.

저는 현재 개발 중인 프로그램에 내장하기 위해 static 라이브러리 버전을 빌드하였습니다.

OpenCV정도는 아니지만, 시간이 꽤 소요되는 편이기 때문에 CPU 코어를 확인하고 알맞은 스레드 옵션을 주는 것이 좋아 보입니다.

```bash
make static -j8 # 자신의 CPU 코어 확인
```

컴파일이 완료되면 object 파일들이 생성됩니다. 그리고나서 install을 실행하면 이것을 통해 static 라이브러리를 만들고 include 목록을 생성해줍니다.

다행히 install 과정은 compile만큼 오래 걸리진 않았습니다.
저는 root 경로(/usr/local)에 설치되는 것이 싫었기 때문에 경로를 지정해줬습니다.

루트 경로를 사용하려면 sudo 권한이 필요한 것 같습니다.

```bash
make install PREFIX=build

## 실행 결과
ls build  # bin  include  lib  share
```

### 테스트
간단한 테스트를 위해 프로젝트를 하나 생성하였습니다.

아래 코드는 평문을 AES256 알고리즘을 통해 암호화 하여 파일로 저장하고, 이것을 다시 파일에서 읽어 반환합니다.

> 대칭키는 고정된 문자열을 사용한다고 가정합니다.

```cpp
#include <iostream>
#include <string>

#include "cryptopp/aes.h"
#include "cryptopp/cryptlib.h"
#include "cryptopp/files.h"
#include "cryptopp/filters.h"
#include "cryptopp/hex.h"
#include "cryptopp/modes.h"
#include "cryptopp/osrng.h"
#include "cryptopp/secblock.h"

namespace crypto {
namespace encrypt {
void stringToFile(const std::string& source_string, const std::string& output_file,
                  const std::string& key, const std::string& iv)
{
  try {
    using namespace CryptoPP;

    CBC_Mode<AES>::Encryption encryption;
    encryption.SetKeyWithIV((const byte*)key.data(), key.size(),
                            (const byte*)iv.data());

    // To file
    StringSource ss(source_string, true,
                    new StreamTransformationFilter(encryption,
                      new FileSink(output_file.c_str()))
    );
#if 0 // To source
    std::string encrypted_string = "";
    StringSource ss(source_string, true,
                    new StreamTransformationFilter(encryption,
                      new FileSink(encrypted_string))
    );
    std::cout << "encrypted_string : " << encrypted_string << std::endl;
#endif  
  } catch (const CryptoPP::Exception& e) {
    std::cerr << "encrypt : exception : " << e.what() << std::endl;
  }
}
}  // namespace encrypt

namespace decrypt {
void fileToString(const std::string& source_file, std::string& output_string,
                  const std::string& key, const std::string& iv)
{
  output_string.clear();

  try {
    using namespace CryptoPP;

    CBC_Mode<AES>::Decryption decryption;
    decryption.SetKeyWithIV((const byte*)key.data(), key.size(),
                            (const byte*)iv.data());

    // From file
    FileSource fs(source_file.c_str(), true,
                  new StreamTransformationFilter(decryption,
                    new StringSink(output_string))
    );
  } catch (const CryptoPP::Exception& e) {
    std::cerr << "decrypt : exception : " << e.what() << std::endl;
  }
}
}  // namespace decrypt
}  // namespace crypto

int main()
{
  std::string plane_text = "My crypto @ Testing 1";

  std::string key = "q6wEdPuv3JctZolAi4MbFcQhst89GBLD";  // 32byte 임의의 문자열
  std::string iv = "abcdefghij123456";  // 16byte 임의의 문자열


  // 암호화하여 파일에 저장
  std::string encrypted_file = "encrypted.dat";
  crypto::encrypt::stringToFile(plane_text, encrypted_file, key, iv);

  // 파일을 복호화하여 문자열로 변환
  std::string decrypted_text;
  crypto::decrypt::fileToString(encrypted_file, decrypted_text, key, iv);
  std::cout << "=== Test 1. normal case ================" << std::endl;
  std::cout << decrypted_text << std::endl;


#if 1 // 일반 파일을 복호화 하는 경우
  std::string plane_text_file = "plane-text.txt";
  std::ofstream ofs(plane_text_file);
  ofs << plane_text;
  ofs.close();

  crypto::decrypt::fileToString(plane_text_file, decrypted_text, key, iv);
  std::cout << "=== Test 2. plane text decryption ================" << std::endl;
  std::cout << decrypted_text << std::endl;
#endif

#if 1 // 복호화할 때 키가 잘못된 경우
  key = "abcdefghijklmlnopqrstuvwxyz12345";
  crypto::decrypt::fileToString(encrypted_file, decrypted_text, key, iv);
  std::cout << "=== Test 3. decrypt with wrong key ================" << std::endl;
  std::cout << decrypted_text << std::endl;
#endif
}
```
```
=== Test 1. normal case ================
My crypto @ Testing 1
decrypt : exception : StreamTransformationFilter: ciphertext length is not a multiple of block size
=== Test 2. plane text decryption ================

decrypt : exception : StreamTransformationFilter: invalid PKCS #7 block padding found
=== Test 3. decrypt with wrong key ================
�zc��7[|�[�.F�
```

보다 간단한 이해를 위해 코드에서 예외처리는 대부분 누락되어 있습니다.

실제 사용 시에는 Key의 길이를 확인하거나, 초기 벡터(`iv`)의 길이를 확인하는 로직이 포함되어야 하며, 파일의 존재 유무 역시 판단되어야 합니다.



