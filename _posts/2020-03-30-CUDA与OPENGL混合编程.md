---
layout:     post
title:      CUDA与OPENGL 混合编程
subtitle:   
date:       2020-03-30
author:     infinityyf
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - CUDA
    - openGL
---

## 在cuda中注册一个texture 资源

使用`cudaGraphicsGLRegisterImage `，使用`cudaGraphicsResource**`来作为内存的映射指针，该函数接受GL的textureID或renderbufferID.
说明：  
```c
cudaError_t cudaGraphicsGLRegisterImage( 
  struct cudaGraphicsResource**  resource,
  GLuint image, //ID
  GLenum target,    
  unsigned int flags
)
```  

*target*:  If image is a texture resource, then target must be GL_TEXTURE_2D, GL_TEXTURE_RECTANGLE, GL_TEXTURE_CUBE_MAP, GL_TEXTURE_3D, or GL_TEXTURE_2D_ARRAY. If the image refers to a render-buffer object, then target must be GL_RENDERBUFFER.  

*flag*:  cudaGraphicsRegisterFlagsNone:可读可写
cudaGraphicsRegisterFlagsReadOnly: 只读
cudaGraphicsRegisterFlagsWriteDiscard:只写
cudaGraphicsRegisterFlagsSurfaceLoadStore:

## 在cuda中注册一个VBO或PBO资源  

使用`cudaGraphicsGLRegisterBuffer `
说明：  
```c
cudaError_t cudaGraphicsGLRegisterBuffer(
  struct cudaGraphicsResource**  resource,  
  GLuint  buffer,  
  unsigned int  flags   //见上面说明
) 
```  


## 在cuda中使用资源，map操作

使用`cudaGraphicsMapResources `  
说明：  
```c
cudaError_t cudaGraphicsMapResources ( 
  int count,  //需要映射的资源量
  cudaGraphicsResource_t* resources,  //一个对应资源的指针列表
  cudaStream_t stream = 0       
)
```

获取设备的内存指针以访问数据，如果是texture or render-buffer resource，使用`cudaGraphicsSubResourceGetMappedArray `映射成一个二维CUDA列表，如果是 vertex buffer or pixel buffer object使用`cudaGraphicsResourceGetMappedPointer `得到内存地址。  

说明：
```c
cudaError_t cudaGraphicsResourceGetMappedPointer( 
  void** devPtr,  
  size_t* size,  
  cudaGraphicsResource_t resource//之前被map过
) 
```
在进行map之后，资源会被lock 需要在处理完成之后使用`cudaGraphicsUnmapResources`函数进行解除锁定

## 分配全局内存  
使用`cudaMalloc`，使用`cudaFree`释放
```c
uchar4* dstBuffer = NULL;
size_t bufferSize = width * height * sizeof(uchar4);
cudaMalloc( &dstBuffer, bufferSize );
```
使用`cudaMemcpy`进行数据通信
```c
cudaError_t cudaMemcpy(
    void* dst, 
    const void* src, 
    size_t count, //字节数
    cudaMemcpyKind kind //复制方向（cudaMemcpyHostToHost，cudaMemcpyHostToDevice，cudaMemcpyDeviceToHost，cudaMemcpyDeviceToDevice）
    )
```
`cudaMemcpyAsync`同样可以进行通信，但是是进行异步的数据同行，使用`cudaFreeHost`释放


cuda的内存包括本地内存，共享内存和全局内存。其中本地内存为线程私有，共享内存为线程块包含，所有线程都可以访问全局内存。  

`cudaMallocHost`用于在host上开辟空间，具有比malloc更好的性能（从交换页面角度考虑）


## 核函数  

__global__：在device上执行，从host中调用（一些特定的GPU也可以从device上调用），返回类型必须是void，不支持可变参数参数，不能成为类成员函数。注意用__global__定义的kernel是异步的，这意味着host不会等待kernel执行完就执行下一步。
__device__：在device上执行，单仅可以从device中调用，不可以和__global__同时用。
__host__：在host上执行，仅可以从host上调用，一般省略不写，不可以和__global__同时用，但可和__device__，此时函数会在device和host都编译。

## CUDA 和opengl 交互流程

1. 初始化VBO等数据，并开辟空间  
2. 在CUDA中进行注册  
3. 在CUDA中map,并获取数据进行计算    
4. unmap并传回opengl中的对应内存