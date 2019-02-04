---
layout: post
title: Vim 8.0 vs Mojave
---
If you're using VIM, you have probably noticed, that after update to Mojave your VIM has some problems..

![_config.yml]({{ site.baseurl }}/images/posts/vim80vsmojave/mojave.png)
{: style="text-align: center;"}

## Solution 1: Update VIM to 8.1
You can update VIM to [latest version (8.1)](https://www.vim.org/vim-8.1-released.php), but be carreful. If you are using plugins, it may cause errors. My main problem was the Command-T plugin. VIM 8.1 resolves problem with this plugin, but unfortunately brokes other plugins. You need to check if vim 8.1 is right for you.

## Solution 2: Install missing SDK headers for MacOS 10.14 Mojave
You even don't need to download them:
```
sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /
```

Thanks to this your VIM should back to life! If you use the command-t, you may need to recompile the plugin:
```
cd ~/.vim/bundle/command-t/
rake make clean
rake make
```

That's all, your VIM is ready to work.
