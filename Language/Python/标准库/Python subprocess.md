Python `subprocess` 模块是用于**创建和管理子进程**的标准库，允许在 Python 程序中启动新的进程、连接到它们的输入/输出/错误管道，并获取返回值。

## 常用函数

1. **`subprocess.run()`** - 推荐方式（Python 3.5+）
```python
import subprocess
# 执行命令，返回 CompletedProcess 对象
result = subprocess.run(['ls', '-l'], capture_output=True, text=True)
print(result.stdout)  # 输出结果
print(result.returncode)  # 返回码
```

2. **`subprocess.Popen()`** - 更底层的控制
```python
import subprocess
# 更灵活的控制
proc = subprocess.Popen(['ping', 'google.com'], 
                        stdout=subprocess.PIPE,
                        stderr=subprocess.PIPE)
stdout, stderr = proc.communicate()  # 等待进程结束
```

3. **旧函数**（现在推荐用 `run()` 替代）
	- `subprocess.call()` - 执行命令并等待完成
	- `subprocess.check_output()` - 获取命令输出
	- `os.system()` - 简单但功能较弱（不推荐）

## 常用参数

|参数|说明|
|---|---|
|`args`|要执行的命令（字符串或列表）|
|`shell`|是否通过 shell 执行|
|`capture_output`|是否捕获 stdout/stderr|
|`text`|是否以文本模式返回（否则是 bytes）|
|`timeout`|超时时间（秒）|
|`cwd`|工作目录|
|`env`|环境变量|

## 实际示例

```python
import subprocess
# 1. 执行简单命令
subprocess.run(['echo', 'Hello World'])
# 2. 获取输出
result = subprocess.run(['ls', '-la'], 
                        capture_output=True, 
                        text=True)
print(f"输出: {result.stdout}")
# 3. 错误处理
try:
    subprocess.run(['false'], check=True)  # check=True 会抛出异常
except subprocess.CalledProcessError as e:
    print(f"命令失败，返回码: {e.returncode}")
# 4. 使用 shell 模式
subprocess.run('echo $HOME', shell=True)
# 5. 管道输入
proc = subprocess.Popen(['grep', 'python'], 
                        stdin=subprocess.PIPE,
                        stdout=subprocess.PIPE,
                        text=True)
stdout, _ = proc.communicate(input='python\njava\ngo\npython')
print(stdout)  # 输出: python\npython
```