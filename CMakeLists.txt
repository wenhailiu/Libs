project(Libs)
cmake_minimum_required(VERSION 3.16)

set(CMAKE_CXX_STANDARD 11)

include(${CMAKE_ROOT}/Modules/ExternalProject.cmake)

#Thirdparties root dir: 
set(ThirdPartyLibs ${CMAKE_CURRENT_BINARY_DIR}/ThirdPartyLibs)

#Settings: 
set(CMAKE_BUILD_TYPE Release)
set(Use_System_Qt5 ON)
set(Use_System_VTK OFF)
set(Use_System_OpenIGTLink OFF)
set(Use_System_OpenCV OFF)
set(Use_CUDA OFF)

#Check CUDA: 
if(Use_CUDA)
    find_package(CUDA)
    if(CUDA_FOUND)
        set(CMAKE_CUDA_COMPILER ${CUDA_TOOLKIT_ROOT_DIR}/bin/nvcc.exe)
        enable_language(CUDA)
        include_directories(${CUDA_INCLUDE_DIRS})
    endif()
endif()

#Check Qt5: 
if(Use_System_Qt5)
    # set(Qt5_DIR C:/Qt/Qt5.12.7/5.12.7/msvc2015_64/lib/cmake/Qt5)
    set(Qt5_DIR C:/Qt/Qt5.9.1/5.9.1/msvc2015/lib/cmake/Qt5)
    find_package(Qt5 COMPONENTS Core Widgets Xml OpenGL REQUIRED)
    include_directories(${QT_INCLUDES})
    set(CMAKE_AUTOUIC ON)
    set(CMAKE_AUTOMOC ON)
endif()

#VTK: 
if(Use_System_VTK)
    find_package(VTK REQUIRED)
    include_directories(${VTK_INCLUDE_DIRS})
else()
    set(VTK_Install ${ThirdPartyLibs}/VTKInstall)
    set(VTK_Repo ${ThirdPartyLibs}/VTK)
    set(VTK_DIR ${ThirdPartyLibs}/VTKBuild)
    set(VTK_Prefix ${ThirdPartyLibs}/VTKPrefix)
    ExternalProject_Add(
        VTK
        PREFIX ${VTK_Prefix}
        SOURCE_DIR ${VTK_Repo}
        BINARY_DIR ${VTK_DIR}
        GIT_REPOSITORY https://github.com/Kitware/VTK.git
        GIT_TAG v8.2.0
        CMAKE_ARGS 
            -DCMAKE_INSTALL_PREFIX=${VTK_Install}
            -DVTK_WRAP_PYTHON:BOOL=OFF
            -DVTK_WRAP_JAVA:BOOL=OFF
            -DBUILD_SHARED_LIBS:BOOL=OFF
            -DVTK_USE_GUISUPPORT:BOOL=ON
            -DVTK_USE_QVTK_QTOPENGL:BOOL=ON
            -DVTK_Group_Qt:BOOL=ON 
            -DBUILD_TESTING:BOOL=OFF
            -DQt5_DIR=${Qt5_DIR}
            -DModule_vtkVTKm:BOOL=OFF
            -DModule_vtkAcceleratorsVTKm:BOOL=OFF
    )
endif()

#OpenIGTLink: 
if(Use_System_OpenIGTLink)
    find_package(OpenIGTLink REQUIRED)
    include_directories(${OpenIGTLink_INCLUDE_DIRS})
else()
    set(OpenIGTLink_Install ${ThirdPartyLibs}/OpenIGTLinkInstall)
    set(OpenIGTLink_Repo ${ThirdPartyLibs}/OpenIGTLink)
    set(OpenIGTLink_DIR ${ThirdPartyLibs}/OpenIGTLinkBuild)
    set(OpenIGTLink_Prefix ${ThirdPartyLibs}/OpenIGTLinkPrefix)
    ExternalProject_Add(
        OpenIGTLink
        PREFIX ${OpenIGTLink_Prefix}
        SOURCE_DIR ${OpenIGTLink_Repo}
        BINARY_DIR ${OpenIGTLink_DIR}
        GIT_REPOSITORY https://github.com/openigtlink/OpenIGTLink.git
        CMAKE_ARGS 
            -DCMAKE_INSTALL_PREFIX=${OpenIGTLink_Install}
            -DBUILD_SHARED_LIBS:BOOL=OFF
    )
endif()

