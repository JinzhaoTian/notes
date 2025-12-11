在 Windows 编程中，`WinMain` 是图形界面应用程序的主入口点，类似于控制台程序中的 `main()` 函数。

```c++
int WINAPI WinMain(
    HINSTANCE hInstance,      // 当前实例句柄
    HINSTANCE hPrevInstance,  // 先前实例句柄（已废弃，总是 NULL）
    LPSTR lpCmdLine,          // 命令行参数
    int nCmdShow              // 窗口初始显示状态（如最大化、最小化）
);
```

## 基本示例

```c++
#include <windows.h>

int WINAPI WinMain(
    HINSTANCE hInstance,
    HINSTANCE hPrevInstance,
    LPSTR lpCmdLine,
    int nCmdShow)
{
    // 1. 注册窗口类
    WNDCLASS wc = {};
    wc.lpfnWndProc = WindowProc;  // 窗口过程函数
    wc.hInstance = hInstance;
    wc.lpszClassName = "MyWindowClass";
    
    RegisterClass(&wc);
    
    // 2. 创建窗口
    HWND hwnd = CreateWindow(
        "MyWindowClass",    // 窗口类名
        "我的窗口",         // 窗口标题
        WS_OVERLAPPEDWINDOW, // 窗口样式
        CW_USEDEFAULT, CW_USEDEFAULT,  // 位置
        800, 600,           // 大小
        NULL, NULL, hInstance, NULL);
    
    // 3. 显示窗口
    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);
    
    // 4. 消息循环
    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    
    return (int)msg.wParam;
}

// 窗口过程函数（处理消息）
LRESULT CALLBACK WindowProc(
    HWND hwnd, UINT uMsg, 
    WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    case WM_DESTROY:
        PostQuitMessage(0);
        return 0;
    default:
        return DefWindowProc(hwnd, uMsg, wParam, lParam);
    }
}
```