---
layout: post
title:  "Introduction to GPU Programming with D3D12: Part 4 - API Basics"
date:   2025-04-02 3:00:00 +0900
categories: Introduction_to_GPU_Programming_with_D3D12
lang: en
---

[Previous Post](Introduction-to-GPU-Programming-with-D3D12-Part-3-History.html)

Before using D3D12 APIs, you should understand how they are structured. There are multiple traits shared by relevant APIs that you should be aware of.

## API Components

There are 3 separate APIs that we are going to cover: DXGI, D3D12 and DirectStorage. There are also adjacent APIs that we won't cover in this series: DirectSR, DirectML.

### DXGI

DXGI (DirectX Graphics Infrastructure) is an API that is shared between all versions of DirectX starting with 10. Included functionality is:

- Enumerating and querying info about GPUs
- Enumerating and querying info about displays
- Setting up display output
- Setting display modes
- etc.

You can technically skip using DXGI, but only if you don't need to enumerate GPUs or display anything on screen.

### D3D12

D3D12 is short for Direct3D 12. Despite the name, it can be used for 2D graphics and general purpose computations. It is the main API that you are going to use. Included functionality is:

- Creating GPU resources (Buffers, Textures, etc.)
- Compiling Shaders (Vertex, Pixel, Compute, etc.)
- Recording commands for the GPU work
- Submitting work to the GPU
- Synchronizing CPU and GPU
- etc.

### DirectStorage

DirectStorage is an API that allows you to load data from disk. Despite common misconception, DirectStorage does not require NVMe SSDs, or any special kind of disk. It supports *anything* you can read file from: SSDs, HDDs, network storage, optical disks, *floppy disks*, etc. In this series it'll be referred to as "DStorage". Included functionality is:

- Reading from disk to CPU memory
- Reading from disk to GPU memory
- On-the-fly decompression
- Synchronizing disk with CPU or GPU
- etc.

Its usage is completely optional, as you could read from disk using any other API. There are a few benefits though:

- It is simpler to use DStorage rather than perform all those operations manually
- It exposes option to set read request priority
- It has special optimization for NVMe SSDs, allowing for lower I/O overhead
- It includes high performance GPU decompression
- You will be able to benefit from future updates to DStorage

## nano-COM

COM (Component Object Model) - is Microsoft's programming interface standard. All aforementioned APIs use simplified version of COM referred to as "nano-COM". You don't need prior understanding of COM to use nano-COM.

But if you are already familiar with COM, or will stumble upon materials about COM on the internet, here are important differences between them:

- nano-COM does not need initialization.
  - Full COM requires you to initialize it with `CoInitialize`\[`Ex`\]/`CoUninitialize`\[`Ex`\].
- nano-COM is always multithreaded.
  - Full COM can be initialized in either single or multithreaded mode.
  - Note that it doesn't mean that all API objects are multithreaded.
- You can not create nano-COM objects yourself.
  - Full COM allows you to create objects with `CoCreateInstance`.
  - In nano-COM, objects are either created with a free function available in the API, or using methods of other objects.

### HRESULT

HRESULT is a status code that indicates result of a method. Most nano-COM methods return this.

There are 2 helpful macros to determine whether result is success or failure: `SUCCEEDED(x)` and `FAILED(x)`. Here's an example of checking for an error using `FAILED` macro:

```cpp
HRESULT hr = obj->Method();
if (FAILED(hr))
{
    // Error handling here
}
```

Another common pattern is to create a macro that adds "default" error handling and use it with calls that you never expect to fail.

```
#define ASSERT_D3D12_SUCCEEDED(expression) \
    { \
        HRESULT hr = expression; \
        assert(SUCCEEDED(hr)); \
    }
ASSERT_D3D12_SUCCEEDED(obj->Method());
```

### Interfaces

Primary method of exposing functionality to an app via nano-COM are interfaces. When you create an object, you get a pointer to the interface. You can never dereference those pointers, as all those interfaces are abstract classes.

Every nano-COM interface has the following traits:

- Their name starts with `I`, which stands for Interface.
- They inherit from `IUnknown` interface
- They have unique Interface ID (IID)
- They use reference counting for lifetime management (using `IUnknown` interface)
- They have dynamic casting function (using `IUnknown` interface)

#### IID

Interface ID (IID) is unique identifier of an interface. It is primarily used to dynamically pass interface type. If you want to create an object, you would pass IID of relevant type to the creation method. That way single method can return pointers for different interfaces. To perform dynamic cast you would also use IID to specify target type.

You can get IID in multiple different ways:

- By using a constant. For interface `ISampleInterface`, such constant IID would be named `IID_ISampleInterface`.
  - This option may require linking of additional libraries.