#OpenCV: 
if(Use_System_OpenCV)
    find_package(OpenCV REQUIRED)
    include_directories(${OpenCV_INCLUDE_DIRS})
else(Use_System_OpenCV)
    if(Use_CUDA)
        if(CUDA_FOUND)
            #Extra_contrib: 
            set(OpenCV_Extra ${ThirdPartyLibs}/OpenCVContrib)
            ExternalProject_Add(
                OpenCVContrib
                PREFIX ${OpenCV_Extra}
                GIT_REPOSITORY https://github.com/opencv/opencv_contrib.git
                GIT_TAG 4.2.0
                CONFIGURE_COMMAND ""
                BUILD_COMMAND ""
                INSTALL_COMMAND ""
            )
        endif(CUDA_FOUND)
    endif(Use_CUDA)
    
    set(OpenCV_Install ${ThirdPartyLibs}/OpenCVInstall)
    set(OpenCV_Repo ${ThirdPartyLibs}/OpenCV)
    set(OpenCV_DIR ${ThirdPartyLibs}/OpenCVBuild)
    set(OpenCV_Prefix ${ThirdPartyLibs}/OpenCVPrefix)
    if(Use_CUDA)
        ExternalProject_Add(
            OpenCV
            PREFIX ${OpenCV_Prefix}
            SOURCE_DIR ${OpenCV_Repo}
            BINARY_DIR ${OpenCV_DIR}
            GIT_REPOSITORY https://github.com/opencv/opencv.git
            GIT_TAG 4.2.0
            
            CMAKE_ARGS 
                -DCMAKE_INSTALL_PREFIX=${OpenCV_Install}
                -DBUILD_JAVA:BOOL=OFF
                -DBUILD_SHARED_LIBS:BOOL=ON
                -DBUILD_opencv_legacy=OFF 
                -DOPENCV_DNN_CUDA:BOOL=ON
                -DWITH_CUDA:BOOL=ON
                -DOPENCV_EXTRA_MODULES_PATH=${OpenCV_Extra}/src/OpenCVContrib/modules 
        )
    else()
        ExternalProject_Add(
            OpenCV
            PREFIX ${OpenCV_Prefix}
            SOURCE_DIR ${OpenCV_Repo}
            BINARY_DIR ${OpenCV_DIR}
            GIT_REPOSITORY https://github.com/opencv/opencv.git
            GIT_TAG 4.2.0
            
            CMAKE_ARGS 
                -DCMAKE_INSTALL_PREFIX=${OpenCV_Install}
                -DBUILD_JAVA:BOOL=OFF
                -DBUILD_SHARED_LIBS:BOOL=ON
                -DBUILD_opencv_legacy=OFF 
        )
    endif()
endif(Use_System_OpenCV)

#CTK: 
if(Use_System_Qt5)
    set(CTK_Install ${ThirdPartyLibs}/CTKInstall)
    set(CTK_Repo ${ThirdPartyLibs}/CTK)
    set(CTK_DIR ${ThirdPartyLibs}/CTKBuild)
    set(CTK_Prefix ${ThirdPartyLibs}/CTKPrefix)
    ExternalProject_Add(
        CTK
        PREFIX ${CTK_Prefix}
        SOURCE_DIR ${CTK_Repo}
        BINARY_DIR ${CTK_DIR}
        GIT_REPOSITORY https://github.com/commontk/CTK.git
        CMAKE_ARGS 
            -DCMAKE_INSTALL_PREFIX=${CTK_Install}
            -DCTK_QT_VERSION:STRING=5
            -DQT_QMAKE_EXECUTABLE=${QT_QMAKE_EXECUTABLE}
            -DQt5_DIR=${Qt5_DIR}
            -DCTK_ENABLE_Widgets:BOOL=ON
            -DCTK_LIB_Core:BOOL=ON
            -DCTK_LIB_Widgets:BOOL=ON
            -DCTK_LIB_DICOM/CORE:BOOL=ON
            -DCTK_LIB_DICOM/Widgets:BOOL=ON
            -DCTK_LIB_PluginFramework:BOOL=OFF
            -DBUILD_SHARED_LIBS:BOOL=OFF
            #-DCTK_USE_SYSTEM_VTK:BOOL=ON
            #-DCTK_LIB_Visualization/VTK/Widgets:BOOL=ON
            #-DCTK_LIB_Visualization/VTK/Widgets_USE_TRANSFER_FUNCTION_CHARTS:BOOL=ON
            #-DVTK_DIR=${VTK_DIR}
    )
endif()
