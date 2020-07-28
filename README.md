# Build Instruction (32-bit architecture): 
```console
$ git glone https://github.com/wenhailiu/Libs.git
$ cd Libs && mkdir build && cd build
$ cmake -DQT_QMAKE_EXECUTABLE:PATH=C:/Qt/Qt5.9.1/5.9.1/msvc2015/bin/qmake -DQt5_DIR:PATH=C:/Qt/Qt5.9.1/5.9.1/msvc2015/lib/cmake/Qt5 -S ../ -B ./ -G “Visual Studio 14 2015” -A x86
```

Then build using Libs.sln in Visual Studio. 
