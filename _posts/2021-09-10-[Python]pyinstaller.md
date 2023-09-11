---
layout: post
title: "[Python] py 파일을 exe 실행 파일로 변환"
date: 2021-09-10
categories: [Language, Python]
tags: [Python]
comments: true
---

친구가 만든 똥피하기 게임을 exe파일로 만들어달라는 부탁을 받았다. 
한번도 해보진 않았지만...!! 만들어서 보내주었다. 뿌듯해 ☺

1. py파일을 exe 파일로 만들어 주기 위한 pyinstaller 패키지를 설치 한다. 

```python
pip install pyinstaller
```

1. pyinstaller를 사용해서 exe 파일을 만든다.

```python
pyinstaller -F game.py
```

1. 실행했을 때 아래와 같은 메시지가 나온다면 데이터 경로가 문제일 수 있다. 

```python
pygame 2.0.1 (SDL 2.0.14, Python 3.8.10)
Hello from the pygame community. https://www.pygame.org/contribute.html
Traceback (most recent call last):
  File "game.py", line 51, in <module>
TypeError: expected str, bytes or os.PathLike object, not _io.BytesIO
[132844] Failed to execute script 'game' due to unhandled exception!
```

1. py에 사용되는 이미지 파일을 spec 파일에 더 해준다. 

```python
# -*- mode: python ; coding: utf-8 -*-

block_cipher = None

a = Analysis(['D:\\personal\\python_from_ym\\game.py'],
             pathex=['C:\\Users\\user\\AppData\\Local\\Packages\\PythonSoftwareFoundation.Python.3.8_qbz5n2kfra8p0\\LocalCache\\local-packages\\Python38\\Scripts'],
             binaries=[],
             datas=[],
             hiddenimports=[],
             hookspath=[],
             hooksconfig={},
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)
             
**a.datas += [('background.png', 'D:\\personal\\python_from_ym\\background.png', "DATA")]
a.datas += [('character.png', 'D:\\personal\\python_from_ym\\character.png', "DATA")]
a.datas += [('atti.png', 'D:\\personal\\python_from_ym\\atti.png', "DATA")]**

pyz = PYZ(a.pure, a.zipped_data,
             cipher=block_cipher)

exe = EXE(pyz,
          a.scripts,
          a.binaries,
          a.zipfiles,
          a.datas,  
          [],
          name='game',
          debug=False,
          bootloader_ignore_signals=False,
          strip=False,
          upx=True,
          upx_exclude=[],
          runtime_tmpdir=None,
          console=True,
          disable_windowed_traceback=False,
          target_arch=None,
          codesign_identity=None,
          entitlements_file=None )
```

1. py 코드에서는 절대경로로 찾을 수 있도록 아래 코드를 추가한다.

```python
import sys, os 

def resource_path(relative_path):
    if hasattr(sys, '_MEIPASS'):
        return os.path.join(sys._MEIPASS, relative_path)
    return os.path.join(os.path.abspath("."), relative_path)

background = pygame.image.load(resource_path("background.png"))
```

출처 : [https://datatofish.com/executable-pyinstaller/](https://datatofish.com/executable-pyinstaller/)