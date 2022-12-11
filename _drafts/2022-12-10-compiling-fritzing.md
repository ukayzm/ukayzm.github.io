---
title:  "Windows에서 Fritzing을 직접 컴파일하여 사용하기"
tags:   arduino
header:
  teaser: /assets/img/fritzing/compiling_fritzing.png
  image: /assets/img/fritzing/compiling_fritzing.png
date:   2022-12-11 12:00:00 +0900
---

Fritzing은 간단한 회로를 그리고 표현하는데 가장 간편한 툴이다. 0.9.3 버전까지는 인터넷에서 무료로 구할 수 있지만 그 이후 버전은 유료이다. 하지만 소스코드가 공개되어 있기 때문에 직접 컴파일을 하면 무료로 사용할 수 있다. 심지어 Fritzing의 공식 GitHub에서 컴파일 방법까지 친절하게 설명하고 있다. [Building Fritzing](https://github.com/fritzing/fritzing-app/wiki/1.-Building-Fritzing)

이 글에서는 Fritzing 소스코드를 Windows 11에서 다운로드 받고 컴파일하는 과정을 기술한다. Windows 10에서도 동일하게 설치 가능할 것으로 생각한다.

Fritzing을 빌드하는데는 여러 단계가 필요하다. 하지만 각 단계가 그리 어렵지는 않으니 하나 하나 따라가다 보면 어느새 빌드가 완성된다. 

빌드 과정에서 여러 서브디렉토리들이 생성되므로, 별도의 디렉토리를 만들고 시작하기를 추천한다. 나는 작업 디렉토리 용도로 `D:\fritzing` 디렉토리를 만들었고, 거기에 코드를 다운로드했다.

# 1. Visual Studio 설치

[Visual Studio Download](https://visualstudio.microsoft.com/downloads/)에서 Visual Studio를 다운로드 받아 설치한다. 이 글을 쓰는 시점의 최신 버전인 Visual Studio Community 2022를 받아서 설치했다. Community 버전을 설치하면 무료로 설치하고 사용할 수 있다. 

중요한 점은 아래와 같이 Python과 C++ 지원을 설치해야 한다는 것이다.

![Including_Python_CPP](/assets/img/fritzing/installing_visual_studio.png){: .align-center}

나머지는 그냥 NEXT를 누르면 설치가 끝난다. PC와 네트워크 성능에 따라 다르겠지만, 대략 10분 쯤 걸린다.

이미 Visual Studio가 설치되어 있는 경우에도 Installer를 이용하여 Python과 C++ 지원을 추가할 수 있다.

# 2. Qt 설치

다음으로 Qt를 설치한다. [Qt Download](https://www.qt.io/download-qt-installer) 페이지에 접속하고 "Download"를 눌러 설치파일을 다운로드하고 실행한다.

Qt를 설치하려면 Qt 계정이 있어야 하는데, 설치 과정에서 무료로 만들 수 있다.

[Fritzing 공식 빌드 가이드](https://github.com/fritzing/fritzing-app/wiki/1.-Building-Fritzing) 페이지에서는 Qt 버전 5.15.2를 설치하라고 나와 있다. "Next"를 몇 번 누르면 아래와 같이 "Select Components" 화면이 나오는데, 여기에서 5.15.2 버전을 선택한다.

![Qt_setup](/assets/img/fritzing/qt_setup.png){: .align-center}

나머지는 "Next"를 누르면 된다. 

필요한 파일을 다운로드 받아서 설치하는데 대략 서너시간 정도 걸린다. 너무 오래 걸리니까 다운로드 하도록 내버려 두고 다음 단계로 넘어가도 된다.

# 3. Fritzing Source Code 다운로드

[GitHub fritzing-app](https://github.com/fritzing/fritzing-app)과 [GitHub fritzing-parts](https://github.com/fritzing/fritzing-parts)에서 코드를 다운로드 받는다. 

각 페이지에서 "Code"를 누르고 "Download zip" (아래 그림의 초록색 동그라미)를 누르면 된다.

![Code Download](/assets/img/fritzing/code_download.png){: .align-center}

또는 파란색 동그라미를 눌러, git을 써서 다운로드 받을 수도 있다. 아래 명령은 command line으로 다운로드 받는 방법이다.

```bash
git clone https://github.com/fritzing/fritzing-app.git
git clone https://github.com/fritzing/fritzing-parts.git
```

Git을 쓰든, zip으로 다운로드 받아서 압축을 쓰든, 아래와 같이 두 개의 디렉토리가 만들어져야 한다.
* fritzing-app
* fritzing-parts

# 4. Boost 다운로드

Boost는 C++에서 널리 사용되는 유명한 라이브러리이다. [Boost Downloads](https://www.boost.org/users/download/) 페이지에 접속하여 최신 버전의 Windows 용 Boost 압축파일을 다운로드 받는다. 이 글을 쓰는 시점에는 boost_1_80_0.zip이 최신 버전이었다.

압축파일을 작업디렉토리에 푼다. 

# 5. Libgit2 다운로드

[LibGit2 Release](https://github.com/libgit2/libgit2/releases) 페이지에서 최신 코드를 다운로드 받는다. 스크롤을 조금 내리고 아래와 같이 Assets에서 "Source code (zip)"을 누르면 된다.

![LIbgit2](/assets/img/fritzing/libgit2.png){: .align-center}

작업디렉토리에서 압축을 푼다. 디렉토리 이름을 `libgit2`로 바꾼다. 그래야 Qt에서 디렉토리를 인식할 수 있다.

이제 코드 다운로드는 끝났다. 여기까지 진행되면 아래와 같은 디렉토리 구조가 나와야 한다.

![Directory Structure](/assets/img/fritzing/directory.png)

```
D:\fritzing
├── boost_1_80_0
│   ├── INSTALL
│   ├── Jamroot
│   ├── LICENSE_1_0.txt
|   ...
├── fritzing-app
│   ├── Fritzing.1
│   ├── Fritzing.sh
│   ├── FritzingInfo.plist
|   ...
├── fritzing-parts
│   ├── CONTRIBUTING.md
│   ├── LICENSE.txt
│   ├── README.md
|   ...
└── libgit2
    ├── AUTHORS
    ├── CMakeLists.txt
    ├── COPYING
    ...
```

# 6. Cmake 설치

Cmake는 빌드를 위한 파일을 만들어 주는 (즉, Makefile을 만들어 주는) 유명한 툴이다. [Cmake Download](https://cmake.org/download/) 페이지에 접속하여 최신 버전의 Windows용 설치 파일을 다운로드 받아서 설치한다.

이 글을 쓰는 시점에는 cmake-3.25.1-windows-x86_64.msi 파일을 다운로드 받고 실행했다. 설치 과정에서 아래와 같이 cmake를 PATH에 등록을 시켜 준다.

![Cmake](/assets/img/fritzing/cmake.png){: .align-center}

나머지는 그냥 "Next"만 눌러서 진행했다.

# 7. Libgit2 빌드

설치한 cmake를 이용해서 libgit2를 컴파일한다.

명령 프롬프트 창을 열고, 작업디렉토리로 이동하고 아래 명령으로 컴파일을 한다. 빌드 디렉토리 이름을 `build64`로 해 주어야 Qt에서 인식한다. 

```
D:
cd \fritzing\libgit2
mkdir build64
cd build64
cmake ..
cmake --build . --config Release
```

cmake가 실행이 안된다면, PATH 환경 변수에 cmake 경로가 들어있는지 확인한다.

# 8. Qt에서 Fritzing 빌드하기

지금까지의 과정은 모두 이 단계를 위해서 한 일이었다. Qt 설치를 포함하여 위의 모든 단계가 완료되어야 한다.

탐색기에서 `fritzing-app` 디렉토리를 열고 `phoenix.pro` 파일을 더블클릭한다. Qt Creator가 실행되고 Fritzing 프로젝트가 열린다. 

디폴트로 "Desktop Qt 5.15.2 MSVC2015 64bit"이 선택되어 있는데, MSVC2019가 더 최신 버전이므로 "Desktop Qt 5.15.2 MSVC2019 64bit"을 선택한다. "Configure Project" 버튼을 누르면 자동으로 Indexing이 진행된다. 이 작업은 몇 분 정도 걸린다.

왼쪽의 Projects 를 누르고, "Desktop Qt 5.15.2 MSVC2019 64bit"를 선택한다. 위에서 다른 config를 선택했다면 다시 configuring 작업이 진행될 것이다. "Run"을 누르고 "Command line arguments"를 아래와 같이 설정한다.

```
-f "D:\fritzing\fritzing-app" -parts "D:\fritzing\fritzing-parts" -db "D:\fritzing\fritzing-parts\parts.db"
```

<figure>
    <a href="/assets/img/fritzing/qt_cla.png" class="align-center"><img src="/assets/img/fritzing/qt_cla.png"></a>
</figure>

# 10. 그래도 안되면...

[Fritzing 0.9.9](https://github.com/Move2win/Fritzing-0.9.9.64.pc-Compiled-Build)를 누군가가 컴파일해서 GitHub에 올린 것을 다운로드 받아서 실행한다. Windows에서 위험한 파일일 수 있다고 워닝이 나오지만 무시하고 실행하면 된다.


# Reference

* [Fritzing WIKI in GitHug](https://github.com/fritzing/fritzing-app/wiki)
* [Compile Fritzing From Source on Windows 11 Step By Step Guide](https://siytek.com/build-fritzing-windows/)

