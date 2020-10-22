[2020-10-22 10:51]

需求：
1. 在当前 Finder 路径下打开终端
2. 在当前终端路径下打开 Finder (Optional 因为直接在终端输入命令已经足够简单)

尽管在 Finder 中可以通过 Services 实现打开终端，但需要两次点击不够高效。网上有现成的 [Alfred workflow](https://github.com/LeEnno/alfred-terminalfinder) 就直接拿来用了。之前常用的终端是 iTerm 所以刚好可以用，但最近有点想拥抱 Hyper（颜🐶）用它来敲命令感觉自己就是一个小仙女～

整个过程是通过 AppleScript 实现的，我没有系统学过这个只会最最基础的写法，魔改了原来 iTerm 和 Terminal 的代码不成功。于是去网上搜搜有木有好心人造福人类，果真被我找到了！[Change default Terminal to Hyper.is ? - Discussion & Help - Alfred App Community Forum](https://www.alfredforum.com/topic/10444-change-default-terminal-to-hyperis/) （注：这个帖子的讨论中只有 deanishe 贴的代码能成功运行）

附上最后的代码：

```AppleScript
on hyper_win()
	set _running to (application "Hyper" is running)
	tell application "Hyper" to activate
	tell application "System Events"
		log (name of first application process whose frontmost is true)
		repeat while (name of first application process whose frontmost is true) is not "Hyper"
			delay 0.05
		end repeat
		
		set _hyper to first application process whose frontmost is true
		-- If Hyper was running, create a new window to run command
		if _running then
			tell _hyper to set _target to (count windows) + 1
			keystroke "n" using {command down}
		else
			set _target to 1
		end if
		
		-- Wait for wanted window count
		tell _hyper
			repeat while (count windows) < _target
				delay 0.05
			end repeat
			-- do script ("cd " & pathList) in first window
		end tell
	end tell
end hyper_win

on alfred_script(q)
    tell application "Finder"
    		set pathList to (quoted form of POSIX path of (folder of the front window as alias))
  	end tell

	my hyper_win()

	tell application "System Events"
		keystroke q
		key code 36
		do shell script "open -a Hyper " & pathList
	end tell
end alfred_script
```

Enjoy~