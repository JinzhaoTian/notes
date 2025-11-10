
```cpp
thread()
{
    game_initialize();
    
    //game loop
    time_last = now();
    elapsed = 0;
    while(1)
    {
        time_now = now();
        elapsed += time_now - time_last; //上一次更新后游戏时间
        time_last = time_now;
        while(elapsed >= GAME_STEP)
        {
            elapsed -= GAME_STEP;
            game_update();//游戏世界更新
        }
        game_render();//游戏世界渲染
        sleep(0);
    }
}

main()
{
    CreateThread(thread);
    while(system_loop()) 
        sleep(0); //系统循环, * 可获取消息循环中的鼠标按键输入
}
```

## 参考

1. [在游戏编程中，如何设计游戏循环（Game Main Loop）？ - 知乎](https://www.zhihu.com/question/341271673)
