
Win32 API（Windows API）是微软为其Windows操作系统提供的一组应用程序接口，提供了用于操作系统内核和资源的一组功能，可以让开发者创建Windows应用程序，包括窗口、对话框、菜单、文件操作等。

Win32 API 是一个庞大且强大的工具集，可以让开发者利用Windows操作系统的各种功能来创建各种类型的应用程序，从简单的工具到复杂的图形界面程序，甚至是游戏。

如遇不懂的地方，参考：[MSDN 库](https://visualstudio.microsoft.com/zh-hans/msdn-platforms/)

## 概述

Windows 应用程序由许多组件构成，如应用程序的主窗口、菜单、工具栏、滚动条、按钮以及其他的对话框控件，Windows 程序并不能直接访问硬件。

Windows 应用程序需要依从事件驱动编程模型（event-driven programming model），常见的事件有


主要组成部分：
1. **窗口和消息**：Win32 API 允许你创建和管理窗口应用程序，包括创建、显示、隐藏、关闭窗口，以及处理窗口消息（例如鼠标点击、键盘输入等）。
2. **图形设备接口（GDI）**：GDI 提供了用于绘制图形、文本和图像的功能，它包括了画刷、画笔、字体等绘图工具。
3. **用户界面（UI）控件**：Win32 API 提供了一系列基本的用户界面控件，如按钮、文本框、列表框等，以便构建交互式的应用程序界面。
4. **文件和I/O 操作**：API 提供了用于文件的创建、读取、写入、删除等操作的函数。
5. **多媒体支持**：Win32 API 包括了一些用于处理音频、视频和图像的函数。
6. **网络和通信**：提供了用于创建网络连接、发送和接收数据等功能的函数。
7. **注册表和配置**：允许访问Windows的注册表数据库，以保存和检索配置信息。
8. **多线程和同步**：API 提供了多线程编程的支持，包括创建线程、同步访问共享资源等功能。
9. **安全和权限**：包括了用于处理访问控制、权限管理等功能的函数。

随着时间的推移，Microsoft 推出了其他更高级的API和框架，如.NET Framework、UWP（Universal Windows Platform）等，这些框架提供了更现代、高级的开发方式，但Win32 API仍然是许多底层操作的基础。

> 有Win32 API有没有Win64 API？
> "Win64 API" 实际上是一个术语的误用，正式的术语应该是 "Win64" 或 "Windows 64-bit"，指的是运行在 64位版本的Windows操作系统上的应用程序。
> **Win64 平台使用的是相同的 Win32 API**，但是编译器会生成适用于 64 位架构的代码。因此，在 Win64 平台上，你仍然可以使用 Win32 API 来创建和管理窗口。
> 总的来说，Win32 API 是用于 Microsoft Windows 操作系统的应用程序接口，而 Win64 是指运行在 64 位版本的Windows上的程序。在 Win64 平台上，你仍然可以使用 Win32 API 来进行开发。


### 官方教程

[创建 Win32 应用程序 (C++) | Microsoft Learn](https://learn.microsoft.com/zh-cn/previous-versions/visualstudio/visual-studio-2008/bb384843%28v%3dvs.90%29)




### 实现

```C++
#pragma region 窗口实例
int screen_w, screen_h, screen_exit = 0;
int screen_mx = 0, screen_my = 0, screen_mb = 0;

static HWND screen_handle = NULL;													// Windows API [句柄]HWND: Window Handle，指向窗口数据结构的指针，用于唯一标识一个窗口。
static HDC screen_dc = NULL;														//             [句柄]HDC: Device Context Handle，用于标识设备上下文。
static HBITMAP screen_hb = NULL;													//		       [句柄]HBITMAP: Bitmap Handle，对位图对象的引用，用于标识位图对象。
static HBITMAP screen_ob = NULL;                                                    // screen_hb和screen_ob分别表示位图指针和位图对象指针

int screen_keys[512];																// 记录当前键盘按下状态
unsigned char* screen_fb = NULL;													// 记录屏幕位图的地址，frame buffer
long screen_pitch = 0;                                                              // 记录

static LRESULT screen_events(HWND, UINT, WPARAM, LPARAM);

int screen_init(int w, int h, const char* title);
int screen_close(void);
void screen_dispatch(void);
void screen_update(void);

/// <summary>
/// 窗口初始化函数
/// </summary>
int screen_init(int w, int h, const char* title) {

    screen_close();

    // 1. 注册窗口类别
    WNDCLASS wc = {																	// Windows API [结构体]WNDCLASS：窗口类，WNDCLASS（早期版本） 和 WNDCLASSEX（新版本）
        CS_BYTEALIGNCLIENT,															//		                    style			:窗口类的风格
        (WNDPROC)screen_events,														//		                    lpfnWndProc		:窗口处理函数指针，该函数会接收并处理窗口消息
        0,																			//		                    cbClsExtra		:窗口类的额外类别字节数
        0,																			//		                    cbWndExtra		:窗口实例的额外字节数
        GetModuleHandle(nullptr),													//		                    hInstance		:应用程序实例的句柄
        nullptr,																	//		                    hIcon			:窗口类的图标句柄
        LoadCursor(nullptr, IDC_ARROW),												//		                    hCursor			:窗口类的光标句柄
        (HBRUSH)GetStockObject(BLACK_BRUSH),										//		                    hbrBackground	:窗口类的背景刷子句柄
        nullptr,																	//		                    lpszMenuName	:窗口类关联的菜单的名称
        _T("SCREEN3.1415926")														//		                    lpszClassName	:窗口类的名称
    };

    // 注册一个窗口类，以便在调用 CreateWindow 或 CreateWindowEx 函数时使用
    if (!RegisterClass(&wc)) return -1;

    // 2. 创建窗口实例
    screen_handle = CreateWindow(													// Windows API [函数]CreateWindow: 创建窗口实例 
        _T("SCREEN3.1415926"),														//		                    lpClassName     :窗口类别名
        title,																		//		                    lpWindowName    :窗口标题
        WS_OVERLAPPED | WS_CAPTION | WS_SYSMENU | WS_MINIMIZEBOX,					//		                    dwStyle         :窗口风格
        0, 0,																		//		                    x, y            :窗口位置
        0, 0,																		//		                    nWidth, nHeight :窗口大小
        nullptr,																	//		                    hWndParent      :父窗口句柄
        nullptr,																	//		                    hMenu           :菜单句柄
        wc.hInstance,																//		                    hInstance       :应用程序实例句柄
        nullptr																		//		                    lpParam         :附加参数
    );
    if (screen_handle == nullptr) return -2;

    BITMAPINFOHEADER biheader = {                                                   // Windows API [结构体]BITMAPINFOHEADER: 用于描述位图的基本属性，如宽度、高度、颜色平面数等。
        sizeof(BITMAPINFOHEADER),                                                   //                          biSize          :结构体的大小，通常是 sizeof(BITMAPINFOHEADER)。
        w, -h,                                                                      //                          biWidth, biHeight   :位图的宽度和高度
        1,                                                                          //                          biPlanes        :位图的颜色平面数
        32,                                                                         //                          biBitCount      :每个像素的位数
        BI_RGB,				                                                        //                          biCompression   :图像数据的压缩类型
        w * h * 4,                                                                  //                          biSizeImage     :图像数据的大小
        0, 0,                                                                       //                          biXPelsPerMeter, biYPelsPerMeter    :水平和垂直分辨率
        0,                                                                          //                          biClrUsed       :实际使用的颜色数
        0                                                                           //                          biClrImportant  :对于显示有重要影响的颜色数
    };

    BITMAPINFO bi = {                                                               // Windows API [结构体]BITMAPINFO：位图
        biheader,
    };

    LPVOID ptr;																		// Windows API [类型]LPVOID：通常用于传递内存块的指针，类似于C语言中的 void * 指针。
    HDC hDC;																		// Windows API [类型]HDC：把窗口绘制在屏幕上时使用

    screen_exit = 0;
    hDC = GetDC(screen_handle);                                                     // Windows API [函数]GetDC: 用于获取指定窗口或设备上的设备上下文（Device Context，DC）
    screen_dc = CreateCompatibleDC(hDC);                                            //             [函数]CreateCompatibleDC: 用于创建一个与指定设备环境（Device Context，DC）兼容的设备上下文, 通
                                                                                    //                                       常用于在内存中创建一个临时的设备上下文，以便进行绘图操作而无需将图形
                                                                                    //                                       绘制到屏幕上。
    ReleaseDC(screen_handle, hDC);                                                  //             [函数]ReleaseDC: 在使用GetDC获取设备上下文后，需要在不再使用时调用ReleaseDC函数来释放设备上下文

    screen_hb = CreateDIBSection(                                                   // Windows API [函数]CreateDIBSection: 用于创建一种设备无关的位图（DIB，Device-Independent Bitmap）
        screen_dc,                                                                  //                          hdc             :设备上下文句柄，用于指定与位图兼容的设备。
        &bi,                                                                        //                          pbmi            :指向一个 BITMAPINFO 结构体的指针，该结构体描述了位图的格式。
        DIB_RGB_COLORS,                                                             //                          usage           :指定位图的使用方式，可以是 DIB_PAL_COLORS 或 DIB_RGB_COLORS。
        &ptr,                                                                       //                          ppvBits         :指向一个指针的指针，用于接收位图的内存地址。
        0,                                                                          //                          hSection        :指定一个文件映射对象句柄，可选。
        0                                                                           //                          offset          :文件映射对象的偏移量，通常为零。
    );

    if (screen_hb == NULL) return -3;

    screen_ob = (HBITMAP)SelectObject(                                              // Windows API [函数]SelectObject: 主要用于在设备上下文（Device Context，DC）中选取一个图形对象
        screen_dc,                                                                  //                          hdc             :设备上下文的句柄。
        screen_hb                                                                   //                          h               :要选取的图形对象的句柄
    );
    screen_fb = (unsigned char*)ptr;
    screen_w = w;
    screen_h = h;
    screen_pitch = w * 4;

    RECT rect = {                                                                   // Windows API [结构体]RECT: 用于表示矩形区域
        0, 0,                                                                       //                          left, top       :矩形左上角的 x 坐标和 y 坐标
        w, h                                                                        //                          right, bottom   :矩形右下角的 x 坐标和 y 坐标
    };

    AdjustWindowRect(                                                               // Windows API [函数]AdjustWindowRect: 用于调整窗口的客户区矩形的大小，以适应指定的窗口样式。
        &rect,                                                                      //                          lpRect          :指向一个 RECT 结构体的指针，用于传入期望的客户区矩形，以及接收
                                                                                    //                                           计算后的窗口矩形
        GetWindowLong(screen_handle, GWL_STYLE),                                    //                          dwStyle         :窗口样式，通常是通过 CreateWindow 函数中指定的窗口样式参数。
        0                                                                           //                          bMenu           :指示窗口是否有菜单的标志，如果有菜单则为 TRUE，否则为 FALSE。
    );

    int wx = rect.right - rect.left;
    int wy = rect.bottom - rect.top;
    int sx = (GetSystemMetrics(SM_CXSCREEN) - wx) / 2;
    int sy = (GetSystemMetrics(SM_CYSCREEN) - wy) / 2;
    if (sy < 0) sy = 0;

    SetWindowPos(                                                                   // Windows API [函数]SetWindowPos: 用于改变指定窗口的位置、大小、Z轴顺序或外观。
        screen_handle,                                                              //                          hWnd            :指定要设置位置和大小的窗口句柄。 
        NULL,                                                                       //                          hWndInsertAfter :指定在 Z 轴上窗口的位置，可以是其他窗口的句柄，也可以是以下特
                                                                                    //                                           殊值之一：HWND_BOTTOM、HWND_NOTOPMOST、HWND_TOP、HWND_TOPMOST。
        sx, sy,                                                                     //                          X, Y            :窗口的新位置的左上角坐标。
        wx, wy,                                                                     //                          cx, cy          :窗口的新宽度和高度。
        (SWP_NOCOPYBITS | SWP_NOZORDER | SWP_SHOWWINDOW)                            //                          uFlags          :一组标志，用于控制窗口的大小和定位的方式。
    );

    SetForegroundWindow(screen_handle);                                             // Windows API [函数]SetForegroundWindow: 用于将指定窗口设置为前台窗口（即激活窗口）。

    // 3. 显示窗口
    ShowWindow(                                                                     // Windows API [函数]ShowWindow: 用于显示或隐藏指定窗口。
        screen_handle,                                                              //                          hWnd            :指定要显示或隐藏的窗口的句柄。
        SW_NORMAL                                                                   //                          nCmdShow        :指定窗口的显示状态，有若干定义的常量
    );

    screen_dispatch();

    memset(screen_keys, 0, sizeof(int) * 512);
    memset(screen_fb, 0, w * h * 4);

    return 0;
}

/// <summary>
/// 窗口关闭函数，将依次释放相关指针（注意顺序）
/// </summary>
int screen_close(void) {
    if (screen_dc) {                                                                // 释放设备上下文指针
        if (screen_ob) {                                                            // 释放位图对象指针
            SelectObject(screen_dc, screen_ob);
            screen_ob = NULL;
        }
        DeleteDC(screen_dc);
        screen_dc = NULL;
    }
    if (screen_hb) {                                                                // 释放位图指针
        DeleteObject(screen_hb);
        screen_hb = NULL;
    }
    if (screen_handle) {                                                            // 释放窗口句柄
        CloseWindow(screen_handle);
        screen_handle = NULL;
    }
    return 0;
}

/// <summary>
/// 窗口消息事件处理函数
/// </summary>
void screen_dispatch(void) {
    MSG msg;
    // 事件处理循环
    while (true) {
        if (!PeekMessage(                                                           // Windows API [函数]PeekMessage: 用于从消息队列中检查是否有消息，并将消息复制到消息结构中。该函数不会等待消
                                                                                    //                                息到达，而是立即返回。
                &msg,																//                          lpMsg           :指向 MSG 结构的指针，用于接收消息。
                NULL,																//                          hWnd            :指定窗口的句柄，如果为 NULL，则检查线程消息队列中的所有消息。
                0, 0,																//                          wMsgFilterMin, wMsgFilterMax :指定要检查的消息范围。 
                PM_NOREMOVE)														//                          wRemoveMsg      :指定是否从队列中移除消息，通常设为 PM_REMOVE 表示移除。
            )
            break;
        if (!GetMessage(                                                            // Windows API [函数]GetMessage: 用于从消息队列中检取一个消息，并将其复制到消息结构中。与 PeekMessage 不同，
                                                                                    //                               GetMessage会阻塞（等待）直到有消息到达为止。
                &msg,                                                               //                          lpMsg           :指向 MSG 结构的指针，用于接收消息。
                NULL,                                                               //                          hWnd            :指定窗口的句柄，如果为 NULL，则检查线程消息队列中的所有消息。
                0, 0)                                                               //                          wMsgFilterMin, wMsgFilterMax :指定要检查的消息范围。
            )
            break;
        DispatchMessage(&msg);                                                      // Windows API [函数]DispatchMessage: 用于将一个消息发送给窗口过程进行处理。
    }
}

void screen_update(void) {
    HDC hDC = GetDC(screen_handle);
    BitBlt(                                                                         // Windows API [函数]BitBlt: 用于在设备上下文之间执行位块传输（Bit Block Transfer）。它可以在不同的设备上下
                                                                                    //                           文之间复制图像数据，包括从一个设备上下文（例如内存中的位图）到另一个设备上下文
                                                                                    //                           （例如屏幕）。
        hDC,                                                                        //                          hdcDest         :目标设备上下文的句柄。
        0, 0,                                                                       //                          xDest, yDest    :目标矩形的左上角坐标。
        screen_w, screen_h,                                                         //                          width, height   :要复制的区域的宽度和高度。
        screen_dc,                                                                  //                          hdcSrc          :源设备上下文的句柄。
        0, 0,                                                                       //                          xSrc, ySrc      :源矩形的左上角坐标。
        SRCCOPY                                                                     //                          wRop            :光栅操作码，用于指定如何将源矩形的颜色与目标矩形的颜色进行合并。
    );
    ReleaseDC(screen_handle, hDC);
    screen_dispatch();
}

/// <summary>
/// 窗口处理事件函数
/// </summary>
/// <param name="hWnd">窗口句柄</param>
/// <param name="msg">消息类型</param>
/// <param name="wParam">附加参数</param>
/// <param name="lParam">附加参数</param>
/// <returns>LRESULT 一个长整型（LONG_PTR）的别名，用于存储处理消息后的结果。</returns>
static LRESULT screen_events(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    switch (msg) {
    case WM_CLOSE:
        screen_exit = 1;
        break;
    case WM_KEYDOWN:
        screen_keys[wParam & 511] = 1;
        break;
    case WM_KEYUP:
        screen_keys[wParam & 511] = 0;
        break;
    default:
        return DefWindowProc(hWnd, msg, wParam, lParam);
    }
    return 0;
}
#pragma endregion
```