- By using MSVC specific [`__uuidof`](https://learn.microsoft.com/en-us/cpp/cpp/uuidof-operator?view=msvc-170) operator.
  - It supports following inputs
    - Interface type - e.g. `__uuidof(ISampleInterface)`
    - Object pointer - e.g. `__uuidof(sampleObjectPointer)`
    - Object reference - e.g. `__uuidof(*sampleObjectPointer)`
  - Similarly to `sizeof`, `__uuidof` operator doesn't evaluate passed expression, it only uses its type to find IID. So despite `__uuidof(*sampleObjectPointer)` looking like a dereference of an interface pointer, it won't be evaluated, and so it is safe to dereference in this case.

Here's an example of how we can use IIDs:

```cpp
IDXGIFactory7* factory = nullptr; // variable that will get created object
HRESULT hr = CreateDXGIFactory2(0, IID_IDXGIFactory7, (void**)&factory);
```

This code sample creates `IDXGIFactory7` object via `CreateDXGIFactory2` function. To specify that we want specifically `IDXGIFactory7`, we've passed `IID_IDXGIFactory7`. We could also use all other methods to get IID:

```cpp
__uuidof(IDXGIFactory7)
__uuidof(factory)
__uuidof(*factory)
```

Argument following IID is an address of interface pointer variable. Pattern of IID followed by the address of an output pointer is quite common in nano-COM methods, so it would be nice if we could simplify passing both pointer and its type. Luckily there's a macro that does exactly that - `IID_PPV_ARGS`. It expands into IID deduced from the pointer type, together with the address of a pointer itself. Using `IID_PPV_ARGS` we can rewrite that call as:

```cpp
IDXGIFactory7* factory = nullptr; // variable that will get created object
HRESULT hr = CreateDXGIFactory2(0, IID_PPV_ARGS(&factory));
```

#### IUnknown

[`IUnknown`](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nn-unknwn-iunknown) is a base class for all nano-COM interfaces.

`IUnknown` has 3 methods:

- [`ULONG AddRef()`](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nf-unknwn-iunknown-addref)
- [`ULONG Release()`](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nf-unknwn-iunknown-release)
- [`HRESULT QueryInterface(REFIID riid, void **ppvObject)`](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nf-unknwn-iunknown-queryinterface(refiid_void))

First 2 methods are responsible for reference counting, the last one allows you to perform dynamic casts.

#### Reference counting

When object is created, reference count of 1. You can increment it with `AddRef` method, and you decrement it with `Release` method. When reference count reaches 0, the object is destroyed, and you are no longer allowed to use it.

Here's how this can look in code. You want to create `IDXGIFactory7` object with `CreateDXGIFactory2` function, use it and then destroy it.

```cpp
IDXGIFactory7* factory = nullptr; // variable that will get created object
HRESULT hr = CreateDXGIFactory2(0, IID_PPV_ARGS(&factory));
if (SUCCEEDED(hr))
{
    // If creation succeeded, factory now has reference count of 1
    // Can do something with the factory here
    ...
    // We no longer need the object and want to destroy it
    factory->Release(); // This will decrement reference count to 0, and destroy the object.
    factory = nullptr; // Overwriting pointer to be safe, since we can no longer use old one
}
```

Here's another example: You have 2 separate subsystems in your app: A and B. A created an object, and passed it to B. B stored pointer to the object for future use and called `AddRef`. When A no longer needs the object, it calls `Release`. Same happens in B. It doesn't matter the order in which A and B stops using the object, it will only get destroyed after both called `Release`. Here's how that can be implemented:

```cpp
// Error checking is skipped to keep this short
// Subsystem A
IDXGIFactory7* g_FactoryA = nullptr;

void Create()
{
    HRESULT hr = CreateDXGIFactory2(0, IID_PPV_ARGS(&g_FactoryA));
    B.ImportFactory(g_FactoryA);
}

void Use()
{
    // Use g_FactoryA here
}

void Release()
{
    g_FactoryA->Release();
    g_FactoryA = nullptr;
}

// Subsystem B
IDXGIFactory7* g_FactoryB = nullptr;

void ImportFactory(IDXGIFactory7* factory)
{
    g_FactoryB = factory;
    g_FactoryB->AddRef();
}

void Use()
{
    // Use g_FactoryB here
}

void Release()
{
    g_FactoryB->Release();
    g_FactoryB = nullptr;
}
```

`AddRef` and `Release` methods return new number of references. However, this value is only supposed to be used for test purposes. It is fine to use this value, e.g. to `assert` that a given `Release` call decremented reference count to 0, and did destroy an object. But it should not be used for application logic.

In D3D12 there's also an internal refcount, separate from the external refcount manipulated by you with `AddRef` and `Release`. In some cases one D3D12 object may internally hold a reference to another one, and in that cases it uses this separate internal refcount. Object will only get destroyed when both of those refcounts reach 0. You generally don't need to worry about it, since you are only responsible to release external references. You can, however, observe objects that have 0 external references but are not yet destroyed with some debug tools.

#### Dynamic casting

If you have an object for a certain interface, method `QueryInterface` allows you to query another interface out of it. This is essentially a dynamic cast. This method has 2 parameters: IID of target interface and address to store result to, so this is another candidate for use of `IID_PPV_ARGS`. If it succeeds, reference counter is also incremented. The logic is that since you have 2 pointers to the same object, so you'll `Release` each one. 

Here's an example. You have the following interfaces:

<img src="{{site.baseurl}}/diagrams/DynamicCast.mmd.svg" height="300" alt="Sample Class Diagram">

Let's say you have a `ID3D12Device` object, and you want to query `ID3D12DebugDevice` interface from it. Here's how we can use `QueryInterface`:

```cpp
ID3D12Device* device = /* valid pointer here */;
ID3D12DebugDevice* debugDevice = nullptr; // variable that will get the pointer after dynamic cast
if (SUCCEEDED(device->QueryInterface(IID_PPV_ARGS(&debugDevice))))
{
    // Dynamic cast succeeded
    // Can use debugDevice here
    ...
}
```

There's, however, a bug in this code snippet. **On success, `QueryInterface` increments reference counter**. That means that we now have to `Release` after successful `QueryInterface`. As you've just seen it is quite easy to forget that. Luckily, there's an approach that eliminates this problem.

### ComPtr

[`Microsoft::WRL::ComPtr`](https://learn.microsoft.com/en-us/cpp/cppcx/wrl/comptr-class?view=msvc-170) is a smart pointer class that calls `AddRef` and `Release` for you. It is same idea as `std::shared_ptr` with a difference that it uses built-in nano-COM object's reference count instead of allocating separate one.

#### Adding ComPtr to your project

`ComPtr` is a class in the `Microsoft::WRL` namespace. You can use `using Microsoft::WRL::ComPtr;` to skip specifying that namespace each time. Since such `using` directive makes only `ComPtr` visible outside its namespace, it doesn't have the same downside as `using namespace`, and so it is safe to use globally.

Since `ComPtr` is not part of aforemetioned APIs, you need to include it separately. It is available via `wrl/client.h` header file.

So, to include it in your project, you could use the following code snippet:

```cpp
#include <wrl/client.h>
using Microsoft::WRL::ComPtr;
```

#### Methods

`ComPtr` has many methods, we'll cover the most important ones:

- `ComPtr::ComPtr` (constructor)
  - Default constructor - Initializes pointer to `nullptr`.
  - Copy constructor - Copies pointer and if it is not `nullptr`, increments reference count.
  - Move constructor - Copies pointer, then set pointer in other object to nullptr. Does not change reference count
  - From raw pointer - Stores pointer and if it is not `nullptr`, *increments reference count*.
- `ComPtr::~ComPtr` (destructor) - If stored pointer is not `nullptr`, decrements reference count.
- `Reset` - If stored pointer is not `nullptr`, decrements reference count, and sets it to `nullptr`
- `operator&`/`ReleaseAndGetAddressOf` - Performs `Reset`, then returns address of a member that stores the pointer. *This means that after calling this method, the pointer inside will always be `nullptr`.*
- `Attach` - Performs `Reset`, and stores passed pointer, *without incrementing reference count*.
- `operator->`/`Get` - Returns raw pointer. `operator->` allows call interface methods directly via `ComPtr`.
- `As` - Performs `QueryInterface` storing result in another `ComPtr`. Passes down return value (`HRESULT`) of `QueryInterface`.

You can see the full list of methods in the [ComPtr documentation](https://learn.microsoft.com/en-us/cpp/cppcx/wrl/comptr-class?view=msvc-170#public-constructors).

#### Example

Let's rewrite our previous example using `ComPtr`:

```cpp
ComPtr<ID3D12Device> device = /* valid pointer here */;
ComPtr<ID3D12DebugDevice> debugDevice; // Initialized with nullptr
// "As" method uses type of debugDevice ComPtr to find appropriate IID, and return QueryInterface result
if (SUCCEEDED(device.As(&debugDevice))) 
{
    // Dynamic cast succeeded
    // Can use debugDevice here
    ...
    // Don't need to manually release references
}
```

With `ComPtr` we can't forget to call `Release`, and it is simpler to perform `QueryInterface`. Note that you can still use `IID_PPV_ARGS` macro with `ComPtr` as if it was a raw pointer.

There are however, few more things to note:

- If you need to get raw pointer, with `ComPtr` you'll need to call the `Get` method. You'll need it for certain nano-COM methods that expect raw pointers to other nano-COM objects.
- Ideally you should create nano-COM objects directly in the `ComPtr`. But if you already have raw pointer that you want to convert to `ComPtr`, you likely need to use `Attach` method, to prevent reference counter increment.
- `operator&` releases stored object before returning the address of a pointer variable. This makes it convenient to create a new object and save pointer into `ComPtr`. You need to be careful with this operator to make sure you only call it when you intend to not preserve pointer inside.

## Versioning

Since release of D3D12 almost 10 years ago a lot of new features were added. That means that new functionality had to be exposed in some way. In all APIs that we are going to cover it is done in the same way.

Let's take `ID3D12GraphicsCommandList` as an example. This interface exposes different kinds of GPU work. Some features, such as Raytracing, were added after initial release of D3D12, and so they had to be added to that interface somehow. The way it is done is by creating new interface, that inherits previous one:

<img src="{{site.baseurl}}/diagrams/ID3D12GraphicsCommandList.mmd.svg" height="800" alt="Class diagram for ID3D12GraphicsCommandList7">

Base functionality is in `ID3D12GraphicsCommandList`, Raytracing was added in `ID3D12GraphicsCommandList4`, Mesh Shaders were added in `ID3D12GraphicsCommandList6`, etc. Other interfaces are updated in same way, if you have `ISampleInterface`, updates would add `ISampleInterface2`, `ISampleInterface3`, and so on. If you have pointer to an interface, you can also use all methods from previous versions, since newer versions inherit older ones.

Support for those newer interfaces is either defined by the OS version, or by version of `Agility SDK` (more on Agility SDK in a later post). So if you're running on an older OS or an outdated Agility SDK version, you may not be able to use latest interface. It does not depend on the hardware present in the system. If GPU does not support raytracing, you can still use `ID3D12GraphicsCommandList4` as an interface, given recent enough OS or Agility SDK version, you just won't be allowed to call raytracing related methods.

To get newer interface you have 2 options:

- Directly create object for a new interface. Since most creation methods accept IID's, you can just swap the type with a newer one.
- Create object for an old version of an interface, and then use `QueryInterface` to get new version.

Despite the fact that you ask to create an object for a specific version of an interface, you'll alway get an object that implements newest supported interface. That's why approach with `QueryInterface` works, if you create object for `ID3D12GraphicsCommandList` interface, you will get object that implements all interfaces from that class diagram, as long as OS or Agility SDK support them.

If a certain version of an interface isn't supported, attempting to create an object for that version will fail, and no object will be returned. However, if you first create an object using an older version of the interface that you know is supported, the creation will succeed. You can then call `QueryInterface` to request the newer interface and handle the error gracefully if it's not available.

So recommendations on usage of versioned interfaces are:

- Create objects for latest interface that will be supported or that you require to run your app.
- Use `QueryInterface` to get newer interface and handle potential errors.

## Useful materials

### Documentation

- [Microsoft Learn](https://learn.microsoft.com) - Contains API Reference for all 3 of those APIs. It is most useful when you need to look up details of specific method, class, structure or enum. This is also the only resource that contains DXGI documentation.
- [DirectX-Specs](https://microsoft.github.io/DirectX-Specs/) - Contains all documentation for D3D12, including the newest features.
- [Direct3D 11.3 Functional Specification](https://microsoft.github.io/DirectX-Specs/d3d/archive/D3D11_3_FunctionalSpec.htm) - This is a huge document with all details of DirectX 10-11.3. Unfortunately, D3D12 does not have an equivalent document, and so in cases when something is not described in D3D12 documentation, it is assumed to work in the same way as it did in D3D11.
- [DirectStorage Developer Guidance](https://github.com/microsoft/DirectStorage/blob/main/Docs/DeveloperGuidance.md) - Guide on using DStorage combined with best practices and partial API reference.

### Samples

- [DirectX-Graphics-Samples](https://github.com/Microsoft/DirectX-Graphics-Samples) - Collection of D3D12 samples covering many different features.
- [DirectStorage Samples](https://github.com/microsoft/DirectStorage/tree/main/Samples) - Collection of DStorage samples.

### Other links

- [DirectX landing page](https://devblogs.microsoft.com/directx/landing-page/) - List of links to DirectX related materials.
- [DirectX (Developers) Discord Server](https://discord.gg/directx) - Discord server where you can ask DirectX related questions. Highly recommend joining.
- [NVIDIA's Advanced API Performance](https://developer.nvidia.com/blog/tag/advanced-api-performance) - Best practices for D3D12 that help maximize performance. Note that those may include Nvidia specific suggestions that won't neccecarilly translate to better performance across all hardware.
