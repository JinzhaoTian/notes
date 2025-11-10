
## Terminal

Windows上Terminal的体验不是很好，我找了一个比较好的解决方案：PowerShell+ Oh My Posh，主题选为catppuccin_macchiato还是有一个比较好的体验的，接近于原生的Linux Bash。
![](_imgs/Pasted%20image%2020230417111035.png)


### PowerShell

不是微软自带的Microsoft PowerShell，是就叫PowerShell，安装命令：

```powershell
winget search powershell

winget install Microsoft.PowerShell
winget uninstall Microsoft.PowerShell
```

然后在PowerShell里面配置默认开启：

![](_imgs/Pasted%20image%2020230417113425.png)

#### winget

在 Windows 上使用 winget 命令行工具来发现、安装、升级、删除和配置应用程序。 此工具是 Windows 程序包管理器服务的客户端接口。

安装：[从 Microsoft Store 获取应用安装程序](https://www.microsoft.com/p/app-installer/9nblggh4nns1#activetab=pivot:overviewtab)



#### gsudo

Windows下的gsudo类似于Linux下的sudo，用来提权，不用重新使用管理员打开PowerShell了。使用如下命令安装：

```PowerShell
winget install gsudo
```


### Oh My Posh

Mac 下面有一个Oh My Zsh，这个Oh My Posh和那个类似，官网是[Home | Oh My Posh](https://ohmyposh.dev/)

安装命令：

```powershell
winget install JanDeDobbeleer.OhMyPosh -s winget

winget upgrade JanDeDobbeleer.OhMyPosh -s winget
```

主题配置：

首先使用命令：`notepad $profile`，打开配置文件，输入以下两行：

```
oh-my-posh init pwsh | Invoke-Expression
& ([ScriptBlock]::Create((oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\catppuccin_macchiato.omp.json" --print) -join "`n"))
```

![](_imgs/Pasted%20image%2020230417113101.png)


有的主题需要字体库的支持才可以显示图标，因此还需要安装字体库，一般选用 Nerd Font 类型的字体库。根据官网上的说明，使用以下命令：

```powershell
oh-my-posh font install
```

![](_imgs/Pasted%20image%2020230417113949.png)

选择一个喜欢的字体安装，或者去[Nerd Fonts 官网](https://www.nerdfonts.com/font-downloads) 直接下载一个字体，然后解压缩，全选，安装即可。


然后在 Terminal 界面使用快捷命令 `Ctrl + Shift + ,` 打开settings.json文件，输入如下设置：
```json
{  
	"profiles":  
	{  
		"defaults":  
		{  
			"font":  
			{  
				"face": "MesloLGM NF"  
			}  
		}  
	}  
}
```
填入你选择的字体。
